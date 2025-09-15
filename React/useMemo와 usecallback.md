# useMemo와 useCallback - 성능 최적화

React는 기본적으로 빠르지만, 애플리케이션이 복잡해지면 성능 문제가 발생할 수 있습니다. `useMemo`와 `useCallback`은 **불필요한 연산과 리렌더링을 방지**하여 React 애플리케이션의 성능을 최적화하는 Hook입니다.

<br />

## 리렌더링의 이해

성능 최적화를 이헤하려면 먼저 React의 리렌더링을 이해해야 합니다.

### 컴포넌트가 리렌더링되는 경우

1. **State가 변경될 때**

2. **Props가 변경될 때**

3. **부모 컴포넌트가 리렌더링될 때**

4. **Context value가 변경될 때**

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  console.log("Parent 렌더링");

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child /> {/* Parent가 리렌더링되면 Chid도 리렌더링 */}
    </div>
  );
}

function Child() {
  console.log("Child 렌더링"); // Parent의 state가 변경될 때마다 출력
  return <div>자식 컴포넌트</div>;
}
```

<br />

## useMemo - 계산 결과 메모이제이션

`useMemo`는 **비용이 큰 계산의 결과값을 메모이제이션(기억)** 합니다. 의존성 배열의 값이 변경되지 않으면 이전에 계산한 값을 재사용합니다.

### 기본 문법

```jsx
const memoizedValue = useMemo(() => {
  // 비용이 큰 계산
  return computeExpensiveValue(a, b);
}, [a, b]); // 의존성 배열
```

### 언제 사용해야 할까?

**1. 비용이 큰 계산 최적화**

```jsx
function ExpensiveComponent({ items, filter }) {
  // 나쁜 예시: 매 렌더링마다 필터링 수행
  const filteredItems = items.filter((item) => item.category === filter && item.price > 10000);

  // 좋은 예시: items나 filter가 변경될 때만 필터링
  const filterdItems = useMemo(() => {
    console.log("필터링 실행");
    return itens.filter((item) => item.category === filter && item.price > 10000);
  }, [items, filter]);

  return (
    <ul>
      {filteredItems.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// React.memo로 최적화된 자식 컴포넌트
const ExpensiveChild = React.memo(({ config }) => {
  console.log("ExpensiveChild 렌더링");
  return <div>설정: {JSON.stringify(config)}</div>;
});
```

### 예제: 복잡한 데이터 변환

```jsx
function DataTable({ data, sortBy, sortOrder, filterText }) {
    // 여러 단계의 데이터 처리를 각각 메모이제이션

    // 1단계: 필터링
    const filteredData = useMemo(() => {
        if(!filterText) return data;

        return data.filter(item =>
            item.name.toLowerCase().includes(filterText.toLowerCase()) ||
            item.description.toLowerCase().includes(filterText.toLowerCase())
        );
    }, [data, filterText]);

    // 2단계: 정령
    const sortedData = useMemo(() => {
        const sorted = [...filteredData];

        sorted.sort((a, b) => {
            const aVal = a[sortBy];
            const bVal = b[sortBy];

            if(sortOrder === 'asc') [
                return aVal > bVal ? 1 : -1;
            ] else {
                return aVal < bVal ? 1 : -1;
            }
        });

        return sorted;
    }, [filteredData, sortBy, sortOrder]);

    // 3단께: 동켸 계산
    const statistics = useMemo(() => {
        return {
      total: sortedData.length,
      average: sortedData.reduce((sum, item) => sum + item.value, 0) / sortedData.length,
      max: Math.max(...sortedData.map(item => item.value)),
      min: Math.min(...sortedData.map(item => item.value))
        };
    }, sortedData);

    return (
        <div>
      <div>총 {statistics.total}개, 평균: {statistics.average}</div>
      <table>
        {sortedData.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.value}</td>
          </tr>
        ))}
      </table>
        </div>
    )
}
```

<br />

## useCallback - 함수 메모이제이션

`useCallback`은 **함수 자체를 메모이제이션**합니다. 의존성이 변경되지 않으면 같은 함수 참조를 유지합니다.

### 기본 문법

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]); // 의존성 배열
```

### 언제 사용해야 할까?

**1. 자식 컴포넌트에 함수를 props로 전달할 때**

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState("");

  // 나쁜 예시: 매 렌더링마다 새 함수 생성
  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input }]);
    setInput("");
  };

  // 좋은 예시: 의존성이 변경될 때만 새 함수 생성
  const addTodo = useCallback(() => {
    setTodos((prev) => [...prev, { id: Date.now(), text: input }]);
    setInput("");
  }, [input]); // input이 변경될 때만 새 함수 생성

  const deleteTodo = useCallback((id) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  }, []); // 의존성 없음: 한 번만 생성

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={addTodo}>추가</button>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onDelete={deleteTodo} // 같은 함수 참조 유지
        />
      ))}
    </div>
  );
}

// React.memo로 최적화된 자식 컴포넌트
const TodoItem = React.memo(({ todo, onDelete }) => {
  console.log(`TodoItem ${todo.id} 렌더링`);

  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </div>
  );
});
```

### 2. useEffect의 의존성으로 사용될 때

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  // 함수를 useCallback으로 메모이제이션
  const fetchResults = useCallback(async () => {
    if (!query) return;

    setLoading(true);
    try {
      const response = await fetch(`/api/search?q=${query}`);
      const data = await response.json();
      setresults(data);
    } finally {
      setLoading(false);
    }
  }, [query]); //  query가 변경될 때만 새 함수

  // useEffect의 의존성으로 사용
  useEffect(() => {
    fetchResults();
  }, [fetchResults]); //  안정적인 의존성

  return (
    <div>
      {loading ? "검색 중..." : results.map((item) => <div key={item.id}>{item.title}</div>)}
    </div>
  );
}
```

### 3. 디바운싱/쓰로틀링과 함께 사용

```jsx
function SearchInput() {
  const [value, setValue] = useState("");

  // 디바운싱된 검색 함수
  const debouncedSearch = useCallback(
    debounce((searchTerm) => {
      console.log("검색:", searchTerm);
      // API 호출
    }, 500),
    []
  );

  const handleChange = (e) => {
    const newValue = e.target.value;
    setValue(newValue);
    debouncedSearch(newValue);
  };

  return <input value={value} onChange={handleChange} />;
}
```

<br />

## useMemo vs useCallback

두 Hook은 본질적으로 같은 목적(메모이제이션)을 가지지만 사용 대상이 다릅니다.

```jsx
// useMemo: 값을 메모이제이션
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])

// useCallback: 함수를 메모이제이션
const memoizedCallback = useCallback(() => {
    doSomething(a, b);
}, [a, b]);

// useCallback은 useMemo로 구현 가능
const memoizedCallback = useMemo(() => {
    return () => {
        doSomething(a, b);
    };
}. [a, b]);
```

| 구분              | useMemo        | useCallback         |
| ----------------- | -------------- | ------------------- |
| 메모이제이션 대상 | 계산된 값      | 함수 자체           |
| 반환값            | 계산 결과      | 함수                |
| 사용 시점         | 비용이 큰 연산 | 함수를 props로 전달 |

<br />

## 성능 최적화 예제

### 복잡한 폼 컴포넌트 최적화

```jsx
function OptimizedForm() {
  const [formData, setFormData] = useState({
    name: "",
    email: "",
    phone: "",
    addres: "",
  });
  const [errors, setErrors] = useState({});

  // 유효성 검사 함수 메모이제이션
  const validators = useMemo(
    () => ({
      name: (value) => (value.length >= 2 ? "" : "이름은 2자 이상이어야 합니다"),
      email: (value) => (/\S+@\S+\.\S+/.test(value) ? "" : "올바른 이메일을 입력하세요"),
      phone: (value) => (/^\d{3}-\d{4}-\d{4}$/.test(value) ? "" : "올바른 전화번호를 입력하세요"),
      address: (value) => (value.length >= 5 ? "" : "주소는 5자 이상이어야 합니다"),
    }),
    []
  );

  // 입력 핸들러 메모이제이션
  const handleChange = useCallback(
    (field) => (e) => {
      const value = e.target.value;

      setFormData((prev) => ({
        ...prev,
        [field]: value,
      }));

      //  실시간 유효성 검사
      const error = validators[field](value);
      setErrors((prev) => ({
        ...prev,
        [field]: error,
      }));
    },
    [validators]
  );

  // 폼 유효성 상태 계산
  const isFormValid = useMemo(() => {
    return Object.keys(formData).every((field) => {
      const value = formData[field];
      const error = validators[field](value);
      return value && !error;
    });
  }, [formData, validators]);

  //   제출 핸들러
  const handleSubmit = useCallback(
    (e) => {
      e.preventDefault();

      if (isFormValid) {
        console.log("폼 제출:", formData);
        // API 호출
      }
    },
    [formData, isFormValid]
  );

  return (
    <form onSubmit={handleSubmit}>
      <FormField
        label="이름"
        value={formData.name}
        onChange={handleChange("name")}
        error={errors.name}
      />
      <FormField
        label="이메일"
        value={formData.email}
        onChange={handleChange("email")}
        error={errors.email}
      />
      <FormField
        label="전화번호"
        value={formData.phone}
        onChange={handleChange("phone")}
        error={errors.phone}
      />
      <FormField
        label="주소"
        value={formData.address}
        onChange={handleChange("address")}
        error={errors.address}
      />
      <button type="submit" disabled={!isFormValid}>
        제출
      </button>
    </form>
  );
}

// 메모이제이션된 폼 필드 컴포넌트
const FormField = React.memo(({ label, value, onChange, error }) => {
  console.log(`FormField ${label} 렌더링`);

  return (
    <div>
      <label>{label}</label>
      <input value={value} onChange={onChange} />
      {error && <span classname="error">{error}</span>}
    </div>
  );
});
```

<br />

## 최적화 시 주의사항

### 1. 과도한 최적화는 피하자

```jsx
// 과도한 최적화: 간단한 계산에 useMemo 사용
const doubled = useMemo(() => number * 2, [number]);

// 그냥 계산
const doubled = number * 2;
```

### 2. 의존성 배열을 정확히 설정하자

```jsx
// 잘못된 의존성: count를 사용하지만 의존성에 없음
const calculat = useCallback(() => {
  return count * 2; //  ESLint 경고 발생
}, []); // count가 빠짐

// 올바른 의존성
const calculate = useCallback(() => {
  return count * 2;
}, [count]);
```

### 3. 메모이제이션 비용 고려

메모이제이션도 비용이 듭니다. 다음 경우에만 사용합니다.

- 정말로 비용이 큰 연산
- 참조 동일성이 중요한 경우
- 자주 리렌더링되는 컴포넌트

### 4. React DevTools Profiler 활용

```jsx
// 성능 측정을 위한 Profiler 사용
import { Profiler } from "react";

function onRenderCallback(id, phase, actualDuration, baseDuration) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <ExpensiveComponent />
    </Profiler>
  );
}
```

<br />

## 성능 최적화 리스트

- **먼저 측정하기** - 실제로 성능 문제가 있는지 확인

- **React.memo 먼저 고려** - 컴포넌트 리렌더링 방지

- **비용이 큰 계산만 useMemo** - 단순 계산은 그냥 수행

- **props로 전달되는 함수만 useCallback** - 내부에서만 쓰는 함수는 불필요

- **의존성 배열 정확히** - ESLint의 exhaustive-deps 규칙 활용

- **과도한 최적화 피하기** - 코드 복잡도와 성능의 균형

<br />

# 참고 자료

> [React 공식 문서 - useMemo](https://ko.react.dev/reference/react/useMemo)

> [React 공식 문서 - useCallback](https://ko.react.dev/reference/react/useCallback)

> [React 공식 문서 - React.memo](https://ko.react.dev/reference/react/memo)
