# 1. 변수와 상수 (Variables & Constant)

## 변수

**변수(variable)**는 값(value)을 저장(할당)하고 그 저장된 값을 참조하기 위해 사용합니다. 한번 쓰고 버리는 값이 아닌 유지할 필요가 있는 값은 변수에 담아 사용합니다.

`let` 선언은 블록 유효 범위의 지역 변수를 선업합니다.
선언과 동시에 임의의 값으로 초기화 할 수 있고 이후 값 변경도 가능합니다.

```JavaScript
let y; // 변수의 선언
y = 15; // y는 15의 정수값의 할당
```

<br />

## 상수

상수에 대한 스코프 규칙은 `let` 블록 스코프 변수와 동일합니다. 만약 `const` 키워드가 생략된 경우에는, 식별자는 변수를 나타내는 것으로 간주됩니다.

`const` 선언은 블록 범위의 상수를 선언합니다. 상수의 값은 재할당할 수 없으며 다시 선언할 수도 없습니다.

```JavaScript
const myBirthday = '26.05.2001';
myBirthday = '09.08.2007'; // error, can't reassign the constant
```

<br />

## 변수와 상수의 차이점

**변수**는 저장한 값을 변경할 수 있습니다.(재할당 한다)

```JavaScript
let age = 30;

age = 31; // 값 변경
age = 32; // 값 변경
```

**상수**는 저장한 값을 변경할 수 없습니다.(재할당 할 수 없다)

```JavaScript
const PI = 3.14159;

// ReferenceError 발생
PI = 6.28318;
```

그러나, `const`변수의 타입이 객체인 경우, 객체에 대한 참조(Reference)를 변경하지 못하지만 **객체의 프로퍼티는 보호되지 않는다**. 다시 말하자면 재할당은 불가능하지만 할당된 객체의 내용(프로퍼티 추가, 삭제, 값의 변경)은 변경할 수 있다.

```JavaScript
// const는 오브젝트에도 잘 동작합니다.
const MY_OBJECT = {key: "value"};

// 오브젝트를 덮어쓰면 에러가 발생합니다.
MY_OBJECT = {OTHER_KEY: "value"};

// 객체의 내용(property)은 변경가능
MY_OBJECT.key = "other value";
```

<br />

### 변수선언 vs 상수선언

변수 선언에는 기본적으로 `const`를 사용하고 `let`은 재할당이 필요한 경우에 한정해 사용하는 것이 좋습니다. 원시 값의 경우, 가급적 상수를 사용하는 편이 좋다. 왜냐하면 변수라고 하는 것은 값이 변한다는 뜻인데 값 자체가 변화 된다는 거 자체가 굉장히 많은 불안정성을 내포하고 있다. `const` 키워드를 사용하면 의도칙 않은 재할당을 방지할 수 있기 때문에 보다 안전하다.

`var`,`let` 그리고 `const`는 다음처럼 사용하는 것을 추천한다.

- ES6을 사용한다면 var키워드는 사용하지 않는다.
- 재할당이 필요한 경우에 한정해 `let` 키워드를 사용한다. 이때 변수의 스코프는 최대한 좁게 만든다.
- 변경이 발생하지 않는(재할당이 필요 없는 상수) 원시 값과 객체에는 `const` 키워드를 사용한다. `const` 키워드는 재할당을 금지하므로 `var`,`let` 보다 안전하다.

변하지 않는 값으로 프로그래밍을 하는 습관을 길들이자. `const`로 선언하고 값이 변한다고 할 때만 `let`으로 변수를 만들어서 사용해야 합니다.

<br />

## 참고자료

> https://ko.javascript.info/variables

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/let

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/const

> https://poiemaweb.com/es6-block-scope
