# State와 생명주기

`Props`가 부모 컴포넌트로부터 전달받는 읽기 전용 데이터라면, `State`는 **컴포넌트 내부에서 자체적으로 관리되고, 변화할 수 있는 데이터**입니다. 사용자의 입력이나 네트워크 응답처럼 시간에 따라 UI가 변해야 할 때 `State`를 사용합니다.

State의 가장 중요한 특징은 **State가 변경되면, 리액트가 컴포넌트를 다시 렌더링(re-render)하여 UI를 자동으로 업데이트**한다는 점입니다.

<br />

## `useState` Hook: 컴포넌트에 State 추가

`useState`는 함수 컴포넌트에서 State를 사용할 수 있게 해주는 리액트 **Hook**입니다.

```jsx
import { useState } from "react";

function Counter() {
  // useState를 호출하여 state 변수를 선언합니다.
  const [count, setCount] = useState(0); // 0은 count의 초깃값입니다.

  function handleClick() {
    // state를 변경할 때는 항상 setState 함수를 사용해야 합니다.
    setCount(count + 1);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

### `useState`의 구조

`useState`를 호출하면 두 가지 값을 가진 배열을 반환합니다.

1. 현재 State 값 (`count`): State의 현재 값을 담고 있는 변수입니다.

2. State 설정 함수 (`setCount`): State를 새로운 값으로 업데이트하고, 컴포넌트의 리렌더링을 유발하는 함수입니다.

`const [count, setCount] = ...` 와 같은 문법은 자바스크립트의 **"배열 구조 분해 할당"** 입니다. `useState(0)`이 반환하는 배열의 첫 번째와 두 번째 요소를 각각 `count`와 `setCount`라는 이름의 변수에 할당하는 것입니다.

<br />

## State 사용의 핵심 규칙

1.**State는 직접 수정하면 안 된다 (Immutability)**

**가장 중요한 규칙입니다.** State 변수를 직접 변경하려고 시도하면 리액트가 변화를 감지하지 못해 리렌더링이 일어나지 않습니다.

- **잘못된 예**

```jsx
// 이렇게 직접 수정하면 안 됩니다!
count = count + 1;
```

- **올바른 예**

```jsx
// 항상 setState 함수를 사용해야 합니다.
setCount(count + 1);
```

이 원칙을 **불변성(Immutability)** 이라고 합니다. 리액트는 `setCount`와 같은 설정 함수가 호출되어야만 State가 변경되었다고 인지하고 화면을 다시 그릴 준비를 합니다.

### State 업데이트는 비동기적일 수 있다.

리액트는 성능을 위해 여러 State 업데이트를 하나로 묶어서 처리(batching)할 수 있습니다. `setCount`를 호출한 직후에 State 값이 바로 반영되지 않을 수 있습니다.

```jsx
function handleClick() {
  setCount(count + 1);
  console.log(count); // 여기서는 이전 count 값이 출력될 수 있습니다.
}
```

만약 이전 State 값을 기반으로 다음 State를 계산해야 한다면, **함수형 업데이트**를 사용하는 것이 안전합니다.

- **안전한 예(함수형 업데이트)**

```jsx
// setCount에 값을 직접 넣는 대신, 함수를 전달합니다.
// 이 함수의 첫 번째 인자(prevCount)는 항상 가장 최신 상태의 state를 보장받습니다.
setCount((prevCount) => prevCount + 1);
```

<br />

## 컴포넌트의 생명주기 (Lifecycle)

컴포넌트는 생성되고(mount), 업데이트되며(update), 사라지는(unmount) 과정을 거칩니다. 이를 **컴포넌트의 생명주기**라고 합니다.

- **Mount (생성)**: 컴포넌트가 처음으로 DOM에 렌더링될 때.

- **Update (업데이트)**: `props`나 `state`가 변경되어 컴포넌트가 리렌더링될 때.

- **Unmount (소멸)**: 컴포넌트가 DOM에서 제거될 때.

과거에는 클랙스 컴포넌트의 생명주기 메소드(e.g., `componentDidMount`)를 사용했지만 함수 컴포넌트에서는 `useEffect` **Hook**을 사용하여 이러한 생명주기의 특정 시점에 원하는 작업을 수행할 수 있습니다.

<br />

### Props vs State

| 구분 | Props                                | State                                                |
| ---- | ------------------------------------ | ---------------------------------------------------- |
| 소유 | 부모 컴포넌트                        | 컴포넌트 자신                                        |
| 변경 | 변경할 수 없음 (읽기 전용)           | 변경할 수 있음 (setState 함수 사용)                  |
| 역할 | 부모에서 자식으로 데이터와 기능 전달 | 컴포넌트 내부의 상호작용으로 인해 변하는 데이터 관리 |
| 흐름 | 단방향 데이터 흐름 (Top-down)        | 컴포넌트 내부에서 시작됨                             |

<br />

# 참고 자료

> [React 공식 문서 - State: 컴포넌트의 메모리](https://ko.react.dev/learn/state-a-components-memory)

> [React 공식 문서 - State를 사용해 상호작용 추가하기](https://ko.react.dev/learn/adding-interactivity)
