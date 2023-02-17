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

https://www.youtube.com/watch?v=8aGhZQkoFbQ
자바스크립트 런타임 동작 방식을 알기 쉽게 설명한 글

https://dev.to/_staticvoid/node-js-under-the-hood-1-getting-to-know-our-tools-1465
Node.js의 동작방식을 상세하게 설명한 글

https://stackoverflow.com/a/63653361
Javascript의 thread는 하나이다. 그리고 대부분의 async operation는 그 thread에서 실행된다.

최초에는 Promise.all을 쓸 때 parameter로 들어가는 개별 Promise가 event queue로 들어간다고 생각했던 것 같다. 그러나 그렇게 된다면 Promise.all이 순서대로 실행되지 않고, 마치 병렬로 실행되는 듯 보이는 내용을 설명할 수 없다. 

그러다 봤던 내용이 Javascript가 사실상 single thread가 아니라는 것이다. 브라우저의 경우 web api가, node js의 경우 엔진 내 c++ api가 별도의 thread로 작동한다는 것이다. 이 내용에서 혼란이 왔던 것 같다.
