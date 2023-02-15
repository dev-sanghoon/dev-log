### 문제
내가 작성한 테이블은 특정 재료에 대해 단위가 고정되는 형태였다. 그러나 가루 설탕에서 이슈가 발생했다. 가루 설탕에 단위가 2가지 (bar spoon, tea spoon)의 형태로 있을 수 있던 것이다. 물론 바스푼과 티스푼의 용량은 비슷하지만, 향후 유저가 직접 칵테일을 입력할 수 있는 기능을 도입했을 때 단위가 자유롭지 않다면 문제가 발생할 수 있다. 

어떻게 대응할 것인가?
우선 현재 테이블 형태는 다음과 같다.

```sql
CREATE TABLE ingredients (
  id int NOT NULL,
  name varchar(255) NOT NULL,
  unit int NOT NULL,
  type int,
  PRIMARY KEY (id),
  FOREIGN KEY (unit) REFERENCES units.id,
  FOREIGN KEY (type) REFERENCES types.id
)
```

### 시도 1 (진행중)
다음과 같은 선택지가 있다.
- 그냥 ml로 변경시키고, 유저에게 다양한 input을 drop down으로 두고 서버에서 일괄로 ml로 변경시켜 관리
- 테이블을 분리하기. unit을 제거하고 ingredients-unit 테이블을 생성하여 ingredient id를 foreign key로 두는 것이다. unit 컬럼은 이미 foreign key니까 상관없고. 사실 이게 정석적인 해결방식인 듯 하다.
- 바스푼과 티스푼을 통일하기. 경직된 DB지만 좀더 뭐랄까... normalized? opinionated?한 것이 장점이 될 수도 있다.

결정: 바스푼과 티스푼을 통일하는 것으로 했다. 우선 테이블 분리는 스키마가 지나치게 복잡해진다. 또한, ml로 변경시키고 통일하는 것은 설탕을 1 티스푼이라고 했을 때 설탕이 5ml가 되는 살짝 비현실적인 결과를 낸다.

데이터를 확인한 결과, 5ml 이하의 액체도 바스푼으로 계량을 한다. 따라서 해당 데이터들은 모두 ml로 변경 후, 서버에서 5ml 이하일 시 스푼으로 변경해주는 로직을 추가하는 것이 맞다고 판단된다.
