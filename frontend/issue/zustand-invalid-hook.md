# 🚀 Zustand `Invalid Hook Call` 오류 해결 방법

## 🔍 오류 개요
Zustand 스토어(`useAuth`) 내부에서 `useAlertStore()`를 호출하면 다음과 같은 **React Hook 규칙 위반 오류**가 발생할 수 있습니다.

```shell
Invalid Hook Call. Hooks can only be called inside of the body of a function component.
```

### 🔥 **오류 원인**
- Zustand의 `useAlertStore()`는 **React Hook**이므로 **React 컴포넌트나 다른 Hook 내부에서만 사용 가능**합니다.
- `create()` 내부에서 Hook을 호출하면 **React Hook 규칙을 위반**하게 됩니다.

---

## ✅ **해결 방법**
### **❌ 잘못된 코드 (오류 발생)**
```typescript
import { create } from 'zustand';
import { useAlertStore } from './useAlertStore';

export const useAuth = create<IAuthState>((set) => {
  const { setAlert } = useAlertStore(); // ❌ React Hook 규칙 위반!

  return {
    login: async (email, password) => {
      try {
        const response = await loginUser(email, password);
        if (response.data.access_token) {
          set({ isAuthenticated: true, token: response.data.access_token });
          return true;
        } else {
          setAlert('로그인 실패'); // ❌ Hook을 스토어 내부에서 호출
          return false;
        }
      } catch (error) {
        setAlert('서버 오류');
        return false;
      }
    },
  };
});
```

### ✅ **올바른 코드 (`useAlertStore.setState()` 사용)**
```typescript
import { create } from 'zustand';
import { loginUser } from '@/api/userApi';
import { useAlertStore } from './useAlertStore';

export const useAuth = create<IAuthState>((set) => {
  const token = localStorage.getItem('token');

  return {
    isAuthenticated: !!token,
    token,
    login: async (email, password) => {
      try {
        const response = await loginUser(email, password);
        const token = response.data.access_token;

        if (token) {
          set({ isAuthenticated: true, token });
          localStorage.setItem('token', token);
          return true;
        } else {
          useAlertStore.setState({ message: '로그인 실패: 이메일 또는 비밀번호를 확인하세요.', isOpen: true });
          return false;
        }
      } catch (error) {
        useAlertStore.setState({ message: '서버 오류: 다시 시도해주세요.', isOpen: true });
        return false;
      }
    },
    logout: () => {
      set({ isAuthenticated: false, token: null });
      localStorage.removeItem('token');
    },
  };
});
```

---

## 🎯 **정리**
| 문제 | 해결 방법 |
|------|-----------|
| **Zustand Store 내부에서 Hook (`useAlertStore()`) 호출 시 오류 발생** | `useAlertStore.setState({...})` 사용 ✅ |
| **React Hook 규칙 위반 (Invalid Hook Call)** | Hook은 React 컴포넌트 내부에서만 사용 가능 ✅ |
| **Zustand 상태를 변경하려면?** | `useAlertStore.setState({...})`을 사용해야 함 ✅ |

### ✅ **이제 `useAlertStore`를 활용하여 로그인 에러 메시지를 정상적으로 표시할 수 있습니다!** 🚀

---

### 📌 **추가 참고 링크**
- [React 공식 문서: Invalid Hook Call](https://react.dev/link/invalid-hook-call)
- [Zustand 공식 문서](https://docs.pmnd.rs/zustand/getting-started/introduction)

---
