# 자료형

자바스크립트의 모든 데이터는 크게 원시 타입과 객체 타입 두 가지로 나뉩니다. 이 둘의 가장 큰 차이점은 **"값이 어떻게 저장되고 복사되는가"** 입니다.

## 원시 타입

변수에 값(Value) 자체가 저장됩니다. 다른 변수에 할당하면 값 자체가 복사되어, 서로에게 영향을 주지 않습니다. 한 번 생성되면 값을 변경할 수 없는 **불변성(Immutable)** 을 가집니다.

- `String`: "Hello World"와 같은 문자열.

- `Number`: 정수, 실수를 포함한 모든 숫자.

- `Boolean`: `true` 또는 `false`.

- `null`: "값이 없음"을 개발자가 의도적으로 명시한 값.

- `undefined`: 값이 할당되지 않은 상태.

- `Symbol`: 유일하고 변경 불가능한 값으로, 주로 객체 속성의 충돌을 방지하기 위해 사용됩니다.

- `BigInt`: `Number`가 표현할 수 있는 한계를 넘어서는 매우 큰 정수를 다룰 때 사용합니다.

```JavaScript
// 원시 타입은 값 자체가 복사됩니다.
let a = 100;
let b = a;

b = 200; // b를 변경해도 a는 영향을 받지 않습니다.

console.log(a); // 100
console.log(b); // 200
```

<br />

## 객체 타입

데이터의 실제 값이 아닌, 데이터가 저장된 메모리의 위치(참조, Reference)가 저장됩니다. 다른 변수에 할당하면 이 메모리 주소가 복사됩니다.

- `Object`: `{}` 키와 값으로 이루어진 데이터의 집합.

- `Array`: `[]` 순서가 있는 데이터의 집합.

- `Function`: 실행 가능한 코드 블록.

```JavaScript
// 객체 타입은 메모리 주소(참조)가 복사됩니다.
let user1 = { name: 'Alex' };
let user2 = user1; // user1의 메모리 주소를 user2에 복사

user2.name = 'Bob'; // user2를 통해 값을 변경하면...

// ...user1과 user2가 같은 메모리 주소를 바라보고 있으므로, user1도 변경됩니다.
console.log(user1.name); // 'Bob'
```

<br />

# 참고 자료

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Data_structures
