# 스코프(Scope)와 호이스팅(Hoisting)

## 스코프란?

**스코프(Scope)**는 변수나 함수에 접근할 수 있는 **유효 범위**를 의미합니다. 코드를 작성할 때 변수나 함수가 서로 충돌하지 않고 안전하게 사용될 수 있는 영역을 지정하는 규칙입니다.

JavaScript는 스코프는 크게 3가지로 나눌 수 있습니다.

**블록 스코프 (Block Scope)**

- `if` , `for` , `while` 등 중괄호`{}`로 감싸진 영역입니다. `let`과 `const`로 선언된 변수는 블록 스코프를 따르며, 이는 의도치 않은 변수 값 변경을 막아주기 때문에 권장합니다.

**전역 스코프(Global Scope)**

- 코드의 가장 바깥 영역입니다. 여기서 선언된 변수나 함수는 코드 어디서든 접근할 수 있습니다.

**지역 스코프(Local Scope or Function-Level Scope)**

- 함수 코드 블록`{}` 내부 영역입니다. var로 선언된 변수는 이 함수 스코프를 따릅니다.

```JavaScript
const global = '전역'; // 전역 스코프(Global Scope)

function myFunction () {
    const func = '함수'; // 함수 스코프(Function-Level Scope) 안에서 사용가능
    if (true) {
        const block = '블록'; // 블록 스코프 (Block Scope)
        console.log(global); // 전역 스코프 접근 가능
        console.log(func);   // 함수 스코프 접근 가능
    }

    // console.log(block); // ReferenceError 발생. 블록 밖에서 접근 불가
}

myFunction();
```

**`var`사용 시 주의점**  
`var`는 블록 스코프를 따르지 않고 함수 스코프를 따르기 때문에, 개발자의 의도와 다르게 동작할 수 있습니다.

```JavaScript
var b = 10; // 전역 변수

console.log(b); // 10
{
  var b = 20; // 이 변수는 블록 스코프가 아닌 전역 스코프의 b를 가리킴
}

console.log(b); // 20 (전역 변수 b의 값이 재할당되어 덮어써짐)
```

이러한 예측하기 어려운 문제 때문에, 모던 자바스크립트에서는 `var`사용을 지양하고 `let`과 `const`를 사용하는 것이 좋습니다.

<br />

## 호이스팅이란?

호이스팅(Hoisting)은 **자바스크립트 엔지이 코드를 실행하기 전, 변수와 함수의 선언을 해당 스코프의 최상단으로 끌어올리는 것 같은 현상**을 말합니다.

### 변수 호이스팅

`var`,`let`,`const`모두 호이스팅이 발생하지만, 동작 방식이 다릅니다.

- `var`: 선언과 동시에 `undefined`로 초기화됩니다. 따라서 선언 전에 호출해도 에러가 발생하지 않고 `undefined`가 반환됩니다.

```JavaScript
console.log(myVar); // undefined (에러가 발생하지 않음)
var myVar = 'Hello';
```

- **`let`, `const`와 TDZ(Temporal Dead Zone)**: 선언만 위로 끌어올려질 뿐, 초기화는 실제 선언 라인에서 이루어집니다. 이 때문에 **스코프의 시작 지점부터 변수 선언 라인까지 변수에 접근할 수 없는 '일시적 사각지대(TDZ)'**가 생깁니다. 이 구간에서 변수를 호출하면 `ReferenceError`가 발생합니다.

```JavaScript
console.log(myVar); // undefined (에러가 발생하지 않음)
var myVar = 'Hello';
```

<br>

### 함수 호이스팅

함수 또한 선언 방식에 따라 호이스팅 동작이 다릅니다.

- **함수 선언식 (Function Declaration): 함수 전체가 호이스팅됩니다.** 따라서 코드 상에서 선언된 위치보다 앞에서 호출해도 정상적으로 작동합니다.

```JavaScript
sayHello(); // "Hello" (정상 작동)

function sayHello() {
    console.log('Hello');
}
```

- **함수 표현식(Function Expression): 변수 호이스팅 규칙을 그대로 따릅니다.** `const`나 `let`으로 선언하면 TDZ의 영향을 받습니다. `var`로 선언하면 변수는 `undefined`로 초기화되므로 함수가 아니라는 `TypeError`가 발생합니다.

```JavaScript
sayHi(); // ❌ ReferenceError: Cannot access 'sayHi' before initialization (TDZ)

const sayHi = function() {
  console.log('Hi!');
};
```

호이스팅으로 인한 혼란을 줄이기 위해, \*\*변수와 함수는 항상 해당 스코프의 최상단에 선언하고 사용하는것이 좋습니다.이는 코드으 가독성을 높이고 버그를 줄이는 좋은 습관입니다.
