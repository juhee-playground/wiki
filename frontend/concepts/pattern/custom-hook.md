# Custom Hook 패턴

## 개요

Custom Hook 패턴은 React의 로직을 재사용 가능한 함수로 분리하는 디자인 패턴입니다.
이 패턴을 사용하면 컴포넌트의 로직을 분리하여 재사용성과 유지보수성을 크게 향상시킬 수 있습니다.

- **Custom Hook**: 컴포넌트의 로직을 분리한 재사용 가능한 함수
- **컴포넌트**: Custom Hook을 사용하여 UI를 렌더링

## 패턴 설명

### Custom Hook

- **역할**:
  - 컴포넌트의 로직을 분리하여 재사용 가능한 형태로 만듦
  - 상태 관리, 이벤트 핸들러, 부수 효과 등을 캡슐화
- **특징**:
  - `use` 접두사로 시작하는 함수
  - React의 기본 Hook들을 조합하여 새로운 로직을 만듦
  - 컴포넌트와 독립적으로 동작

### 컴포넌트에서의 사용

- **역할**:
  - Custom Hook을 사용하여 필요한 로직을 가져옴
  - UI 렌더링에 집중
- **특징**:
  - 로직이 분리되어 있어 코드가 깔끔해짐
  - 비즈니스 로직과 UI 로직이 명확히 구분됨

## 폴더 구조 예시

```
src
├─ hooks
│   ├─ dashboard
│   │   ├─ useColumns.ts
│   │   ├─ useDashboardDate.ts
│   │   └─ useGridScroll.ts
│   └─ index.ts
│
├─ components
│   ├─ dashboard
│   │   ├─ container
│   │   │   └─ DashboardContainer.tsx
│   │   └─ ui
│   │       ├─ GridHeader.tsx
│   │       ├─ GridMain.tsx
│   │       └─ ...
│   └─ ...
│
└─ App.tsx
```

## 코드 예시

### Custom Hook (useColumns.ts)

```typescript
export const useColumns = (scheduleCardsData: IScheduleCardData[]) => {
  const [columns, setColumns] = useState<IGridColumns[]>([
    {
      id: 1,
      name: "Tindlo Project",
      isLocked: false,
      isShared: false,
      collapsed: false,
    },
    // ...
  ]);

  const handleColumnAction = (id: number, action: TColumnActionType) => {
    switch (action) {
      case "add":
        setColumns((prev) => [
          ...prev,
          {
            id: prev.length + 1,
            name: "NEW GROUP",
            isLocked: false,
            isShared: false,
            collapsed: false,
          },
        ]);
        break;
      // ... 다른 액션 처리
    }
  };

  return { columns, handleColumnAction };
};
```

### 컴포넌트에서 사용 (DashboardContainer.tsx)

```typescript
function DashboardContainer() {
  const { columns, handleColumnAction } = useColumns(scheduleCardsData);
  const { currentDate, viewMode } = useDashboardDate();
  const { headerScrollRef, gridScrollRef } = useGridScroll();

  return (
    <div className="w-full h-full flex flex-col">
      <GridHeader columns={columns} onColumnAction={handleColumnAction} />
      {/* ... 나머지 UI */}
    </div>
  );
}
```

## 장점

- **재사용성**: 여러 컴포넌트에서 같은 로직을 재사용할 수 있습니다.
- **관심사 분리**: 로직과 UI가 명확히 분리되어 코드가 깔끔해집니다.
- **테스트 용이성**: Hook 단위로 테스트가 가능합니다.
- **유지보수성**: 로직이 분리되어 있어 수정이 용이합니다.

## 단점

- **과도한 분리**: 너무 많은 Hook으로 분리하면 오히려 코드 이해가 어려워질 수 있습니다.
- **의존성 관리**: Hook 간의 의존성이 복잡해질 수 있습니다.

## 추가 고려사항

- **Hook 네이밍**: `use` 접두사를 사용하여 Hook임을 명확히 합니다.
- **의존성 주입**: 외부 의존성은 파라미터로 주입하여 유연성을 높입니다.
- **타입 안정성**: TypeScript를 사용하여 타입 안정성을 확보합니다.

## 결론

Custom Hook 패턴은 React 애플리케이션에서 로직을 재사용하고 관리하는 강력한 방법입니다.
적절히 사용하면 코드의 재사용성과 유지보수성을 크게 향상시킬 수 있습니다.
