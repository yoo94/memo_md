# React Hooks 완벽 가이드 (React 16.8 ~ React 19)

**작성일**: 2025-12-08  
**대상**: React 16.8 이상  
**커버 범위**: 기본 Hook ~ React 19 최신 Hook

---

## 📑 목차

1. [기본 Hooks (React 16.8)](#기본-hooks-react-168)
   - useState
   - useEffect
   - useContext
2. [추가 Hooks (React 16.8)](#추가-hooks-react-168)
   - useReducer
   - useCallback
   - useMemo
   - useRef
   - useLayoutEffect
3. [Hooks 규칙 (Rules of Hooks)](#hooks-규칙-rules-of-hooks)
4. [React 18 Hooks](#react-18-hooks)
   - useTransition
   - useDeferredValue
   - useId
   - useInsertionEffect
   - useSyncExternalStore
5. [React 19 New Hooks](#react-19-new-hooks)
   - useActionState (새로움)
   - useFormStatus (새로움)
   - useOptimistic (새로움)
   - use() (새로움)
   - Enhanced useTransition
6. [Hook 동작 원리](#hook-동작-원리)
7.  [최적화 전략](#최적화-전략)
8. [Hook 선택 가이드](#hook-선택-가이드)

---

## 🎯 기본 Hooks (React 16.8)

### 1. useState

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | 함수형 컴포넌트에서 상태 관리 (클래스형의 `this.state` 대체) |
| **사용 목적** | 컴포넌트의 상태값 관리 |
| **언제 사용** | 컴포넌트가 변경되어야 하는 데이터를 다룰 때 |

#### 💻 사용법

```javascript
import { useState } from 'react';

// 1. 기본 사용
function Counter() {
  const [count, setCount] = useState(0);
  // count: 현재 상태값
  // setCount: 상태값을 변경하는 함수
  // 0: 초기값

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}

// 2. 여러 상태 관리
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input value={age} onChange={(e) => setAge(Number(e.target.value))} />
    </div>
  );
}

// 3. 함수형 초기값 (지연 초기화)
function ExpensiveCounter() {
  // 복잡한 계산은 함수로 감싸기 (첫 렌더링에만 실행)
  const [count, setCount] = useState(() => {
    return expensiveCalculation(); // 한 번만 실행
  });

  return <p>{count}</p>;
}

// 4. 이전 상태를 기반으로 업데이트
function GoodCounter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(prev => prev + 1)}>
      {/* 이렇게 하면 클로저 문제 해결 */}
      증가
    </button>
  );
}

// 5. 객체 상태 관리
function UserProfile() {
  const [user, setUser] = useState({ name: 'John', age: 30 });

  const updateName = (newName) => {
    setUser(prev => ({ ...prev, name: newName }));
  };

  return <p>{user.name}, {user.age}</p>;
}
```

#### ⚙️ 동작 원리

```javascript
// useState 내부 원리
let componentHooks = [];
let hookIndex = 0;

function useState(initialValue) {
  const index = hookIndex; // 각 Hook의 인덱스 저장 (클로저)
  hookIndex++;

  // 초기값 설정 (첫 렌더링에만)
  if (componentHooks[index] === undefined) {
    componentHooks[index] = {
      state: typeof initialValue === 'function' 
        ? initialValue() 
        : initialValue,
      queue: [] // 상태 업데이트 큐
    };
  }

  const hook = componentHooks[index];

  // 큐에 있는 모든 업데이트 적용
  hook.queue.forEach(action => {
    hook.state = typeof action === 'function'
      ? action(hook.state)
      : action;
  });
  hook.queue = [];

  const setState = (action) => {
    const currentHook = componentHooks[index];
    const newState = typeof action === 'function'
      ? action(currentHook.state)
      : action;

    // 상태가 변경되었는지 확인 (Object.is 사용)
    if (! Object.is(newState, currentHook.state)) {
      currentHook.queue.push(action);
      scheduleRender(); // 리렌더링 스케줄
    }
  };

  return [hook.state, setState];
}

// 렌더링 순서
// 1. Counter() 함수 호출
// 2. useState(0) → [0, setCount] 반환
// 3. JSX 반환
// 4. 버튼 클릭
// 5. setCount(1) 호출
// 6. scheduleRender() → 리렌더링 스케줄
// 7. Counter() 재호출
// 8. useState(0) → hook.queue 적용 → [1, setCount] 반환
```

---

### 2. useEffect

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | 함수형 컴포넌트에서 side effects 처리 (componentDidMount, componentDidUpdate, componentWillUnmount 통합) |
| **사용 목적** | API 호출, 구독, 타이머 등 부수 효과 관리 |
| **언제 사용** | 컴포넌트가 렌더링된 후 실행되어야 할 작업이 있을 때 |

#### 💻 사용법

```javascript
import { useEffect, useState } from 'react';

// 1. 의존성 배열 없음 - 매 렌더링마다 실행
function Effect1() {
  useEffect(() => {
    console. log('매 렌더링마다 실행');
  });
  // ⚠️ 성능 문제 주의! 
  return <div>Effect 1</div>;
}

// 2. 의존성 배열 [] - 마운트 시에만 실행 (componentDidMount)
function Effect2() {
  useEffect(() => {
    console. log('컴포넌트 마운트됨');
    // 초기 데이터 로드, 구독 설정 등
  }, []);

  return <div>Effect 2</div>;
}

// 3.  의존성 배열 [dependency] - dependency가 변경될 때만 실행
function Effect3() {
  const [userId, setUserId] = useState(1);
  
  useEffect(() => {
    console.log(`userId ${userId}가 변경되었습니다`);
    // userId에 따라 데이터 로드
  }, [userId]);

  return <button onClick={() => setUserId(userId + 1)}>변경</button>;
}

// 4.  Cleanup 함수 (componentWillUnmount)
function Effect4() {
  useEffect(() => {
    const timer = setInterval(() => {
      console. log('타이머 실행');
    }, 1000);

    // cleanup: 컴포넌트 언마운트 또는 다음 effect 실행 전에 호출
    return () => {
      clearInterval(timer);
      console.log('타이머 정지');
    };
  }, []);

  return <div>Effect 4</div>;
}

// 5. API 호출 패턴
function DataFetcher() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isMounted = true; // 언마운트 감지

    const fetchData = async () => {
      try {
        const response = await fetch('/api/data');
        const result = await response.json();
        if (isMounted) {
          setData(result);
        }
      } catch (err) {
        if (isMounted) {
          setError(err);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchData();

    // cleanup
    return () => {
      isMounted = false;
    };
  }, []);

  if (loading) return <p>로딩 중...</p>;
  if (error) return <p>에러 발생</p>;
  return <div>{JSON.stringify(data)}</div>;
}

// 6. 이벤트 리스너 등록/제거
function EventListener() {
  useEffect(() => {
    const handleResize = () => {
      console.log(`Window size: ${window.innerWidth}x${window.innerHeight}`);
    };

    window.addEventListener('resize', handleResize);

    // cleanup
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>Event Listener</div>;
}

// 7. 다중 effect
function MultipleEffects() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Effect 1
  useEffect(() => {
    console.log('count 변경:', count);
  }, [count]);

  // Effect 2
  useEffect(() => {
    console.log('name 변경:', name);
  }, [name]);

  // Effect 3
  useEffect(() => {
    console.log('마운트됨');
    return () => console.log('언마운트됨');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Name: {name}</p>
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useEffect 내부 원리
let effects = [];
let effectIndex = 0;

function useEffect(callback, dependencies) {
  const index = effectIndex++;
  const lastDependencies = effects[index]?.dependencies;

  // 의존성 배열 비교
  const hasChanged = ! lastDependencies || 
    dependencies. some((dep, i) => 
      !Object.is(dep, lastDependencies[i])
    );

  if (hasChanged) {
    // 이전 effect의 cleanup 함수 실행
    if (effects[index]?.cleanup) {
      effects[index].cleanup();
    }

    // 새로운 effect 실행 (마이크로태스크 큐에 등록)
    Promise.resolve().then(() => {
      const cleanup = callback();
      
      // effect 저장
      effects[index] = {
        callback,
        dependencies,
        cleanup
      };
    });
  }
}

// 실행 순서
// 1. 컴포넌트 렌더링
// 2.  JSX 반환
// 3. DOM 업데이트
// 4. 브라우저가 페인트 (화면에 그려짐)
// 5. useEffect 콜백 실행 (브라우저 페인트 후)
```

---

### 3. useContext

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | Props Drilling을 피하고 전역 데이터를 쉽게 공유 |
| **사용 목적** | 컴포넌트 트리의 깊은 곳까지 데이터 전달 |
| **언제 사용** | 테마, 사용자 인증 정보, 다국어 설정 등 전역 상태가 필요할 때 |

#### 💻 사용법

```javascript
import { createContext, useContext, useState } from 'react';

// 1. Context 생성
const ThemeContext = createContext();

// 2. Context Provider 설정
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const value = {
    theme,
    setTheme,
    toggleTheme: () => setTheme(prev => prev === 'light' ?  'dark' : 'light')
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext. Provider>
  );
}

// 3. Custom Hook 작성 (선택사항이지만 권장)
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme는 ThemeProvider 내부에서만 사용 가능');
  }
  return context;
}

// 4. 컴포넌트에서 useContext 사용
function Button() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      style={{
        background: theme === 'light' ?  '#fff' : '#333',
        color: theme === 'light' ?  '#000' : '#fff'
      }}
      onClick={toggleTheme}
    >
      테마 변경
    </button>
  );
}

// 5. App에서 Provider로 감싸기
function App() {
  return (
    <ThemeProvider>
      <Button />
      <Content />
    </ThemeProvider>
  );
}

// 6. 다중 Context
const UserContext = createContext();
const NotificationContext = createContext();

function AppProviders({ children }) {
  return (
    <UserContext.Provider value={{ user: 'John' }}>
      <NotificationContext.Provider value={{ notifications: [] }}>
        {children}
      </NotificationContext.Provider>
    </UserContext.Provider>
  );
}

// 7. Context 값 변경 시 리렌더링
function ContextValue() {
  const [count, setCount] = useState(0);
  const CountContext = createContext();

  // 주의: 매번 새로운 객체를 생성하면 불필요한 리렌더링 발생
  const value = { count, setCount }; // ❌ 매번 새로운 객체

  // ✅ useMemo로 감싸기
  const memoizedValue = useMemo(() => 
    ({ count, setCount }),
    [count]
  );

  return (
    <CountContext.Provider value={memoizedValue}>
      {/* ... */}
    </CountContext.Provider>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// Context 생성 원리
function createContext(defaultValue) {
  const context = {
    Provider: function Provider({ value, children }) {
      context.currentValue = value;
      return children;
    },
    consumers: new Set()
  };
  
  return context;
}

// useContext 원리
function useContext(context) {
  // 가장 가까운 Provider의 value를 찾음
  const value = findNearestProviderValue(context);
  
  if (value === undefined) {
    throw new Error('useContext는 Provider 내부에서만 사용 가능');
  }
  
  return value;
}

// Provider 체이닝 - 가장 가까운 Provider를 사용
<MyContext.Provider value={{ theme: 'light' }}>
  <Component1 /> {/* 'light' */}
  
  <MyContext.Provider value={{ theme: 'dark' }}>
    <Component2 /> {/* 'dark' - 더 가까운 Provider 사용 */}
  </MyContext.Provider>
</MyContext.Provider>

// Context 값 변경 시 리렌더링
// Provider의 value 객체가 변경되면 모든 useContext 소비자가 리렌더링됨
```

---

## 📚 추가 Hooks (React 16.8)

### 4. useReducer

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | 복잡한 상태 로직을 관리 (useState의 고급 버전) |
| **사용 목적** | 여러 관련된 상태를 하나로 관리 |
| **언제 사용** | 상태 전환이 복잡하거나 Redux 패턴이 필요할 때 |

#### 💻 사용법

```javascript
import { useReducer } from 'react';

// 1.  Reducer 함수 정의
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + (action.payload || 1) };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    case 'SET':
      return { count: action. payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const initialState = { count: 0 };
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <div>
      <p>Count: {state. count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        증가
      </button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>
        감소
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>
        초기화
      </button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>
        10으로 설정
      </button>
    </div>
  );
}

// 2. 복잡한 상태 관리
function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        [action.field]: action.value
      };
    case 'RESET':
      return { name: '', email: '', age: '' };
    case 'SET_ERROR':
      return { ...state, error: action.error };
    default:
      return state;
  }
}

function Form() {
  const [state, dispatch] = useReducer(formReducer, {
    name: '',
    email: '',
    age: '',
    error: null
  });

  const handleChange = (e) => {
    dispatch({
      type: 'SET_FIELD',
      field: e.target.name,
      value: e.target.value
    });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    // 유효성 검사
    if (!state.name) {
      dispatch({ type: 'SET_ERROR', error: '이름을 입력하세요' });
      return;
    }
    // 제출
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={state.name}
        onChange={handleChange}
      />
      <input
        name="email"
        value={state.email}
        onChange={handleChange}
      />
      {state.error && <p>{state.error}</p>}
      <button type="submit">제출</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>
        초기화
      </button>
    </form>
  );
}

// 3. 지연 초기화
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [... state, { id: Date.now(), text: action.text }];
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.id);
    default:
      return state;
  }
}

function initTodos(initialCount) {
  // 초기값을 함수로 계산 (비용이 큼)
  return Array.from({ length: initialCount }, (_, i) => ({
    id: i,
    text: `Todo ${i}`
  }));
}

function TodoList() {
  const [todos, dispatch] = useReducer(
    todoReducer,
    10, // initialArg
    initTodos // init 함수 (선택사항)
  );

  return (
    <div>
      {todos.map(todo => (
        <div key={todo.id}>
          {todo.text}
          <button onClick={() => dispatch({ type: 'REMOVE_TODO', id: todo.id })}>
            삭제
          </button>
        </div>
      ))}
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useReducer 내부 원리
function useReducer(reducer, initialState, init) {
  let state = init ?  init(initialState) : initialState;

  const dispatch = (action) => {
    // reducer 함수 호출: (현재상태, action) => 새로운상태
    state = reducer(state, action);
    scheduleRender(); // 리렌더링 스케줄
  };

  return [state, dispatch];
}

// 동작 순서
// 1. useReducer(reducer, initialState) 호출
// 2.  state = initialState, dispatch 반환
// 3. 버튼 클릭 → dispatch({ type: 'INCREMENT' })
// 4. reducer(state, action) 호출
// 5. new state 반환
// 6.  state 업데이트, 리렌더링
```

---

### 5. useCallback

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | 함수의 메모이제이션으로 불필요한 렌더링 최적화 |
| **사용 목적** | 콜백 함수를 메모이제이션하여 자식 컴포넌트에 안정적으로 전달 |
| **언제 사용** | 콜백이 자식 컴포넌트의 의존성 배열에 들어갈 때 |

#### 💻 사용법

```javascript
import { useCallback, useState, memo } from 'react';

// 1. 기본 사용
function Parent() {
  const [count, setCount] = useState(0);

  // useCallback 없음 - 렌더링마다 새로운 함수 생성
  // const handleClick = () => {
  //   setCount(count + 1);
  // };

  // useCallback 사용 - count가 변경될 때만 새로운 함수 생성
  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]); // 의존성 배열

  return (
    <div>
      <p>Count: {count}</p>
      <Child onClick={handleClick} />
    </div>
  );
}

// 2. 자식 컴포넌트 최적화
const Child = memo(function Child({ onClick }) {
  console.log('Child 렌더링');
  return <button onClick={onClick}>증가</button>;
});

// 3. 여러 콜백
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleNameChange = useCallback((e) => {
    setName(e. target.value);
  }, []); // 의존성 없음 - 항상 같은 함수

  const handleEmailChange = useCallback((e) => {
    setEmail(e.target.value);
  }, []); // 의존성 없음

  return (
    <div>
      <input value={name} onChange={handleNameChange} />
      <input value={email} onChange={handleEmailChange} />
    </div>
  );
}

// 4.  함수형 업데이트로 의존성 줄이기
function Counter() {
  const [count, setCount] = useState(0);

  // ❌ count를 의존성에 포함
  // const handleClick = useCallback(() => {
  //   setCount(count + 1);
  // }, [count]);

  // ✅ 함수형 업데이트로 의존성 제거
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // 의존성 없음

  return <button onClick={handleClick}>{count}</button>;
}

// 5. 복잡한 콜백
function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = useCallback((text) => {
    setTodos(prev => [
      ...prev,
      { id: Date.now(), text }
    ]);
  }, []); // 의존성 없음

  const removeTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []); // 의존성 없음

  return (
    <div>
      <button onClick={() => addTodo('New Task')}>추가</button>
      {todos.map(todo => (
        <div key={todo.id}>
          {todo.text}
          <button onClick={() => removeTodo(todo. id)}>삭제</button>
        </div>
      ))}
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useCallback 내부 원리
let memoizedCallbacks = [];
let callbackIndex = 0;

function useCallback(callback, dependencies) {
  const index = callbackIndex++;
  const lastDependencies = memoizedCallbacks[index]?.dependencies;

  // 의존성 비교
  const hasChanged = !lastDependencies ||
    dependencies.some((dep, i) =>
      !Object.is(dep, lastDependencies[i])
    );

  if (hasChanged) {
    // 의존성이 변경되었으면 새 함수 생성
    memoizedCallbacks[index] = {
      callback,
      dependencies
    };
  }

  // 이전 함수 참조 반환 (의존성이 변경되지 않았으면)
  return memoizedCallbacks[index]. callback;
}

// 렌더링 시나리오
// 1. 초기: count=0
// 2. name 변경 → 리렌더링
//    - handleClick은 count가 변경되지 않았으므로 같은 함수 참조
//    - Child는 props 변경 없음 → 리렌더링 안 됨 (memo)
// 3. count 변경 → 리렌더링
//    - handleClick은 새로운 함수 (의존성 변경)
//    - Child는 handleClick이 변경되었으므로 리렌더링됨
```

---

### 6. useMemo

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | 비용이 큰 계산 결과를 메모이제이션하여 성능 최적화 |
| **사용 목적** | 복잡한 계산 결과를 캐시 |
| **언제 사용** | 렌더링할 때마다 비용이 큰 계산이 필요할 때 |

#### 💻 사용법

```javascript
import { useMemo, useState } from 'react';

// 1. 기본 사용
function ExpensiveCalculation() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // useMemo 없음 - 매 렌더링마다 계산
  // const expensiveValue = fibonacci(count);

  // useMemo 사용 - count가 변경될 때만 계산
  const expensiveValue = useMemo(() => {
    console.log('복잡한 계산 중...');
    return fibonacci(count);
  }, [count]); // 의존성 배열

  function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="이름 입력 (계산에 영향 없음)"
      />
      <p>Count: {count}</p>
      <p>Fibonacci: {expensiveValue}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}

// 2. 객체 메모이제이션
function UserProfile() {
  const [userId, setUserId] = useState(1);
  const [filter, setFilter] = useState('');

  // 객체가 변경되어도 리렌더링되지 않도록
  const user = useMemo(() => ({
    id: userId,
    name: 'John',
    email: 'john@example.com'
  }), [userId]); // userId 변경 시만 새 객체 생성

  return (
    <div>
      <Profile user={user} />
      <input value={filter} onChange={(e) => setFilter(e. target.value)} />
    </div>
  );
}

// 3. 배열 메모이제이션
function FilteredList() {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  const [filter, setFilter] = useState(2);

  // 배열이 변경되어도 리렌더링되지 않도록
  const filteredItems = useMemo(() => {
    console.log('필터링 중...');
    return items.filter(item => item >= filter);
  }, [items, filter]); // items 또는 filter 변경 시만 계산

  return (
    <div>
      <List items={filteredItems} />
      <button onClick={() => setFilter(filter + 1)}>필터 증가</button>
    </div>
  );
}

// 4. useMemo vs 단순 계산
function Comparison() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // 방법 1: 단순 계산 (name 변경 시마다 재계산)
  const value1 = fibonacci(count);

  // 방법 2: useMemo (name 변경 시 재계산 안 함)
  const value2 = useMemo(() => fibonacci(count), [count]);

  function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
  }

  return (
    <div>
      <p>단순 계산: {value1}</p>
      <p>useMemo: {value2}</p>
      <input value={name} onChange={(e) => setName(e.target. value)} />
    </div>
  );
}

// 5. 주의: 과도한 최적화 피하기
function GoodOptimization() {
  const [count, setCount] = useState(0);

  // ❌ 과도한 최적화 - 간단한 계산은 useMemo 불필요
  const simpleValue = useMemo(() => count * 2, [count]);

  // ✅ 필요한 최적화만 - 비용이 큰 계산만
  const complexValue = useMemo(() => {
    // 복잡한 계산... 
    return heavyComputation(count);
  }, [count]);

  return <div>{simpleValue} / {complexValue}</div>;
}
```

#### ⚙️ 동작 원리

```javascript
// useMemo 내부 원리
let memoizedValues = [];
let memoIndex = 0;

function useMemo(computeValue, dependencies) {
  const index = memoIndex++;
  const lastMemo = memoizedValues[index];

  // 의존성 비교
  const hasChanged = !lastMemo ||
    dependencies.some((dep, i) =>
      !Object. is(dep, lastMemo. dependencies[i])
    );

  if (hasChanged) {
    // 의존성이 변경되었으면 계산 실행
    memoizedValues[index] = {
      value: computeValue(),
      dependencies
    };
  }

  // 이전 값 반환
  return memoizedValues[index].value;
}

// 메모리 사용
// - 계산 결과를 메모리에 저장
// - 의존성 배열도 저장
// - useMemo 자체도 비용이 있으므로 중요한 경우만 사용
```

---

### 7.  useRef

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | DOM에 직접 접근하거나 변경되어도 리렌더링하지 않을 값을 저장 |
| **사용 목적** | DOM 조작, 타이머 ID 저장, 이전 값 저장 |
| **언제 사용** | 비제어 컴포넌트(input), 포커스 관리, 라이브러리 통합 |

#### 💻 사용법

```javascript
import { useRef, useState, useEffect } from 'react';

// 1. DOM 요소 직접 접근
function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current. focus();
  };

  const clearInput = () => {
    inputRef.current.value = '';
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>포커스</button>
      <button onClick={clearInput}>초기화</button>
    </div>
  );
}

// 2. 비제어 컴포넌트
function Form() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e. preventDefault();
    console.log('Name:', nameRef.current.value);
    console.log('Email:', emailRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} type="text" placeholder="Name" />
      <input ref={emailRef} type="email" placeholder="Email" />
      <button type="submit">제출</button>
    </form>
  );
}

// 3.  렌더링 횟수 카운트
function RenderCount() {
  const renderCount = useRef(0);

  // ❌ 렌더링되지 않음 (renderCount. current++ 자체가 리렌더링을 트리거하지 않음)
  renderCount.current++;

  return <p>렌더링 횟수: {renderCount.current}</p>;
}

// 4. 이전 값 저장
function PrevValue() {
  const [value, setValue] = useState('');
  const prevValueRef = useRef('');

  useEffect(() => {
    prevValueRef.current = value; // 렌더링 후 업데이트
  }, [value]);

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <p>현재: {value}</p>
      <p>이전: {prevValueRef.current}</p>
    </div>
  );
}

// 5.  타이머 ID 저장
function Timer() {
  const intervalRef = useRef(null);
  const [seconds, setSeconds] = useState(0);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef. current) {
      clearInterval(intervalRef.current);
    }
  };

  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={startTimer}>시작</button>
      <button onClick={stopTimer}>정지</button>
    </div>
  );
}

// 6. 라이브러리 인스턴스 저장
function VideoPlayer() {
  const videoRef = useRef(null);

  const play = () => {
    videoRef.current?. play();
  };

  const pause = () => {
    videoRef.current?.pause();
  };

  return (
    <div>
      <video ref={videoRef} src="movie.mp4" />
      <button onClick={play}>재생</button>
      <button onClick={pause}>정지</button>
    </div>
  );
}

// 7. 포커스 관리
function FocusManager() {
  const firstInputRef = useRef(null);
  const secondInputRef = useRef(null);

  const focusFirst = () => {
    firstInputRef.current?.focus();
  };

  const focusSecond = () => {
    secondInputRef.current?.focus();
  };

  return (
    <div>
      <input ref={firstInputRef} placeholder="First" />
      <input ref={secondInputRef} placeholder="Second" />
      <button onClick={focusFirst}>첫 번째 포커스</button>
      <button onClick={focusSecond}>두 번째 포커스</button>
    </div>
  );
}

// 8. useRef vs useState
function Comparison() {
  const [stateValue, setStateValue] = useState(0);
  const refValue = useRef(0);

  const increaseState = () => {
    setStateValue(prev => prev + 1); // 리렌더링 ✅
  };

  const increaseRef = () => {
    refValue. current++; // 리렌더링 없음 ✓
  };

  return (
    <div>
      <p>State: {stateValue}</p>
      <p>Ref: {refValue.current}</p>
      <button onClick={increaseState}>State 증가</button>
      <button onClick={increaseRef}>Ref 증가</button>
      {/* Ref 증가는 화면에 반영되지 않음 */}
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useRef 내부 원리
let refs = [];
let refIndex = 0;

function useRef(initialValue) {
  const index = refIndex++;

  // 처음 한 번만 객체 생성 (이후는 같은 객체 반환)
  if (!refs[index]) {
    refs[index] = {
      current: initialValue
    };
  }

  return refs[index];
}

// 특징
// 1. 매번 호출할 때마다 동일한 객체 반환
// 2. current 속성 변경해도 리렌더링 안 됨
// 3. 렌더링과 무관하게 값이 유지됨

// 타이밍
// 렌더링 직후에 DOM에 ref가 연결됨
// useEffect 내에서 ref. current 접근 가능
```

---

### 8. useLayoutEffect

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **생긴 이유** | DOM 변경 후 **동기적으로** 실행되어야 할 side effects 처리 |
| **사용 목적** | DOM 측정, 스타일 조정 등 레이아웃 계산 후 작업 |
| **언제 사용** | useEffect보다 먼저 실행되어야 하는 작업이 필요할 때 |

#### 💻 사용법

```javascript
import { useLayoutEffect, useEffect, useRef, useState } from 'react';

// 1. useEffect vs useLayoutEffect
function Timing() {
  useLayoutEffect(() => {
    console. log('2.  useLayoutEffect 실행 (화면 그려지기 전)');
  }, []);

  useEffect(() => {
    console.log('3. useEffect 실행 (화면 그려진 후)');
  }, []);

  console.log('1. 렌더링 함수 실행');

  return <div>Timing</div>;
}
// 출력 순서:
// 1. 렌더링 함수 실행
// 2. useLayoutEffect 실행
// [DOM 업데이트, 화면에 그려짐]
// 3. useEffect 실행

// 2. DOM 측정
function Tooltip() {
  const ref = useRef(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });

  useLayoutEffect(() => {
    const element = ref.current;
    const rect = element.getBoundingClientRect();

    setPosition({
      top: rect.top,
      left: rect.left
    });
  }, []);

  return (
    <div
      ref={ref}
      style={{
        position: 'absolute',
        top: position.top,
        left: position.left,
        background: 'yellow'
      }}
    >
      Tooltip
    </div>
  );
}

// 3. 스타일 조정 (깜빡임 없음)
function StyledComponent() {
  const ref = useRef(null);

  useLayoutEffect(() => {
    // 화면에 그려지기 전에 스타일 조정
    if (ref.current) {
      ref.current.style.opacity = '1';
    }
  }, []);

  return (
    <div ref={ref} style={{ opacity: '0' }}>
      내용
    </div>
  );
}

// 4. 주의: 성능 영향
function Performance() {
  // ❌ useLayoutEffect는 브라우저 페인팅을 블로킹함
  useLayoutEffect(() => {
    // 비용이 큰 계산
    for (let i = 0; i < 1000000; i++) {
      // ... 
    }
  }, []);

  // ✅ 가능하면 useEffect 사용
  useEffect(() => {
    // 비용이 큰 계산
    for (let i = 0; i < 1000000; i++) {
      // ...
    }
  }, []);

  return <div>Performance</div>;
}

// 5. 여러 항목 측정
function MultipleElements() {
  const firstRef = useRef(null);
  const secondRef = useRef(null);
  const [measurements, setMeasurements] = useState(null);

  useLayoutEffect(() => {
    const firstRect = firstRef.current.getBoundingClientRect();
    const secondRect = secondRef.current.getBoundingClientRect();

    setMeasurements({
      first: firstRect,
      second: secondRect
    });
  }, []);

  return (
    <div>
      <div ref={firstRef}>첫 번째</div>
      <div ref={secondRef}>두 번째</div>
      {measurements && (
        <p>
          첫 번째 높이: {measurements.first.height}
          <br />
          두 번째 높이: {measurements.second.height}
        </p>
      )}
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useLayoutEffect 실행 순서
// 1. 컴포넌트 함수 실행
// 2. JSX 반환
// 3. React가 가상 DOM 업데이트
// 4. DOM에 실제 변경 적용
// 5. useLayoutEffect 콜백 실행 (동기적) ← 브라우저 페인팅 블로킹
// 6. cleanup 함수 실행 (다음 effect 직전)
// 7. 브라우저가 페인팅 (화면에 그려짐)
// 8. useEffect 콜백 실행 (비동기)

// 성능 영향
useLayoutEffect(() => {
  // 이 코드가 실행 중인 동안 브라우저는 페인팅할 수 없음
  // → 사용자 입장에서는 응답 없음으로 느낄 수 있음
}, []);

// 규칙
// - 가능하면 useEffect 사용
// - DOM 측정, 스타일 조정이 필요한 경우만 useLayoutEffect 사용
// - 비용이 큰 작업은 useLayoutEffect에서 피하기
```

---

## 📋 Hooks 규칙 (Rules of Hooks)

### 1️⃣ Hook은 최상위에서만 호출하세요

```javascript
// ❌ 잘못된 예
function BadComponent({ condition }) {
  if (condition) {
    const [state, setState] = useState(0); // ❌ 조건부 호출
  }
  const [name, setName] = useState('');
}

// ✅ 올바른 예
function GoodComponent({ condition }) {
  const [state, setState] = useState(0);
  const [name, setName] = useState('');

  if (condition) {
    // Hook 외부에서 조건부 로직
  }
}
```

### 2️⃣ Hook은 함수형 컴포넌트와 Custom Hook에서만 호출하세요

```javascript
// ❌ 잘못된 예 - 클래스 컴포넌트
class BadComponent extends React.Component {
  render() {
    const [state, setState] = useState(0); // ❌ 오류
  }
}

// ❌ 잘못된 예 - 일반 함수
function notAComponent() {
  const [state, setState] = useState(0); // ❌ 오류
}

// ✅ 올바른 예 - 함수형 컴포넌트
function GoodComponent() {
  const [state, setState] = useState(0);
}

// ✅ 올바른 예 - Custom Hook
function useCustomHook() {
  const [state, setState] = useState(0);
  return state;
}
```

### 3️⃣ Hook 호출 순서는 항상 일정해야 합니다

```javascript
// ❌ 잘못된 예
function BadComponent() {
  const [name, setName] = useState('');

  if (someCondition) {
    // 조건에 따라 Hook 호출 순서 변경
    const [age, setAge] = useState(0);
  }

  const [email, setEmail] = useState('');
  // someCondition = true: name(0), age(1), email(2)
  // someCondition = false: name(0), email(1)
  // ← 인덱스 불일치! 
}

// ✅ 올바른 예
function GoodComponent() {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  const [email, setEmail] = useState('');

  if (someCondition) {
    // 조건부 로직은 Hook 호출 후
  }
}
```

### 왜 이런 규칙이 있나? 

```javascript
// React는 Hook의 호출 순서로 상태를 추적함
// 각 Hook은 인덱스 기반으로 상태에 연결됨

// 예:
function Component() {
  const [state1, setState1] = useState(0); // hooks[0]
  const [state2, setState2] = useState(''); // hooks[1]
  const [state3, setState3] = useState(null); // hooks[2]
}

// 호출 순서가 바뀌면:
function Component() {
  const [state3, setState3] = useState(null); // hooks[0] ← 잘못됨! 
  const [state1, setState1] = useState(0); // hooks[1] ← 잘못됨!
  const [state2, setState2] = useState(''); // hooks[2] ← 잘못됨!
}
```

---

## 🚀 React 18 Hooks

### 9. useTransition

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **생긴 이유** | 긴급하지 않은 상태 업데이트를 분리하여 UI 반응성 개선 |
| **사용 목적** | 검색, 필터링 같은 작업의 성능 최적화 |
| **언제 사용** | 빠른 응답이 필요한 업데이트와 느린 업데이트를 분리할 때 |

#### 💻 사용법

```javascript
import { useTransition, useState } from 'react';

// 1. 기본 사용
function SearchUsers() {
  const [input, setInput] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const value = e.target.value;
    setInput(value); // 긴급 업데이트 - 즉시 반영

    // 느린 업데이트 - isPending 동안 이전 결과 표시
    startTransition(() => {
      const filtered = users.filter(user =>
        user.name.includes(value)
      );
      setResults(filtered);
    });
  };

  return (
    <div>
      <input value={input} onChange={handleSearch} />
      {isPending && <p>검색 중...</p>}
      <ul>
        {results.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// 2. 탭 전환
function Tabs() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const selectTab = (nextTab) => {
    startTransition(() => {
      setTab(nextTab);
    });
  };

  return (
    <div>
      <button
        onClick={() => selectTab('home')}
        disabled={isPending}
      >
        Home
      </button>
      <button
        onClick={() => selectTab('posts')}
        disabled={isPending}
      >
        Posts
      </button>
      <button
        onClick={() => selectTab('about')}
        disabled={isPending}
      >
        About
      </button>
      
      {isPending ?  <p>로딩 중...</p> : <TabContent tab={tab} />}
    </div>
  );
}

// 3. React 19: async transition
function AsyncTransition() {
  const [data, setData] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleFetch = () => {
    startTransition(async () => {
      const response = await fetch('/api/data');
      const result = await response. json();
      setData(result);
    });
  };

  return (
    <div>
      <button onClick={handleFetch} disabled={isPending}>
        {isPending ? '로딩 중...' : '데이터 로드'}
      </button>
      {data && <p>{JSON.stringify(data)}</p>}
    </div>
  );
}

// 4.  느린 리스트
function SlowList({ items }) {
  let startTime = Date.now();
  while (Date.now() - startTime < 500) {
    // 의도적으로 느리게 함
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item. name}</li>
      ))}
    </ul>
  );
}

function ListExample() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value); // 긴급 - 즉시 반영

    startTransition(() => {
      const newList = generateList(value);
      setList(newList);
    });
  };

  return (
    <div>
      <input value={input} onChange={handleChange} />
      {isPending ?  (
        <p>생성 중...</p>
      ) : (
        <SlowList items={list} />
      )}
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useTransition 동작 순서
// 1. startTransition() 호출
// 2. 콜백 함수 실행
// 3. 콜백 내 setState 호출 → 업데이트를 "전환" 상태로 표시
// 4. isPending = true
// 5. React가 우선순위 낮은 작업으로 처리
// 6. 다른 긴급 작업이 있으면 일시 중지
// 7. 업데이트 완료
// 8. isPending = false

// 시간대
input 값 변경 (긴급)
└─ setInput() → 즉시 렌더링
└─ startTransition() 콜백 (비긴급)
   └─ setList() → 나중에 렌더링
```

---

### 10.  useDeferredValue

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **생긴 이유** | 값의 업데이트를 지연시켜 성능 최적화 |
| **사용 목적** | 느린 렌더링이 필요한 컴포넌트의 성능 개선 |
| **언제 사용** | 리스트 필터링, 검색 결과 표시 같은 상황 |

#### 💻 사용법

```javascript
import { useDeferredValue, useState, useMemo } from 'react';

// 1. 기본 사용
function SearchApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const results = useMemo(() => {
    // deferredQuery가 변경될 때만 계산
    return searchItems(deferredQuery);
  }, [deferredQuery]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색..."
      />
      {query !== deferredQuery && <p>검색 중...</p>}
      <SearchResults results={results} />
    </div>
  );
}

// 2. useTransition과의 차이
// useTransition: 상태 업데이트 함수를 감싸기
// useDeferredValue: 값 자체를 지연시키기

function Comparison() {
  const [query, setQuery] = useState('');
  
  // 방법 1: useTransition
  const [isPending, startTransition] = useTransition();
  const handleSearch = (e) => {
    const value = e.target.value;
    setQuery(value);
    startTransition(() => {
      // 필터링 로직
    });
  };

  // 방법 2: useDeferredValue
  const [query2, setQuery2] = useState('');
  const deferredQuery = useDeferredValue(query2);
  const handleSearch2 = (e) => {
    setQuery2(e.target.value);
  };
}

// 3. 느린 컴포넌트 최적화
function SlowComponent({ items }) {
  let startTime = Date.now();
  while (Date.now() - startTime < 1000) {
    // 의도적으로 느리게 함
  }

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

function OptimizedList() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const items = useMemo(() => {
    return filterItems(deferredQuery);
  }, [deferredQuery]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {query !== deferredQuery && <p>렌더링 중...</p>}
      <SlowComponent items={items} />
    </div>
  );
}

// 4. 여러 deferred 값
function MultipleDeferred() {
  const [query, setQuery] = useState('');
  const [filter, setFilter] = useState('');

  const deferredQuery = useDeferredValue(query);
  const deferredFilter = useDeferredValue(filter);

  const results = useMemo(() => {
    return filterResults(deferredQuery, deferredFilter);
  }, [deferredQuery, deferredFilter]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <input
        value={filter}
        onChange={(e) => setFilter(e.target. value)}
      />
      <Results data={results} />
    </div>
  );
}
```

#### ⚙️ 동작 원리

```javascript
// useD
