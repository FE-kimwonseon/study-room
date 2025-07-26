# JS핵심 메서드 (React 활용)

| `map()` | `filter()` | `reduce()` | `find()` | `some()vsevery()` | `Object`메서드 |
| ------- | ---------- | ---------- | -------- | ----------------- | -------------- |

<br />

## `map()`

배열의 모든 요소를 순회하면서, 각 요소를 **다른 형태의 값으로 변환하여 새로운 배열을 만들고 싶을 때** 사용합니다.

- **동작 방식**

  1. 새로운 빈 배열을 생성합니다.
  2. 원본 배열의 각 요소를 처음부터 끝까지 순서대로 순회합니다.
  3. 각 요소마다 콜백 함수를 실행하고, 그 반환값을 새로운 배열에 순서대로 추가합나다.
  4. 순회가 끝나면, 새로운 배열을 최종적으로 반환합니다.(원본 배열은 절대 변경되지 않습니다.)

- **ex)** 서버에서 받은 사용자 객체 배열에서 이름만 추출하여 새로운 배열 만들기

```JavaScript
const user = [
    {id : 1, name: 'Alex', age: 30},
    {id : 2, name: 'Bob', age: 26},
    {id : 3, name: 'Chris', age: 35},
];

const userNames = user.map(user => user.name);
// 결과: ['Alex', 'Bob', 'Chris']
```

**React 활용 팁**

배열 데이터를 UI 컴포넌트 목록으로 렌더링 할 때 절대적으로 필요한 메서드입니다. `map`을 사용하여 각 데이터 요소를 JSX 엘리먼트로 변환하고, `key`prop을 각 엘리먼트에 부여해야합니다

```JavaScript
<ul>
  {users.map(user => (
    <li key={user.id}>{user.name} ({user.age}세)</li>
  ))}
</ul>
```

### 참고자료

> [MDN Web Docs - Array.prototype.map()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

> [React 공식 문서](https://ko.react.dev/learn/rendering-lists)

<br />

## `filter()`

배열의 모든 요소 중에서 **특정 조건을 만족하는 요소들만 걸러내어 새로운 배열을 만들고 싶을 때** 사용합니다.

- **동작 방식**

  1. 새로운 빈 배열을 생성합니다.
  2. 원본 배열의 각 요소를 순회합니다.
  3. 각 요소마다 콜백 함수를 실행하여 **반환값이 `true`인 요소만** 새로운 배열에 추가합니다.
  4. 순회가 끝나면, 조건에 맞는 요소들로 구성된 새로운 배열을 반환합니다.

- **ex)** 사용자 목록에서 30세 이상인 사용자만 필터링하기

```JavaScript
const users = [
    {id : 1, name: 'Alex', age: 30},
    {id : 2, name: 'Bob', age: 26},
    {id : 3, name: 'Chris', age: 35},
];

const adults = users.filter(user => user.age >= 30);
// 결과: [{id: 1, name: 'Alex', age: 30}, {id: 3, name: 'Chris', age: 35}]
```

**React 활용 팁**

검색 기능이나 카테고리 필터링 등 사용자의 입력에 따라 보여줄 목록을 동적으로 변경할 때 매우 유용합니다. 전체 목록(원본 상태)은 그대로 유지한 채, 필터링된 결과만 화면에 렌더링할 수 있습니다.

### 참고 자료

> [MDN Web Docs - Array.prototype.filter()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

<br />

## `reduce()`

배열의 모든 요소를 순회하며 하나의 결과값을 만들고 싶을 때 사용합니다. (예: 총합, 평균, 복잡한 객체 생성 등)

- **동작 방식**

  1. `reduce(callback, initialValue)` 두 개의 인자를 받습니다.
  2. `callback` 함수는 `(accumulator, currentValue)` 두 개의 주요 인자를 받습니다.

  - `accumulator` (누산기): 이전 콜백 호출의 반환값이 누적되는 변수. `initialValue`가 있으면 그 값으로, 없으면 배열의 첫 번째 요소로 초기화됩니다.
  - `currentValue` (현재 값): 현재 처리 중인 배열 요소.

  3. 배열의 각 요소를 순회하며 a`ccumulator`에 값을 계속 누적한 뒤, 최종 `accumulator`값을 반환합니다.

- **ex)** 장바구니에 담긴 상품들의 총 가격 계산하기

```JavaScript
const cartItems = [
  { name: 'Laptop', price: 1200 },
  { name: 'Mouse', price: 25 },
  { name: 'Keyboard', price: 75 }
];

const totalPrice = cartItems.reduce((sum, item) => sum + item.price, 0); // 초기값 0이 중요
// 결과: 1300
```

**React 활용 팁**

복잡한 계산이 필요하거나, 배열을 다른 형태의 객체로 변환해야 할 때 유용합니다. 예를 들어, `['a', 'b', 'c']` 배열을 `{ a: true, b: true, c: true }` 객체로 변환할 수 있습니다.

### 참고 자료

> [MDN Web Docs - Array.prototype.reduce()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

> [javascript.info - 배열과 메서드](https://www.google.com/search?q=https://ko.javascript.info/array-methods%23reduce-reduceright)

<br />

## `find{}`

- 배열에서 특정 조건을 만족하는 **첫 번째 요소 하나**를 찾고 싶을 대 사용합니다.

- **동작 방식**

  1. 배열의 각 요소를 순회하며 콜백 함수를 실행합니다.
  2. 콜백 함수의 **반환값이 `true`가 되는 첫 번째 요소를 발견하는 즉시, 그 요소를 반환하고 순회를 멈춥니다.**
  3. 조건에 맞는 요소가 없으면 `undefined`를 반환합니다.

- **ex)** ID가 2인 사용자 정보 찾기

```JavaScript
const users = [
  { id: 1, name: 'Alex' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Chris' }
];

const targetUser = users.find(user => user.id === 2);
// 결과: { id: 2, name: 'Bob' }
```

**React 활용 팁**

URL 파라미터(e.g., `/users/2`)에 해당하는 데이터를 전체 목록에서 찾아 상세 페이지에 표시할 때 매우 유용합니다.

### 참고 자료

> [MDN Web Docs - Array.prototype.find()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/find)

<br />

## `some()` vs `every()`

- `some()` : 배열의 요소 중 하나라도 특정 조건을 만족하는지 확인하고 싶을 때(`true` / `false` 반환).
- `every()` : 배열의 모든 요소가 특정 조건을 만족하는지 확인하고 싶을 때 (`true` / `false` 반환)

- **동작 방식**

  - `some()` : 조건을 만족하는 요소를 하나라도 찾으면 즉시 `true`를 반환하고 순회를 멈춥니다.
  - `every()` : 조건을 만족하지 않는 요소를 하나라도 찾으면 즉시 `false`를 반환하고 순회를 멈춥니다.

- **ex)**

```JavaScript
const scores = [80, 95, 72, 100, 68];

// 100점 맞은 학생이 있는지?
const hasPerfectScore = scores.some(score => score === 100); // true

// 모든 학생이 과락(60점)을 면했는지?
const allPassed = scores.every(score => score >= 60); // true
```

**React 활용 팁**
사용자 권한 체크나 폼 유효성 검사 등에서 특정 조건을 만족하는지 여부를 간단히 확인하여 UI를 다르게 보여주거나 특정 동작을 막는 데 사용합니다. (예: 권리자 권한을 가진 사용자가 한 명이라도 있는가?)

### 참고자료

> [MDN Web Docs - Array.prototype.some()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/some)

> [MDN Web Docs - Array.prototype.every()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/every)

<br />

# 객체 메서드

## `Object.keys()`, `Object.values()`, `Object.entries()`

객체의 키, 값, 또는 [키, 값] 쌍을 배열 형태로 다루고 싶을 때 사용합니다. 객체는 map이나 filter 같은 배열 메서드를 직접 사용할 수 없으므로, 이 메서드들을 통해 배열로 변환한 후 처리하는 경우가 많습니다.

- **동작 방식**

  - `Object.keys(obj)`: 객체의 키(key)들만 모아서 새로운 배열을 반환합니다.
  - `Object.values(obj)`: 객체의 값(value)들만 모아서 새로운 배열을 반환합니다.
  - `Object.entries(obj)`: `[key, value]` 쌍으로 이루어진 2차원 배열을 반환합니다.

- **ex)**

```JavaScript
const user = {
  name: 'Alex',
  age: 30,
  country: 'Korea'
};

const keys = Object.keys(user);       // ['name', 'age', 'country']
const values = Object.values(user);   // ['Alex', 30, 'Korea']
const entries = Object.entries(user); // [['name', 'Alex'], ['age', 30], ['country', 'Korea']]
```

**React 활용 팁**

`Object.entries()`는 객체를 map으로 순회하여 UI를 렌더링할 때 특히 유용합니다.

```JavaScript
<div>
  {Object.entries(user).map(([key, value]) => (
    <p key={key}><strong>{key}:</strong> {value}</p>
  ))}
</div>
```

### 참고 자료

> [MDN Web Docs - Object static methods](https://www.google.com/search?q=https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object%23static_methods)

> [javascript.info - 객체를 배열처럼 다루기](https://www.google.com/search?q=https://ko.javascript.info/object-keys-values-entries)

<br />

## JSON.stringify() vs JSON.parse()

- `JSON.stringify()`: 자바스크립트 객체나 배열을 JSON 문자열로 변환할 때 사용합니다. (주로 서버로 데이터를 보내거나, `localStorage`에 저장할 때)

- `JSON.parse()`: JSON 문자열을 다시 자바스크립트 객체나 배열로 변환할 때 사용합니다. (주로 서버에서 받은 데이터나, `localStorage`에서 읽은 데이터를 사용할 때)

- **동작 방식**

  - `JSON.stringify()`: 객체의 참조 관계를 끊고 값만 가진 문자열로 직렬화합니다.
  - `JSON.parse()`: 문자열을 파싱하여 새로운 객체나 배열을 생성합니다.

- **ex)** 객체의 깊은 복사(Deep Copy)

객체는 참조로 복사되기 때문에, 스프레드 문법(`...`)은 1단계 깊이의 복사만 가능합니다. 중첩된 객체까지 완벽하게 복사하여 참조를 끊고 싶을 때 이 조합을 간단하게 사용할 수 있습니다.

```JavaScript
const originalUser = {
  name: 'Alex',
  address: {
    city: 'Seoul'
  }
};

const copiedUser = JSON.parse(JSON.stringify(originalUser));

copiedUser.address.city = 'Busan';

// copiedUser를 변경해도 originalUser는 영향을 받지 않음
console.log(originalUser.address.city); // 'Seoul'
```

> **주의할 점** : 이 방법은 함수, `undefined`, `Symbo`l 등 JSON으로 표현할 수 없는 속성들은 복사 과정에서 유실되므로, 순수한 데이터 객체에만 사용하는 것이 안전합니다.

### 참고자료

> [MDN Web Docs - JSON](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON)

> [The Modern JavaScript Tutorial - JSON methods](https://javascript.info/json)
