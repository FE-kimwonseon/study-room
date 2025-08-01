# 컴포넌트와 prop

<br />

## 컴포넌트란?

컴포넌트는 **UI를 구성하는 독립적이고 재사용 가능한 조각**입니다. 웹 페이지를 하나의 거대한 HTML 파일로 만드는 대신, 헤더, 사이드바, 게시물 목록, 버튼 등 의미 있는 단위로 잘게 쪼개어 관리합니다.

### 컴포넌트 기반 개발의 장점:

- **재사용성**: 버튼이나 프로필 카드처럼 반복적으로 사용되는 UI를 한 번만 만들어두고 여러 곳에서 재사용할 수 있습니다.

- **유지보수**: UI의 특정 부분을 수정해야 할 때, 전체 코드를 헤맬 필요 없이 해당 컴포넌트 파일만 수정하면 됩니다.

- **가독성**: 앱의 구조를 쉽게 파악할 수 있으며, 각 컴포넌트는 자신만의 역할에만 집중하므로 코드가 간결해집니다.

<br />

## 함수 컴포넌트

현재 리액트에서는 **함수 컴포넌트**를 사용하는 것이 표준입니다. 이름 그대로 자바스크립트 함수 형태로 작성하며, Props라는 객체를 인자로 받아 UI를 설명하는 **리액트 엘리먼트(JSX)** 를 반환합니다.

```jsx
// App.js (부모 컴포넌트)
import React from "react";
import Welcome from "./Welcome"; // 자식 컴포넌트를 불러옴

function App() {
  return (
    <div>
      {/* Welcome 컴포넌트를 세 번 재사용 */}
      <Welcome name="Alex" />
      <Welcome name="Bob" />
      <Welcome name="Chris" />
    </div>
  );
}

export default App;
```

```jsx
// Welcome.js (자식 컴포넌트)
import React from "react";

// props 객체를 인자로 받음
function Welcome(props) {
  // props 객체에서 name 값을 추출하여 사용
  return <h1>Hello, {props.name}</h1>;
}

export default Welcome;
```

### 중요

- 컴포넌트의 이름은 항상 **대문자**로 시작해야 합니다. (e.g., `Welcome`, `MyButton`) 리액트는 소문자로 시작하는 태그를 일반적인 HTML 태그로 인식합니다.

- 컴포넌트는 항상 UI 조각(JSX)를 반환해야 합니다.

<br />

## props: 컴포넌트에 데이터 전달

**Props(Properties의 줄임말)** 는 부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하기 위한 **읽기 전용(Read-Only)** 객체입니다.

자식 컴포넌트는 마치 함수의 인자처럼 Props를 전달받아, 그 값에 따라 다른 내용이나 모양의 UI를 보여줄 수 있습니다.

### 동작 방식

1. 부모 컴포넌트에서 자식 컴포넌트를 사용할 때, HTML의 속성(attribute)처럼 `key="value"` 형식으로 데이터를 전달합니다.

```jsx
<Profile username="Alex" age={30} />
```

2. 리액트는 이 속성들을 모아 하나의 자바스크립트 객체, 즉 **Props 객체**로 만들어 자식 컴포넌트에 전달합니다.

```jsx
// Profile 컴포넌트가 받게 될 props 객체
{
  username: "Alex",
  age: 30
}
```

3. 자식 컴포넌트는 `props` 매개변수를 통해 이 객체에 접근하여 값을 사용합니다.

```jsx
// Profile.js
import React from "react";

// props를 통해 username과 age를 받음
function Profile(props) {
  return (
    <div>
      <h2>{props.username}</h2>
      <p>나이: {props.age}</p>
    </div>
  );
}

// 구조 분해 할당을 사용하면 더 깔끔하게 작성할 수 있습니다.
function Profile({ username, age }) {
  return (
    <div>
      <h2>{username}</h2>
      <p>나이: {age}</p>
    </div>
  );
}

export default Profile;
```

 <br />

## Props는 읽기 전용이다 (Props are Read-Only)

**매우 중요한 원칙입니다.** 자식 컴포넌트는 **절대 전달받은 Props를 직접 수정해서는 안 됩니다.** 모든 리액트 컴포넌트는 자신의 Props에 관해서는 순수 함수(Pure Function)처럼 동작해야 합니다. 즉, 동일한 입력(Props)에 대해서는 항상 동일한 결과(UI)를 반환해야 합니다.

- 잘못된 예 (Props룰 직접 수정하려는 시도):

```jsx
function Welcome(props) {
  props.name = "Bob"; // 절대 이렇게 하면 안 됩니다! 에러 발생.
  return <h1>Hello, {props.name}</h1>;
}
```

UI를 변경하고 싶다면, Props를 바꾸는 것이 아니라 다음 단계에서 배울 `State`를 사용해야 합니다. 데이터의 흐름은 항상 **"위에서 아래로 (Top-down)"**, 즉 부모에서 자식으로만 향합니다.

<br />

## `children` prop

컴포넌트 태그를 열고 닫는 사이에 작성된 내용은 `children`이라는 특별한 Prop으로 전달됩니다. 이는 다른 컴포넌트를 감싸는 "컨테이너"나 "박스" 형태의 컴포넌트를 만들 때 매우 유용합니다.

```jsx
// Card.js
function Card(props) {
  return (
    <div className="card">
      {props.children} {/* 자식 요소들이 이 위치에 렌더링됨 */}
    </div>
  );
}

// App.js
function App() {
  return (
    <Card>
      {/* 이 내용이 Card 컴포넌트의 children prop으로 전달됨 */}
      <h1>오늘의 명언</h1>
      <p>리액트는 재미있다.</p>
    </Card>
  );
}
```

<br />

# 참고 자료

> [React 공식 문서 - 첫 컴포넌트](https://ko.react.dev/learn/your-first-component)

> [React 공식 문서 - 컴포넌트에 Props 전달하기](https://ko.react.dev/learn/passing-props-to-a-component)
