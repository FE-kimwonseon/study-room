# useReducer - 복잡한 상태 관리

`useReducer`는 `useState`의 대안으로, **복잡한 상태 로직을 관리**하는데 사용되는 Hook입니다.
Redux의 핵심 개념인 Reducer 패턴을 React 컴포넌트에서 사용할 수 있게 해줍니다.

<br />

## useReducer의 기본 개념

`useReducer`는 **(state, action) => newState** 형태의 reducer 함수를 받아서, 현재 state와 dispatch 함수를 반환합니다.

### 기본 문법

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### Reducer함수의 구조

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "ACTION_TYPE":
      return newState; // 항상 새로운 state를 반환
    default:
      return state; // 기본값은 현재 state로 반환
  }
}
```

### 간단한 카운터 예제

```jsx
import { useReducer } from "react";

// Reducer 함수 정의
function CounterReducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    case "RESET":
      return { count: 0 };
    case "SET":
      return { count: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// 컴포넌트에서 사용
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
      <button onClick={() => dispatch({ type: "SET" })}>Set to 10</button>
    </div>
  );
}
```

<br />

## useState vs useReducer

### 언제 useState를 사용할까?

```jsx
// useState가 적합한 경우: 단순한 상태
const [isOpen, setIsOpen] = useState(false);
const [name, setName] = useState("");
const [count, setCount] = useState(0);
```

### 언제 useReducer를 사용할까?

```jsx
// useReducer가 적합한 경우: 복잡한 상태 로직

// 1. 여러 하위 값을 포함하는 복잡한 state
initialState = {
  loading: false,
  data: null,
  error: null,
};

// 2. 다음 state가 이전 state에 의존하는 경우
function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return { ...state, loading: true, error: null };
    case "FETCH_SUCCESS":
      return { loading: false, data: action.payload, error: null };
    case "FETCH_ERROR":
      return { loading: false, data: null, error: action.payload };
  }
}
```

| useState        | useReducer              |
| --------------- | ----------------------- |
| 단순한 상태     | 복잡한 상태 로직        |
| 독립적인 상태들 | 서로 연관된 상태들      |
| 간단한 업데이트 | 복잡한 업데이트 로직    |
| 작은 컴포넌트   | 큰 컴포넌트나 전역 상태 |

<br />

## Todo List

```jsx
// Todo 앱의 복잡한 상태 관리
const initialTodos = {
  todos: [],
  filter: "all", // all, active, completed
  sortBy: "date", // date, priority, name
};

function todoReducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false,
            createdAt: new Date(),
            priority: "medium",
          },
        ],
      };

    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        ),
      };

    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case "EDIT_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id ? { ...todo, text: action.payload.text } : todo
        ),
      };

    case "SET_FILTER":
      return {
        ...state,
        filter: action.payload,
      };

    case "SET_SORT":
      return {
        ...state,
        sortBy: action.payload,
      };

    case "CLEAR_COMPLETED":
      return {
        ...state,
        todos: state.todos.filter((todo) => !todo.completed),
      };

    case "SET_PRIORITY":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id ? { ...todo, priority: action.payload.priority } : todo
        ),
      };

    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);
  const [input, setInput] = useState("");

  // 필터링된 todos 계산
  const filteredTodos = useMemo(() => {
    let filtered = state.todos;

    // 필터 적용
    if (state.filter === "active") {
      filtered = filtered.filter((todo) => !todo.completed);
    } else if (state.filter === "completed") {
      filtered = filtered.filter((todo) => todo.completed);
    }

    // 정렬 적용
    return filtered.sort((a, b) => {
      switch (state.sortBy) {
        case "date":
          return b.createdAt - a.createdAt;
        case "priority":
          const priorityOrder = { high: 0, medium: 1, low: 2 };
          return priorityOrder[a.priority] - priorityOrder[b.priority];
        case "name":
          return a.text.localeCompare(b.text);
        default:
          return 0;
      }
    });
  }, [state.todos, state.filter, state.sortBy]);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (input.trim()) {
      dispatch({ type: "ADD_TODO", payload: input });
      setInput("");
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="할 일 추가..."
        />
        <button type="submit">추가</button>
      </form>

      <div>
        <select
          value={state.filter}
          onChange={(e) => dispatch({ type: "SET_FILTER", payload: e.target.value })}>
          <option value="all">전체</option>
          <option value="active">진행중</option>
          <option value="completed">완료</option>
        </select>

        <select
          value={state.sortBy}
          onChange={(e) => dispatch({ type: "SET_SORT", payload: e.target.value })}>
          <option value="date">날짜순</option>
          <option value="priority">우선순위순</option>
          <option value="name">이름순</option>
        </select>
      </div>

      <ul>
        {filteredTodos.map((todo) => (
          <TodoItem key={todo.id} todo={todo} dispatch={dispatch} />
        ))}
      </ul>

      <button onClick={() => dispatch({ type: "CLEAR_COMPLETED" })}>완료 항목 삭제</button>
    </div>
  );
}

function TodoItem({ todo, dispatch }) {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleEdit = () => {
    dispatch({
      type: "EDIT_TODO",
      payload: { id: todo.id, text: editText },
    });
    setIsEditing(false);
  };

  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => dispatch({ type: "TOGGLE_TODO", payload: todo.id })}
      />

      {isEditing ? (
        <input
          value={editText}
          onChange={(e) => setEditText(e.target.value)}
          onBlur={handleEdit}
          onKeyPress={(e) => e.key === "Enter" && handleEdit()}
        />
      ) : (
        <span
          style={{ textDecoration: todo.completed ? "line-through" : "none" }}
          onDoubleClick={() => setIsEditing(true)}>
          {todo.text}
        </span>
      )}

      <select
        value={todo.priority}
        onChange={(e) =>
          dispatch({
            type: "SET_PRIORITY",
            payload: { id: todo.id, priority: e.target.value },
          })
        }>
        <option value="high">높음</option>
        <option value="medium">보통</option>
        <option value="low">낮음</option>
      </select>

      <button onClick={() => dispatch({ type: "DELETE_TODO", payload: todo.id })}>삭제</button>
    </li>
  );
}
```

<br />

## 폼 상태 관리

```jsx
// 복잡한 폼의 상태와 유효성 검사
const initialFormState = {
  values: {
    username: "",
    email: "",
    password: "",
    confirmPassword: "",
  },
  errors: {},
  touched: {},
  isSubmitting: false,
  isValid: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case "SET_FIELD_VALUE":
      return {
        ...state,
        values: {
          ...state.values,
          [action.payload.field]: action.payload.value,
        },
      };

    case "SET_FIELD_TOUCHED":
      return {
        ...state,
        touched: {
          ...state.touched,
          [action.payload]: true,
        },
      };

    case "SET_FIELD_ERROR":
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.payload.field]: action.payload.error,
        },
      };

    case "SET_ERRORS":
      return {
        ...state,
        errors: action.payload,
      };

    case "SUBMIT_START":
      return {
        ...state,
        isSubmitting: true,
      };

    case "SUBMIT_SUCCESS":
      return {
        ...initialFormState,
        isSubmitting: false,
      };

    case "SUBMIT_FAILURE":
      return {
        ...state,
        isSubmitting: false,
        errors: action.payload,
      };

    case "RESET_FORM":
      return initialFormState;

    default:
      return state;
  }
}

// Custom Hook으로 추출
function useForm(initialValues, validate) {
  const [state, dispatch] = useReducer(formReducer, {
    ...initialFormState,
    values: initialValues,
  });

  // 값 변경 핸들러
  const handleChange = useCallback(
    (e) => {
      const { name, value, type, checked } = e.target;
      const fieldValue = type === "checkbox" ? checked : value;

      dispatch({
        type: "SET_FIELD_VALUE",
        payload: { field: name, value: fieldValue },
      });

      // 실시간 유효성 검사
      if (validate && state.touched[name]) {
        const errors = validate({ ...state.values, [name]: fieldValue });
        dispatch({
          type: "SET_FIELD_ERROR",
          payload: { field: name, error: errors[name] || "" },
        });
      }
    },
    [state.values, state.touched, validate]
  );

  // 블러 핸들러
  const handleBlur = useCallback(
    (e) => {
      const { name } = e.target;
      dispatch({ type: "SET_FIELD_TOUCHED", payload: name });

      if (validate) {
        const errors = validate(state.values);
        dispatch({
          type: "SET_FIELD_ERROR",
          payload: { field: name, error: errors[name] || "" },
        });
      }
    },
    [state.values, validate]
  );

  // 제출 핸들러
  const handleSubmit = useCallback(
    async (onSubmit) => {
      return async (e) => {
        e.preventDefault();

        if (validate) {
          const errors = validate(state.values);
          dispatch({ type: "SET_ERRORS", payload: errors });

          if (Object.keys(errors).length > 0) {
            return;
          }
        }

        dispatch({ type: "SUBMIT_START" });

        try {
          await onSubmit(state.values);
          dispatch({ type: "SUBMIT_SUCCESS" });
        } catch (error) {
          dispatch({
            type: "SUBMIT_FAILURE",
            payload: { submit: error.message },
          });
        }
      };
    },
    [state.values, validate]
  );

  const resetForm = useCallback(() => {
    dispatch({ type: "RESET_FORM" });
  }, []);

  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
  };
}

function RegistrationForm() {
  const validate = (values) => {
    const errors = {};

    if (!values.username) {
      errors.username = "사용자명을 입력하세요";
    } else if (values.username.length < 3) {
      errors.username = "사용자명은 3자 이상이어야 합니다";
    }

    if (!values.email) {
      errors.email = "이메일을 입력하세요";
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = "올바른 이메일 형식이 아닙니다";
    }

    if (!values.password) {
      errors.password = "비밀번호를 입력하세요";
    } else if (values.password.length < 8) {
      errors.password = "비밀번호는 8자 이상이어야 합니다";
    }

    if (values.password !== values.confirmPassword) {
      errors.confirmPassword = "비밀번호가 일치하지 않습니다";
    }

    return errors;
  };

  const form = useForm({ username: "", email: "", password: "", confirmPassword: "" }, validate);

  const onSubmit = async (values) => {
    // API 호출
    console.log("폼 제출:", values);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <div>
        <input
          name="username"
          value={form.values.username}
          onChange={form.handleChange}
          onBlur={form.handleBlur}
          placeholder="사용자명"
        />
        {form.errors.username && form.touched.username && <span>{form.errors.username}</span>}
      </div>

      <div>
        <input
          type="email"
          name="email"
          value={form.values.email}
          onChange={form.handleChange}
          onBlur={form.handleBlur}
          placeholder="이메일"
        />
        {form.errors.email && form.touched.email && <span>{form.errors.email}</span>}
      </div>

      <div>
        <input
          type="password"
          name="password"
          value={form.values.password}
          onChange={form.handleChange}
          onBlur={form.handleBlur}
          placeholder="비밀번호"
        />
        {form.errors.password && form.touched.password && <span>{form.errors.password}</span>}
      </div>

      <div>
        <input
          type="password"
          name="confirmPassword"
          value={form.values.confirmPassword}
          onChange={form.handleChange}
          onBlur={form.handleBlur}
          placeholder="비밀번호 확인"
        />
        {form.errors.confirmPassword && form.touched.confirmPassword && (
          <span>{form.errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? "제출 중..." : "회원가입"}
      </button>
      <button type="button" onClick={form.resetForm}>
        초기화
      </button>
    </form>
  );
}
```

<br />

## lmmer와 함께 사용

```jsx
import { useImmerReducer } from "use-immer";

function todoReducer(draft, action) {
  switch (action.type) {
    case "ADD_TODO":
      // 직접 수정하는 것처럼 보이지만 immer가 불변성을 보장
      draft.todos.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
      break;

    case "TOGGLE_TODO":
      const todo = draft.todos.find((t) => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
      break;

    case "DELETE_TODO":
      const index = draft.todos.findIndex((t) => t.id === action.payload);
      if (index !== -1) {
        draft.todos.splice(index, 1);
      }
      break;

    default:
      break;
  }
}

function TodoApp() {
  const [state, dispatch] = useImmerReducer(todoReducer, { todos: [] });
  // 사용법은 동일
}
```

<br />

## useReducer + Context: 전역 상태 관리

```jsx
// 전역 상태 관리를 위한 Context + useReducer 조합
const StateContext = createContext();
const DispatchContext = createContext();

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);

  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>{children}</DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Custom Hooks로 사용 편의성 증대
function useAppState() {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error("useAppState must be used within AppProvider");
  }
  return context;
}

function useAppDispatch() {
  const context = useContext(DispatchContext);
  if (!context) {
    throw new Error("useAppDispatch must be used within AppProvider");
  }
  return context;
}

// 사용
function SomeComponent() {
  const state = useAppState();
  const dispatch = useAppDispatch();

  return <button onClick={() => dispatch({ type: "SOME_ACTION" })}>{state.someValue}</button>;
}
```

<br />

## useReducer 장단점

### 장점

- 복잡한 상태 로직을 컴포넌트 외부로 분리

- 상태 업데이트 로직을 한 곳에서 관리

- 테스트하기 쉬움(순수 함수)

- 상태 변경 추적이 용이(action type으로 디버깅)

- TypeScript와 잘 어울림

### 단점

- 간단한 상태에는 과도한 보일러플레이트

- 초기 학습 곡선

- action type 관리 필요

<br />

## 팁

### 1. Action Type을 상수로 관리

```jsx
const ActionTypes = {
  ADD_TODO: "ADD_TODO",
  TOGGLE_TODO: "TOGGLE_TODO",
  DELETE_TODO: "DELETE_TODO",
};
```

### 2. Action Creator 사용

```jsx
const actions = {
  addtodo: (text) => ({ type: "ADD_TODO", payload: text }),
  toggleTodo: (id) => ({ type: "TOGGLE_TODO", payload: id }),
  deleteTodo: (id) => ({ type: "DELETE_TODO", payload: id }),
};
```

### 3. 초기 상태를 함수로 지연 초기화

```jsx
function init(initialCount) {
  return { count: initialCount };
}

const [state, dispatch] = useReducer(reducer, initialArg, init);
```

<br />

# 참고 자료

> [React 공식 문서 - useReducer](https://ko.react.dev/reference/react/useReducer)

> [React 공식 문서 - useState에서 useReducer로 마이그레이션하기](https://ko.react.dev/learn/extracting-state-logic-into-a-reducer)

> [React 공식 문서 - Reducer와 Context로 확장하기](https://ko.react.dev/learn/scaling-up-with-reducer-and-context)
