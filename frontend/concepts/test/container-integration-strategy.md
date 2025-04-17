# 🧪 Container 연동 테스트 전략

이 문서는 컨테이너 컴포넌트(Container Component)에 대한 연동 테스트 전략을 정의합니다. 해당 전략은 컴포넌트와 내부 상태, props, 콜백 간의 연동을 테스트할 때 일관된 기준을 제공합니다.

---

## ✅ 테스트 목적

컨테이너 컴포넌트는 다음과 같은 책임을 가지므로, 연동 테스트는 아래 목적을 달성해야 합니다:

1. **UI 구성 요소를 올바르게 렌더링하는지 검증**
2. **props에 따라 적절한 상태/컴포넌트가 표시되는지 확인**
3. **내부 상태 변화 시 적절한 동작이 발생하는지 확인**
4. **외부 콜백 함수(onXxx 등)가 올바르게 호출되는지 검증**

---

## 🧩 테스트 구성 예시

```tsx
render(
  <DayDashboardContainer
    gridContainerRef={{ current: container }}
    projectColumns={mockColumns}
    hours={mockHours}
    scheduleCards={mockCards}
    onProjectColumnAction={vi.fn()}
    onAddCard={vi.fn()}
    onUpdateCard={vi.fn()}
    onDeleteCard={vi.fn()}
  />
);
```

---

## 🧪 필수 테스트 항목 체크리스트

| 항목            | 설명                                                                       |
| ------------- | ------------------------------------------------------------------------ |
| ✅ 초기 렌더링      | 모든 props 기반 UI 구성 요소들이 정상적으로 보이는지 확인                                     |
| ✅ props 변경 반영 | 외부에서 전달된 scheduleCards / columns 등이 반영되는지 확인                             |
| ✅ 콜백 호출 여부    | 카드 추가/수정/삭제 등에서 `onAddCard`, `onUpdateCard`, `onDeleteCard` 등이 호출되는지 테스트 |
| ✅ 내부 상태 반영    | 모달 열림/닫힘, 선택된 카드 등의 내부 상태 변화 확인                                          |

---

## 🧠 팁 & 패턴

- **Testable Wrapper** 사용

  - 내부 상태 기반 테스트가 필요한 경우 `TestableDayDashboardContainer`처럼 상태를 관리하는 테스트 전용 컴포넌트를 작성

- **mockCards + events 조합**

  - 기본 카드 목록을 주입하고, 클릭/입력 이벤트로 상태 전이 시나리오 확인

- **BoundingClientRect mocking**

  - 시간 위치 기반 클릭 계산이 필요한 경우 `getBoundingClientRect()` 를 override 해서 테스트

```ts
cell.getBoundingClientRect = () => ({
  top: 0,
  height: 73,
  width: 192,
  bottom: 73,
  left: 0,
  right: 192,
  x: 0,
  y: 0,
  toJSON: () => ({}),
});
```

---

## ✅ 결론

- 컨테이너는 UI와 로직을 연결하는 핵심 계층이므로 테스트 우선순위가 높음
- 콜백 중심, 상태 변화 중심의 테스트를 중심으로 구성
- `Testable 컴포넌트`를 통해 내부 상태 확인을 쉽게 만들 것

