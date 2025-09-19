# 에러 바운더리 (Error Boundaries)

에러 바운더리는 **자식 컴포넌트 트리에서 발생한 JavaScrit 에러를 잡아내고, 에러를 로깅하며, 풀백 UI**를 표시하는 React 컴포넌트입니다. 애플리케이션 전체가 크래시되는 것을 방지하고 사용자에게 더 나은 경험을 제공합니다.

<br />

## 에러 바운더리의 필요성

React 16 이전에는 컴포넌트 내부의 JavaScript 에러가 React의 내부 상태를 손상시켜 애플리케이션 전체를 중단시켰습니다. 에러 바운더리는 이러한 문제를 해결합니다.

```jsx
// 에러 바운더리가 없을 때: 전체 앱이 크래시
function BuggyComponent() {
  const [counter, setCounter] = useState(0);

  if (counter === 3) {
    throw new Error("Counter가 3이 되면 크래시!");
  }

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={() => setCounter(counter + 1)}>증가</button>
    </div>
  );
}

// 앱 전체가 백색 화면이 되고 에러만 표시됨
```

<br />

## 에러 바운더리 구현

에러 바운더리는 반드시 **클래스 컴포넌트**로 작성해야 합니다. 현재 함수 컴포넌트에서는 에러 바운더리를 구현할 수 없습니다.

### 기본 에러 바운더리

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  // 에러가 발생하면 state를 업데이트
  static getDerivedStateFromError(error) {
    // 다음 렌더링에서 폴백 UI가 보이도록 state 업데이트
    return { hasError: true };
  }

  // 에러 정보를 로깅
  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 로그할 수 있습니다
    console.error("에러 발생:", error);
    console.error("에러 정보:", errorInfo);

    // state에 에러 정보 저장 (선택사항)
    this.setState({
      error: error,
      errorInfo: errorInfo,
    });
  }

  render() {
    if (this.state.hasError) {
      // 폴백 UI 렌더링
      return (
        <div className="error-fallback">
          <h2>앗, 무언가 잘못되었습니다!</h2>
          <details style={{ whiteSpace: "pre-wrap" }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo && this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }

    // 에러가 없으면 자식 컴포넌트들을 정상적으로 렌더링
    return this.props.children;
  }
}

// 사용법
function App() {
  return (
    <ErrorBoundary>
      <Header />
      <MainContent />
      <BuggyComponent />
    </ErrorBoundary>
  );
}
```

<br />

## 실용적인 에러 바운더리 구현

### 재사용 가능한 에러 바운더리

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      errorCount: 0,
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    const { onError, errorReporter } = this.props;

    // 에러 로깅
    console.error("ErrorBoundary caught an error:", error, errorInfo);

    // 외부 에러 리포팅 서비스로 전송 (예: Sentry)
    if (errorReporter) {
      errorReporter.reportError(error, {
        componentStack: errorInfo.componentStack,
        props: this.props,
        state: this.state,
      });
    }

    // 부모 컴포넌트에 에러 알림
    if (onError) {
      onError(error, errorInfo);
    }

    // 에러 카운트 증가
    this.setState((prevState) => ({
      error,
      errorInfo,
      errorCount: prevState.errorCount + 1,
    }));
  }

  resetErrorBoundary = () => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
    });
  };

  render() {
    const { hasError, error, errorInfo, errorCount } = this.state;
    const { fallback, fallbackComponent: FallbackComponent } = this.props;

    if (hasError) {
      // 사용자 정의 폴백 컴포넌트
      if (FallbackComponent) {
        return (
          <FallbackComponent
            error={error}
            errorInfo={errorInfo}
            resetErrorBoundary={this.resetErrorBoundary}
            errorCount={errorCount}
          />
        );
      }

      // 사용자 정의 폴백 렌더 함수
      if (fallback) {
        return fallback(error, errorInfo, this.resetErrorBoundary);
      }

      // 기본 폴백 UI
      return (
        <div className="error-boundary-default-ui">
          <h2>오류가 발생했습니다</h2>
          <p>페이지를 새로고침하거나 나중에 다시 시도해주세요.</p>
          <button onClick={this.resetErrorBoundary}>다시 시도</button>
          {process.env.NODE_ENV === "development" && (
            <details style={{ marginTop: "20px" }}>
              <summary>에러 상세 정보 (개발 모드)</summary>
              <pre>{error?.toString()}</pre>
              <pre>{errorInfo?.componentStack}</pre>
            </details>
          )}
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 풀백 컴포넌트 예제

```jsx
function ErrorFallback({ error, errorInfo, resetErrorBoundary, errorCount }) {
  // 에러가 반복적으로 발생하는 경우
  if (errorCount > 3) {
    return (
      <div className="error-fallback-fatal">
        <h2>반복적인 오류가 발생했습니다.</h2>
        <p>고객센터에 문의해주세요</p>
        <button onClick={() => window.location.reload()}>페이지 새로고침</button>
      </div>
    );
  }

  return (
    <div className="error-fallback">
      <h2>일시적인 오류가 발생했습니다</h2>
      <p>불편을 드려 죄송합니다. 다시 시도해주세요.</p>
      <button onClick={resetErrorBoundary}>다시 시도</button>
      <button onClick={() => window.history.back()}>뒤로 가기</button>
    </div>
  );
}

// 사용
<ErrorBoundary fallbackComponent={ErrorFallback}>
  <App />
</ErrorBoundary>;
```

<br />

## 여러 에러 바운더리 전략

### 세분화된 에러 바운더리

```jsx
function App() {
  return (
    <div>
      {/* 전체 앱 레벨 에러 바운더리 */}
      <ErrorBoundary fallback={<AppErrorFallback />}>
        <Header />

        {/* 페이지 레벨 에러 바운더리 */}
        <ErrorBoundary fallback={<PageErrorFallback />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route
              path="/profile"
              element={
                // 특정 기능 레벨 에러 바운더리
                <ErrorBoundary fallback={<ProfileErrorFallback />}>
                  <Profile />
                </ErrorBoundary>
              }
            />
          </Routes>
        </ErrorBoundary>

        <Footer />
      </ErrorBoundary>
    </div>
  );
}

// 각 레벨별 다른 폴백 UI
function AppErrorFallback() {
  return (
    <div className="app-error">
      <h1>애플리케이션 오류</h1>
      <button onClick={() => (window.location.href = "/")}>홈으로 돌아가기</button>
    </div>
  );
}

function PageErrorFallback() {
  return (
    <div className="page-error">
      <h2>페이지를 불러올 수 없습니다</h2>
      <Link to="/">홈으로</Link>
    </div>
  );
}

function ProfileErrorFallback() {
  return (
    <div className="feature-error">
      <p>프로필을 불러올 수 없습니다</p>
      <button onClick={() => window.location.reload()}>새로고침</button>
    </div>
  );
}
```

<br />

## Hook과 함께 사용하기

함수 컴포넌트에서 에러 바운더리 기능을 사용하기 위한 Custom Hook을 만들 수 있습니다.

```jsx
// Custom Hook: useErrorHandler
function useErrorHandler() {
  const [error, setError] = useState(null);

  const resetError = () => setError(null);

  const captureError = useCallback((error) => {
    setError(error);
  }, []);

  // 에러가 있으면 throw (가장 가까운 에러 바운더리가 캐치)
  if (error) throw error;

  return { captureError, resetError };
}

// 사용 예제
function RiskyComponent() {
  const { captureError } = useErrorHandler();

  const fetchData = async () => {
    try {
      const response = await fetch("/api/data");
      if (!response.ok) throw new Error("데이터 로드 실패");
      const data = await response.json();
      // 데이터 처리
    } catch (error) {
      captureError(error); // 에러 바운더리로 전파
    }
  };

  return <button onClick={fetchData}>데이터 불러오기</button>;
}

// react-error-boundary 라이브러리 사용
import { ErrorBoundary, useErrorHandler } from "react-error-boundary";

function MyApp() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => window.location.reload()}
      onError={(error, errorInfo) => {
        console.log("에러 로깅:", error, errorInfo);
      }}>
      <ComponentThatMayError />
    </ErrorBoundary>
  );
}

function ComponentThatMayError() {
  const handleError = useErrorHandler();

  const riskyOperation = async () => {
    try {
      await doSomethingRisky();
    } catch (error) {
      handleError(error); //  에러 바운더리로 전파
    }
  };

  return <button onClick={riskyOperation}>위험한 작업</button>;
}
```

<br />

## 에러 바운더리가 잡지 못하는 에러

에러 바운더리는 다음과 같은 경우의 에러를 잡을 수 없습니다.

### 1. 이벤트 핸들러 내부 에러

```jsx
// 에러 바운더리가 잡을 수 없음
function MyComponent() {
  const handleClick = () => {
    throw new Error("클릭 핸들러 에러"); // 에러 바운더리가 못 잡음
  };

  return <button onClick={handleClick}>클릭</button>;
}

// 해결 방법: try-catch 사용
function MyComponent() {
  const [error, setError] = useState(null);

  const handleClick = () => {
    try {
      // 위험한 작업
      doSomethingRisky();
    } catch (error) {
      setError(error);
      // 또는 에러 리포팅
      errorReporter.log(error);
    }
  };

  if (error) throw error; // 에러 바운더리로 전파

  return <button onClick={handleClick}>클릭</button>;
}
```

### 2. 비동기 코드 (setTimeout, Promise)

```jsx
// 에러 바운더리가 잡을 수 없음
function MyComponent() {
  useEffect(() => {
    setTimeout(() => {
      throw new Error("비동기 에러!"); // 에러 바운더리가 못 잡음
    }, 1000);
  }, []);

  return <div>컴포넌트</div>;
}

// 해결 방법: 에러 상태 관리
function MyComponent() {
  const [error, setError] = useState(null);

  useEffect(() => {
    setTimeout(() => {
      try {
        doSomethingAsync();
      } catch (err) {
        setError(err);
      }
    }, 1000);
  }, []);

  if (error) throw error; // 에러 바운더리로 전파

  return <div>컴포넌트</div>;
}
```

### 3. 서버 사이드 렌더링 (SSR) 에러

```jsx
// SSR에서는 componentDidCatch가 호출되지 않음
// 서버에서는 다른 에러 처리 메커니즘 필요
```

### 4. 에러 바운더리 자체의 에러

```jsx
// 에러 바운더리 컴포넌트 자체에서 발생한 에러는 잡을 수 없음
// 상위 에러 바운더리가 필요
```

<br />

## 실전 패턴과 Best Practices

### 1. 로딩과 에러 상태 통합 관리

```jsx
class AsyncBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      isLoading: true,
      error: null,
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, isLoading: false, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error("AsyncBoundary Error:", error, errorInfo);
  }

  componentDidMount() {
    // 로딩 완료 시뮬레이션
    setTimeout(() => {
      this.setState({ isLoading: false });
    }, 1000);
  }

  render() {
    const { hasError, isLoading, error } = this.state;
    const { children, loadingFallback, errorFallback } = this.props;

    if (isLoading) {
      return loadingFallback || <div>로딩 중...</div>;
    }

    if (hasError) {
      return errorFallback ? errorFallback(error) : <div>에러 발생!</div>;
    }

    return children;
  }
}
```

### 2. 네트워크 에러 처리

```jsx
class NetworkErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      isOnline: navigator.onLine,
      retryCount: 0,
    };
  }

  componentDidMount() {
    window.addEventListener("online", this.handleOnline);
    window.addEventListener("offline", this.handleOffline);
  }

  componentWillUnmount() {
    window.removeEventListener("online", this.handleOnline);
    window.removeEventListener("offline", this.handleOffline);
  }

  handleOnline = () => {
    this.setState({ isOnline: true });
    if (this.state.hasError) {
      this.retry();
    }
  };

  handleOffline = () => {
    this.setState({ isOnline: false });
  };

  static getDerivedStateFromError(error) {
    // 네트워크 에러인지 확인
    if (error.name === "NetworkError" || !navigator.onLine) {
      return { hasError: true, isNetworkError: true };
    }
    return { hasError: true, isNetworkError: false };
  }

  retry = () => {
    this.setState((prevState) => ({
      hasError: false,
      retryCount: prevState.retryCount + 1,
    }));
  };

  render() {
    const { hasError, isOnline, isNetworkError } = this.state;

    if (hasError && !isOnline) {
      return (
        <div className="offline-error">
          <h2>📡 오프라인 상태입니다</h2>
          <p>인터넷 연결을 확인해주세요</p>
        </div>
      );
    }

    if (hasError && isNetworkError) {
      return (
        <div className="network-error">
          <h2>🔌 네트워크 오류</h2>
          <p>서버에 연결할 수 없습니다</p>
          <button onClick={this.retry}>다시 시도</button>
        </div>
      );
    }

    if (hasError) {
      return (
        <div className="general-error">
          <h2>오류가 발생했습니다</h2>
          <button onClick={this.retry}>다시 시도</button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 3. 에러 리포팅 통합

```jsx
// 에러 리포팅 서비스 설정 (예: Sentry)
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: "YOUR_SENTRY_DSN",
  environment: process.env.NODE_ENV,
});

// Sentry와 통합된 에러 바운더리
const SentryErrorBoundary = Sentry.withErrorBoundary(MyApp, {
  fallback: ({ error, resetError }) => (
    <div>
      <h2>오류가 발생했습니다</h2>
      <pre>{error.message}</pre>
      <button onClick={resetError}>다시 시도</button>
    </div>
  ),
  showDialog: true, // 사용자 피드백 다이얼로그 표시
});

// 또는 수동으로 리포팅
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Sentry로 에러 전송
    Sentry.withScope((scope) => {
      scope.setExtras(errorInfo);
      Sentry.captureException(error);
    });
  }

  render() {
    // 렌더링 로직
  }
}
```

<br />

### 테스팅

```jsx
// 에러 바운더리 테스트
import { render, screen } from "@testing-library/react";

// 에러를 발생시키는 컴포넌트
const ThrowError = ({ shouldThrow }) => {
  if (shouldThrow) {
    throw new Error("Test error");
  }
  return <div>정상 동작</div>;
};

describe("ErrorBoundary", () => {
  test("에러 없을 때 children 렌더링", () => {
    render(
      <ErrorBoundary>
        <ThrowError shouldThrow={false} />
      </ErrorBoundary>
    );

    expect(screen.getByText("정상 동작")).toBeInTheDocument();
  });

  test("에러 발생 시 폴백 UI 표시", () => {
    // console.error 일시 억제
    const spy = jest.spyOn(console, "error").mockImplementation(() => {});

    render(
      <ErrorBoundary>
        <ThrowError shouldThrow={true} />
      </ErrorBoundary>
    );

    expect(screen.getByText(/오류가 발생했습니다/)).toBeInTheDocument();

    spy.mockRestore();
  });

  test("reset 기능 동작 확인", () => {
    const { rerender } = render(
      <ErrorBoundary>
        <ThrowError shouldThrow={true} />
      </ErrorBoundary>
    );

    const resetButton = screen.getByText("다시 시도");
    resetButton.click();

    rerender(
      <ErrorBoundary>
        <ThrowError shouldThrow={false} />
      </ErrorBoundary>
    );

    expect(screen.getByText("정상 동작")).toBeInTheDocument();
  });
});
```

<br />

## 정리

### 에러 바운더리 사용 시 장점

- 애플리케이션 안정성 향상
- 더 나은 사용자 경험 제공
- 에러 추적 및 모니터링 용이
- 부분적 실패 처리 가능

### 주의사항

- 클래스 컴포넌트로만 구현 가능
- 이벤트 핸들러, 비동기 코드 에러는 잡지 못함
- 개발/프로덕션 환경별 다른 전략 필요
- 과도한 에러 바운더리는 복잡성 증가

<br />

# 참고 자료

> [React 공식 문서 - Error Boundaries](https://ko.react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)

> [React Error Boundary 라이브러리](https://github.com/bvaughn/react-error-boundary)

> [Sentry React 통합 가이드](https://docs.sentry.io/platforms/javascript/guides/react/)
