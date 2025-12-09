## 🎛️ Hook 선택 가이드

### 상태 관리

| 상황 | 추천 Hook |
|------|----------|
| 단순 상태 | `useState` |
| 복잡한 상태 로직 | `useReducer` |
| 전역 상태 | `useContext` + `useState` |
| 외부 라이브러리 | `useSyncExternalStore` |

### 부수 효과 (Side Effects)

| 상황 | 추천 Hook |
|------|----------|
| API 호출, 구독 | `useEffect` |
| DOM 측정 | `useLayoutEffect` |
| CSS-in-JS 스타일 | `useInsertionEffect` |

### 성능 최적화

| 상황 | 추천 Hook |
|------|----------|
| 함수 메모이제이션 | `useCallback` |
| 계산 결과 캐시 | `useMemo` |
| 느린 업데이트 분리 | `useTransition` |
| 값 업데이트 지연 | `useDeferredValue` |

### React 19 비동기

| 상황 | 추천 Hook |
|------|----------|
| Form 제출 상태 | `useActionState` |
| 비동기 업데이트 제어 | `useTransition` (async) |
| Form 상태 감지 | `useFormStatus` |
| 낙관적 업데이트 | `useOptimistic` |
| Promise/Context 처리 | `use()` |

---

## 📊 Hook 비교표

| Hook | 용도 | 언제 사용 | 의존성 배열 |
|------|------|---------|-----------|
| useState | 상태 관리 | 항상 | X |
| useEffect | 부수 효과 | 렌더링 후 | O |
| useContext | 전역 상태 | Props Drilling 방지 | X |
| useReducer | 복잡한 상태 | 여러 관련 상태 | X |
| useCallback | 함수 메모이제이션 | 자식에 콜백 전달 | O |
| useMemo | 값 메모이제이션 | 비용이 큰 계산 | O |
| useRef | DOM 접근 | 비제어 컴포넌트 | X |
| useLayoutEffect | 동기 부수 효과 | DOM 측정 | O |
| useTransition | 비긴급 업데이트 | 검색/필터링 | X |
| useDeferredValue | 값 지연 | 느린 렌더링 | X |
| useId | 고유 ID 생성 | form/aria 속성 | X |
| useInsertionEffect | 스타일 삽입 | CSS-in-JS | O |
| useSyncExternalStore | 외부 store | Redux/Zustand | X |
| useActionState | Form 상태 | Server Action | X |
| useFormStatus | Form 제출 상태 | Form 내부 컴포넌트 | X |
| useOptimistic | 낙관적 업데이트 | 즉시 UI 업데이트 | X |
| use() | Promise/Context | 비동기 처리 | X |
