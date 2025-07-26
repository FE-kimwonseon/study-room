# 클로저와 this

<br />

## 클로저 (Closure)

클로저는 함수와 그 **함수가 선언된 렉시컬 환경(Lexical Environment)의 조합**입니다.

**핵심 원리**

- 함수는 자신이 선언된 환경(스코프)을 **"기억"** 합니다. 따라서 그 함수가 외부 스코프에서 호출되더라도, 자신이 선언되었던 환경의 변수에 계속 접근할 수 있습니다.

```JavaScript
function createCounter() {
  let count = 0; // 외부 함수의 변수 (자유 변수)

  return function() { // 내부 함수 (클로저)
    count++;
    console.log(count);
  };
}

const counter = createCounter(); // createCounter 함수 실행은 끝났지만,
                               // 내부 함수는 count 변수가 있는 환경을 기억합니다.

counter(); // 1
counter(); // 2
counter(); // 3
```

**사용 사례:**

- **상태 은닉 (Information Hiding)**: 위 예제처럼 `count` 변수는 외부에서 직접 접근할 수 없고, 오직 `counter` 함수를 통해서만 제어할 수 있습니다. 이를 통해 비공개(private) 변수를 흉내 낼 수 있습니다.

- **데이터 보존**: 특정 값을 기억하고 유지해야 할 때 사용됩니다.

<br />

## `this`

`this`는 함수가 호출되는 방식에 따라 동적으로 결정되는 특별한 키워드입니다. `this`는 함수 자신을 호출한 객체를 가리킵니다.

### `this`가 결정되는 4가지 규칙

1. **객체의 메서드로 호출될 때 (`Method Invocation`)**

   `this`는 해당 메서드를 소유한 객체를 가리킵니다.

```JavaScript
const user = {
  name: 'Alex',
  sayHi() {
    console.log(`Hi, I'm ${this.name}`); // this는 user 객체
  }
};
user.sayHi(); // "Hi, I'm Alex"
```

2. **일반 함수로 호출될 때 (`Function Invocation`)**

   `this`는 전역 객체(`window` in browser)를 가리킵니다. 단, `strict mode`(엄격 모드)에서는 `undefined`가 됩니다. 이 동작 방식은 종종 버그의 원인이 됩니다.

```JavaScript
function sayName() {
  console.log(this.name); // this는 window 객체
}
window.name = 'Global';
sayName(); // "Global"
```

3. **생성자 함수(`new`)로 호출될 때 (`Constructor Invocation`)**

   `new` 키워드로 함수를 호출하면, `this`는 새로 생성되는 **인스턴스**를 가리킵니다.

```JavaScript
function User(name) {
  this.name = name; // this는 새로 생성될 인스턴스 (alex)
}
const alex = new User('Alex');
console.log(alex.name); // "Alex"
```

4. **화살표 함수에서 사용될 때 (`Arrow Function`)**

   화살표 함수는 자신만의 `this`를 갖지 않습니다. 대신, 자신을 감싸고 있는 **외부 스코프의 `this`를 그대로 물려받습니다**(**Lexical** `this`).

```JavaScript
const user = {
  name: 'Alex',
  friends: ['Bob', 'Chris'],
  printFriends() {
    this.friends.forEach(friend => {
      // forEach의 콜백 함수가 화살표 함수이므로,
      // this는 바깥 함수인 printFriends의 this(user 객체)를 그대로 사용합니다.
      console.log(`${this.name} and ${friend} are friends.`);
    });
  }
};
user.printFriends();
```

<br />

# 참고 자료

> [MDN - 클로저](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures)

> [MDN - this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)
