# 🧩 Custom Hook 구성 가이드

React + TypeScript 환경에서 Custom Hook을 구성할 때 가독성, 유지보수성, 일관성을 고려한 추천 구성 순서를 정리합니다.

---

## ✅ Custom Hook 추천 구성 순서

1. **상수 및 외부 데이터 준비**
   - 예: `mockData`, 초기값, 상수 등

2. **State 관련 훅** (`useState`, `useRef`, 등)
   - 가장 먼저 상태 정의

3. **Effect / Memo 관련 훅** (`useEffect`, `useMemo`, `useCallback` 등)
   - 상태에 의존한 부가 동작 정의

4. **핸들러 함수** (`handleX`, `onY`, `doZ`)
   - 유저 이벤트나 내부 상태 변경 처리 로직

5. **유틸리티 함수 / 액션 맵핑 함수**
   - `createActionMap`, `mapDataToSomething` 등

6. **최종 반환 값**
   - `return {}` 구문

---

## 🧠 예시: `useColumns`

```ts
export const useColumns = () => {
  // 1. State
  const [columns, setColumns] = useState<Column[]>([]);
  const [isOpen, setIsOpen] = useState(false);

  // 2. Effect
  useEffect(() => {
    console.log('columns changed');
  }, [columns]);

  // 3. 핸들러
  const handleAdd = () => { ... };
  const handleDelete = (id: number) => { ... };

  // 4. 유틸 함수
  const getSortedColumns = () => [...columns].sort(...);

  // 5. Return
  return {
    columns,
    handleAdd,
    handleDelete,
    getSortedColumns,
  };
};
```

---

## ✍️ 네이밍 규칙

- `handleX`, `onX`, `setX`: 액션을 의미하는 명확한 prefix 사용
- `getX`, `isX`, `shouldX`: 조건 판단 함수는 의미 있는 네이밍 유지
- `createX`, `mapXToY`: 매핑 / 변환 로직 함수에 사용

---

## 📌 리턴 순서 정리 팁

리턴 객체의 순서도 다음처럼 정리하면 좋습니다:

1. 상태 변수 (`columns`, `selectedId`, ...)
2. 주요 핸들러 (`handleAdd`, `handleDelete`, ...)
3. 유틸 함수 (`getSortedColumns`, `createActionMap`, ...)

---

## ✅ 결론

> Custom Hook은 **일관된 구조 + 명확한 네이밍 + 정리된 리턴** 만으로도 생산성과 유지보수성이 크게 향상됩니다.

---

## 📦 참고로 자동 정렬을 위한 Lint Rule

Hook 내부 선언 순서에 대한 Lint 규칙은 `eslint-plugin-react-hooks`로는 제어할 수 없습니다.  
`@typescript-eslint/member-ordering`도 Hook에는 직접 적용되지 않습니다.  

그러므로 아래처럼 **정렬 체크를 위한 커스텀 룰 또는 ESLint 플러그인**을 활용하는 방식이 필요합니다:

### ▶ Lint 추천

1. **@tanstack/eslint-plugin-query**
   - React Query 기준으로 훅의 위치 정리 가능

2. **eslint-plugin-custom-hooks-order** _(직접 작성 필요)_
   - `useState → useEffect → 핸들러 → 유틸리티 → 리턴` 패턴 검출 가능

---
