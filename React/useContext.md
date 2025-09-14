# useContext - 전역 상태 관리

`useContext`는 React의 Context API를 함수 컴포넌트에서 쉽게 사용할 수 있게 해주는 Hook입니다. **Props Drilling** 문제를 해결하고, 여러 컴포넌트가 공유해야 하는 데이터를 효율적으로 관리할 수 있게 해줍니다.

<br>

## Props Drilling 문제

props Drilling은 데이터를 필요로 하는 컴포넌트까지 props를 여러 단계에 걸쳐 전달해야 하는 상황을 말합니다.

### Props Drilling 문제 상황

```jsx
// 최상위 컴포넌트
function App() {
    const [user, setUser] useState({name: 'Alex', role: 'admin'})

    return <Dashboard user={user} setUser={setUser} />;
}

// 중간 컴포넌트 (user를 사용하지 않지만 전달만 함)
function Dashboard({ user. setUser }) {
    return (
        <div>
        <h1>대시보드</h1>
        <Sidebar user={user} setUser={setUser} />
        </div>
    );
}

// 또 다른 중간 컴포넌트
function Sidebar({ user, setUser }) {
    return (
        <div>
            <UserProfile user={user} />
            <UserSettings user={user} setUser={setUser}/>
        </div>
    );
}

// 실제로 user를 사용하는 컴포넌트
function UserProfile({ user }) {
    return <div>안녕하세요, {user.name}님!</div>;
}
```

위 코드에서 `Dashboard`와 `Sidebar`는 `user`를 사용하지 않지만, 하위 컴포넌트에 전달하기 위해 props로 받아야 합니다. 이것이 **Props Drilling**문제입니다.

<br />

## Context API 기본 개념

Context는 컴포넌트 트리 전체에 데이터를 공유할 수 있는 방법을 제공합니다.

### Context 사용 3단계

1. **Context 생성** - `createContext()`

2. **Context 제공** - `<Context.Provider>`

3. **Context 사용** - `useContext()`

```jsx
import { createContext(), useContext, useState } from 'react';

// 1. Context 생성
const UserContext = createContext();

// 2. Provider 컴포넌트 (Context 제공)
function App() {
    const [user, setUser] = useState({ name: 'Alex', role: 'admin'});

    return (
        <UserContext.Provider value={{ user, setUser }}>
            <Dashboard />
        </UserContext.Provider>
    );
}

// 중간 컴포넌트들은 props를 전달할 필요 없음
function Dashboard() {
    return (
        <div>
            <h1>대시보드</h1>
            <Sidebar />
        </div>
    );
}

function Sidebar() {
    return (
        <div>
            <UserProfile />
            <UserSettings />
        </div>
    );
}

// 3. Context 사용 (필요한 곳에서만)
function UserProfile() {
    const { user } = useContext(UserContext);
    return <div>안녕하세요, {user.name}님!</div>
}

function UserSettings() {
    const { user, setUser } = useContext(UserContext);

    const handleRoleChange = () => {
        setUser({ ...user, role: 'user'});
    };

    return (
        <div>
            <p>현재 권한: {user.role}</p>
            <button onClick={handleRoleChange}>권한 변경</button>
        </div>
    );
}
```

<br />

## Custom Provider 패턴

Context를 더 체계적으로 관리하기 위해 Custom Provider 컴포넌트를 만드는 것이 좋습니다.

```jsx
// contexts/UserContext.js
import { createContext, useContext, useState } from "react";

// Context 생성
const UserContext = createContext();

// Custom Provider
export function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => {
    setUser(userData);
    // 로그인 로직 (API 호출 등)
  };

  const updateProfile = (updates) => {
    setUser((prev) => ({ ...prev, ...update }));
  };

  const value = {
    user,
    login,
    logout,
    updateProfile,
    isAuthenticated: !!user,
  };

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

// Custom Hook
export function useUser() {
  const context = useContext(UserContext);

  // Context가 Provider 외부에서 사용되는 것을 방지
  if (context === undefined) {
    throw new Error("useUser must be used within a UserProvider");
  }

  return context;
}
```

### Custom Provider 사용

```jsx
// App.js
import { UserProvider } from "./contexts/UserContext";

function App() {
  return (
    <UserProvider>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Router>
    </UserProvider>
  );
}

// components/Profile.js
import { useUser } from "../contexts/UserContext";

function Profile() {
  const { user, updateProfile, logout } = useUser();

  if (!user) {
    return <div>로그인이 필요합니다.</div>;
  }

  return (
    <div>
      <h1>{user.name}의 프로필</h1>
      <button onClick={() => updateProfile({ name: "새이름" })}>이름 변경</button>
      <button onClick={logout}>로그아웃</button>
    </div>
  );
}
```

<br />

## 다크 모드 구현 예제

실제로 자주 사용되는 테마(다크 모드) Context 예제

```jsx
// contexts/ThemeContext.js
import { createContext, useContext, useState, useEffect } from "react";

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  // 로컬 스토리지에서 초기값 일기
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem("theme");
    return saved || "light";
  });

  // 테마 변경 시 로컬 스토리지에 저장
  useEffect(() => {
    localStorage.setItem("theme", theme);
    document.documentElement.setAttribute("data-theme", theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  return <ThemeContext.Provider value={{ theme, toggleTheme }}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within ThemeProvider");
  }
  return context;
}

// 사용 예시
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return <button onClick={toggleTheme}>{theme === "light" ? "dark" : "light"}</button>;
}
```

## 여러 Context 조합하기

복잡한 애플리케이션에서는 여러 Context를 함께 사용합니다.

```jsx
// 여러 Provider를 중첩해서 사용
function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <CartProvider>
          <NotificationProvider>
            <MainApp />
          </NotificationProvider>
        </CartProvider>
      </UserProvider>
    </ThemeProvider>
  );
}

// 또는 하나의 Provider로 합치기
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <UserProvider>
        <CartProvider>
          <NotificationProvider>{children}</NotificationProvider>
        </CartProvider>
      </UserProvider>
    </ThemeProvider>
  );
}

function App() {
  return (
    <AppProviders>
      <MainApp />
    </AppProviders>
  );
}
```

<br />

## Context 성능 최적화

### 문제: 불필요한 리렌더링

Context의 value가 변경되면 해당 Contxet를 사용하는 **모든 컴포넌트가 리렌더링**됩니다.

```jsx
// 비효율적인 예시
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  // 이 객체는 매 렌더링마다 새로 생성됨
  const value = {
    user,
    setUser,
    theme,
    setTheme,
  };

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}
```

### 해결책 1: Context 분리

관련 없는 데이터는 별도의 Context로 분리합니다.

```jsx
// UserContext와 ThemeContext를 분리
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <MainApp />
      </ThemeProvider>
    </UserProvider>
  );
}

// 이제 theme 변경 시 user를 사용하는 컴포넌트는 리렌더링되지 않음
```

### 해결책 2: useMemo 사용

Provider의 value를 메모이제이션합니다.

```jsx
import { useMemo } from "react";

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  // value 객체를 메모이제이션
  const value = useMemo(
    () => ({
      user,
      login: (userData) => setUser(userData),
      logout: () => setUser(null),
    }),
    [user]
  ); // user가 변경될 때만 새 객체 생성

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}
```

### 해결책 3: State와 Dispatch 분리

상태와 업데이트 함수를 별도 Context로 분리합니다.

```jsx
const UserStateContext = createContext();
const UserDispatchContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={setUser}>{children}</UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}

// 상태만 필요한 컴포넌트
function UserProfile() {
  const user = useContext(UserStateContext);
  return <div>{user?.name}</div>;
}

// 업데이트만 필요한 컴포넌트
function LogoutButton() {
  const setUser = useContext(UserDispatchContext);
  return <button onClick={() => setUser(null)}>로그아웃</button>;
}
```

<br />

## 장바구니 Context

```jsx
// contexts/CartContext.js
import { createContext, useContext, useReducer } from "react";

const CartContext = createContext();

// Reducer를 사용한 복잡한 상태 관리
function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      const existingItem = state.find((item) => item.id === action.payload.id);
      if (existingItem) {
        return state.map((item) =>
          item.id === action.payload.id ? { ...item, quantity: item.quantity + 1 } : item
        );
      }
      return [...state, { ...action.payload, quantity: 1 }];

    case "REMOVE_ITEM":
      return state.filter((item) => item.id !== action.payload);

    case "UPDATE_QUANTITY":
      return state.map((item) =>
        item.id === action.payload.id ? { ...item, quantity: action.payload.quantity } : item
      );

    case "CLEAR_CART":
      return [];

    default:
      return state;
  }
}

export function CartProvider({ children }) {
  const [cart, dispatch] = useReducer(cartReducer, []);

  // 유용한 계산값들
  const totalItems = cart.reduce((sum, item) => sum + item.quantity, 0);
  const totalPrice = cart.reduce((sum, item) => sum + item.price * item.quantity, 0);

  const addToCart = (product) => {
    dispatch({ type: "ADD_ITEM", payload: product });
  };

  const removeFromCart = (productId) => {
    dispatch({ type: "REMOVE_ITEM", payload: productId });
  };

  const updateQuantity = (productId, quantity) => {
    if (quantity <= 0) {
      removeFromCart(productId);
    } else {
      dispatch({
        type: "UPDATE_QUANTITY",
        payload: { id: productId, quantity },
      });
    }
  };

  const clearCart = () => {
    dispatch({ type: "CLEAR_CART" });
  };

  const value = {
    cart,
    totalItems,
    totalPrice,
    addToCart,
    removeFromCart,
    updateQuantity,
    clearCart,
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

export function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error("useCart must be used within CartProvider");
  }
  return context;
}

// 사용 예시
function ProductCard({ product }) {
  const { addToCart } = useCart();

  return (
    <div>
      <h3>{product.name}</h3>
      <p>{product.price}원</p>
      <button onClick={() => addToCart(product)}>장바구니에 추가</button>
    </div>
  );
}

function CartIcon() {
  const { totalItems } = useCart();

  return <div>🛒 {totalItems > 0 && <span>({totalItems})</span>}</div>;
}
```

<br />

## Context 사용 시 주의사항

1. **Context는 반드시 Provider 내부에서만 사용합니다.**

   - Custom Hook에 에러 처리 추가

2. **너무 많은 것을 하나의 Context에 넣지 않습니다.**

   - 관심사 분리 원칙 적용

3. **자주 변경되는 데이터는 Context에 적합하지 않을 수 있습니다.**

   - 매우 빈번한 업데이트는 성능 문제 야기

4. **모든 전역 상태를 Context로 관리할 필요는 없습니다.**

   - 단순한 prop 전달이 더 명확할 수 있음

5. **Context vs 상태 관리 라이브러리**
   - 간단한 경우: Context API
   - 복잡한 경우: Redux, Zustand, Recoil 등 고려

<br />

## 언제 Context를 사용해야 할까?

### Context가 적합한 경우

- 테마 (다크 모드)
- 사용자 인증 정보
- 언어 설정
- 장바구니 상태
- 알림/토스트 메시지

### Context가 부적합한 경우

- 자주 변경되는 데이터 (예: 마우스 위치)
- 특정 컴포넌트에만 필요한 상태
- 복잡한 상태 로직 (상태 관리 라이브러리 고려)

<br />

# 참고 자료

> [React 공식 문서 - useContext](https://ko.react.dev/reference/react/useContext)

> [React 공식 문서 - Context로 데이터 깊게 전달하기](https://ko.react.dev/learn/passing-data-deeply-with-context)

> [React 공식 문서 - Context 확장하기](https://ko.react.dev/learn/scaling-up-with-reducer-and-context)
