# React 18 ~ 19 Hooks 완벽 가이드

**작성일**: 2025-12-09  
**대상**:  React 18 이상  
**커버 범위**: React 18 ~ React 19 최신 Hooks

---

## 📑 목차

1. [React 18 Hooks](#react-18-hooks)
   - useTransition
   - useDeferredValue
   - useId
   - useInsertionEffect
   - useSyncExternalStore

2. [React 19 New Hooks](#react-19-new-hooks)
   - useActionState
   - useFormStatus
   - useOptimistic
   - use()
   - Enhanced useTransition

3. [Hooks 비교 및 선택 가이드](#hooks-비교-및-선택-가이드)

---
## 🚀 React 18 Hooks

# `useDeferredValue` vs `useTransition` 비교

| 특징 | `useDeferredValue` | `useTransition` |
|------|-------------------|-----------------|
| **목적** | 값(state)을 지연시킴 | 렌더링을 지연시킴 |
| **사용 대상** | Props나 State 값 | 상태 업데이트 함수 |
| **반환값** | 지연된 값 | (isPending, startTransition) |
| **제어 방식** | 자동으로 값 지연 | 명시적으로 함수 래핑 |

### 1. useTransition

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **목적** | 긴급하지 않은 상태 업데이트를 분리하여 UI 반응성 개선 |
| **사용처** | 검색, 필터링, 탭 전환 등의 느린 업데이트 |
| **반환값** | [isPending, startTransition] |

#### 💻 기본 사용법

```javascript
import { useTransition, useState } from 'react';

// 1. 검색 기능 최적화
function SearchUsers() {
  const [input, setInput] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const value = e.target.value;
    
    // 긴급 업데이트 - 즉시 반영 (입력값)
    setInput(value);

    // 비긴급 업데이트 - 나중에 렌더링 (검색 결과)
    startTransition(() => {
      const filtered = users.filter(user =>
        user.name.includes(value)
      );
      setResults(filtered);
    });
  };

  return (
    <div>
      <input 
        value={input} 
        onChange={handleSearch}
        placeholder="사용자 검색..."
      />
      {isPending && <p>검색 중... </p>}
      <ul>
        {results.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// 2. 탭 전환 최적화
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
        style={{
          opacity: tab === 'home' ? 1 : 0.6
        }}
      >
        Home
      </button>
      <button 
        onClick={() => selectTab('posts')}
        disabled={isPending}
        style={{
          opacity: tab === 'posts' ?  1 : 0.6
        }}
      >
        Posts
      </button>
      <button 
        onClick={() => selectTab('about')}
        disabled={isPending}
        style={{
          opacity: tab === 'about' ?  1 : 0.6
        }}
      >
        About
      </button>
      
      {isPending && <p>로딩 중...</p>}
      <TabContent tab={tab} />
    </div>
  );
}

// 3. 여러 상태 업데이트
function FilteredList() {
  const [items, setItems] = useState([]);
  const [filter, setFilter] = useState('');
  const [sort, setSort] = useState('name');
  const [isPending, startTransition] = useTransition();

  const handleFilter = (value) => {
    setFilter(value);
    
    startTransition(() => {
      // 복잡한 필터링 및 정렬
      const filtered = items
        .filter(item => item.name.includes(value))
        .sort((a, b) => {
          if (sort === 'name') return a.name.localeCompare(b.name);
          return a.date - b.date;
        });
      setItems(filtered);
    });
  };

  return (
    <div>
      <input 
        onChange={(e) => handleFilter(e.target.value)}
        placeholder="필터..."
      />
      {isPending && <div className="spinner" />}
    </div>
  );
}
```

#### ⚙️ useTransition vs useState

```javascript
// ❌ useTransition 없이 - 입력이 느림
function BadSearch() {
  const [input, setInput] = useState('');
  const [results, setResults] = useState([]);

  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value);
    // 입력 중에도 무거운 필터링이 일어남
    setResults(
      largeDataset.filter(item => item.name.includes(value))
    );
  };

  return <input value={input} onChange={handleChange} />;
}

// ✅ useTransition 사용 - 입력이 즉시 반응
function GoodSearch() {
  const [input, setInput] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value); // 즉시 반영
    
    startTransition(() => {
      // 나중에 처리
      setResults(
        largeDataset.filter(item => item.name.includes(value))
      );
    });
  };

  return (
    <>
      <input value={input} onChange={handleChange} />
      {isPending && <LoadingSpinner />}
    </>
  );
}
```

---

### 2. useDeferredValue

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **목적** | 값의 업데이트를 지연시켜 성능 최적화 |
| **사용처** | 느린 렌더링을 하는 자식 컴포넌트에 값 전달 |
| **반환값** | 지연된 값 |

#### 💻 기본 사용법

```javascript
import { useDeferredValue, useState, useMemo } from 'react';

// 1. 기본 사용 - 검색
function SearchApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const results = useMemo(() => {
    return searchItems(deferredQuery);
  }, [deferredQuery]);

  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색..."
      />
      {isStale && <p>검색 중...</p>}
      <SearchResults results={results} />
    </div>
  );
}

// 2. 느린 리스트 렌더링
function SlowList({ items }) {
  let startTime = Date.now();
  
  // 의도적으로 느리게 함
  while (Date.now() - startTime < 500) {
    // 루프
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
        placeholder="필터..."
      />
      {query !== deferredQuery && <p>렌더링 중...</p>}
      <SlowList items={items} />
    </div>
  );
}

// 3. 여러 deferred 값
function AdvancedFilter() {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  const [priceRange, setPriceRange] = useState([0, 1000]);

  const deferredQuery = useDeferredValue(query);
  const deferredCategory = useDeferredValue(category);
  const deferredPrice = useDeferredValue(priceRange);

  const results = useMemo(() => {
    return filterProducts(
      deferredQuery,
      deferredCategory,
      deferredPrice
    );
  }, [deferredQuery, deferredCategory, deferredPrice]);

  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <select 
        value={category}
        onChange={(e) => setCategory(e.target.value)}
      >
        <option value="all">모두</option>
        <option value="electronics">전자제품</option>
        <option value="clothing">의류</option>
      </select>
      <ProductList results={results} />
    </div>
  );
}
```

#### useTransition vs useDeferredValue

```javascript
// useTransition:  상태 업데이트 함수를 감싼다
function WithTransition() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    startTransition(() => {
      // 비긴급 업데이트
      processData(value);
    });
  };
}

// useDeferredValue: 값 자체를 지연시킨다
function WithDeferredValue() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  // query는 즉시 업데이트
  // deferredQuery는 나중에 업데이트
}
```

---

### 3. useId

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **목적** | 접근성 및 SSR 안전 고유 ID 생성 |
| **사용처** | form label, aria 속성, 리스트 식별자 |
| **반환값** | 고유한 문자열 ID |

#### 💻 기본 사용법

```javascript
import { useId } from 'react';

// 1. Form 라벨 연결
function LoginForm() {
  const emailId = useId();
  const passwordId = useId();

  return (
    <form>
      <div>
        <label htmlFor={emailId}>이메일:</label>
        <input id={emailId} type="email" />
      </div>
      
      <div>
        <label htmlFor={passwordId}>비밀번호:</label>
        <input id={passwordId} type="password" />
      </div>
      
      <button type="submit">로그인</button>
    </form>
  );
}

// 2. 여러 입력 필드
function SignUpForm() {
  const firstNameId = useId();
  const lastNameId = useId();
  const emailId = useId();
  const phoneId = useId();

  return (
    <form>
      <input id={firstNameId} type="text" placeholder="이름" />
      <label htmlFor={firstNameId}>이름</label>
      
      <input id={lastNameId} type="text" placeholder="성" />
      <label htmlFor={lastNameId}>성</label>
      
      <input id={emailId} type="email" placeholder="이메일" />
      <label htmlFor={emailId}>이메일</label>
      
      <input id={phoneId} type="tel" placeholder="전화번호" />
      <label htmlFor={phoneId}>전화번호</label>
    </form>
  );
}

// 3. 접근성(aria) 속성
function DropdownMenu() {
  const menuId = useId();
  const buttonId = useId();

  return (
    <div>
      <button 
        id={buttonId}
        aria-haspopup="true"
        aria-controls={menuId}
        aria-expanded={isOpen}
      >
        메뉴 ▼
      </button>
      
      <ul 
        id={menuId}
        role="menu"
        aria-labelledby={buttonId}
        hidden={!isOpen}
      >
        <li role="menuitem">옵션 1</li>
        <li role="menuitem">옵션 2</li>
        <li role="menuitem">옵션 3</li>
      </ul>
    </div>
  );
}

// 4. 동적 리스트 ID
function DynamicList({ items }) {
  const id = useId();

  return (
    <ul>
      {items.map((item, index) => (
        <li key={item.id} id={`${id}-${index}`}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}

// 5. 복잡한 폼 구조
function AddressForm() {
  const streetId = useId();
  const cityId = useId();
  const stateId = useId();
  const zipId = useId();

  return (
    <fieldset>
      <legend>주소</legend>
      
      <div>
        <label htmlFor={streetId}>거리: </label>
        <input id={streetId} type="text" />
      </div>
      
      <div>
        <label htmlFor={cityId}>도시:</label>
        <input id={cityId} type="text" />
      </div>
      
      <div>
        <label htmlFor={stateId}>주: </label>
        <input id={stateId} type="text" />
      </div>
      
      <div>
        <label htmlFor={zipId}>우편번호:</label>
        <input id={zipId} type="text" />
      </div>
    </fieldset>
  );
}
```

#### 주요 특징

```javascript
// ✅ useId의 장점
function GoodId() {
  const id = useId();
  // 1. 각 컴포넌트 인스턴스마다 고유한 ID
  // 2. SSR 환경에서도 일관성 유지
  // 3. hydration mismatch 방지
  
  return <input id={id} />;
}

// ❌ 잘못된 사용법
function BadId() {
  // Math.random() - 서버/클라이언트에서 다를 수 있음
  const id = Math.random().toString(36).substr(2, 9);
  
  // UUID - 매번 새로 생성됨
  const id2 = crypto.randomUUID();
  
  return <input id={id} />;
}
```

---

### 4. useInsertionEffect

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **목적** | CSS-in-JS 라이브러리용 스타일 동적 삽입 |
| **사용처** | styled-components, emotion, 기타 CSS-in-JS |
| **주의** | 일반 개발자는 거의 사용하지 않음 |

#### 💻 기본 사용법

```javascript
import { useInsertionEffect } from 'react';

// 1. 동적 스타일 삽입
function StyledButton({ color }) {
  useInsertionEffect(() => {
    const style = document.createElement('style');
    style.textContent = `
      .btn-${color} {
        background-color: ${color};
        color: white;
        padding: 10px 20px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }
    `;
    
    document.head.appendChild(style);

    return () => {
      document.head.removeChild(style);
    };
  }, [color]);

  return <button className={`btn-${color}`}>버튼</button>;
}

// 2. 테마 적용
function ThemedComponent({ isDark }) {
  useInsertionEffect(() => {
    const style = document.createElement('style');
    
    if (isDark) {
      style.textContent = `
        : root {
          --bg:  #1a1a1a;
          --text: #ffffff;
        }
      `;
    } else {
      style.textContent = `
        :root {
          --bg: #ffffff;
          --text: #000000;
        }
      `;
    }
    
    document.head.appendChild(style);

    return () => {
      document.head.removeChild(style);
    };
  }, [isDark]);

  return <div style={{
    background: 'var(--bg)',
    color: 'var(--text)'
  }}>
    테마 적용됨
  </div>;
}

// 3. 실행 순서
function ExecutionOrder() {
  useInsertionEffect(() => {
    console.log('1️⃣ useInsertionEffect - 가장 먼저');
  }, []);

  useLayoutEffect(() => {
    console.log('3️⃣ useLayoutEffect - DOM 읽기/쓰기');
  }, []);

  useEffect(() => {
    console.log('4️⃣ useEffect - 비동기 작업');
  }, []);

  console.log('2️⃣ 컴포넌트 렌더링');

  return <div>실행 순서 확인</div>;
}
```

#### 실행 타이밍

```
렌더링 함수 실행
  ↓
useInsertionEffect 콜백 (스타일 삽입) ⭐ 가장 먼저
  ↓
DOM 업데이트
  ↓
useLayoutEffect 콜백 (DOM 측정/조작)
  ↓
브라우저 페인팅
  ↓
useEffect 콜백 (데이터 페칭 등)
```

# `useInsertionEffect` vs `useLayoutEffect` 비교

| 특징 | `useInsertionEffect` | `useLayoutEffect` |
|------|-------------------|-----------------|
| **실행 시점** | DOM 커밋 전 | DOM 커밋 후, 화면 그리기 전 |
| **DOM 접근** | ❌ 불가능 | ✅ 가능 |
| **용도** | CSS-in-JS 라이브러리 | 레이아웃 측정, DOM 조작 |
| **성능** | ⚡ 가장 빠름 | 중간 |

---

### 5. useSyncExternalStore

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 18+ |
| **목적** | 외부 상태 저장소(store) React와 동기화 |
| **사용처** | Redux, Zustand, MobX 등 상태 관리 라이브러리 |
| **반환값** | store의 현재 값 |

#### 💻 기본 사용법

```javascript
import { useSyncExternalStore } from 'react';

// 1. 간단한 store 예제
const store = {
  state: { count: 0 },
  listeners: new Set(),

  // subscribe:  변경 감지
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },

  // getSnapshot: 현재 상태 반환
  getSnapshot() {
    return this.state;
  },

  // setState: 상태 변경
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.listeners.forEach(listener => listener());
  }
};

function Counter() {
  const state = useSyncExternalStore(
    store.subscribe. bind(store),
    store.getSnapshot.bind(store)
  );

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => store.setState({ count: state.count + 1 })}>
        증가
      </button>
    </div>
  );
}

// 2. 복잡한 store
const todoStore = {
  todos: [],
  listeners: new Set(),

  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },

  getSnapshot() {
    return {
      todos: this.todos,
      total: this.todos.length,
      completed: this.todos.filter(t => t.completed).length
    };
  },

  addTodo(text) {
    this.todos.push({
      id: Date.now(),
      text,
      completed: false
    });
    this.notifyListeners();
  },

  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
      this.notifyListeners();
    }
  },

  removeTodo(id) {
    this.todos = this.todos.filter(t => t.id !== id);
    this.notifyListeners();
  },

  notifyListeners() {
    this.listeners.forEach(listener => listener());
  }
};

function TodoApp() {
  const { todos, total, completed } = useSyncExternalStore(
    todoStore.subscribe.bind(todoStore),
    todoStore.getSnapshot.bind(todoStore)
  );

  return (
    <div>
      <h2>할 일 목록 ({completed}/{total})</h2>
      
      <form onSubmit={(e) => {
        e.preventDefault();
        const input = e.target.elements.todo;
        todoStore.addTodo(input. value);
        input.value = '';
      }}>
        <input name="todo" placeholder="할 일 입력..." />
        <button>추가</button>
      </form>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo. completed}
              onChange={() => todoStore.toggleTodo(todo. id)}
            />
            <span style={{
              textDecoration: todo. completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => todoStore.removeTodo(todo.id)}>
              삭제
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// 3. SSR 지원
function ServerComponent() {
  const snapshot = useSyncExternalStore(
    store.subscribe.bind(store),
    store.getSnapshot.bind(store),
    () => ({ count: 0 }) // SSR에서 사용할 초기값
  );

  return <div>Count: {snapshot.count}</div>;
}

// 4. 선택적 구독
function SelectiveSubscription() {
  const count = useSyncExternalStore(
    store.subscribe.bind(store),
    () => store.getSnapshot().count // count만 구독
  );

  return <p>{count}</p>;
}

// 5. Redux 통합 (내부적으로 useSyncExternalStore 사용)
// import { useSelector } from 'react-redux';
// Redux는 이미 useSyncExternalStore를 사용하고 있습니다
```

---

## 💡 React 19 New Hooks

### 1. useActionState

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 19+ |
| **목적** | Server Action과 함께 사용하는 form 상태 관리 |
| **사용처** | Form 제출, 로딩 상태, 에러 처리 자동화 |
| **반환값** | [state, formAction, isPending] |

#### 💻 기본 사용법

```javascript
import { useActionState } from 'react';

// 1. 기본 form 제출
async function submitForm(previousState, formData) {
  const name = formData.get('name');
  const email = formData.get('email');

  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify({ name, email })
    });

    if (!response.ok) {
      return {
        message: '제출 실패했습니다',
        success: false,
        errors: {}
      };
    }

    const data = await response. json();
    return {
      message: '제출 성공! ',
      success: true,
      data
    };
  } catch (error) {
    return {
      message: '서버 오류가 발생했습니다',
      success: false,
      errors: { submit: error.message }
    };
  }
}

function ContactForm() {
  const [state, formAction, isPending] = useActionState(
    submitForm,
    {
      message: '',
      success: false,
      errors: {}
    }
  );

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="name">이름:</label>
        <input 
          id="name"
          name="name" 
          type="text"
          disabled={isPending}
          required
        />
      </div>

      <div>
        <label htmlFor="email">이메일:</label>
        <input 
          id="email"
          name="email" 
          type="email"
          disabled={isPending}
          required
        />
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? '제출 중...' : '제출'}
      </button>

      {state.message && (
        <p style={{ 
          color: state.success ?  'green' : 'red' 
        }}>
          {state.message}
        </p>
      )}
    </form>
  );
}

// 2. 유효성 검사 포함
async function validateAndSubmit(previousState, formData) {
  const email = formData.get('email');
  const password = formData. get('password');
  const errors = {};

  // 클라이언트 유효성 검사
  if (!email. includes('@')) {
    errors.email = '유효한 이메일을 입력하세요';
  }

  if (password.length < 8) {
    errors.password = '비밀번호는 8자 이상이어야 합니다';
  }

  if (Object.keys(errors).length > 0) {
    return {
      success: false,
      message: '유효성 검사 실패',
      errors
    };
  }

  // 서버로 제출
  try {
    const response = await fetch('/api/register', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      const errorData = await response.json();
      return {
        success:  false,
        message: errorData.message,
        errors: errorData.errors || {}
      };
    }

    return {
      success: true,
      message: '가입 완료! ',
      errors: {}
    };
  } catch (error) {
    return {
      success: false,
      message: error.message,
      errors: {}
    };
  }
}

function RegisterForm() {
  const [state, formAction, isPending] = useActionState(
    validateAndSubmit,
    { success: false, message: '', errors:  {} }
  );

  return (
    <form action={formAction}>
      <div>
        <label htmlFor="email">이메일:</label>
        <input 
          id="email"
          name="email" 
          type="email"
          disabled={isPending}
        />
        {state.errors?. email && (
          <span style={{ color: 'red' }}>{state.errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">비밀번호:</label>
        <input 
          id="password"
          name="password" 
          type="password"
          disabled={isPending}
        />
        {state.errors?. password && (
          <span style={{ color: 'red' }}>{state.errors.password}</span>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? '가입 중...' : '가입'}
      </button>

      {state.message && (
        <p style={{ color: state.success ? 'green' : 'red' }}>
          {state.message}
        </p>
      )}
    </form>
  );
}

// 3. Server Action과의 통합
// server-actions.js
'use server'

export async function updateProfile(previousState, formData) {
  const userId = formData.get('userId');
  const name = formData.get('name');

  try {
    const user = await db.users.update(userId, { name });
    
    // 캐시 재검증 (ISR)
    revalidatePath(`/profile/${userId}`);

    return {
      success: true,
      message: '프로필이 업데이트되었습니다',
      user
    };
  } catch (error) {
    return {
      success: false,
      message: error.message
    };
  }
}

// components/ProfileForm.jsx
'use client'

import { useActionState } from 'react';
import { updateProfile } from '@/server-actions';

export function ProfileForm({ userId, initialName }) {
  const [state, formAction, isPending] = useActionState(
    updateProfile,
    { success: false, message: '' }
  );

  return (
    <form action={formAction}>
      <input type="hidden" name="userId" value={userId} />
      
      <input 
        name="name"
        defaultValue={initialName}
        disabled={isPending}
      />
      
      <button type="submit" disabled={isPending}>
        {isPending ?  '저장 중...' : '저장'}
      </button>

      {state.message && (
        <p style={{ color: state.success ? 'green' :  'red' }}>
          {state.message}
        </p>
      )}
    </form>
  );
}
```

---

### 2. useFormStatus

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 19+ |
| **목적** | Form 제출 상태 추적 |
| **사용처** | Form 내부 컴포넌트에서 제출 상태 감지 |
| **제약** | form 내부에서만 사용 가능 |

#### 💻 기본 사용법

```javascript
import { useFormStatus } from 'react-dom';

// 1. 기본 submit 버튼
function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? '제출 중...' : '제출'}
    </button>
  );
}

function MyForm({ action }) {
  return (
    <form action={action}>
      <input type="text" name="name" />
      <SubmitButton />
    </form>
  );
}

// 2. 로딩 인디케이터
function LoadingIndicator() {
  const { pending } = useFormStatus();

  return (
    <>
      {pending && (
        <div className="spinner">
          <p>처리 중...</p>
        </div>
      )}
    </>
  );
}

function FormWithIndicator({ action }) {
  return (
    <form action={action}>
      <input type="email" name="email" />
      <LoadingIndicator />
      <button type="submit">제출</button>
    </form>
  );
}

// 3. 입력 필드 비활성화
function FormFields() {
  const { pending } = useFormStatus();

  return (
    <>
      <input
        type="text"
        name="firstName"
        disabled={pending}
        placeholder="이름"
      />
      <input
        type="text"
        name="lastName"
        disabled={pending}
        placeholder="성"
      />
      <input
        type="email"
        name="email"
        disabled={pending}
        placeholder="이메일"
      />
      <button type="submit" disabled={pending}>
        {pending ? '저장 중...' : '저장'}
      </button>
    </>
  );
}

function CompleteForm({ action }) {
  return (
    <form action={action}>
      <FormFields />
    </form>
  );
}

// 4. 여러 버튼 구분
function MultiActionButtons() {
  const { pending, data } = useFormStatus();

  return (
    <>
      <button 
        type="submit" 
        name="action" 
        value="save"
        disabled={pending}
      >
        {pending ? '저장 중...' : '저장'}
      </button>
      
      <button 
        type="submit" 
        name="action" 
        value="draft"
        disabled={pending}
      >
        {pending ? '저장 중...' : '임시 저장'}
      </button>

      <button type="reset" disabled={pending}>
        초기화
      </button>
    </>
  );
}

function MultiActionForm({ action }) {
  return (
    <form action={action}>
      <textarea name="content" />
      <MultiActionButtons />
    </form>
  );
}

// 5. 상황별 메시지
function ContextualButton() {
  const { pending, data } = useFormStatus();
  const actionType = data?.get('action');

  let message = '제출';
  if (pending) {
    if (actionType === 'save') message = '저장 중...';
    else if (actionType === 'delete') message = '삭제 중...';
    else message = '처리 중...';
  }

  return <button type="submit" disabled={pending}>{message}</button>;
}
```

---

### 3. useOptimistic

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 19+ |
| **목적** | 서버 응답 전에 UI를 미리 업데이트 (낙관적 업데이트) |
| **사용처** | 메시지 전송, 좋아요, 삭제 등 빠른 피드백 필요 시 |
| **반환값** | [optimisticState, addOptimistic] |

#### 💻 기본 사용법

```javascript
import { useOptimistic } from 'react';

// 1. 메시지 전송 (낙관적 업데이트)
function MessageThread({ initialMessages, sendMessage }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    initialMessages,
    (state, newMessage) => [
      ...state,
      {
        ... newMessage,
        id: Math.random(),
        pending: true
      }
    ]
  );

  const handleSend = async (formData) => {
    const message = formData.get('message');

    // 즉시 UI 업데이트
    addOptimisticMessage({
      text: message,
      sender: 'me',
      timestamp: new Date()
    });

    try {
      // 서버에 저장
      await sendMessage(message);
    } catch (error) {
      console.error('메시지 전송 실패:', error);
      // 실패하면 자동으로 UI가 원래 상태로 돌아감
    }
  };

  return (
    <div className="chat">
      <div className="messages">
        {optimisticMessages.map((msg, idx) => (
          <div
            key={idx}
            className="message"
            style={{ opacity: msg.pending ? 0.6 : 1 }}
          >
            <strong>{msg.sender}:</strong> {msg.text}
          </div>
        ))}
      </div>

      <form action={handleSend}>
        <input
          type="text"
          name="message"
          placeholder="메시지 입력..."
        />
        <button type="submit">전송</button>
      </form>
    </div>
  );
}

// 2. 좋아요 버튼
function LikeButton({ postId, initialLikes, onLike }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    initialLikes,
    (state) => state + 1
  );

  const handleLike = async () => {
    addOptimisticLike();

    try {
      await onLike(postId);
    } catch (error) {
      console.error('좋아요 실패:', error);
    }
  };

  return (
    <button onClick={handleLike}>
      ♥️ {optimisticLikes}
    </button>
  );
}

// 3. 할 일 리스트 (완료 표시)
function TodoList({ initialTodos, onUpdate, onDelete }) {
  const [optimisticTodos, updateOptimisticTodo] = useOptimistic(
    initialTodos,
    (state, action) => {
      switch (action.type) {
        case 'complete':
          return state.map(todo =>
            todo.id === action.id
              ? { ...todo, completed: true }
              : todo
          );
        case 'delete':
          return state.filter(todo => todo.id !== action. id);
        default:
          return state;
      }
    }
  );

  const handleComplete = async (id) => {
    updateOptimisticTodo({ type: 'complete', id });
    try {
      await onUpdate(id, { completed: true });
    } catch (error) {
      console.error('업데이트 실패:', error);
    }
  };

  const handleDelete = async (id) => {
    updateOptimisticTodo({ type: 'delete', id });
    try {
      await onDelete(id);
    } catch (error) {
      console.error('삭제 실패:', error);
    }
  };

  return (
    <ul>
      {optimisticTodos.map(todo => (
        <li
          key={todo.id}
          style={{
            opacity: todo.pending ? 0.6 : 1,
            textDecoration:  todo.completed ? 'line-through' : 'none'
          }}
        >
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => handleComplete(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => handleDelete(todo.id)}>
            삭제
          </button>
        </li>
      ))}
    </ul>
  );
}

// 4. 복잡한 폼 제출
async function submitTodo(previousState, formData) {
  const text = formData.get('todo');
  const response = await fetch('/api/todos', {
    method: 'POST',
    body: JSON. stringify({ text })
  });
  return response.json();
}

function TodoForm({ initialTodos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    initialTodos,
    (state, newTodo) => [...state, newTodo]
  );

  const handleSubmit = async (formData) => {
    const text = formData.get('todo');

    const newTodo = {
      id:  Math.random(),
      text,
      completed: false
    };

    addOptimisticTodo(newTodo);

    try {
      const result = await submitTodo(null, formData);
      // 성공 시 처리
    } catch (error) {
      console.error('저장 실패:', error);
    }
  };

  return (
    <div>
      <form action={handleSubmit}>
        <input
          type="text"
          name="todo"
          placeholder="할 일 입력..."
        />
        <button type="submit">추가</button>
      </form>

      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo. id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### 4. use()

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 19+ |
| **목적** | Promise와 Context를 일관되게 처리 |
| **사용처** | Server Component의 Promise 처리, 동적 Context |
| **반환값** | Promise 결과 또는 Context 값 |

#### 💻 기본 사용법

```javascript
import { use } from 'react';

// 1. Promise 처리
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

function UserCard({ userPromise }) {
  const user = use(userPromise);

  return (
    <div className="user-card">
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}

// Suspense와 함께 사용
function App() {
  const userPromise = fetchUser(1);

  return (
    <Suspense fallback={<div>사용자 로딩 중...</div>}>
      <UserCard userPromise={userPromise} />
    </Suspense>
  );
}

// 2. 여러 Promise 처리
async function fetchData(id) {
  const [user, posts, comments] = await Promise.all([
    fetch(`/api/users/${id}`).then(r => r.json()),
    fetch(`/api/posts?userId=${id}`).then(r => r.json()),
    fetch(`/api/comments?userId=${id}`).then(r => r.json())
  ]);

  return { user, posts, comments };
}

function Dashboard({ dataPromise }) {
  const { user, posts, comments } = use(dataPromise);

  return (
    <div>
      <h1>{user.name}의 대시보드</h1>
      <section>
        <h2>게시물 ({posts.length})</h2>
        {posts.map(post => (
          <article key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
          </article>
        ))}
      </section>
      <section>
        <h2>댓글 ({comments.length})</h2>
        {comments.map(comment => (
          <div key={comment.id}>
            <p>{comment. text}</p>
          </div>
        ))}
      </section>
    </div>
  );
}

// 3. 동적 Context 값
const UserContext = React.createContext();

function UserProvider({ userId, children }) {
  const userPromise = fetchUser(userId);

  return (
    <UserContext.Provider value={userPromise}>
      {children}
    </UserContext.Provider>
  );
}

function UserInfo() {
  const userPromise = use(UserContext);
  const user = use(userPromise);

  return <p>{user.name}</p>;
}

// 4. 에러 처리
async function fetchUserSafe(id) {
  const response = await fetch(`/api/users/${id}`);
  
  if (!response.ok) {
    throw new Error(`사용자를 찾을 수 없습니다: ${id}`);
  }
  
  return response.json();
}

function SafeUserCard({ userPromise }) {
  try {
    const user = use(userPromise);
    return <div>{user.name}</div>;
  } catch (error) {
    return <div className="error">에러:  {error.message}</div>;
  }
}

// 5. 조건부 Promise
function ConditionalData({ shouldFetch, id }) {
  const dataPromise = shouldFetch
    ? fetchData(id)
    : Promise.resolve(null);

  const data = use(dataPromise);

  return data ? (
    <div>{data.title}</div>
  ) : (
    <div>데이터를 선택하세요</div>
  );
}

// 6. Server Component에서 사용
// app/page.js
async function Page() {
  const users = await fetchUsers();

  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <UserList initialUsers={users} />
    </Suspense>
  );
}

// app/components/UserList.js
'use client'

function UserList({ initialUsers }) {
  const users = use(Promise.resolve(initialUsers));

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

### 5. Enhanced useTransition (React 19)

#### 📌 개요
| 항목 | 설명 |
|------|------|
| **버전** | React 19+ |
| **기능 강화** | Async 함수 직접 지원 추가 |
| **사용처** | Server Action 통합, 복잡한 비동기 로직 |
| **반환값** | [isPending, startTransition] (변화 없음) |

#### 💻 기본 사용법

```javascript
import { useTransition } from 'react';

// 1. Async 함수 지원 (React 19의 새로운 기능)
function UpdateProfile() {
  const [isPending, startTransition] = useTransition();

  const handleUpdate = () => {
    startTransition(async () => {
      // 비동기 작업을 직접 처리
      const response = await fetch('/api/profile', {
        method: 'PUT',
        body: JSON.stringify({ name: 'New Name' })
      });

      if (!response.ok) {
        throw new Error('업데이트 실패');
      }

      const data = await response.json();
      console.log('업데이트 완료:', data);
    });
  };

  return (
    <button onClick={handleUpdate} disabled={isPending}>
      {isPending ? '저장 중...' : '저장'}
    </button>
  );
}

// 2. Server Action과 통합
'use server'

export async function updateUser(formData) {
  const name = formData.get('name');

  // 데이터베이스 업데이트
  const user = await db.users.update({ name });

  // ISR 재검증
  revalidatePath('/profile');

  return { success: true, user };
}

// 클라이언트 컴포넌트
'use client'

function UserProfile() {
  const [isPending, startTransition] = useTransition();

  const handleSubmit = (formData) => {
    startTransition(async () => {
      const result = await updateUser(formData);
      if (result.success) {
        console.log('프로필 업데이트:', result.user);
      }
    });
  };

  return (
    <form action={handleSubmit}>
      <input name="name" type="text" />
      <button type="submit" disabled={isPending}>
        {isPending ? '저장 중...' : '저장'}
      </button>
    </form>
  );
}

// 3. 여러 비동기 작업
function ComplexUpdate() {
  const [isPending, startTransition] = useTransition();

  const handleLoad = () => {
    startTransition(async () => {
      try {
        // 여러 API 호출을 순차적으로 처리
        const userData = await fetch('/api/user').then(r => r.json());
        const postsData = await fetch('/api/posts').then(r => r.json());
        const settingsData = await fetch('/api/settings').then(r => r.json());

        console.log('모든 데이터 로드 완료:', {
          userData,
          postsData,
          settingsData
        });
      } catch (error) {
        console.error('데이터 로드 실패:', error);
      }
    });
  };

  return (
    <button onClick={handleLoad} disabled={isPending}>
      {isPending ? '로딩 중...' : '데이터 로드'}
    </button>
  );
}

// 4. 에러 처리
function SafeUpdate() {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState(null);

  const handleSubmit = () => {
    setError(null);

    startTransition(async () => {
      try {
        const response = await fetch('/api/submit', {
          method: 'POST',
          body: JSON.stringify({ data: 'test' })
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        console.log('성공:', result);
      } catch (err) {
        setError(err.message);
      }
    });
  };

  return (
    <div>
      {error && <p style={{ color: 'red' }}>오류: {error}</p>}
      <button onClick={handleSubmit} disabled={isPending}>
        {isPending ? '처리 중...' : '제출'}
      </button>
    </div>
  );
}

// 5. React 18 vs React 19 비교
// React 18: startTransition은 동기 함수만 지원
function React18Example() {
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(() => {
      // 동기 setState만 가능
      setState(newValue);
    });
  };
}

// React 19: startTransition은 async 함수 지원
function React19Example() {
  const [isPending, startTransition] = useTransition();

  const handleClick = () => {
    startTransition(async () => {
      // async/await 사용 가능
      await someAsyncTask();
      setState(newValue);
    });
  };
}
```

---

## 🎯 Hooks 비교 및 선택 가이드

### React 18 vs React 19 비교표

| Hook | React 18 | React 19 | 용도 |
|------|----------|----------|------|
| useTransition | ✅ | ✅ + async 지원 | 비긴급 업데이트 분리 |
| useDeferredValue | ✅ | ✅ | 값 업데이트 지연 |
| useId | ✅ | ✅ | 고유 ID 생성 |
| useInsertionEffect | ✅ | ✅ | CSS 동적 삽입 |
| useSyncExternalStore | ✅ | ✅ | 외부 store 동기화 |
| useActionState | ❌ | ✅ | Form 상태 관리 |
| useFormStatus | ❌ | ✅ | Form 제출 감지 |
| useOptimistic | ❌ | ✅ | 낙관적 업데이트 |
| use() | ❌ | ✅ | Promise/Context 처리 |

---

### 상황별 Hook 선택 가이드

#### 상태 업데이트 분리

```javascript
// 검색 기능:  useTransition 사용
function Search() {
  const [isPending, startTransition] = useTransition();
  // 입력은 즉시, 검색은 나중에
}

// 느린 렌더링: useDeferredValue 사용
function SlowList() {
  const deferredValue = useDeferredValue(value);
  // value는 즉시, deferredValue는 나중에
}
```

#### Form 처리

```javascript
// React 18: useState + useTransition
function Form18() {
  const [state, setState] = useState();
  const [isPending, startTransition] = useTransition();
}

// React 19: useActionState
function Form19() {
  const [state, formAction, isPending] = useActionState(action, {});
  // Form 상태를 자동으로 관리
}
```

#### 외부 상태 관리

```javascript
// Redux/Zustand 연동:  useSyncExternalStore
function Component() {
  const state = useSyncExternalStore(subscribe, getSnapshot);
}

// 자체 store: 직접 구현 또는 Context API
function Component() {
  const state = useContext(StoreContext);
}
```

---

### 성능 최적화 전략

```javascript
// 1. 긴급 업데이트 vs 비긴급 업데이트 분리
function OptimizedSearch() {
  const [input, setInput] = useState('');
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    // 긴급:  입력 필드 즉시 반응
    setInput(e.target.value);

    // 비긴급:  검색 결과 나중에 처리
    startTransition(() => {
      updateResults(e.target.value);
    });
  };
}

// 2. 값 지연으로 렌더링 최적화
function OptimizedList() {
  const [value, setValue] = useState('');
  const deferredValue = useDeferredValue(value);

  // value 변경 시:  입력 필드만 업데이트
  // deferredValue 변경 시: 전체 리스트 업데이트 (나중에)
}

// 3. 낙관적 업데이트로 UX 개선
function OptimisticUI() {
  const [optimisticItems, addOptimistic] = useOptimistic(items);

  const handleAdd = async (item) => {
    // 즉시 UI 업데이트
    addOptimistic(item);

    try {
      // 그 다음 서버 요청
      await saveItem(item);
    } catch {
      // 실패하면 자동으로 원래대로
    }
  };
}
```

---

## 📝 마이그레이션 팁

### React 17 → React 18

```javascript
// 의존성 배열 주의
// React 18에서 Strict Mode가 강화됨
useEffect(() => {
  // 이 함수가 두 번 호출될 수 있음 (개발 환경에서)
}, []);
```

### React 18 → React 19

```javascript
// 1. useActionState 활용
// React 18: useState + useTransition
const [state, setState] = useState();
const [isPending, startTransition] = useTransition();

// React 19: useActionState
const [state, formAction, isPending] = useActionState(action, {});

// 2. useFormStatus로 form 상태 추적
const { pending } = useFormStatus();

// 3. useOptimistic으로 낙관적 업데이트
const [optimistic, addOptimistic] = useOptimistic(data);

// 4. use()로 Promise 처리 단순화
const data = use(dataPromise);
```

---

## ✅ 체크리스트

React 18/19 Hooks를 효과적으로 사용하기 위한 체크리스트:

- [ ] 검색/필터링 성능 이슈가 있는가?  → `useTransition` 사용
- [ ] 느린 컴포넌트 렌더링이 문제인가? → `useDeferredValue` 사용
- [ ] Form에 ID가 필요한가? → `useId` 사용
- [ ] 외부 store를 사용하는가? → `useSyncExternalStore` 사용
- [ ] Server Action을 사용하는가?  → React 19의 `useActionState` 사용
- [ ] Optimistic update가 필요한가? → `useOptimistic` 사용
- [ ] Promise를 처리해야 하는가? → `use()` 사용
- [ ] form 상태를 추적해야 하는가? → `useFormStatus` 사용

---

**최종 업데이트**: 2025-12-09  
**React 버전**: 18 ~ 19  
**대상**:  중급~고급 개발자
