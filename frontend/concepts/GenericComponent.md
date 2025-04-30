# 📘 제네릭 컨포넌트(Generic Component) 이해하기

## 1. 제네릭(Generic)이란?

**제네릭(Generic)**은 TypeScript에서 타입을 파라미터처럼 사용할 수 있게 해주는 문맥입니다.  
동일한 로직을 다양한 타입에 대해 안전하게 재사용할 수 있도록 도움을 줍니다.

### ✅ 예시: 제네릭 함수

```ts
function identity<T>(value: T): T {
  return value;
}

identity<string>('hello'); // 결과: 'hello'
identity<number>(123);     // 결과: 123
```

---

## 2. 제네릭 컨포넌트란?

React에서의 **제네릭 컨포넌트**는 컨포넌트의 props나 내부 로직이 타입에 따라 유력하게 다른 방식으로 동작하도록 만들는 컨포넌트입니다.

### ✅ 예시: 제네릭 리스트 컨포넌트

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

const List = <T,>({ items, renderItem }: ListProps<T>) => {
  return <ul>{items.map(renderItem)}</ul>;
};
```

보내어 사용:

```tsx
<List
  items={[1, 2, 3]}
  renderItem={item => <li key={item}>{item}</li>}
/>
```

---

## 3. ❌ JSX에서 제네릭 컨포넌트를 직접 사용할 수 없는 이유

### 다음과 같은 코드는 **불가능**합니다:

```tsx
// ❌ 오류 발생
<List<number> items={[1, 2, 3]} renderItem={...} />
```

### 이유:

JSX는 `<List<number>>`를 HTML 태그로 해석하려고 시도하기 때문에 **문맥 오류가 발생**합니다.

> JSX는 `<Component<T>>` 형태의 타입 파라미터 전달을 지원하지 않습니다.

---

## 4. ✅ 해결 방법

### 방법 1: 함수처럼 호출 (JSX에서는 사용 못함)

```tsx
const TypedComponent = GenericComponent<MyType>;
return <TypedComponent ... />;
```

> JSX가 아니라 함수에서는 가능하지만, 일반적으로 잘 사용하지 않는 방식입니다.

---

### 방법 2: 래프 컨포넌트 사용 (✅ 가장 일반적인 방식)

```tsx
// CardContextMenu.tsx
import GenericContextMenu from './GenericContextMenu';
import type { TCardActionType } from '@/types';

const CardContextMenu = (props: {
  items: { label: string; action: TCardActionType }[];
  onAction: (action: TCardActionType) => void;
  children: React.ReactNode;
}) => {
  return (
    <GenericContextMenu<TCardActionType> {...props} />
  );
};
```

> JSX에서는 `<CardContextMenu>`만 사용하고 내부적으로만 제네릭을 고정해줍니다.

---

## 5. 제네릭 컨포넌트를 사용할 때

| 상황 | 예시 |
|--------|-------|
| 다양한 데이터 타입 다루기 | `<List<T>>` |
| 테이블 컨포넌트 | `<DataTable<T>>` |
| 유력한 Context Provider | `<ContextProvider<T>>` |
| 선택형 드롭다운 | `<Select<T>>` |
| 메뉴/폼/필드 및 유력 값 | `<ContextMenu<T>>`, `<DynamicForm<T>>` |

---

## ✅ 결론

- 제네릭 컨포넌트는 **유력하며 타입 안전성이 높은** 컨포넌트를 만들 때 유용합니다.
- JSX에서는 제네릭을 직접 전달할 수 없기 때문에, **래프 컨포넌트가 필요**합니다.
- React + TypeScript 프로젝트에서 **공통 UI 컨포넌트**를 만들 때 매우 자주 사용됩니다.

---
