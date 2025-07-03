# 이벤트

이벤트(Event)는 무언가 일어났다는 신호입니다. 모든 DOM 노드는 이런 신호를 만들어 냅니다. 참고로, 이벤트는 DOM에만 한정되진 않습니다. 사용자의 특정 행동(클릭, 스크롤, 키 입력 등)에 반응하여 코드가 실행됩니다.

<br />

## 이벤트의 종류

**마우스 이벤트:**

* `click` – 마우스 버튼을 클릭했을 때
* `dbclick` – 마우스 버튼을 더블 클릭했을 때
* `mousedown`과 `mouseup` – 마우스 버튼을 누르고 있을 때, 누르고 있던 마우스 버튼을 뗄 때
* `mousemove` – 마우스를 움직일 때 (터치스크린에서 동작하지 않는다.)
* `mouseover`와 `mouseout` – 마우스를 요소 위로 움직였을 때, 마우스를 요소 밖으로 움직였을 때 (터치스크린에서 동작하지 않는다.)
* `mouseenter` - 해당 요소에 마우스 커서를 올려다 놓았을때
* `mouseleave` - 해당 요소에 마우스 커서를 빼낼 떄

**키보드 이벤트**

* `keydown` – 사용자가 키보드 버튼을 누를 때 발생. 계속 누르고 있는 경우에는 계속 실행
* `keyup` – 사용자가 누른 키를 뗄 때 발생
* `keypress` -  `keydown`과 동일하나 '입력할 수 있는 키보드'를 눌렀을 때 발생

 + **`keydown`과 `keypress`의 차이점**
 `keypress`는 한글 입력, 방향키, Delete키와 같은 즉시 Text에 입력이 반영되는 키보드가 아닌 것에는 반응 하지 않는다.
반면에, `keydown`은 키보드의 어떤 키가 눌러지더라도 반응한다.

**폼 요소 이벤트**

* `submit` – 사용자가 `<form>`을 제출할 때 발생
* `input` - input 또는 textarea 요소의 값이 변경되었을 때
contenteditable 어트리뷰트를 가진 요소가 값이 변경되었을 때
* `focus` – 사용자가 `<input>`과 같은 요소에 포커스 할 때 발생
* `change` - 사용자가 폼 컨트롤의 값이 변경 되었을 때 발생
* `blur` - 사용자가 `<input>`과 같은 포커스가 사라졌을 때 발생

**윈도우 이벤트**

* `load` - 모든 리소스 (이미지, 스타일 등)가 로드된 후 발생
페이지가 완전히 준비되고, 실행해야 할 코드가 있으면 이 이벤트에 연결 
* `DOMContentLoaded` - HTML 문서 구조가 로드되고 파싱 된 직후 발생
DOM 조작을 빠르게 수행해야 할 때 유용합니다. 
* `resize` - 창 크기가 변경될 때 발생
반응형 디자인 구현이나 레이아웃 변경에 사용
* `scroll` - 스크롤 위치가 변경될 때 발생
무한 스크롤이나 특정 위치에 도달했을 때 동작을 트리거하는 데 사용
* `unload` - 사용자가 페이지를 떠날 때 발생
팝업 창을 닫는 것과 같이 딜레이 없는 작업에 사용

<br />

## 이벤트 핸들러 등록

**inline 방식**

인라인 방식은 이벤트를 이벤트 대상의 태그 속성으로 지정하는 것이다.
다음은 버튼을 클릭했을 때 'Hello World'를 경고창에 출력한다.

```HTML
<input type="button" onclick="alert('Hello world');" value="button">
```
script탭에서 함수를 만들고, 태그에 지정하는 방식도 있다
```HTML
  <button onclick="myHandler1(); myHandler2();">Click me</button>
  <script>
    function myHandler1() {
      alert('myHandler1');
    }
    function myHandler2() {
      alert('myHandler2');
    }
  </script>
```

***

**프로퍼티 방식**

이벤트 대상에 해당하는 객체의 프로퍼티로 이벤트를 등록하는 방식이다.

이벤트 핸들러 프로퍼티 방식은 이벤트에 **오직 하나의 이벤트 핸들러만을 바인딩**할 수 있다.

```HTML
  <button class="btn">Click me</button>
  <script>
    const btn = document.querySelector('.btn');

    // 이벤트 핸들러 프로퍼티 방식은 이벤트에 하나의 이벤트 핸들러만을 바인딩할 수 있다
    // 첫번째 바인딩된 이벤트 핸들러 => 실행되지 않는다.
    btn.onclick = function () {
      alert('① Button clicked 1');
    };

    // 두번째 바인딩된 이벤트 핸들러
    btn.onclick = function () {
      alert('① Button clicked 2');
    };

    // addEventListener 메소드 방식
    // 첫번째 바인딩된 이벤트 핸들러
    btn.addEventListener('click', function () {
      alert('② Button clicked 1');
    });

    // 두번째 바인딩된 이벤트 핸들러
    btn.addEventListener('click', function () {
      alert('② Button clicked 2');
    });
```

***

## addEventListener 메소드 방식

`addEventListener` 메소드를 이용하여 대상 DOM 요소에 이벤트를 바인딩하고 해당 이벤트가 발생했을 때 실행될 콜백 함수(이벤트 핸들러)를 지정한다.

```JavaScript
    EventTarget.addEventListener('eventType', functionName [, useCapture]);
/**
 * EventTarget: 대상요소
 * eventType: 대상요소에 바인딩될 이벤트를 나타내는 문자열
 * functionName: 이벤트 발생시에 호출될 함수명 또는 함수 자체
 * useCapture: capture 사용 여부    true: Capturing  /  false: Bubbling (Default) 
*/
```

addEventListener 함수 방식은 이전 방식에 비해 아래와 같이 보다 나은 장점을 갖는다.

* 하나의 이벤트에 대해 하나 이상의 이벤트 핸들러를 추가할 수 있다.
* 캡처링과 버블링을 지원한다.
* HTML 요소뿐만아니라 모든 DOM 요소(HTML, XML, SVG)에 대해 동작한다. 브라우저는 웹 문서(HTML, XML, SVG)를 로드한 후, 파싱하여 DOM을 생성한다.

```JavaScript
addEventListener('click', function () {
    alert('Hello world')
});

```

<br />

## 이벤트 객체(e)와 주요 프로퍼티/메소드

이벤트 핸들러 함수는 실행될 때, 이벤트에 관한 모든 정보가 담긴 이벤트 객체 (Event Object)를 인자로 받습니다. 보통 `e`또는 `event`로 이름을 붙여 사용합나디.

```JavaScript
element.addEventListener('click', function(e) { //  e: event object
  // 여기서 'e'를 사용할 수 있습니다.
});
```
**`e`의 핵심 프로퍼티 및 메소드:**
* `e.target`: 이벤트가 **실제로 시작된 바로 그 요소**를 가리킵니다. 가장 많이 사용된다.
* `e.currentTarget`: 이벤트 리스너가 **부착된 요소**를 가리킵니다. `this`와 동일하게 동작합니다.
* `e.type`: 발생한 이벤트의 타입(ex: 'click')을 문자열로 반환합니다.
* `e.key`: 키보드 이벤트에서 누른 키의 값을 문자열로 반환합니다. (ex: 'Enter', 'a')
* `e.preventDefault()`: **기본 동작을 중단**시키는 메소드입니다.
 * `<a>`태그의 페이지 이동 방지
 * `<form>`의`submit`시 새로고침 방지
* `e.stopPropagation()`: **이벤트 버블링을 중단**시키는 메소드입니다.

<br />

## 이벤트 위임

여러 자식 요소에 개별적으로 이벤트를 바인딩하지 않고, 부모 요소에 한번에 이벤트를 바인딩하여 버블링을 이용해 자식의 이벤트를 처리하는 방식으로, 서능 및 메모리 관리 측면에서 유리합니다.

```HTML
<ul id="list">
    <li>항목 1</li>
    <li>항목 2</li>
    <li>항목 3</li>
    <li>항목 4</li>
    <li>항목 5</li>
    <li>항목 6</li>
</ul>
<script>
const list = document.getElementBtId('list');
list.addEventListener('click', function(e) {
    if (e.target.tagName === 'LI') {
        alert(e.target.textContent + '클릭됨');
    }
});
</script>
```

이러한 경우 이벤트 위임을 사용한다.

부모 요소(`#list`)에 이벤트 핸들러를 바인딩함으로써, 여러 자식 요소(`li`)에 개별 핸들러를 등록하는 번거로운을 해결합니다.
새로운 `li`요소가 추가될 때마다 핸들러를 재등록할 필요없이, 이벤트 버블링을 통해 부모요소가 모든 하위 이벤트를 자동으로 처리합니다.

실제 이벤트 발생 지점은 `e.target`으로 정확히 식별할 수 있습니다. 

<br />

# 참고 자료

> https://ko.javascript.info/introduction-browser-events

> https://ramincoding.tistory.com/entry/
