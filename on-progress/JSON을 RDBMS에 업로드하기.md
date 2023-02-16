### 문제
puppeteer을 활용하여, 다음과 같은 칵테일 데이터를 얻을 수 있었다.

```json
[
  {
    "code": "rusty-nail",
    "recipe": {
      "ingredients": [
        { "amount": 45, "unit": "ml", "ingredient": "scotch whisky" },
        { "amount": 25, "unit": "ml", "ingredient": "drambuie" }
      ],
      "method": "stir",
      "garnish": "Garnish with lemon zest."
    }
  },
  ...
]
```

이를 RDMBS에 업로드하려면 어떻게 해야 할까?

### 시도 1 (실패)
개별 TABLE을 위한 Array 객체를 만들고, 각 객체를 업로드하려 했다.
```json
[
  { "ingredient": "scotch whiskey", "type": 1, "unit": 1, "amount": 45 },
  ...
]
```

```sql
INSERT INTO recipes (ingredient, type, unit, amount) VALUES ("whiskey", 1, 1, 30);
```

1회성 작업이라 느껴져서 자동화하지 않았다.  
그러나 향후 개별 칵테일을 추가해야 할 상황이 생길 때에도 위와 같은 작업을 해야 한다.  
즉, 1회성 작업이 아니었던 것이다.  

테이블을 모두 업로드하고 확인해보니 같은 재료는 같은 단위를 사용할거라는 가정을 하고 코드를 구현해서 같은 재료인데 다른 단위를 사용할 경우 데이터가 잘못 올라가는 문제를 발견하였다.
  
### 시도 2 (완료)
좀 더 머리를 써보기로 했다.  
- DB의 구조가 주어졌을 때, 객체를 업로드하면 복수의 테이블에 마법같이 데이터가 입력되는 기능이 있을까?  

`typeorm crud foreign key` 등으로 검색해보았으나, 존재하지 않는 것으로 보였다.
chatGPT에도 질문을 해보았다. `is it possible to upload json data on rdbms which has many tables linked each other?`
만약 기능이 있었다면 알려주지 않았을까? 답변은 `JSON을 파싱 후 구현된 RDBMS의 테이블의 형태에 유의하며 올리세요.`로 요약된다.
결국 이 부분은 내가 구현해야 하는 부분인 것이다.

일단 express 서버를 생생해서, 모든 테이블을 truncate하는 엔드포인트와 위의 rusty nail 레시피를 rdbms에 업로드하는 코드를 작성해보고자 한다. 현재 unit 테이블이 별도로 생성되어 있는데, 업로드 시 unit이 존재하는지 존재하지 않는지 확인하고 업로드하는 코드가 중요할 것 같다.

그 전에 truncate할 db를 새로운 db에 생성하여 백업해두었다. 생각보다 쉽다.
```json
$> mysqldump db1 > dump.sql
$> mysqladmin create db2
$> mysql db2 < dump.sql
```

결과, 성공적으로 DB에 데이터 업로드가 가능하였다.
진행 과정에서 몇가지 문제가 불거졌지만... 이 글의 주제에서 벗어나니 별도로 작성하기로 하고. 

진행방식은 foreign key가 존재하지 않는 테이블을 우선적으로 array 형태로 정리하였다. 그리고 sql에 업로드하면 AUTO_INCREMENT PRIMARY KEY로 고유한 아이디가 생성된다. 그럼 이 아이디를 받아와서, 업로드 된 테이블의 값을 foreign key로 갖는 테이블의 값을 받아온 아이디로 변경해준다. 그리고 다시 업로드 하고, 또 ID를 받아오고. 다시 그 ID를 foreign key로 활용하는 테이블을 변경해주고, 업로드 하는 식이다.

일단 성공을 하긴 했는데, 코드가 복잡한 점이 좀 찝찝하다. 다들 백엔드 작업을 할 때 RDBMS에 데이터를 업로드할 때면 이렇게 변경을 해주어야 하는 걸까? 이전 직장에서는 별도의 RDBMS를 사용하지 않고 js SDK를 활용하여 S3에 업로드했는데, 이 경우는 JSON으로 업로드하니까 변경 과정이 필요가 없긴 하다. 실제 RDBMS와 서버가 어떻게 소통하는지 확인하지 못한 점이 못내 아쉬우려는 찰나.

realworld 예제가 떠올랐다. 확인해봐야겠다.

