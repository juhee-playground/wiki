# 📚 Frontend 폴더 구조 & Export 규칙

이 문서는 프로젝트 내 폴더 구조와 `index.ts` 관리, import/export 규칙을 정의합니다.  
Tindlo 프론트엔드 프로젝트의 일관성과 유지보수를 돕기 위한 기준입니다.

---

## 📁 폴더 구조 규칙

### 1. Feature 기반 폴더 나누기

- `components/` 하위는 도메인 또는 공통 기능 기준으로 나눈다.
> ```
> components/
> ├── dashboard/
> │   ├── container/
> │   ├── view/
> │   ├── ui/
> │   │   ├── card/
> │   │   ├── grid/
> │   │   └── modal/
> ├── profile/
> │   ├── container/
> │   ├── view/
> │   └── ui/
> ├── common/
> │   ├── ui/
> │   ├── form/
> │   ├── feedback/
> │   ├── modal/
> │   ├── menu/
> │   └── select/
> ```

- `dashboard`, `profile` 등 도메인 단위로 구분
- `common/` 폴더는 범용 컴포넌트만 관리 (ex: Button, Input, Modal)

---

## 📦 common 폴더 세부 구조

- `ui/` : 레이아웃 및 기본 요소 (Avatar, Button, Card 등)
- `form/` : 입력 폼 관련 컴포넌트 (Input, Checkbox, Radio 등)
- `feedback/` : 사용자 알림 요소 (Toast, AlertDialog 등)
- `modal/` : 모달 창 관련 컴포넌트 (Modal, ConfirmModal 등)
- `menu/` : 메뉴, 드롭다운 컴포넌트
- `select/` : 셀렉트박스 관련 컴포넌트

---

## 🧩 index.ts 작성 규칙

### 1. 기본 원칙

- **모든 폴더에는 반드시 `index.ts`를 만든다.**
- 해당 폴더에 있는 컴포넌트를 통합 export한다.

### 2. export 방식

| 상황 | 규칙 | 예시 |
|:---|:---|:---|
| default export인 경우 | `export { default as XXX } from './XXX';` | `export { default as Input } from './Input';` |
| named export인 경우 | `export * from './XXX';` | `export * from './Tabs';` |

### 3. import 시 규칙

- 컴포넌트 import는 반드시 `index.ts`를 경유한다.
- 타입(import type)은 원본 파일에서 직접 가져온다.

```tsx
// 컴포넌트 import
import { Input } from '@/components/common/form';

// 타입 import
import type { IInputProps } from '@/components/common/form/Input';
```

(필요 시 index.ts에서 타입 export를 추가할 수 있다.)

---

## 📋 기타 주의사항

- 스토리 파일(.stories.tsx), 테스트 파일(.test.tsx)은 index.ts에서 export하지 않는다.
- 파일명과 컴포넌트명은 반드시 PascalCase를 따른다. (ex: ScheduleCard.tsx, ModalRightPanel.tsx)
- 폴더명은 소문자와 kebab-case를 기본으로 하지만, 도메인 단위(dashboard, calendar)는 단일 소문자 사용.

## ✨ 요약

- 폴더는 feature/domain 단위로 나눈다.
- common은 ui/form/feedback/modal 등으로 세분화한다.
- index.ts는 반드시 작성하고, 규칙에 맞게 export한다.
- 컴포넌트는 index.ts 경유, 타입은 원본 파일에서 직접 import한다.

---
