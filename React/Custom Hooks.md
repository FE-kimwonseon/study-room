# Custom Hooks - 로직 재사용

Custom Hook은 **컴포넌트 로직을 재사용 가능한 함수로 추출**하는 React의 강력한 패턴입니다. `use`로 시작하는 JavaScript 함수이며, 다른 Hook들을 호출할 수 있습니다. UI가 아닌 **로직만을 재사용**할 수 있게 해주는 것이 핵심입니다.

<br/>

## Custom Hook이란?

Custom Hook은 다음과 같은 특징을 가집니다.

1. **이름이 `use`로 시작**해야 합니다. (예: `useCounter`, `useFetch`)

2. 다른 Hook을 내부에서 호출할 수 있습니다.

3. 일반 JavaScript 함수이지만, **Hook의 규칙**을 따라야 합니다.

4. 각 컴포넌트에서 독립적인 state를 가집니다.

### useCounter

```jsx
// hooks/useCounter.js
import { useState } from "react";

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount((prev) => prev + 1);
  const decrement = () => setCount((prev) => prev - 1);
  const reset = () => setCount(initialValue);

  return {
    count,
    increment,
    decrement,
    reset,
  };
}

// 컴포넌트에서 사용
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// 같은 Hook을 여러 컴포넌트에서 독립적으로 사용
function AnotherCounter() {
  const { count, increment } = useCounter(10); // 다른 초기값
  // 이 count는 위 Counter의 count와 완전히 독립적
  return <div>Another: {count}</div>;
}
```

<br />

## 실용적인 Custom Hook

### 1. useLocalStorage - 로컬 스토리지 동기화

```jsx
import { useState, useEffect } from "react";

function useLocalStorage(key, initialValue) {
  // 초기값 설정 (로컬 스토리지에서 읽기)
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error loading ${key} from localStorage:`, error);
      return initialValue;
    }
  });

  // 값 저장 함수
  const setValue = (value) => {
    try {
        // 함수형 업데이트 지원
        const valueToStore = value insanceof Function ? value(storedValue) : setStoredValue(valueToStore);
        window.localStorage.setItem(key, JSON.stringify(valueToStore))
    } catch (error) {
        console.error(`Error saving ${key} to localStorage:`, error);
    }
  };

    // 다른 탭에서의 변경사항 감지
    useEffect(() => {
        const handleStorageChange = (e) => {
            if(e.key === key && e.newValue) {
                setStoredValue(JSON.parse(e.newValue));
            }
        };

        window.addEventListener('storage', handleStorageChange);
        return () => window.removeEventListener('storage', handleStorageChange);
    }, [key])
    return [storedValue, setValue];
}

// 사용 예제
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);

  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        테마 변경: {theme}
      </button>
      <button onClick={() => setFontSize(prev => prev + 2)}>
        글자 크기: {fontSize}px
      </button>
    </div>
  );
}
```

### 2. useFetch - 데이터 페칭

```jsx
import { useState, useEffect } from "react";

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // AbortController로 취소 가능하게 만들기
    const abortController = new AbortController();

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, {
          ...options,
          signal: abortController.signal,
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const jsonData = await response.json();
        setData(jsonData);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    // Cleanup: 컴포넌트 언마운트 시 요청 취소
    return () => abortController.abort();
  }, [url]); // URL이 변경될 때마다 다시 페칭

  return { data, loading, error, refetch: () => fetchData() };
}

// 사용 예제
function UserProfile({ userId }) {
  const { data: user, loading, error, refetch } = useFetch(`/api/users/${userId}`);

  if (loading) return <div>로딩 중...</div>;
  if (error) return <div>에러: {error}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={refetch}>새로고침</button>
    </div>
  );
}
```

### 3. useToggle - 불리언 상태 관리

```jsx
import { useState, useCallback } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue((prev) => !prev);
  }, []);

  const setTrue = useCallback(() => {
    setValue(true);
  }, []);

  const setFalse = useCallback(() => {
    setValue(false);
  }, []);

  return {
    value,
    toggle,
    setTrue,
    setFalse,
  };
}

function Modal() {
  const {
    value: isOpen,
    toggle: toggleModal,
    setTrue: openModal,
    setFalse: closeModal,
  } = useToggle();

  return (
    <>
      <button onClick={openModal}>모달 열기</button>
      {isOpen && (
        <div className="modal">
          <div className="modal-content">
            <h2>모달 제목</h2>
            <button onClick={closeModal}>닫기</button>
          </div>
        </div>
      )}
    </>
  );
}
```

### 4. useDebounce - 디바운싱

```jsx
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// 검색 자동완성
function SearchBox() {
  const [searchTerm, setSearchTerm] = useState("");
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
  const [results, setResults] = useState([]);

  // 디바운싱된 값이 변경될 때만 API 호출
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API 호출
      searchAPI(debouncedSearchTerm).then(setResults);
    } else {
      setResults([]);
    }
  }, [debouncedSearchTerm]);
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="검색어를 입력하세요"
      />
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 5, useOnClickOutside - 외부 클릭 감지

```jsx
import { useEffect, useRef } from "react";

function useOnClickOutside(handler) {
  const ref = useRef();

  useEffect(() => {
    const listener = (event) => {
      // ref 내부 클릭은 무시
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };

    document.addEventListener("mousedown", listener);
    document.addEventListener("touchstart", listener);

    return () => {
      document.removeEventListener("mousedown", listener);
      document.removeEventListener("touchstart", listener);
    };
  }, [handler]);

  return ref;
}

// 드롭다운 메뉴
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useOnClickOutside(() => {
    setIsOpen(false);
  });

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>메뉴 {isOpen ? "▲" : "▼"}</button>
      {isOpen && (
        <ul className="dropdown-menu">
          <li>프로필</li>
          <li>설정</li>
          <li>로그아웃</li>
        </ul>
      )}
    </div>
  );
}
```

<br />

## 고급 Custom Hook 패턴

### 1. useAsync - 비동기 작업 관리

```jsx
import { useState, useEffect, useCallback } from "react";

function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState("idle");
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  // execute 함수를 useCallback으로 메모이제이션
  const execute = useCallback(
    async (...params) => {
      setStatus("pending");
      setValue(null);
      setError(null);

      try {
        const response = await asyncFunction(...params);
        setValue(response);
        setStatus("success");
        return response;
      } catch (error) {
        setError(error);
        setStatus("error");
        throw error;
      }
    },
    [asyncFunction]
  );

  // immediate가 true면 즉시 실행
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return {
    execute,
    status,
    value,
    error,
    isLoading: status === "pending",
    isSuccess: status === "success",
    isError: status === "error",
  };
}

function UserList() {
  const {
    execute: fetchUsers,
    value: users,
    isLoading,
    isError,
    error,
  } = useAsync(
    async () => {
      const response = await fetch("/api/users");
      return response.json();
    },
    true // 컴포넌트 마운트 시 자동 실행
  );

  if (isLoading) return <div>로딩 중...</div>;
  if (isError) return <div>에러: {error.message}</div>;

  return (
    <div>
      <button onClick={fetchUsers}>새로고침</button>
      <ul>
        {users?.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 2. usePrevious - 이전 값 추적

```jsx
import { useRef, useEffect } from "react";

function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>
        현재: {count}, 이전: {prevCount ?? "N/A"}
      </p>
      <p>변화량: {prevCount !== undefined ? count - prevCount : "N/A"}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

### 3. useMediaQuery - 반응형 디자인

```jsx
import { useState, useEffect } from "react";

function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);

    // 초기값 설정
    if (media.matches !== matches) {
      setMatches(media.matches);
    }

    // 리스너 설정
    const listener = (e) => setMatches(e.matches);

    // 모던 브라우저는 addEventListener 사용
    if (media.addEventListener) {
      media.addEventListener("change", listener);
      return () => media.removeEventListener("change", listener);
    } else {
      // 구형 브라우저 대응
      media.addListener(listener);
      return () => media.removeListener(listener);
    }
  }, [matches, query]);

  return matches;
}

function ResponsiveLayout() {
  const isMobile = useMediaQuery("(max-width: 768px)");
  const isTablet = useMediaQuery("(min-width: 769px) and (max-width: 1024px)");
  const isDesktop = useMediaQuery("(min-width: 1025px)");

  return (
    <div>
      {isMobile && <MobileLayout />}
      {isTablet && <TabletLayout />}
      {isDesktop && <DesktopLayout />}
    </div>
  );
}
```

<br />

## Custom Hook 작성 가이드라인

### 1. 네이밍 규칙

```jsx
// 좋은 예시
useAuth();
useLocalStorage();
useWindowSize();

// 나쁜 예시
getAuth(); // use로 시작하지 않음
UseAuth(); // 대문자로 시작
use_auth(); // 스네이크 케이스
```

### 2. 반홥값 설계

```jsx
// 단일 값 반환
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  // ...
  return width;
}

// 배열 반환 (useState처럼)
function useToggle(initial) {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue(!value);
  return [value, toggle];
}

// 객체 반환 (더 많은 값)
function useForm(initial) {
  // ...
  return {
    values,
    errors,
    handleChange,
    handleSubmit,
    reset,
  };
}
```

### 3. 의존성 관리

```jsx
function useCustomHook(dependency) {
  // 의존성이 변경될 때만 effect 재실행
  useEffect(() => {
    // effect logic
  }, [dependency]);

  // 콜백 함수 메모이제이션
  const memoizedCallback = useCallback(() => {
    // callback logic
  }, [dependency]);

  return memoizedCallback;
}
```

<br />

## Custom Hook 테스팅

```jsx
// hooks/useCounter.test.js
import { renderHook, act } from "@testing-library/react-hooks";
import useCounter from "./useCounter";

describe("useCounter", () => {
  test("should increment counter", () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  test("should reset to initial value", () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.increment();
      result.current.reset();
    });

    expect(result.current.count).toBe(5);
  });
});
```

<br />

## Custom Hook 과 일반함수

| Custom Hook                     | 일반함수           |
| ------------------------------- | ------------------ |
| `use`로 시작                    | 아무 이름 가능     |
| 다른 Hook 호출 가능             | Hook 호출 불가     |
| 컴포넌트/Custom Hook에서만 호출 | 어디서든 호출 가능 |
| React의 상태 관리 가능          | 순수 로직만 처리   |
| 각 호출마다 독립적인 상태       | 상태 없음          |

<br />

## 언제 Custom Hook을 만들어야 하나?

1. 같은 로직이 2개 이상의 컴포넌트에서 사용될 때

2. 복잡한 로직을 컴포넌트에서 분리하고 싶을 때

3. 테스트하기 쉬운 코드로 만들고 싶을 때

4. 관심사를 분리하고 싶을 때

### 주의사항

- 과도한 추상화는 피해야합니다.

- 너무 일찍 Custom Hook을 만들면 안됩니다. (YAGNI 원칙)

- Hook의 규칙을 반드시 따릅니다.

<br />

# 참고 자료

> [React 공식 문서 - 커스텀 Hook으로 로직 재사용하기](https://ko.react.dev/learn/reusing-logic-with-custom-hooks)

> [React 공식 문서 - Hook의 규칙](https://ko.react.dev/reference/rules/rules-of-hooks)

> [usehooks.com - Custom Hook 예제 모음](https://usehooks.com/)
