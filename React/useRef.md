# useRef - DOM 접근과 값 보존

`useRef`는 **렌더링 간에 값을 유지**하면서도 **값이 변경되어도 리렌더링을 발생시키지 않는** 특별한 Hook입니다. 주로 DOM 요소에 직접 접근하거나, 컴포넌트의 전체 생명주기 동안 변경 가능한 값을 저장하는 데 사용됩니다.

<br />

## useRef의 기본 개념

`useRef`는 `.current`프로퍼티를 가진 변경 가능한(mutable) ref 객체를 반환합니다.

```jsx
const refContainer = useRef(initialValue);
```

### ref 객체의 특징

1. **컴포넌트의 전체 생명주기 동안 유지**됩니다.
2. **`.current`값을 변경해도 리렌더링이 발생하지 않습니다.**

3. **리렌더링이 되어도 값이 유지됩**니다.

```jsx
function RefExample() {
  const countRef = useRef(0);
  const [renderCount, setRenderCount] = useState(0);

  const handleClick = () => {
    // ref 값 변경 - 리렌더링 발생 안 함
    countRef.current += 1;
    console.log(`Ref value: ${countRef.current}`);
  };

  const forceRender = () => {
    // state 변경 - 리렌더링 발생
    setRenderCount((prev) => prev + 1);
  };

  return (
    <div>
      <p>Render count: {renderCount}</p>
      <p>Ref current: {countRef.current}</p> {/* 화면에 안보임 */}
      <button onClick={handleClick}>Ref 증가</button> {/* 리렌더링 안 함 */}
      <button onClick={forceRender}>강제 리렌더링</button>
    </div>
  );
}
```

<br />

## 주요 사용 사례 1. DOM 요소 접근

### 기본 DOM 접근

```jsx
function TextInputWithFocusButton() {
    const inputRef = useRef(null);

    const focusInput = () => {
        // DOM 요소에 직접 접근
        unputRef.current.focus();
    };

    return (
        <div>
            <input ref={inputRef} type="text">
            <button onClick={focusInput}>입력창에 포커스</button>
        </div>
    )
}
```

### 여러 DOM 요소 관리

```jsx
function MultipleRefs() {
  const inputRefs = useRef({});

  const focusInput = (id) => {
    inputrefs.current[id]?.focus();
  };

  const inputs = ["name", "email", "phone"];

  return (
    <div>
      {inputs.map((id) => (
        <div key={id}>
          <label>{id}:</label>
          <input ref={(el) => (inputRefs.current[id] = el)} type="text" />
          <button onClick={() => focusInput(id)}>Focus</button>
        </div>
      ))}
    </div>
  );
}
```

### 스크롤 제어

```jsx
function ScrollController() {
  const scrollRef = useRef(null);
  const itemRefs = useRef({});

  const scrollToTop = () => {
    scrollref.current.scrollTo({
      top: 0,
      behavior: "smooth",
    });
  };

  return (
    <div>
      <button onClick={scrollToTop}>맨 위로</button>
      <button onClick={() => scrollToItem("item-50")}>50번으로</button>

      <div ref={scrollRef} style={{ height: "400px", overflow: "auto" }}>
        {Array.from({ length: 100 }, (_, i) => (
          <div
            key={i}
            ref={(el) => (itemRefs.current[`item-${i}`] = el)}
            style={{ padding: "10px", border: "1px solid #ccc" }}>
            Item {i}
          </div>
        ))}
      </div>
    </div>
  );
}
```

<br />

## 값 보존

### 이전 값 추적

```jsx
function PreviousValue() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    // 렌더링 후에 현재 값을 ref에 저장
    prevCountRef.current = count;
  }, [count]);

  const prevCount = prevCountRef.current;

  return (
    <div>
      <p>현재 값: {count}</p>
      <p>이전 값: {prevCount ?? "N/A"}</p>
      <p>변화량: {prevCount !== undefined ? count - prevCount : "N/A"}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
      <button onClick={() => setCount(count - 1)}>감소</button>
    </div>
  );
}
```

### 타이머/인터벌 ID 저장

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const intervaluRef = useRef(null);

  const startTimer = () => {
    if (!isRunning) {
      intervalRef.current = setInterval(() => {
        setSeconds((prev) => prev + 1);
      }, 1000);
      setIsRunning(true);
    }
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
      setIsRunning(false);
    }
  };

  const resetTimer = () => {
    stopTimer();
    setSeconds(0);
  };

  // 컴포넌트 언마운트 시 정리
  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <h2>타이머: {seconds}초</h2>
      <button onClick={startTimer} disabled={isRunning}>
        시작
      </button>
      <button onClick={stopTimer} disabled={!isRunning}>
        정지
      </button>
      <button onClick={resetTimer}>리셋</button>
    </div>
  );
}
```

### 렌더링 카운트 추적

```jsx
function RenderCounter() {
  const renderCount = useRef(1);
  const [value, setValue] = useState("");

  useEffect(() => {
    renderCount.current += 1;
  });

  return (
    <div>
      <p>이 컴포넌트는 {renderCount.current}번 렌더링되었습니다.</p>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="타이핑하면 리렌더링됩니다"
      />
    </div>
  );
}
```

<br />

## 최신 값 캡처

### 클로저 문제 해결

```jsx
function LatestValueCapture() {
  const [message, setMessage] = useState("");
  const latestMessage = useRef(message);

  // ref를 최신 값으로 유지
  useEffect(() => {
    latestMessage.current = message;
  }, [message]);

  const showMessage = useCallback(() => {
    setTimeout(() => {
      // 3초 후에도 최신 메시지를 보여줌
      alert(`최신 메시지: ${latestMessage.current}`);
    }, 3000);
  }, []); // 의존성 배열이 비어있어도 최신 값 접근 가능

  return (
    <div>
      <input
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="메시지 입력"
      />
      <button onClick={showMessage}>3초 후 메시지 표시 {/* 입력 계속 가능 */}</button>
    </div>
  );
}
```

<br />

## 비디오 플레이어 제어

```jsx
function VideoPlayer({ src }) {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);
  const [currentTime, setCurrentTime] = useState(0);
  const [duration, setDuration] = useState(0);

  useEffect(() => {
    const video = videoRef.current;
    if (!video) return;

    const updateTime = () => setCurrentTine(video.currentTime);
    const updateDuration = () => setDuration(video.duration);

    video.addEventListener("timeupdate", updateTime);
    video.addEventListener("loadedmetadata", updateDuration);

    return () => {
      video.removeEventListener("timeupdate", updateTime);
      video.removeEventListener("loadedmetadata", updateDuration);
    };
  }, [src]);

  const togglePlay = () => {
    const video = videoref.current;
    if (isPlaying) {
      video.pause();
    } else {
      video.play();
    }
    setIsPlaying(!isPlaying);
  };

  const handleSeek = (e) => {
    const video = videoRef.current;
    const newTime = (e.target.value / 100) * duration;
    video.currentTime = newTime;
    setCurrentTime(newTime);
  };

  const skip = (seconds) => {
    const video = videoRef.current;
    video.currentTime = Math.max(0, Math.min(duration, video.currentTime + seconds));
  };

  const formatTime = (time) => {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60);
    return `${minutes}:${seconds.toString().padStart(2, "0")}`;
  };

  return (
    <div className="video-player">
      <video ref={videoRef} src={src} style={{ width: "100%", maxWidth: "600px" }} />

      <div className="controls">
        <button onClick={togglePlay}>{isPlaying ? "재생" : "정지"}</button>

        <button onClick={() => skip(-10)}>⏪ 10s</button>
        <button onClick={() => skip(10)}>10s ⏩</button>

        <input
          type="range"
          min="0"
          max="100"
          value={duration ? (currentTime / duration) * 100 : 0}
          onChange={handleSeek}
          style={{ flexGrow: 1 }}
        />

        <span>
          {formatTime(currentTime)} / {formatTime(duration)}
        </span>
      </div>
    </div>
  );
}
```

<br />

## 무한 스크롤

```jsx
function InfiniteScroll() {
  const [items, setItems] = useState(Array.from({ length: 20 }, (_, i) => i));
  const [loading, setLoading] = useState(false);
  const observerRef = useRef(null);
  const lastItemRef = useRef(null);

  // Intersection Observer 설정
  useEffect(() => {
    if (loading) return;

    if (observerRef.current) {
      observerRef.current.disconnect();
    }

    observerRef.current = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          loadMore();
        }
      },
      { threshold: 1.0 }
    );

    if (lastItemRef.current) {
      observerRef.current.observe(lastItemRef.current);
    }

    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, [items, loading]);

  const loadMore = async () => {
    setLoading(true);

    // API 호출 시뮬
    await new Promise((resolve) => setTimeout(resolve, 1000));

    setItems((prev) => [...prev, ...Array.from({ length: 20 }, (_, i) => prev.length + i)]);

    setLoading(false);
  };

  return (
    <div style={{ height: "400px", overflow: "auto" }}>
      {items.map((item, index) => (
        <div
          key={item}
          ref={index === items.length - 1 ? lastItemRef : null}
          style={{
            padding: "20px",
            border: "1px solid #ccc",
            margin: "10px",
          }}>
          Item {item}
        </div>
      ))}
      {loading && <div>Loading more...</div>}
    </div>
  );
}
```

<br />

## forwardRef: 컴포넌트에 ref 전달

컴포넌트는 기본적으로 ref를 props로 받을 수 없습니다. `forwardRef`를 사용해야 합니다.

```jsx
// 작동하지 않음
function MyInput(props) {
  return <input ref={props.ref} />;
}

// forwardRef 사용
const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// 사용
function Parent() {
  const inputRef = useRef(null);

  return (
    <div>
      <MyInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>포커스</button>
    </div>
  );
}
```

### useimperativeHandle: 부모에 노출할 메서드 제한

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);

  // 부모 컴포넌트에 노출할 메서드만 선택적으로 제공
  useImperativeHandle(
    ref,
    () => ({
      focus: () => {
        inputRef.current.focus();
      },
      clear: () => {
        inputRef.current.value = "";
      },
      scrollIntoView: () => {
        inputRef.current.scrollIntoView();
      },
    }),
    []
  );

  return <input ref={inputRef} {...props} />;
});

// 사용
function Parent() {
  const fancyInputRef = useRef(null);

  return (
    <div>
      <FancyInput ref={fancyInputRef} />
      <button onClick={() => fancyInputRef.current.focus()}>포커스</button>
      <button onClick={() => fancyInputRef.current.clear()}>지우기</button>
    </div>
  );
}
```

<br />

## useRef 와 useState 비교

| useRef                    | useState                |
| ------------------------- | ----------------------- |
| 값 변경 시 리렌더링 안 함 | 값 변경 시 리렌더링     |
| `.current`로 접근         | 직접 접근               |
| 동기적 업데이트           | 비동기적 업데이트(배치) |
| DOM조작, 타이머 ID 저장   | UI에 반영되는 상태      |
| 렌더링과 무관한 값        | 렌더링에 필요한 값      |

```jsx
function Comparison() {
  const countRef = useRef(0);
  const [countState, setCountState] = useState(0);

  const incrementRef = () => {
    countRef.current += 1;
    console.log("Ref:", countRef.current); // 즉시 반영
  };

  const incrementState = () => {
    setCountState(countState + 1);
    console.log("State:", countState); // 이전 값 출력 (아직 업데이트 안 됨)
  };

  return (
    <div>
      <p>
        Ref {/* 화면에 안 보임 */}: {countRef.current}
      </p>
      <p>State: {countState}</p>
      <button onClick={incrementRef}>Ref 증가</button>
      <button onClick={incrementState}>State 증가</button>
    </div>
  );
}
```

<br />

## 주의사황과 Best Practices

### 1. 렌더링 중 ref.current 읽기/쓰기 피하기

```jsx
// 나쁜 예: 렌더링 중 ref 읽기
function Bad() {
  const ref = useRef(0);
  ref.current++; // 렌더링 중 수정

  return <div>{ref.current}</div>; // 렌더링 중 읽기
}

// 좋은 예: 이벤트 핸들러나 useEffect에서 사용
function Good() {
  const ref = useRef(0);

  useEffect(() => {
    ref.current++; // Effect에서 수정
  });

  const handleClick = () => {
    console.log(ref.current); // 이벤트 핸들러에서 읽기
  };

  return <button onClick={handleClick}>클릭</button>;
}
```

### 2. ref 객체를 의존성 배열에 넣지 않기

```jsx
// ref 객체는 변경되지 않으므로 의미 없음
useEffect(() => {
  // ...
}, [someRef]); // someRef 자체는 항상 같은 객체

// ref.current를 의존성으로 사용 (필요한 경우)
useEffect(() => {
  // ...
}, [someRef.current]); // 하지만 변경을 감지하지 못할 수 있음
```

### 3. DOM 조작은 최후의 수단

```jsx
// 가능하면 피하기
inputRef.current.value = "new value";

// React의 방식 사용
const [value, setValue] = useState("new value");
<input value={value} onChange={(e) => setValue(e.target.value)} />;
```

<br />

# 참고 자료

> [React 공식 문서 - useRef](https://ko.react.dev/reference/react/useRef)

> [React 공식 문서 - ref로 값 참조하기](https://ko.react.dev/learn/referencing-values-with-refs)

> [React 공식 문서 - ref로 DOM 조작하기](https://ko.react.dev/learn/manipulating-the-dom-with-refs)
