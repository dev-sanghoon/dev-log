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
  
### 시도 2 (진행중)
좀 더 머리를 써보기로 했다.  
- DB의 구조가 주어졌을 때, 객체를 업로드하면 복수의 테이블에 마법같이 데이터가 입력되는 기능이 있을까?  

`typeorm crud foreign key` 등으로 검색해보았으나, 존재하지 않는 것으로 보였다.
chatGPT에도 질문을 해보았다. `is it possible to upload json data on rdbms which has many tables linked each other?`
만약 기능이 있었다면 알려주지 않았을까? 답변은 `JSON을 파싱 후 구현된 RDBMS의 테이블의 형태에 유의하며 올리세요.`로 요약된다.
결국 이 부분은 내가 구현해야 하는 부분인 것이다.










