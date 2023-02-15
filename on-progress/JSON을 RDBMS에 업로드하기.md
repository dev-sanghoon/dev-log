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

