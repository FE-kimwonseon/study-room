# `useEffect`로 부수 효과(Side Effect) 처리하기

리액트 컴포넌트의 주된 역할은 `props`와 `state`를 기반으로 UI를 그리는(렌더링) 것입니다. 하지만 때로는 컴포넌트의 주요 역할과 관계없는 다른 작업들을 처리해야 할 때가 있습니다.

**ex)**

- 서버에서 데이터 가져오기 (Data fetching)

- 외부 라이브러리(e.g., Chart.js)사용

- 타이머 설정 (`setInterval`, `setTimeout`)

- DOM을 직접 조작해야 할 떼

이처럼 **렌더링과 직접적인 관련이 없는 모든 작업을 "부수 효과(Side Effect)"** 라고 부릅니다. `useEffect`는 함수 컴포넌트 안에서 이러한 부수 효과들을 처리할 수 있게 해주는 Hook입니다.

<br />

## `useEffect`의 기본 구조

`useEffect`는 두 개의 인자를 받습니다. **실행할 함수(effect)** 와 **의존성 배열(dependency array).**

```jsx
import { useEffect, useState } from "react";

useEffect(
  () => {
    // 부수 효과를 처리하는 코드 (Effect 함수)
    // 이 함수는 렌더링이 완료된 후에 실행됩니다.
  },
  [
    /* 의존성 배열 */
  ]
);
```

1. **Effect 함수**: 첫 번째 인자로 전달하는 함수입니다. 리액트가 렌더링을 마치고 DOM을 업데이트한 후에 이 함수를 실행합니다.

2. **의존성 배열**: 두 번째 인자로 전달하는 배열입니다. `useEffect`가 언제 다시 실행되어야 하는지를 리액트에게 알려주는 역할을 합니다. **이 배열을 어떻게 설정하느냐에 따라 `useEffect`의 동작 방식이 완전히 달라집니다.**

<br />

## 의존성 배열(Dependency Array)

의존성 배열은 `useEffect`의 실행 시점을 제어하는 핵심적인 부분입니다.

1. **배열을 생략할 경우**

```jsx
useEffect(() => {
  // 컴포넌트가 렌더링될 때마다 실행됩니다.
  console.log("I run on every render");
});
```

의존성 배열을 아예 전달하지 않으면, **컴포넌트가 렌더링될 때마다** Effect 함수가 실행됩니다. 이는 불필요한 재실행을 유발하고, 무한 루프에 빠질 위험이 있어 거의 사용하지 않는 패턴입니다.

2. **빈 배열 `[]`을 전달할 경우**

```jsx
useEffect(() => {
  // 컴포넌트가 처음 렌더링(mount)될 때 딱 한 번만 실행됩니다.
  console.log("I run only once");
});
```

빈 배열을 전달하면, Effect 함수는 **컴포넌트가 처음 화면에 나타날 때 딱 한 번만 실행**됩니다. 주로 최초에 데이터를 불러오거나, 이벤트 리스너를 설정하는 등의 작업에 사용됩니다.

3. 배열에 특정 값 `[prop, state]`을 전달할 경우

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  // 처음 렌더링될 때 + 'count' state가 변경될 때마다 실행됩니다.
  document.title = `You clicked ${count} times`;
}, [count]); // 'count'를 의존성으로 등록
```

배열 안에 특정 props나 state값을 넣으면, Effect 함수는 **처음 렌더링될 때 한 번 실행되고, 그 이후에는 배열 안의 값이 변경될 때 마다 다시 실행**됩니다. 리액트는 이전 렌더링과 현재 렌더링의 의존성 배열 값을 비교하여 함수의 재실행 여부를 결정합니다.

<br />

## 클린업(Cleanup) 함수

컴포넌트는 언젠가 화면에서 사라집니다(unmount). 이때 우리가 설정했던 부수 효과들을 정리해야 할 필요가 있습니다. 예를 들어, 설정했던 타이머를 해제하거나, 추가했던 이벤트 리스너를 제거하지 않으면 메모리 누수(memory leak)가 발생할 수 있습니다.

`useEffect`는 Effect 함수 안에서 **함수를 반환(return)** 함으로써 클린업 함수를 정의할 수 있습니다.

```jsx
useEffect(() => {
  // Effect: 타이머 설정
  const timerId = setInterval(() => {
    console.log("Timer ticking...");
  }, 1000);

  // Cleanup 함수: 컴포넌트가 사라지기 전에 실행됩니다.
  return () => {
    clearInterval(timerId);
    console.log("Timer cleared!");
  };
}, []); // 처음 한 번만 실행
```

### 클린업 함수의 실행 시점

1. 컴포넌트가 화면에서 사라질 때 (Unmount)

2. (의존성 배열이 있는 경우) Effect 함수가 재실행되기 직전에, 이전 Effect를 정리하기 위해 실행됩니다.

<br />

## 정리: `useEffect`는 "동기화"다

`useEffect`를 단순히 '생명주기'에 맞춰 무언가를 실행하는 도구로만 생각하기보다, **"리액트 컴포넌트의 상태(state, props)를 외부세계(API, DOM, 라이브러리)와 동기화하는 도구"** 라고 이해하는 것이 더 정확합니다.

- `document.title`을 `count`state와 **동기화**한다.

- 채팅방 컴포넌트의 상태를 외부 채팅 서버와 **동기화** 한다.

<br />

# 참고 자료

> [React 공식 문서 - Effect: 생명주기 동기화하기](https://ko.react.dev/learn/synchronizing-with-effects)

> [React 공식 문서 - useEffect Hook 참조](https://ko.react.dev/reference/react/useEffect)
