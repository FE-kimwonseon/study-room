# React.memo와 성능 최적화

`React.memo`는 컴포넌트를 메모이제이션하여 **불필요한 리렌더링을 방지**하는 고차 컴포넌트(Higher Order Component)입니다. props가 변경되지 않으면 컴포넌트를 다시 렌더링하지 않고 이전 렌더링 결과를 재사용합니다.

<br />

## React.memo 기본 개념

기본 사용법

```jsx
// React.memo로 컴포넌트 감싸기
const MyComponent = React.memo(function MyComponent({ name, age }) {
  console.log("MyComponent 렌더링");
  return (
    <div>
      <p>이름: {name}</p>
      <p>나이: {age}</p>
    </div>
  );
});

// 또는 화살표 함수
const MyComponent = React.memo(({ name, age }) => {
  console.log("MyComponent 렌더링");
  return (
    <div>
      <p>이름: {name}</p>
      <p>나이: {age}</p>
    </div>
  );
});

// 기존 컴포넌트를 감싸기
function RawComponent({ name, age }) {
  return (
    <div>
      {name}, {age}
    </div>
  );
}
const MemoizedComponent = React.memo(RawComponent);
```

### 동작 원리

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("Alex");

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <button onClick={() => setName("Bob")}>Change Name</button>

      {/* count가 변경되어도 ChildWithMemo는 리렌더링되지 않음 */}
      <ChildWithMemo name={name} />

      {/* count가 변경될 때마다 리렌더링 */}
      <ChildWithoutMemo name={name} />
    </div>
  );
}

// React.memo 사용
const ChildWithMemo = React.memo(({ name }) => {
  console.log("ChildWithMemo 렌더링");
  return <div>Memoized: {name}</div>;
});

// React.memo 미사용
function ChildWithoutMemo({ name }) {
  console.log("ChildWithoutMemo 렌더링");
  return <div>Not Memoized: {name}</div>;
}
```

<br />

## Custom 비교 함수

React.memo는 기본적으로 **얕은 비교(shallow comparisoon)** 를 수행합니다. 복잡한 props의 경우 custom 비교 함수를 제공할 수 있습니다.

```jsx
const MyComponent = React.memo(
  function MyComponent({ data, config }) {
    console.log("렌더링");
    return <div>{JSON.stringify(data)}</div>;
  },
  // custom 비교 함수 (true를 반환하면 리렌더링 건너뛰기)
  (prevProps, nextProps) => {
    // data의 id가 같으면 리렌더링하지 않음
    return (
      prevProps.data.id === nextProps.data.id && prevProps.config.theme === nextProps.config.theme
    );
  }
);

// 더 복잡한 비교 로직
const ExpensiveList = React.memo(
  function ExpensiveList({ item, filter }) {
    console.log("Expensive 렌더링");
    const filtered = items.filter(filter);
    return (
      <ul>
        {filtered.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    );
  },
  (preveProps, nextProps) => {
    // items 배열의 길이와 첫 번째/마지막 요소만 비교 (간단한 체크)
    if (prevProps.items.length !== nextProps.items.length) return false;
    if (prevProps.items.length === 0) return true;

    const prevFirst = prevProps.items[0];
    const nextFirst = nextProps.items[0];
    const prevLast = prevProps.items[prevProps.items.length - 1];
    const nextLast = nextProps.items[nextProps.items.length - 1];

    return (
      prevFirst.id === nextFirst.id &&
      prevLast.id === nextLast.id &&
      prevProps.filter === nextProps.filter
    );
  }
);
```

<br />

## React.memo와 함께 사용하는 최적화 패턴

### 1. useCallback과 함께 사용

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState("all");

  // 나쁜 예: 매 렌더링마다 새 함수 생성
  //   const handleDelete = (id) => {
  //     setTodos(todos.filter((todo) => todo.id !== id));
  //   };

  // 좋은 에: useCallback으로 함수 메모이제이션
  const handleDelete = useCallback((id) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  }, []); // 의존성 없음

  const handleToggle = useCallback((id) => {
    setTodos((prev) =>
      prev.map((todo) => (todo.id === id ? { ...todo, completed: !todo.completed } : todo))
    );
  }, []);

  return (
    <div>
      <FilterBar filter={filter} onFilterChange={setFilter} />
      <TodoList todos={todos} onDelete={handleDelete} onToggle={handleToggle} />
    </div>
  );
}

// 메모이제이션된 컴포넌트들
const FilterBar = React.memo(({ filter, onFilterChange }) => {
  console.log("FilterBar 렌더링");
  return (
    <div>
      <button onClick={() => onFilterChange("all")}>전체</button>
      <button onClick={() => onFilterChange("active")}>진행중</button>
      <button onClick={() => onFilterChange("completed")}>완료</button>
    </div>
  );
});

const TodoList = React.memo(({ todos, onDelete, onToggle }) => {
  console.log("TodoList 렌더링");
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} onDelete={onDelete} onToggle={onToggle} />
      ))}
    </ul>
  );
});

const TodoItem = React.memo(({ todo, onDelete, onToggle }) => {
  console.log(`TodoItem ${todo.id} 렌더링`);
  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={() => onToggle(todo.id)} />
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>삭제</button>
    </li>
  );
});
```

### 2. useMemo와 함께 사용

```jsx
function DataTable({ rawData, sortConfig, filterText }) {
  // 비용이 큰 계산을 메모이제이션
  const processedData = useMemo(() => {
    console.log("데이터 처리 중...");
    let result = [...rawData];

    // 필터링
    if (filterText) {
      result = result.filter((item) => item.name.toLowerCase().includes(filterText.toLowerCase()));
    }

    // 정렬
    if (sortConfig) {
      result.sort((a, b) => {
        if (sortConfig.direction === "asc") {
          return a[sortConfig.key] > b[sortConfig.key] ? 1 : -1;
        }
        return a[sortConfig.key] < b[sortConfig.key] ? 1 : -1;
      });
    }

    return result;
  }, [rawData, sortConfig, filterText]);

  return (
    <div>
      <SearchBar filterText={filterText} />
      <Table data={processedData} />
      <Summary data={processedData} />
    </div>
  );
}

const Table = React.memo(({ data }) => {
  console.log("Table 렌더링");
  return (
    <table>
      {data.map((item) => (
        <tr key={item.id}>
          <td>{item.name}</td>
          <td>{item.value}</td>
        </tr>
      ))}
    </table>
  );
});

const Summary = React.memo(({ data }) => {
  console.log("Summary 렌더링");
  const total = data.reduce((sum, item) => sum + item.value, 0);
  return <div>총합: {total}</div>;
});
```

<br />

## 성능 최적화

### 대용량 리스트 최적화

```jsx
// 가상화(virtualization)와 React.memo를 결합한 최적화
import { useState, useCallback, memo } from "react";

const ITEM_HEIGHT = 50;
const VISIBLE_COUNT = 10;

function VirtualizedList({ items }) {
  const [scrollTop, setScrollTop] = useState(0);

  const handleScroll = useCallback((e) => {
    setScrollTop(e.target.scrollTop);
  }, []);

  // 보이는 항목만 계산
  const startIndex = Math.floor(scrollTop / ITEM_HEIGHT);
  const endIndex = Math.min(startIndex + VISIBLE_COUNT + 1, items.length);
  const visibleItems = items.slice(startIndex, endIndex);

  const totalHeight = items.length * ITEM_HEIGHT;
  const offsetY = startIndex * ITEM_HEIGHT;

  return (
    <div
      className="list-container"
      style={{ height: VISIBLE_COUNT * ITEM_HEIGHT, overflow: "auto" }}
      onScroll={handleScroll}>
      <div style={{ height: totalHeight, position: "relative" }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <ListItem key={startIndex + index} item={item} style={{ height: ITEM_HEIGHT }} />
          ))}
        </div>
      </div>
    </div>
  );
}

const ListItem = memo(({ item, style }) => {
  console.log(`Rendering item ${item.id}`);
  return (
    <div style={style} className="list-item">
      {item.name}
    </div>
  );
});
```

### Context 최적화

```jsx
// Context를 사용할 때 React.memo 최적화
const ThemeContext = createContext();
const UserContext = createContext();

function App() {
  const [theme, setTheme] = useState("light");
  const [user, setUser] = useState({ name: "Alex", role: "admin" });

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <Dashboard />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// 나쁜 예: Context 전체를 구독
const Header = () => {
  const { theme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  return (
    <div>
      {user.name} - {theme}
    </div>
  );
};

// 좋은 예: 필요한 부분만 구독하는 작은 컴포넌트로 분리
const Header = memo(() => {
  return (
    <div>
      <UserName />
      <ThemeIndicator />
    </div>
  );
});

const UserName = memo(() => {
  const { user } = UserContext(UserContext);
  console.log("UserName 렌더링");
  return <span>{user.name}</span>;
});

const ThemeIndicator = memo(() => {
  const { theme } = useContext(ThemeContext);
  console.log("ThemeIndicator 렌더링");
  return <span>{theme}</span>;
});
```

<br />

## 언제 React.memo를 사용해야 할까?

### 사용하면 좋은 경우

1. 같은 props로 자주 렌더링되는 컴포넌트

2. 렌더링 비용이 큰 컴포넌트 (복잡한 계산, 많은 요소)

3. 리스트의 아이템 컴포넌트

4. 모달, 툴팁 같은 조건부 렌더링 컴포넌트

```jsx
// 좋은 사용 예: 차트 컴포넌트
const ExpensiveChart = memo(({ data, options }) => {
  // 복잡한 차트 렌더링 로직
  const chartData = processChartData(data);
  return <Canvas data={chartData} options={options} />;
});

// 좋은 사용 예: 리스트 아이템
const ProductCard = memo(({ product, onAddToCart }) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>장바구니 추가</button>
    </div>
  );
});
```

## 사용하지 않아도 되는 경우

1. props가 자주 변경되는 컴포넌트

2. 매우 간단한 컴포넌트

3. 이미 부모가 거의 리렌더링되지 않는 경우

```jsx
// 불필요한 사용 예: 너무 간단한 컴포넌트
const SimpleText = memo(({ text }) => {
  return <span>{text}</span>; // 메모이제이션 오베헤드가 더 클 수 있음
});

// 불필요한 사용 예: props가 항상 변경
const Timer = memo(({ currentTime }) => {
  return <div>{currentTime}</div>; // 매초 변경되므로 의미 없음
});
```

<br />

## 성능 측정과 디버깅

### React DevTools Profiler 사용

```jsx
import { Profiler } from "react";

function onRenderCallback(
  id, // 프로파일러 id
  phase, // "mount" 또는 "update"
  actualDuration, // 렌더링 소요 시간
  baseDuration, // 메모이제이션 없이 걸리는 예상 시간
  startTime, // 렌더링 시작 시간
  commitTime // 커밋 시간
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <MemoizedComponent />
      <RegularComponent />
    </Profiler>
  );
}
```

## Custom Hook으로 렌더링 추적

```jsx
function useRenderCount(componentName) {
  const renderCount = useRef(0);

  useeffect(() => {
    render.Count.current += 1;
    console.log(`${componentName} rendered ${renderCount.current} times`);
  });

  return renderCount.current;
}

const MyComponent = memo(({ data }) => {
  const renderCount = userenderCount("MyComponent");

  return (
    <div>
      <small>Render #{renderCount}</small>
      {/* 컴포넌트 내용 */}
    </div>
  );
});
```

<br />

## Best Practices

### 1. 측정 먼저, 최적화는 나중에

```jsx
// 성능 문제가 실제로 있는지 확인 후 최적화
```

### 2. Props 구조 최적화

```jsx
// 객체를 직접 전달 X
<Component config={{theme: 'dark', size: 'large'}} />

// 개별 props로 전달하거나 메모이제이션 O
const config = useMemo(() => ({theme: 'dark', size: 'large'}). []);
<Component config={config} />
```

### 3. Children props 주의

```jsx
// children이 매번 새로 생성됨 X
const Parent = memo(({ children }) => {
  return <div>{children}</div>;
});

<Parent>
  <ExpensiveChild /> {/* 매번 새로 생성 */}
</Parent>;

// children도 메모이제이션 O
const memoizedChild = useMemo(() => <ExpensiveChild />, []);
<Parent>{memoizedChild}</Parent>;
```

<br />

# 참고 자료

> [React 공식 문서 - React.memo](https://ko.react.dev/reference/react/memo)

> [React 공식 문서 - 성능 최적화](https://ko.react.dev/learn/render-and-commit#optimizing-with-memo)
