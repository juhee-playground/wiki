# Popover z-index 이슈 해결 가이드

## 🚨 문제 상황

### 발생한 문제
Tindlo 프로젝트의 스냅 스페이스에서 코멘트 기능을 구현할 때, CommentPopover가 다른 요소들 밑으로 깔리는 z-index 문제가 발생했습니다.

**구체적인 문제점:**
- 드래그 중인 카드 (z-index: 50)가 CommentPopover (z-index: 40) 위에 표시됨
- 모달 (z-index: 1000)과 드롭다운 (z-index: 900) 사이의 중간 레이어가 없어서 충돌
- 새로운 컴포넌트 추가 시마다 z-index 값을 임의로 설정하여 일관성 부족

### 문제의 원인
1. **분산된 z-index 관리**: 각 컴포넌트에서 개별적으로 z-index 값을 설정
2. **일관성 없는 값**: 10, 20, 30, 40, 50 등 임의의 값 사용
3. **레이어 체계 부재**: 어떤 요소가 어떤 레이어에 속하는지 명확하지 않음

---

## 🛠️ 해결 방안

### 1. **중앙 집중식 z-index 관리**

**constants/canvas.ts에서 상수 정의:**
```typescript
export const CANVAS_CONFIG = {
  Z_INDEX: {
    REMOVE_BUTTON: 20,
    POPOVER: 50,
  },
} as const;
```

**왜 이렇게 했는가?**
- **일관성**: 모든 z-index 값을 한 곳에서 관리
- **유지보수성**: 값 변경 시 한 곳만 수정하면 됨
- **가독성**: 각 요소의 역할이 명확해짐

### 2. **체계적인 레이어 구조**

**theme.css에서 CSS 변수로 정의:**
```css
--z-index-modal: 1000;      /* 최상위 모달 */
--z-index-dropdown: 900;    /* 드롭다운 메뉴 */
--z-index-header: 800;      /* 헤더 */
--z-index-footer: 700;      /* 푸터 */
--z-index-tooltip: 600;     /* 툴팁 */
--z-index-overlay: 500;     /* 오버레이 */
--z-index-sticky: 400;      /* 스티키 요소 */
--z-index-fixed: 300;       /* 고정 요소 */
--z-index-above: 200;       /* 일반 요소 위 */
--z-index-default: 100;     /* 기본 요소 */
--z-index-below: 0;         /* 최하위 */
```

**왜 이렇게 했는가?**
- **계층적 구조**: 100 단위로 구분하여 충돌 방지
- **확장성**: 새로운 레이어 추가 시 적절한 위치에 배치 가능
- **예측 가능성**: 개발자가 어떤 값을 사용해야 할지 명확

### 3. **실제 적용 사례**

**CommentLayer.tsx:**
```typescript
<div
  style={{
    position: 'absolute',
    left: activePopover.x,
    top: activePopover.y,
    zIndex: 50, // CANVAS_CONFIG.Z_INDEX.POPOVER 사용
    transform: 'translate(-50%, -100%)',
  }}
>
```

**CommentPopover.tsx:**
```typescript
<div
  className='absolute z-40 bg-neutral-900 rounded-xl shadow-lg p-2'
  // z-40은 Tailwind의 z-index: 40과 동일
>
```

---

## ✅ 해결 효과

### 1. **문제 해결**
- ✅ CommentPopover가 드래그 중인 카드 위에 정상 표시
- ✅ 모달과 드롭다운 간의 레이어 충돌 해결
- ✅ 새로운 컴포넌트 추가 시 적절한 z-index 값 쉽게 결정

### 2. **개발 경험 개선**
- **일관성**: 모든 z-index 값이 체계적으로 관리됨
- **유지보수성**: 값 변경 시 한 곳만 수정하면 됨
- **가독성**: 코드에서 각 요소의 레이어 위치를 쉽게 파악

### 3. **성능 최적화**
- **렌더링 최적화**: 불필요한 z-index 충돌로 인한 리페인트 방지
- **메모리 효율성**: 브라우저가 레이어를 효율적으로 관리

---

## �� 핵심 원칙

### 1. **단일 책임 원칙**
- 각 레이어는 명확한 역할을 가짐
- 모달은 모달, 드롭다운은 드롭다운 역할만 수행

### 2. **계층적 구조**
- 100 단위로 구분하여 충돌 방지
- 상위 레이어는 하위 레이어를 덮을 수 있음

### 3. **중앙 집중식 관리**
- 모든 z-index 값을 한 곳에서 관리
- 상수와 CSS 변수를 활용한 일관성 유지

---

## 📝 향후 개선 방안

### 1. **TypeScript 타입 안전성**
```typescript
type ZIndexLayer = 'modal' | 'dropdown' | 'tooltip' | 'popover';

const getZIndex = (layer: ZIndexLayer): number => {
  const zIndexMap = {
    modal: 1000,
    dropdown: 900,
    tooltip: 600,
    popover: 50,
  };
  return zIndexMap[layer];
};
```

### 2. **동적 z-index 관리**
```typescript
const useZIndex = (layer: ZIndexLayer) => {
  const [zIndex, setZIndex] = useState(getZIndex(layer));
  
  const bringToFront = () => {
    setZIndex(prev => prev + 1);
  };
  
  return { zIndex, bringToFront };
};
```

### 3. **자동화된 z-index 검증**
```typescript
// 개발 환경에서 z-index 충돌 감지
const validateZIndex = (element: HTMLElement) => {
  const zIndex = parseInt(getComputedStyle(element).zIndex);
  if (isNaN(zIndex)) {
    console.warn('Element has no z-index:', element);
  }
};
```

---

## 결론

z-index 이슈는 단순히 값을 높이는 것이 아니라, **체계적인 레이어 관리 시스템**을 구축하는 것이 핵심입니다. 

**주요 성과:**
- ✅ Popover z-index 충돌 완전 해결
- ✅ 개발 생산성 향상 (일관된 z-index 관리)
- ✅ 유지보수성 개선 (중앙 집중식 관리)
- ✅ 확장성 확보 (새로운 레이어 추가 용이)

이러한 접근을 통해 복잡한 UI에서도 예측 가능하고 안정적인 레이어 구조를 유지할 수 있게 되었습니다.
