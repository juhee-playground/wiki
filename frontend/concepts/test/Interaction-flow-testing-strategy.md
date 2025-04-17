# 사용자 인터랙션 흐름 테스트 전략

## ✅ 목적
사용자 인터랙션 흐름 테스트(Interaction Flow Test)는 컴포넌트 단위 테스트와 E2E 테스트 사이에서 **실제 사용자 행동 흐름을 시뮬레이션하여 검증하는 테스트 계층**이다. 주로 **컨테이너 레벨**에서 진행되며, 다음과 같은 목적을 갖는다:

- 사용자 입장에서 주요 흐름이 잘 작동하는지 검증
- 복잡한 상호작용(예: 상태 변경, 조건부 렌더링)을 실제처럼 테스트
- 단위 테스트의 보완 및 E2E 테스트 전에 빠르게 오류를 잡는 용도

---

## ✅ 테스트 대상 예시
다음은 인터랙션 흐름 테스트로 확인해야 할 대표적인 흐름이다:

### 1. 카드 생성 흐름
- 셀 클릭 → 카드 생성 → 인풋 포커스 → 제목 입력 → 엔터 또는 blur로 저장 → 콜백 호출 여부

### 2. 카드 수정 흐름
- 카드 제목 클릭 → 인풋 전환 → 수정 → 엔터로 저장 / ESC로 취소 → 상태 변경 확인

### 3. 카드 삭제 흐름
- 카드 우클릭 → 컨텍스트 메뉴 오픈 → 삭제 클릭 → 카드가 DOM에서 사라졌는지 확인

### 4. 카드 모달 열기/닫기 흐름
- 카드 본체 클릭 → 모달 오픈 → 닫기 버튼 클릭 → 모달 닫힘 확인 → 상태 초기화 확인

### 5. 컨텍스트 메뉴에서 수정 클릭 흐름
- 카드 우클릭 → 수정 클릭 → 인풋으로 전환 → 카드 편집 시작

---

## ✅ 테스트 구조 전략

### 1. 테스트 컴포넌트 분리
- 컨테이너 컴포넌트에서 상태 확인이 어려울 경우, 내부 상태를 추적 가능한 `TestableXXXContainer`로 분리
- 해당 컴포넌트에서 상태 변경을 `useState`로 직접 관리

```tsx
// 예시: TestableDayDashboardContainer
const [cards, setCards] = useState(initialCards);
const handleAdd = (card) => setCards(prev => [...prev, card]);
```

### 2. 모킹 전략
- `getBoundingClientRect()` 모킹 필수 (클릭 위치 → 시간 계산하는 경우)
- 외부 콜백은 `vi.fn()`으로 선언

### 3. 유틸 분리
- `mockCards`, `mockColumns`, `mockHours` 등의 데이터는 `__mocks__` 또는 `test-utils` 폴더로 이동해 재사용성 높이기

---

## ✅ 작성 요령

### ⬛ 테스트 이름 작성
- `[카드 생성] 셀 클릭 시 새 카드 생성되고 제목 저장까지의 흐름 확인`

### ⬛ 테스트 시나리오 구성
```tsx
it('[카드 생성] 셀 클릭 후 제목 입력 → 카드 추가', async () => {
  render(<TestableContainer ... />);
  const cell = screen.getByTestId('grid-cell-10:00-1');
  mockCellPosition(cell);
  await userEvent.click(cell);
  const input = await screen.findByTestId('schedule-title-input');
  await userEvent.type(input, '회의 일정{enter}');
  expect(screen.getByText('회의 일정')).toBeInTheDocument();
});
```

### ⬛ DOM 변경 기다릴 땐 `await waitFor()` 사용
```ts
await waitFor(() => {
  expect(screen.queryByText('삭제된 카드 제목')).not.toBeInTheDocument();
});
```

---

## ✅ 좋은 테스트의 조건
- 사용자 입장에서 핵심 흐름을 정확히 시뮬레이션함
- 너무 세부 구현에 의존하지 않음 (예: 내부 상태보단 결과 DOM 기반 검증)
- 필수 유틸과 테스트 컴포넌트는 재사용 가능하게 정리됨

---

## ✅ 추천 폴더 구조
```
src/
├── __mocks__/
│   └── tests/
│       ├── mockCards.test.ts
│       └── mockColumns.test.ts
├── components/
│   └── dashboard/
│       ├── container/
│       │   ├── DayDashboardContainer.tsx
│       │   ├── TestableDayDashboardContainer.tsx
│       │   └── DayDashboardContainer.interaction.test.tsx
```

---

## ✅ 마무리 팁
- 이 테스트는 **비즈니스 로직과 사용자 인터페이스를 연결해주는 브리지 역할**
- 컴포넌트 단위와 E2E 테스트의 중간에서 유지보수 효율을 높이는 데 효과적
- 언제든지 `추가 흐름이 생기면 여기 전략 문서에 등록`하고 함께 관리할 것

