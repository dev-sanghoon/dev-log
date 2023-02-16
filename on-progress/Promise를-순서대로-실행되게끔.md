### 문제
```js
async function upload() {
  // 1. db table의 unique constraint와 업로드 데이터가 충돌하는지 확인
  // 2-1. 충돌 시 충돌되는 row의 id를 반환후 종료
  // 2-2. 충돌하지 않을 시 업로드 후 생성된 row의 id를 반환 후 종료
}

const promises = dataList.map(data => upload(data));
await Promise.all(promises);
```

위의 코드를 실행하는데, 자꾸 데이터가 중복되어 업로드 되고 있었다.
upload function의 로직 내부에서 console을 찍어보면, 분명 이전에 동일한 데이터가 업로드 되었는데도 충돌 확인 로직에서 이전에 업로드된 데이터를 찾지 못하고 있었다.

그러다가 문득 Promise.all이 두번째 실행되었을 때에는 이전에 업로드된 데이터를 제대로 찾는 것을 확인하였다. 이것을 보고 Promise.all이 순서대로 실행되지 않아, 복수의 upload가 동시에 실행되면서... 예를 들어 10개의 Promise가 a라는 데이터를 업로드한다고 할 때, 10개 중 대략 8~9개 정도가 다른 Promise에서 a를 업로드 하기 전에 충돌 확인 로직을 실행하는 식으로 확인하여 중복 데이터가 업로드 되는 것이라는 가설을 세우게 되었다.

Promise.all을 활용하지 않고
```js
for (const item of data) {
  await doSomething(item);
}
```
을 활용하였을 때에는 정상적으로 실행됨을 확인하였다.

흠. node.js는 single thread라고 배웠는데, 어떻게 이런 방식으로 작동하는 걸까? v8 엔진의 작동 방식에 대해 공부해야 할까? 여러개의 promise를 래핑한 promise는, 내부의 promise를 모두 실행한다음 thread를 넘기는게 아니라 최소 단위의 promise를 실행하고 다른 함수에게 thread를 넘긴 뒤 자신이 가진 다른 promise의 차례가 올때까지 기다리는 것일까?
