# 제어문 (Control Flow)

프로그램의 실행 흐름을 조건이나 반복에 따라 제어하는 구문입니다.

- `if...else` 문: 가장 기본적인 조건문입니다. 조건이 복잡해지면 `else if`를 사용하여 여러 조건을 처리할 수 있습니다.

- `switch` 문: 하나의 변수 값을 여러 경우의 수와 비교할 때 `if...else`보다 가독성이 좋습니다. 각 case 끝에 break를 작성하는 것을 잊지 마세요.

- 삼항 연산자: 간단한 `if...else` 문을 한 줄로 표현할 수 있어 코드를 간결하게 만듭니다.

```JavaScript
// 삼항 연산자
// 조건 ? 참일_때_값 : 거짓일_때_값
const message = isLoggedIn ? '환영합니다!' : '로그인이 필요합니다.';
```

- `for` 문: 특정 횟수만큼 코드를 반복 실행할 때 사용합니다.

- `while` 문: 특정 조건이 참인 동안 코드를 계속해서 반복 실행할 때 사용합니다.

<br />

# 함수

함수는 특정 작업을 수행하는 독립적인 코드 블록이며, 여러 번 호출하여 재사용할 수 있습니다.

## 함수 선언 방식

- **함수 선언문 (Function Declaration)**: 가장 일반적인 형태입니다. 함수 전체가 호이스팅되어, 코드 상에서 선언된 위치보다 앞에서 호출할 수 있습니다.

```JavaScript
sayHello(); // "Hello!"

function sayHello() {
  console.log('Hello!');
}
```

- **함수 표현식 (Function Expression)**: 함수를 변수에 할당하는 방식입니다. 변수 호이스팅 규칙을 따르므로, `const`나 `let`으로 선언하면 TDZ의 영향을 받아 선언 전에 호출할 수 없습니다.

```JavaScript
// sayHi(); // ReferenceError

const sayHi = function() {
  console.log('Hi!');
};
```

> 함수 표현식은 호이스팅으로 인한 혼란을 방지하고, `const`를 사용해 함수가 재할당되는 것을 막을 수 있어 더 일관성 있고 안전한 코드를 작성하는 데 도움이 됩니다.

## 화살표 함수 (Arrow Functions)

ES6에서 도입된 더 간결한 함수 표현 방식입니다.

```JavaScript
// 기본 형태
const add = (a, b) => {
  return a + b;
};

// 코드가 한 줄이면 중괄호와 return 생략 가능
const subtract = (a, b) => a - b;
```

**function 키워드 함수와의 가장 큰 차이점:**

1. `this`: 화살표 함수는 자신만의 `this`를 갖지 않습니다. 대신, 자신을 감싸고 있는 외부 스코프(렉시컬 스코프)의 `this`를 그대로 물려받습니다. 이 특징 때문에 콜백 함수 등에서 `this` 컨텍스트를 유지하기 매우 편리합니다.

2. `arguments` 객체: 화살표 함수는 `arguments` 객체를 지원하지 않습니다. 대신, Rest 파라미터를 사용해야 합니다.

### 매개변수와 인자

- **매개변수 기본값 (Default Parameters)**: 매개변수에 기본값을 지정하여, 인자가 전달되지 않았을 때 undefined가 되는 것을 방지할 수 있습니다.

```JavaScript
function greet(name = 'Guest') {
  console.log(`Hello, ${name}!`);
}
greet(); // "Hello, Guest!"
```

- **Rest 파라미터 (`...`)**: 정해지지 않은 개수의 인자들을 배열로 모아서 받을 수 있습니다. 항상 마지막 매개변수여야 합니다.

```JavaScript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}
sum(1, 2, 3); // 6
sum(10, 20, 30, 40); // 100
```

<br />

# 참고자료

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/rest_parameters

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
