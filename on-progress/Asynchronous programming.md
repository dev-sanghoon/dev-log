문제
express.js로 mySQL을 사용하고 있었는데, 다음과 같은 코드를 작성하게 되었다.

```js
app.get("/truncate", (req, res) => {
  connection.query("SHOW TABLES", (err, result) => {
    if (err) return res.send(err.message);
    const tables = result.map(({ Tables_in_test }) => Tables_in_test);
    const response = [];
    tables.forEach((table) => {
      connection.query(`TRUNCATE TABLE ${table}`, (err, result) => {
        if (err)
          return response.push({ success: false, table, message: err.message });
        response.push({ success: true, table });
      });
    });
    res.send(JSON.stringify(response));
  });
});
```

위의 코드는 SQL에 문제가 없다면 정상 작동할 것이다. 그러나express의 response가 empty array일 것이다. 왜냐하면 forEach의 루프가 작동하기 전에 res.send가 작동할 것이기 때문이다. 그렇다면 Promise나 async/await를 사용하면 간단하게 해결되겠지만, 과연 나는 Asynchronous programming을 이해하고 있는걸까? 하는 의문이 들었다.

결론. callback만을 사용하여 forEach iteration을 대기할 수 있게 코드를 짜보자. 그리고 Promise는 대체 어떻게 작동하는가? 내부에 setTimeout이라도 있는걸까? node.js 엔진의 작동방식으로 이해할 수 있을까? 
