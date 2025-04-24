# 프런트엔드 디자인 가이드라인

이 문서는 일관된 프런트엔드 개발을 위해 따라야 할 디자인 가이드라인을 정의합니다. 
읽기 쉬운 코드, 예측 가능한 동작, 높은 응집도, 낮은 결합도를 추구합니다. 원문은 Toss의 Design Guideline을 기반으로 하고, juhee 프로젝트 특성을 반영하여 정리되었습니다.

---

## 📘 Readability (가독성)

### 🔢 매직 넘버는 의미 있는 상수로 대체하기

```ts
const ANIMATION_DELAY_MS = 300;
await delay(ANIMATION_DELAY_MS);
```

- 설명 없는 숫자 대신 의미 부여 → 유지보수성 향상

---

### 🔍 복잡한 로직은 전용 컴포넌트로 분리

```tsx
<AuthGuard>
  <LoginStartPage />
</AuthGuard>
```

- 인증/권한 확인 등은 전용 HOC 또는 Wrapper 컴포넌트로 캡슐화

---

### 🧩 역할별 분기 로직은 별도 컴포넌트로 분리

```tsx
return isViewer ? <ViewerSubmitButton /> : <AdminSubmitButton />;
```

- 조건 분기 로직을 분리해서 컴포넌트 역할 명확화

---

### ❓복잡한 삼항연산자는 IIFE나 if/else로 치환

```ts
const status = (() => {
  if (A && B) return 'BOTH';
  if (A) return 'A';
  if (B) return 'B';
  return 'NONE';
})();
```

---

### 👁️ 눈의 이동 줄이기 (로직 근처에 위치시키기)

```tsx
const policy = {
  admin: { canInvite: true },
  viewer: { canInvite: false },
}[user.role];
```

- 인라인으로 간단한 정책/로직 표현 → 읽기 쉬움

---

### ✅ 조건이 복잡하면 이름 붙이기

```ts
const isSameCategory = ...;
const isPriceInRange = ...;
return isSameCategory && isPriceInRange;
```

- 의미 명확화, 테스트 가능성 향상

---

## 🔄 Predictability (예측 가능성)

### 📦 훅 리턴 타입은 일관성 유지

```ts
function useUser(): UseQueryResult<User, Error> {...}
```

---

### ✅ Validation 함수는 Discriminated Union 사용

```ts
type ValidationResult = { ok: true } | { ok: false; reason: string };
```

---

### 🔍 함수는 자신의 일만 (SRP)

```ts
const balance = await fetchBalance();
logging.log('balance_fetched');
await syncBalance(balance);
```

---

### 🔤 함수명은 명확하게 (e.g. getWithAuth)

```ts
httpService.getWithAuth(...);
```

---

## 🧩 Cohesion (응집도)

### 🧪 Field-Level vs Form-Level Validation 선택 기준

- Field-Level → 재사용성, 독립성 중요할 때
- Form-Level → 관련성 강한 필드, Wizard Form 등

---

### 📁 폴더는 도메인 기준으로 구성 (Tindlo 적용 예)

```
src/
├── components/        # 공통 컴포넌트
├── hooks/             # 공통 훅
├── stores/            # Zustand 상태 관리
├── domains/
│   ├── schedule/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── utils/
│   │   └── types.ts
│   ├── user/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── ...
```

- 기능별로 분리 → 코드 찾기, 유지보수, 삭제 용이

---

### 🔗 상수는 관련 로직 근처에 정의하거나 명확한 이름 사용

---

## 🔓 Coupling (결합도 줄이기)

### 🧱 성급한 추상화 지양

- 로직이 갈릴 가능성 있는 경우 중복 허용 → 미래 확장 고려

---

### 🔍 상태 관리 스코프는 작게 나눠서 사용 (Zustand 적극 활용)

```ts
const [cardId, setCardId] = useCardIdQueryParam();
```

- 불필요한 리렌더 줄이고 관심사 분리

---

### 🧼 Props Drilling → Composition으로 전환

```tsx
<Modal>
  <Input value={keyword} onChange={setKeyword} />
  <ItemEditList keyword={keyword} />
</Modal>
```

- 중간 컴포넌트 제거, 필요한 컴포넌트에서 직접 prop 전달

#### 🔧 현재 Project 상태:

- `DashboardContainer → DayDashboardContainer → ScheduleCard`로 props 전달됨
- `useScheduleStore` 또는 컴포지션 기반 구조로 전환 예정
- 이후: zustand + shallow selector로 필요한 컴포넌트만 state 접근

---
