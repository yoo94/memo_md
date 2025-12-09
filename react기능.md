# React 기능정리

React의 모든 공식 기능들을 전부 정리했습니다.  (Hooks 제외)

---

## 1. Fragment <> 📦

여러 요소를 불필요한 래퍼 없이 그룹화합니다.

### 사용법
```jsx
// 단축 문법
<>
  <Component1 />
  <Component2 />
</>

// 또는 React.Fragment (key가 필요할 때)
<React.Fragment key={item.id}>
  <dt>{item.term}</dt>
  <dd>{item.description}</dd>
</React.Fragment>
```

### 규칙
- DOM에 래퍼 노드를 생성하지 않음
- key props가 필요하면 `<React.Fragment>` 사용
- 단축 문법(`<>`)은 key를 사용할 수 없음

---

## 2. StrictMode ⚠️

개발 환경에서 잠재적 문제를 감지하는 도구입니다.

### 사용법
```jsx
import { StrictMode } from 'react'

root. render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

### 감지 항목
- 안전하지 않은 생명주기 메서드 사용
- 레거시 문자열 ref API 사용
- findDOMNode 사용
- 예상치 못한 부작용 (의도적 이중 렌더링)
- 레거시 Context API 사용

### 규칙
- **프로덕션에서는 영향 없음** (개발 환경 전용)
- 의도적으로 컴포넌트를 두 번 렌더링하여 부작용 감지

---

## 3. Profiler 📊

렌더링 성능을 측정하고 분석하는 도구입니다. 

### 사용법
```jsx
import { Profiler } from 'react'

function onRenderCallback(
  id,             // 프로파일러 ID
  phase,          // "mount" 또는 "update"
  actualDuration, // 렌더링 시간 (ms)
  baseDuration,   // 메모이제이션 없을 시 예상 시간
  startTime,      // 렌더링 시작 시간
  commitTime,     // 렌더링 완료 시간
  interactions    // Set of interactions
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`)
}

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>
```

### 규칙
- 성능 측정용 (프로덕션 배포 시 오버헤드 있음)
- 여러 Profiler를 중첩할 수 있음
- React DevTools Profiler와 함께 사용 권장

---

## 4. createContext 📚

Props drilling 없이 깊은 레벨의 컴포넌트에 데이터를 전달합니다.

### 사용법
```jsx
// Context 생성
const ThemeContext = React.createContext('light')

// Provider 설정
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <MyComponent />
    </ThemeContext.Provider>
  )
}

// Consumer에서 사용 (클래스)
<ThemeContext.Consumer>
  {value => <div>Theme: {value}</div>}
</ThemeContext.Consumer>
```

### 규칙
- 기본값은 `createContext()`의 인자로 지정
- Provider의 value가 변경되면 모든 Consumer가 리렌더링
- 성능 최적화:  값을 분리하거나 useMemo 사용

---

## 5. lazy 🚀

컴포넌트를 동적으로 로딩합니다 (Code Splitting).

### 사용법
```jsx
import { lazy, Suspense } from 'react'

const HeavyComponent = lazy(() => import('./HeavyComponent'))

function App() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <HeavyComponent />
    </Suspense>
  )
}
```

### 규칙
- **default export만 지원**
- **Suspense로 감싸야 함** (필수)
- 번들러 지원 필요 (Webpack, Vite 등)
- 네트워크 요청 후 컴포넌트 렌더링

---

## 6. Suspense ⏳

비동기 작업(데이터 로드, 코드 분할)이 완료될 때까지 로딩 UI를 표시합니다.

### 사용법
```jsx
import { Suspense } from 'react'

<Suspense fallback={<Loading />}>
  <AsyncComponent />
</Suspense>
```

### 규칙
- **fallback props**:  로딩 중 표시할 UI (필수)
- 자식이 Promise를 throw해야 함
- Error Boundary로 에러 처리
- 중첩 가능 (세밀한 로딩 제어)

---

## 7. memo 💾

props가 변경되지 않으면 리렌더링을 스킵합니다.

### 사용법
```jsx
const MyComponent = memo(function MyComponent(props) {
  // props가 같으면 리렌더링 안 함
  return <div>{props. value}</div>
})

// 커스텀 비교 함수
const MyComponent = memo(MyComponent, (prevProps, nextProps) => {
  return prevProps.id === nextProps.id // true면 리렌더링 스킵
})
```

### 규칙
- **얕은 비교(shallow comparison)** 사용
- 객체/배열은 참조 비교하므로 주의
- 과도한 사용은 오히려 성능 저하 가능

---

## 8. forwardRef 🔗

자식 컴포넌트의 DOM 노드에 직접 접근합니다.

### 사용법
```jsx
import { forwardRef } from 'react'

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input ref={ref} />
})

// 사용
const inputRef = useRef()
<MyInput ref={inputRef} />
```

### 규칙
- 함수형 컴포넌트에서 ref를 받으려면 필수
- ref는 props로 전달되지 않음 (별도 처리)
- DOM 조작이 필요한 경우만 사용

---

## 9. createPortal 🌀

DOM의 다른 위치에 컴포넌트를 렌더링합니다. 

### 사용법
```jsx
import { createPortal } from 'react-dom'

function Modal({ children }) {
  return createPortal(
    <div className="modal">
      {children}
    </div>,
    document.getElementById('modal-root')
  )
}

// HTML
<div id="root"></div>
<div id="modal-root"></div>
```

### 규칙
- **이벤트 버블링**:  DOM 트리를 따름 (실제 DOM 위치 무관)
- CSS z-index로 레이어 관리
- Modal, Tooltip, Dropdown 등에 유용

---

## 10. Higher Order Component (HOC) 🎁

컴포넌트 로직을 재사용하기 위한 고급 패턴입니다. 

### 사용법
```jsx
// HOC 정의
function withSubscription(WrappedComponent) {
  return function WithSubscription(props) {
    const [isSubscribed, setIsSubscribed] = useState(false)
    
    return <WrappedComponent isSubscribed={isSubscribed} {... props} />
  }
}

// 사용
const MyComponent = withSubscription(OriginalComponent)
```

### 규칙
- 컴포넌트를 받아 새 컴포넌트를 반환하는 함수
- displayName 설정으로 디버깅 용이
- Ref 전달 불가 (forwardRef 필요)
- 정적 메서드 복사 필요

---

## 11. Render Props 패턴 🎬

렌더링 로직을 props로 전달합니다.

### 사용법
```jsx
function DataFetcher({ render }) {
  const [data, setData] = useState(null)
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(data => setData(data))
  }, [])
  
  return render(data)
}

// 사용
<DataFetcher render={data => <div>{data}</div>} />
```

### 규칙
- render props는 함수여야 함
- 자식 prop으로도 가능
- HOC보다 덜 복잡하고 composable

---

## 12. React Router 🛣️

클라이언트 사이드 라우팅입니다.  (react-router-dom 라이브러리)

### 사용법
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/user/:id" element={<User />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### 규칙
- BrowserRouter:  HTML5 History API 사용 (권장)
- Routes: 첫 번째 매칭되는 Route만 렌더링
- exact path:  정확한 경로만 매칭
- useParams: URL 파라미터 추출
- useNavigate: 프로그래밍 방식 네비게이션

---

---

## 17. Actions (React 19) ⚡

폼 제출, 데이터 변경 등 비동기 작업을 관리합니다.

### 사용법
```jsx
// 서버 액션
async function saveTask(formData) {
  'use server'
  const task = formData.get('task')
  await db.tasks.create({ task })
}

// 클라이언트에서 사용
'use client'

<form action={saveTask}>
  <input name="task" />
  <button type="submit">저장</button>
</form>
```

### 규칙
- 'use server' 함수는 서버에서만 실행
- 폼과 함께 자동 처리
- 대기, 에러, 낙관적 업데이트 지원

---

## 18. Directives (React 19) 📌

클라이언트/서버 경계를 명시합니다.

### 사용법
```jsx
// 클라이언트 컴포넌트
'use client'

// 서버 액션
'use server'
async function handleSubmit(formData) {
  // ...
}
```

### 규칙
- 파일 맨 위에 작성
- 모듈 레벨에서만 유효
- 한 파일에 한 번만

---

## 19. Refs 🔍

DOM 노드나 클래스 인스턴스에 직접 접근합니다.

### 사용법
```jsx
function TextInput() {
  const inputRef = useRef(null)
  
  const focusInput = () => {
    inputRef.current. focus()
  }
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  )
}
```

### 규칙
- 과도한 사용은 지양
- DOM 조작이 필수일 때만 사용
- useRef는 Hook (여기서는 Ref 개념만)

---

## 20. Key Props 🔑

리스트 렌더링 시 요소를 식별합니다.

### 사용법
```jsx
{items.map(item => (
  <div key={item. id}>{item.name}</div>
))}
```

### 규칙
- **고유하고 안정적인 값** 사용
- Index를 key로 사용하지 말 것 (성능 문제)
- 리스트 순서가 바뀔 수 있으면 특히 중요

---

## 21. Props & Attributes 📋

컴포넌트에 데이터를 전달합니다.

### 사용법
```jsx
// Props 전달
<MyComponent name="John" age={30} />

// JSX 속성
<div className="container" id="app">
  Content
</div>
```

### 규칙
- Props는 읽기 전용 (immutable)
- HTML 속성은 camelCase (className, onClick 등)
- defaultProps로 기본값 설정 가능

---

## 22. Children 👶

컴포넌트 내부에 전달된 콘텐츠입니다.

### 사용법
```jsx
function Container({ children }) {
  return <div className="container">{children}</div>
}

<Container>
  <h1>Title</h1>
  <p>Content</p>
</Container>
```

### 규칙
- JSX 요소들이 자동으로 children props에 전달됨
- 여러 자식 가능
- 텍스트, 요소, 컴포넌트 모두 가능

---

## 24. Function Components 🔧

함수 기반 컴포넌트입니다.  (최신, Hook 사용)

### 사용법
```jsx
function MyComponent(props) {
  return <div>{props.message}</div>
}

// 또는 화살표 함수
const MyComponent = (props) => {
  return <div>{props.message}</div>
}
```

### 규칙
- 더 간단하고 읽기 쉬움
- Hook 사용 가능
- 클래스 컴포넌트보다 성능 우수

---

## 25. JSX 문법 📝

JavaScript XML - HTML 같은 문법으로 React UI를 작성합니다.

### 사용법
```jsx
// JSX
const element = <h1 className="greeting">Hello! </h1>

// JavaScript (컴파일 후)
const element = React.createElement('h1', { className: 'greeting' }, 'Hello!')
```

### 규칙
- 단일 root 요소 반환 (또는 Fragment)
- HTML 속성은 camelCase
- JavaScript 표현식은 `{}`로 감싸기
- 주석은 `{/* */}`

---

## 26. Event Handling 🎯

사용자 상호작용 처리합니다.

### 사용법
```jsx
function MyComponent() {
  const handleClick = (e) => {
    console.log(e)
  }
  
  const handleChange = (e) => {
    const value = e.target.value
  }
  
  return (
    <>
      <button onClick={handleClick}>클릭</button>
      <input onChange={handleChange} />
    </>
  )
}
```

### 규칙
- 이벤트명은 camelCase (onClick, onChange 등)
- 핸들러는 함수여야 함
- Event 객체는 합성 이벤트 (SyntheticEvent)

---

## 27. Controlled vs Uncontrolled Components 🎮

폼 입력 제어 방식입니다.

### Controlled (권장)
```jsx
function Form() {
  const [value, setValue] = useState('')
  
  return (
    <input 
      value={value} 
      onChange={e => setValue(e.target. value)} 
    />
  )
}
```

### Uncontrolled
```jsx
function Form() {
  const inputRef = useRef()
  
  const handleSubmit = () => {
    console.log(inputRef.current. value)
  }
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>제출</button>
    </>
  )
}
```

---

## 28. React.createElement ⚙️

JSX 없이 React 요소를 생성합니다.

### 사용법
```jsx
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello World'
)

// JSX와 동일
const element = <h1 className="greeting">Hello World</h1>
```

### 규칙
- 컴포넌트는 PascalCase
- Props는 두 번째 인자
- 자식들은 세 번째 이후 인자

---

## 29. React.cloneElement 🔄

기존 요소를 복제하고 Props를 수정합니다.

### 사용법
```jsx
const clonedElement = React.cloneElement(element, { className: 'updated' })
```

### 규칙
- 기존 요소는 변경되지 않음
- Props를 병합 (기존 + 새로운)

---

## 30. React. isValidElement ✅

유효한 React 요소인지 확인합니다. 

### 사용법
```jsx
React.isValidElement(<MyComponent />) // true
React.isValidElement('string') // false
```

---

## 31. PropTypes 🛡️

Props 타입 검증합니다.  (라이브러리)

### 사용법
```jsx
import PropTypes from 'prop-types'

function MyComponent({ name, age }) {
  return <div>{name}, {age}</div>
}

MyComponent. propTypes = {
  name:  PropTypes.string.isRequired,
  age: PropTypes.number
}

MyComponent.defaultProps = {
  age: 18
}
```

---

## 32. TypeScript 📝

타입 안정성을 제공합니다.

### 사용법
```tsx
interface Props {
  name: string
  age: number
  children?: React.ReactNode
}

function MyComponent({ name, age }: Props) {
  return <div>{name}, {age}</div>
}
```

---

## 33. Conditional Rendering 🔀

조건에 따라 다른 UI를 렌더링합니다. 

### 사용법
```jsx
// if/else
function MyComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome! </h1>
  }
  return <h1>Please log in</h1>
}

// 삼항 연산자
<div>{isLoggedIn ? <Profile /> : <Login />}</div>

// 논리 AND (&&)
<div>{isLoggedIn && <Profile />}</div>

// null 반환 (아무것도 렌더링 안 함)
{condition && <Component />}
```

---

## 34. List Rendering 📝

배열을 컴포넌트로 변환합니다.

### 사용법
```jsx
function TodoList({ items }) {
  return (
    <ul>
      {items. map(item => (
        <li key={item. id}>{item.text}</li>
      ))}
    </ul>
  )
}
```

### 규칙
- key props 필수 (고유값)
- Index 사용 지양

---

## 35. Form Handling (React 19) 📋

폼 제출 및 유효성 검사를 간단하게 처리합니다.

### 사용법
```jsx
'use client'

export default function Form() {
  return (
    <form action={saveTask}>
      <input name="task" required />
      <button type="submit">저장</button>
    </form>
  )
}

// 서버 액션
async function saveTask(formData) {
  'use server'
  const task = formData.get('task')
  // DB 저장
}
```

---

## 36. Optimistic Updates (React 19) ⚡

서버 응답 전에 UI를 즉시 업데이트합니다.

### 개념
```jsx
// 사용자가 클릭
// 1. UI 즉시 업데이트 (낙관적)
// 2. 서버 요청 (비동기)
// 3. 응답 받아 확정 또는 롤백
```

### 규칙
- 사용자 경험 향상
- 실패 시 롤백 필요

---

## 37. Code Splitting 📦

큰 번들을 작은 청크로 분할합니다.

### 사용법
```jsx
// 동적 import
const HeavyComponent = lazy(() => import('./Heavy'))

<Suspense fallback={<Loading />}>
  <HeavyComponent />
</Suspense>
```

### 규칙
- 번들러 지원 필요
- Suspense와 함께 사용
- 라우트별로 분할하는 것이 일반적

---

## 38. Performance Optimization 🚀

불필요한 렌더링을 방지합니다. 

### 방법
```jsx
// memo - Props 비교
const MyComponent = memo(function MyComponent(props) {
  // ...
})

// useMemo - 계산 결과 캐싱 (Hook)
// useCallback - 함수 참조 유지 (Hook)
```

---

## 39. Accessibility (a11y) ♿

웹 접근성을 개선합니다.

### 규칙
- Semantic HTML 사용
- ARIA 속성 추가
- 키보드 네비게이션
- 색상 대비율 준수
- alt 텍스트 제공

```jsx
<button aria-label="close">×</button>
<img src="photo.jpg" alt="Profile photo" />
```

---

## 40. Testing 🧪

React 컴포넌트를 테스트합니다. 

### 도구
- React Testing Library
- Jest
- Vitest
- Cypress

### 예시
```jsx
import { render, screen } from '@testing-library/react'

test('renders greeting', () => {
  render(<Greeting name="John" />)
  expect(screen.getByText('Hello John')).toBeInTheDocument()
})
```

---

## 요약 표

| 기능 | 타입 | 설명 |
|------|------|------|
| Fragment | Built-in | 래퍼 없이 그룹화 |
| StrictMode | Built-in | 개발 환경 문제 감지 |
| Suspense | Built-in | 비동기 대기 |
| Error Boundary | Pattern | 에러 처리 |
| Portal | API | DOM 다른 위치 렌더링 |
| HOC | Pattern | 컴포넌트 로직 재사용 |
| Render Props | Pattern | 렌더링 로직 전달 |
| Context | API | Props drilling 방지 |
| Router | Library | 클라이언트 라우팅 |
| Redux Provider | Library | 상태 관리 |
| Server Components | React 19 | 서버 렌더링 |
| Actions | React 19 | 비동기 작업 |
| lazy | API | 코드 분할 |
| memo | API | 렌더링 최적화 |
| Refs | API | DOM 직접 접근 |

---

이 파일에는 React의 **거의 모든 공식 기능**이 포함되어 있습니다!  🎉
