# 📦 Import 정렬 가이드 (ESLint `import/order`)

이 문서는 `eslint-plugin-import`의 `import/order` 룰을 기반으로, 가독성과 유지보수성을 높이기 위한 **Import 정렬 규칙**을 정의합니다.

---

## ✅ 기본 그룹 순서

```ts
import/order: [
  'error',
  {
    groups: [
      'builtin',         // Node.js 내장 모듈 (fs, path 등)
      'external',        // 외부 라이브러리 (react, axios 등)
      'internal',        // 절대경로 import (@/lib 등)
      ['parent', 'sibling'], // 상대경로 import
      'index',           // ./index 등
      'object',          // rarely used (dynamic imports)
      'type'             // 타입 전용 import (import type)
    ],
  }
]
```

---

## 🧩 pathGroups 예시 (우선순위 정렬)

```ts
pathGroups: [
  { pattern: 'react*', group: 'external', position: 'before' },
  { pattern: '@storybook/*', group: 'internal', position: 'before' },
  { pattern: '@/**/*', group: 'internal', position: 'before' },
  { pattern: './**/*.styles', group: 'sibling', position: 'after' },
  { pattern: './**.css', group: 'sibling', position: 'after' },
  { pattern: '@/assets/**', group: 'internal', position: 'after' },
  { pattern: '@/constants/**', group: 'internal', position: 'before' },
],
```

---

## 🧾 최종 Import 순서 예시

```ts
// 1. Node.js 내장 모듈
import fs from 'fs';

// 2. React 및 외부 라이브러리
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

// 3. Storybook 또는 테스트 라이브러리
import { Meta } from '@storybook/react';

// 4. 내부 절대 경로 (aliases)
import { useSomething } from '@/hooks/useSomething';
import { theme } from '@/styles/theme';
import CONSTANTS from '@/constants/app';
import Logo from '@/assets/logo.svg';

// 5. 상대 경로
import MyComponent from '../MyComponent';
import useUtils from './utils';

// 6. 스타일 관련
import './MyComponent.styles';
import './global.css';
```

---

## ✍️ 기타 옵션

```ts
'newlines-between': 'always-and-inside-groups',

alphabetize: {
  order: 'asc',
  caseInsensitive: true,
},
```

- **newlines-between**: 각 그룹 사이 줄바꿈 강제
- **alphabetize**: 그룹 내부에서도 알파벳 순 정렬

---

## 📌 권장 경로 alias 구조

`tsconfig.json` 또는 `vite.config.ts`, `webpack.config.js` 에서 다음과 같이 alias를 정의해두면 import 정렬이 더 직관적입니다.

```json
"paths": {
  "@/components/*": ["src/components/*"],
  "@/hooks/*": ["src/hooks/*"],
  "@/lib/*": ["src/lib/*"],
  "@/assets/*": ["src/assets/*"]
}
```

---

## ✅ 결론

- `import/order`를 통해 **그룹별 import 정렬**을 강제하면 코드 가독성이 높아집니다.
- `pathGroups`로 세부 우선순위를 정의하고, 스타일/자산 import를 항상 마지막에 유지하세요.
- 팀 내에서 공유되는 alias, 절대경로 규칙을 명확히 유지하고, Storybook/Testing import는 별도 우선순위로 관리하는 것이 좋습니다.

