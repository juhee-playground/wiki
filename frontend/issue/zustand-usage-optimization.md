# 🚀 Zustand에서 `useAuthStore` 사용 최적화

## ✅ **문제: Zustand에서 `useAuthStore()`를 사용할 때 최적의 방식은?**

### ❌ **비효율적인 방식 (구조 분해 할당 사용)**
```tsx
const { logIn } = useAuthStore();
```
### 🚨 **문제점**
- `useAuthStore()`는 상태 전체를 반환하기 때문에, **스토어 내의 모든 상태가 변경될 때 리렌더링됨.**
- `logIn` 함수는 변하지 않지만, 상태가 변경될 때마다 **컴포넌트가 불필요하게 리렌더링**됨.

---

## ✅ **🚀 최적의 방식: 필요한 값만 선택하여 가져오기**
```tsx
const logIn = useAuthStore(state => state.logIn);
```
### 🎯 **이 방식의 장점**
- ✅ **불필요한 리렌더링 방지** → `logIn` 함수만 가져와서 사용
- ✅ **Zustand의 리렌더링 최적화 활용** (shallow 비교 적용 가능)
- ✅ **로그인 상태(`isAuthenticated` 등) 변경과 무관하게 `logIn` 함수 유지**

---

## 📌 **✅ 여러 상태를 가져와야 할 때: `shallow` 사용**
```tsx
import { shallow } from 'zustand/shallow';

const { logIn, isAuthenticated } = useAuthStore(
  state => ({
    logIn: state.logIn,
    isAuthenticated: state.isAuthenticated,
  }),
  shallow
);
```
### 🎯 **이 방식의 장점**
- ✅ `shallow` 비교를 적용하여 **여러 상태를 가져와도 불필요한 리렌더링 방지**
- ✅ `logIn`과 `isAuthenticated` 값이 변경될 때만 컴포넌트 리렌더링

---

## 🚀 **결론: Zustand에서 최적의 사용 방식**
| 방식 | 리렌더링 발생 조건 | 성능 |
|------|------------------|------|
| **`const { logIn } = useAuthStore();`** | `useAuthStore` 내부의 **모든 상태(state)**가 변경될 때 리렌더링 발생 ❌ | 비효율적 ❌ |
| **`const logIn = useAuthStore(state => state.logIn);`** | `logIn` 함수가 변하지 않으므로 **불필요한 리렌더링 방지** ✅ | 최적화됨 ✅ |
| **`shallow`를 활용한 최적화 (`state => ({ ... })`)** | 여러 개의 상태를 가져와도 `shallow` 비교로 변경된 값만 반영 ✅ | 최적화됨 ✅ |

---

## 🎯 **최종 결론**
✅ Zustand에서 상태를 사용할 때는 **`state => state.logIn`** 형식으로 **필요한 값만 가져오는 것이 최적화된 방식**
✅ 여러 개의 상태를 가져와야 한다면 **`shallow` 비교를 사용하여 불필요한 리렌더링 방지**
✅ **Zustand 내부의 상태 변경이 많아질 경우, 구조 분해 할당은 피하는 것이 좋음**

---
