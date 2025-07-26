# 비동기 처리

<br />

## 동기와 비동기

- **동기 (Synchronous)**

  작업이 순서대로 실행됩니다. 하나의 작업이 끝날 때까지 다음 작업은 대기합니다. 코드가 직관적이지만, 오래 걸리는 작업이 있으면 전체 프로그램이 멈추는 **블로킹(Blocking)** 현상이 발생합니다.

- **비동기 (Asynchronous)**

  작업이 순서대로 시작되지만, 끝나는 순서는 보장되지 않습니다. 오래 걸리는 작업을 시작시켜 놓고 바로 다음 코드를 실행합니다. 프로그램이 멈추지 않는 **논블로킹(Non-blocking)** 방식입니다.

<br />

## 이벤트 루프(Event Loop)

**"자바스크립트는 싱글 스레드인데 어떻게 비동기 작업이 가능한가?"**

이 질문에 대한 답이 바로 이벤트 루프입니다. 자바스크립트 엔진 자체는 싱글 스레드지만, 브라우저나 Node.js 환경이 제공하는 API(Web API)와 이벤트 루프를 통해 비동기 작업을 처리합니다.

### 핵심 구성 요소와 동작 흐름

1. **콜 스택 (Call Stack)**: 실행할 코드(함수)가 순서대로 쌓이는 곳. 스택이 비면 프로그램이 종료됩니다.

2. **백그라운드 (Web API)**: `setTimeout`, `fetch` (API 요청) 등 시간이 걸리는 비동기 작업은 즉시 백그라운드로 보내져 처리됩니다. 자바스크립트 엔진의 일부가 아닙니다.

3. **태스크 큐 (Task Queue / Callback Queue)**: 백그라운드에서 완료된 비동기 작업의 콜백 함수가 대기하는 곳입니다.

4. **이벤트 루프 (Event Loop)**: 콜 스택이 비어 있는지 끊임없이 확인합니다. **콜 스택이 비면**, 태스크 큐에서 가장 오래된 작업을 콜 스택으로 옮겨 실행시킵니다.

```JavaScript
console.log('Start'); // 1. 콜 스택에 추가되고 실행

setTimeout(() => {
  console.log('Timeout!'); // 4. 콜 스택이 비었을 때 이벤트 루프에 의해 콜 스택으로 이동하여 실행
}, 1000); // 2. 백그라운드로 보내짐. 1초 후 콜백 함수가 태스크 큐로 이동

console.log('End'); // 3. 콜 스택에 추가되고 실행

// 출력 순서: Start -> End -> Timeout!
```

<br />

## 비동기 처리 패턴

### 콜백함수 (Callback)

비동기 작업이 완료된 후 실행될 함수를 인자로 넘기는 방식입니다. 초기의 비동기 처리 방식이었지만, 여러 비동기 작업을 순차적으로 처리해야 할 때 코드가 깊게 중첩되는 **콜백 헬(Callback Hell)** 문제가 발생합니다.

### `Promise`

콜백 헬 문제를 해결하기 위해 ES6에서 도입된 객체입니다. 비동기 작업의 **최종 성공 또는 실패**를 나타냅니다.

**Promise의 세 가지 상태**

- `pending` **(대기)**: 비동기 작업이 아직 완료되지 않은 초기 상태.

- `fulfilled` **(이행)**: 비동기 작업이 성공적으로 완료된 상태.

- `rejected` **(거부)**: 비동기 작업이 실패한 상태.

```JavaScript
const myPromise = new Promise((resolve, reject) => {
  // 비동기 작업 수행
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('성공!'); // 성공 시 resolve 호출
    } else {
      reject('실패!'); // 실패 시 reject 호출
    }
  }, 1000);
});

myPromise
  .then(result => { // 성공(fulfilled)했을 때 실행
    console.log(result); // "성공!"
  })
  .catch(error => { // 실패(rejected)했을 때 실행
    console.error(error);
  })
  .finally(() => { // 성공/실패 여부와 관계없이 항상 실행
    console.log('작업 완료');
  });

```

<br />

### async / await

`Promise`를 더욱 동기 코드처럼 깔끔하고 직관적으로 작성할 수 있게 해주는 최신 문법(ES2017)입니다.

- `async` 키워드는 함수 앞에 붙이며, 해당 함수는 항상 `Promise`를 반환합니다.

- `await` 키워드는 `async` 함수 내에서만 사용할 수 있으며, `Promise`가 처리될 때까지(성공 또는 실패) 함수의 실행을 일시 중지합니다.

```JavaScript
async function fetchData() {
  console.log('데이터 가져오는 중...');
  try {
    // await는 Promise가 처리될 때까지 기다립니다.
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error('네트워크 응답에 문제가 있습니다.');
    }
    const data = await response.json();
    console.log(data);
  } catch (error) { // Promise의 reject는 catch 블록에서 처리됩니다.
    console.error('데이터를 가져오는 데 실패했습니다:', error);
  }
}

fetchData();
```

`async/await`는 현재 비동기 처리를 위한 가장 현대적이고 권장되는 방식입니다. 내부적으로는 `Promise`를 기반으로 동작합니다.

## 참고자료

> [MDN - 이벤트 루프](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Execution_model)

> [MDN - Promise 사용하기](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Using_promises)

> [MDN - async function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)
