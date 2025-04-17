🧪 단위 테스트 전략 (Unit Test Strategy)

📌 목적

컴포넌트, 유틸 함수, 커스텀 훅 등의 최소 단위 기능이 의도한 대로 동작하는지 확인하기 위함.

단위 테스트는 각 요소를 독립적으로 테스트하여 빠르게 문제를 파악하고, 리팩터링 시 회귀를 방지하는 역할을 한다.

⸻

✅ 단위 테스트가 필요한 대상

테스트 대상	예시 컴포넌트/모듈	주된 테스트 항목
Presentational UI 컴포넌트	<Button />, <Badge />	렌더링 여부, 텍스트/아이콘 표시, 클래스 조건부 적용 등
Form 컴포넌트	<Input />, <Checkbox />	입력 동작, 포커스, 에러 메시지 렌더링 등
커스텀 훅 (Hooks) useGridScrollSync	반환값, 상태 변화, side effect 등
유틸 함수	반환 값 검증이 필요한 함수, 경계 조건 테스트

⸻

🛠️ 단위 테스트 작성 방법

1. 컴포넌트 테스트
	•	UI 렌더링 확인: 필수 요소가 존재하는지 확인
	•	props 기반 동작: prop 변경에 따른 클래스, 텍스트 변화 확인
	•	이벤트 트리거: 버튼 클릭, 인풋 변경 등의 콜백 실행 여부 확인

it('버튼 클릭 시 콜백이 호출되는지 확인', async () => {
  const onClick = vi.fn();
  render(<Button onClick={onClick}>Click</Button>);
  await userEvent.click(screen.getByText('Click'));
  expect(onClick).toHaveBeenCalled();
});

2. 커스텀 훅 테스트
	•	@testing-library/react-hooks 또는 renderHook 활용
	•	상태 변화, 반환값, side-effect 중심으로 확인
	•	UI와 분리된 순수 로직만 테스트

3. 유틸 함수 테스트
	•	describe와 it을 활용해 다양한 입력 케이스 작성
	•	경계값/예외 상황 포함한 테스트 진행

it('시간 계산 유틸: 09:00에서 60분 후는 10:00', () => {
  expect(calculateEndTime('09:00', 60)).toBe('10:00');
});



⸻

🧩 폴더 구조 예시

src/
  components/
    Button/
      Button.tsx
      Button.test.tsx    ← 단위 테스트
  hooks/
    useCustomHook.ts
    useCustomHook.test.ts ← 커스텀 훅 단위 테스트
  utils/
    time.ts
    time.test.ts         ← 유틸 함수 테스트

⸻

🔍 권장 전략
	•	테스트 커버리지보다는 의미 있는 테스트에 집중
	•	상태 기반 렌더링 / props 변화 / 이벤트 처리에 집중
	•	복잡한 로직은 utils 혹은 hooks로 분리 후 단위 테스트 작성

⸻
