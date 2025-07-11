# Portal 사용 가이드

## 개요

Portal은 React에서 제공하는 기능으로, 컴포넌트를 DOM 계층 구조상 현재 위치가 아닌 다른 곳에 렌더링할 수 있게 해주는 메서드입니다. 주로 모달, 드롭다운, 툴팁과 같이 부모 컴포넌트의 스타일 영향을 받지 않아야 하는 UI 요소에 사용됩니다.

## 사용 사례

### 1. 드롭다운 메뉴 (BasicSelect)
```tsx
// 기존 방식 (테이블 내부에 렌더링)
<table>
  <tr>
    <td>
      <select>
        <option>옵션1</option>  // 테이블 레이아웃에 영향을 줌
      </select>
    </td>
  </tr>
</table>

// Portal 사용 (테이블 외부에 렌더링)
import { createPortal } from 'react-dom';

<table>
  <tr>
    <td>
      <select />  // 선택 버튼만 테이블 안에 존재
    </td>
  </tr>
</table>
{createPortal(
  <div className="dropdown">  // 드롭다운은 body에 렌더링
    <option>옵션1</option>
  </div>,
  document.body
)}
```

### 2. 모달 다이얼로그
```tsx
const Modal = ({ isOpen, children }) => {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">
        {children}
      </div>
    </div>,
    document.body
  );
};
```

## Portal 사용 시 고려사항

### 1. 이벤트 버블링
- Portal을 사용해도 React 이벤트는 정상적으로 버블링됨
- DOM 트리가 아닌 React 컴포넌트 트리를 따라 전파

### 2. 위치 계산
```tsx
const [position, setPosition] = useState({ top: 0, left: 0 });

useEffect(() => {
  if (triggerRef.current) {
    const rect = triggerRef.current.getBoundingClientRect();
    setPosition({
      top: rect.bottom + window.scrollY,
      left: rect.left + window.scrollX
    });
  }
}, [isOpen]);
```

### 3. 클린업
```tsx
useEffect(() => {
  // 이벤트 리스너 등록
  window.addEventListener('scroll', updatePosition);
  window.addEventListener('resize', updatePosition);

  return () => {
    // 클린업: 이벤트 리스너 제거
    window.removeEventListener('scroll', updatePosition);
    window.removeEventListener('resize', updatePosition);
  };
}, []);
```

## 성능 최적화

### 1. 조건부 렌더링
```tsx
{isOpen && createPortal(
  <DropdownContent />,
  document.body
)}
```

### 2. 메모이제이션
```tsx
const MemoizedDropdown = useMemo(() => (
  <DropdownContent items={items} />
), [items]);

{isOpen && createPortal(MemoizedDropdown, document.body)}
```

## 스타일링 가이드

### 1. z-index 관리
```css
.portal-dropdown {
  z-index: 9999; /* 항상 최상위에 표시 */
  position: fixed;
}
```

### 2. 애니메이션
```css
.portal-dropdown {
  transition: opacity 200ms ease-in-out;
  opacity: 0;
}

.portal-dropdown.open {
  opacity: 1;
}
```

## 접근성 (A11y)

### 1. 키보드 네비게이션
```tsx
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.key === 'Escape') {
      onClose();
    }
  };
  
  window.addEventListener('keydown', handleKeyDown);
  return () => window.removeEventListener('keydown', handleKeyDown);
}, [onClose]);
```

### 2. ARIA 속성
```tsx
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="modal-title"
>
  {children}
</div>
```

## 테스트

### 1. Portal 테스트 예시
```tsx
import { render, screen } from '@testing-library/react';

test('renders portal content', () => {
  render(<PortalComponent />);
  
  // Portal 내용이 document.body에 렌더링되었는지 확인
  expect(screen.getByRole('dialog')).toBeInTheDocument();
});
```

## 주의사항

1. SSR (Server-Side Rendering)
   - `document`가 없는 환경에서는 Portal을 사용할 수 없음
   - SSR 환경에서는 조건부 렌더링 필요

2. 이벤트 핸들링
   - Portal 외부 클릭 감지 시 ref 사용 필요
   - 여러 Portal이 있는 경우 z-index 관리 중요

3. 메모리 관리
   - Portal 언마운트 시 이벤트 리스너 정리
   - 불필요한 재렌더링 방지

## 참고 자료

- [React 공식 문서 - Portals](https://react.dev/reference/react-dom/createPortal)
- [MDN - createPortal](https://developer.mozilla.org/en-US/docs/Web/API/createPortal)
