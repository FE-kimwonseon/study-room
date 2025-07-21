# 객체와 배열 (고차 함수)

## 객체 (Objects)

객체는 **키(key)와 값(value)으로 구성된 속성(property)들의 집합**입니다. 순서가 없는 데이터 모음이며, 현실 세계의 대상을 표현하는 데 적합합니다.

```JavaScript
const user = {
  name: 'Alex',
  age: 30,
  isAdmin: true,
  'like-color': 'blue'
};
```

### 객체 속성 다루기

- **접근**:

  - 점 표기법: `user.name` (일반적으로 사용)

  - 대괄호 표기법: `user['like-color']` (키에 특수문자가 있거나, 키가 변수일 때 사용)

- **추가/수정**: `user.age = 31;` 또는 `user.country = 'Korea';`

- **삭제**: `delete user.isAdmin;`

### 모던 객체 문법

- **단축 속성 (Property Shorthand)**: 변수 이름과 객체의 키 이름이 같을 때 축약해서 사용할 수 있습니다.

```JavaScript
const name = 'Alex';
const age = 30;
const user = { name, age }; // { name: name, age: age } 와 동일
```

- **계산된 속성명 (Computed Property Name)**: 변수 값을 객체의 키로 동적으로 사용할 수 있습니다.

```JavaScript
const key = 'country';
const user = {
  name: 'Alex',
  [key]: 'Korea' // user 객체는 { name: 'Alex', country: 'Korea' }가 됨
};
```

<br />

## 배열

배열은 **순서가 있는 데이터의 집합**입니다. 각 데이터는 인덱스(index)를 가지며, 0부터 시작합니다.

### 기본 배열 메서드

- `push(item)` / `pop()`: 배열의 **끝**에 요소를 추가 / 제거합니다. (원본 배열 변경 O)

- `unshift(item)` / `shift()`: 배열의 **앞**에 요소를 추가 / 제거합니다. (원본 배열 변경 O)

- `slice(start, end)`: 배열의 일부를 **복사**하여 새로운 배열로 반환합니다. **원본 배열을 변경하지 않습니다.**

```JavaScript
const numbers = [10, 20, 30, 40, 50];
const sliced = numbers.slice(1, 4); // [20, 30, 40]
```

- `splice(start, deleteCount, item?)`: 배열의 특정 위치에서 요소를 삭제하거나 추가합니다. **원본 배열을 직접 변경합니다.**

```JavaScript
const numbers = [10, 20, 30, 40, 50];
numbers.splice(2, 1, 99); // 인덱스 2에서 1개를 삭제하고 99를 추가 -> [10, 20, 99, 40, 50]
```

<br />

## 고차 함수

고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수입니다. 자바스크립트의 배열 메서드들은 데이터의 불변성을 지키며 선언적으로 코드를 작성하는 데 매우 중요합니다.

- `forEach(callback)`: 각 요소에 대해 주어진 콜백 함수를 실행합니다. 반환값이 없으며, 주로 각 요소를 가지고 어떤 동작을 수행할 때 사용합니다.

- `map(callback)`: 각 요소를 콜백 함수를 통해 변환하고, 그 결과로 이루어진 **새로운 배열을 반환**합니다. 원본 배열은 변경되지 않습니다. 리액트에서 배열 데이터를 UI 요소 목록으로 렌더링할 때 핵심적으로 사용됩니다.

```JavaScript
const numbers = [1, 2, 3];
const doubled = numbers.map(num => num * 2); // [2, 4, 6]
```

- `filter(callback)`: 콜백 함수가 true를 반환하는 요소들만 모아서 **새로운 배열을 반환**합니다. 특정 조건에 맞는 데이터만 걸러낼 때 사용합니다.

```JavaScript
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(num => num % 2 === 0); // [2, 4]
```

- `reduce(callback, initialValue)`: 배열의 각 요소를 순회하며 누적된 값을 계산하여 **하나의 결과값을 반환**합니다. 배열 요소의 총합, 평균, 또는 복잡한 객체 생성 등에 사용됩니다.

```JavaScript
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0); // 15
```

- `find(callback)`: 콜백 함수가 `true`를 반환하는 첫 번째 요소 하나를 반환합니다. 조건에 맞는 요소가 없으면 `undefined`를 반환합니다.

<br />

# 참고 자료

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array

> https://ko.javascript.info/array-methods
