# 📌 TypeScript 설정 파일 구조 (`tsconfig.json`)

## ✅ 왜 `tsconfig.app.json`과 `tsconfig.node.json`으로 나누는가?
TypeScript 설정을 나누는 이유는 **프론트엔드 코드와 백엔드/설정 코드(Vite, Node 관련 코드 등)**를 분리하기 위해서입니다.

- **`tsconfig.app.json`** → **앱 실행 코드** (React 코드, UI 관련 코드)
- **`tsconfig.node.json`** → **Vite, ESLint, Storybook 같은 설정 관련 코드** (Node 실행용)

### 💡 이렇게 나누면 좋은 점
1️⃣ **Vite, ESLint 설정(`vite.config.ts` 등)에 불필요한 React 설정이 안 들어감**  
2️⃣ **React 빌드할 때, Vite 설정 같은 Node 관련 파일이 포함되지 않음**  
3️⃣ **TypeScript 성능 최적화 가능 (불필요한 파일 체크 안 함)**  

---

## ✅ 각 설정 파일에 무엇을 써야 하는가?

### 📌 1. `tsconfig.json` (최상위 설정)
- 프로젝트 전체 TypeScript 설정을 관리하는 역할 (Entry Point)
- **`tsconfig.app.json`과 `tsconfig.node.json`을 참조(extends)하는 역할**

**✔ 파일 내용 예시**
```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" }
  ]
}
```
✅ **이 파일은 설정을 직접 추가하는 곳이 아님!**  
➡️ 대신, **`tsconfig.app.json`과 `tsconfig.node.json`을 불러와서 관리하는 역할**을 함.

---

### 📌 2. `tsconfig.app.json` (React 관련 코드)
- **React 앱 실행을 위한 TypeScript 설정**
- **컴포넌트, Zustand Store, Hooks 등** UI 관련 코드 설정을 여기에 추가해야 함.
- `src/` 내부의 코드에 대한 설정을 관리.

**✔ 파일 내용 예시**
```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"]
}
```
✅ **어떤 설정을 넣어야 하나?**  
✅ UI 관련 코드(`src/` 폴더)의 TypeScript 설정 추가  
✅ `@/` 별칭(`alias`)을 `src/`로 연결  

➡️ **✅ React 앱 실행 코드(`src/pages`, `src/components` 등)는 여기 포함!**  
➡️ **🚫 Vite 설정(`vite.config.ts`) 같은 Node 관련 설정은 여기에 넣으면 안 됨!**  

---

### 📌 3. `tsconfig.node.json` (Vite, Storybook, ESLint 설정 관련 코드)
- **Vite, ESLint, Storybook 같은 Node 환경에서 실행되는 코드에 대한 설정**
- **Node.js 환경에서 실행되는 파일들** (`vite.config.ts`, `eslint.config.ts` 등)이 여기 포함됨.

**✔ 파일 내용 예시**
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["vite.config.ts"]
}
```
✅ **어떤 설정을 넣어야 하나?**  
✅ `vite.config.ts`, `eslint.config.ts`, `storybook` 설정 같은 **Node 환경에서 실행되는 파일들**  
✅ Node 관련 실행 코드 (`lib`, `utils` 같은 백엔드 관련 파일들)  

➡️ **🚫 React 컴포넌트 관련 코드(`src/` 폴더)는 여기에 포함하면 안 됨!**  
➡️ **✅ Vite 설정(`vite.config.ts`) 같은 파일만 여기에 포함!**  

---

## ✅ 어디에 뭘 써야 하는지 최종 정리

| 설정 파일 | 포함해야 하는 코드 | 예제 파일 |
|----------|----------------|----------|
| `tsconfig.json` | 전체 TypeScript 설정 (참조용) | - |
| `tsconfig.app.json` | **React UI 관련 코드** (앱 실행) | `src/pages/Home.tsx`, `src/components/Button.tsx` |
| `tsconfig.node.json` | **Node 실행 관련 코드** (설정용) | `vite.config.ts`, `eslint.config.ts` |

📌 **한마디로 정리하면**  
- ✅ **React 실행 코드** → `tsconfig.app.json`  
- ✅ **Vite & 설정 코드** → `tsconfig.node.json`  

---

이제 **완벽하게 이해됐나요?** 혹시 헷갈리는 부분 있으면 언제든지 확인하세요! 🚀

