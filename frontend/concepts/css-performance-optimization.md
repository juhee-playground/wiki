# CSS 성능 최적화 가이드

## 목차
1. [현재 프로젝트의 CSS 성능 문제점](#현재-프로젝트의-css-성능-문제점)
2. [최적화 방안](#최적화-방안)
3. [구현 예시](#구현-예시)
4. [성능 측정 및 개선 효과](#성능-측정-및-개선-효과)

---

## 현재 프로젝트의 CSS 성능 문제점

### 1. **애니메이션에서 비효율적인 속성 사용** ⚠️

**문제 코드:**
```css
/* animations.css - 일부 애니메이션에서 scale과 translate 혼용 */
@keyframes slideUp {
  from {
    opacity: 0;
    transform: scale(0.96); /* 리플로우 유발 가능성 */
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px) scale(0.95); /* 복합 transform */
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
```

**왜 문제인가?**
- `scale`과 `translate`를 동시에 사용하면 브라우저가 복잡한 계산을 수행
- 일부 애니메이션에서 `width`, `height` 변경으로 인한 리플로우 발생 가능성
- GPU 레이어 생성이 비효율적으로 이루어짐

### 2. **과도한 transition 사용** ⚠️

**문제 코드:**
```typescript
// 여러 컴포넌트에서 발견
className="transition-all duration-300 hover:bg-white/10"
className="transition-colors"
className="transition-all duration-75"
```

**왜 문제인가?**
- `transition-all`은 모든 속성 변경을 감지하여 불필요한 계산 발생
- hover 상태에서 매번 transition이 실행되어 성능 저하
- 특히 대량의 카드가 있는 대시보드에서 누적 효과

### 3. **모달 애니메이션의 비효율적 구현** ⚠️

**문제 코드:**
```typescript
// ModalRightPanel.tsx
<Dialog.Content
  className='... transition-transform duration-300 z-[100]'
  style={{ transform: isOpen ? 'translateX(0)' : 'translateX(100%)' }}
>
```

**왜 문제인가?**
- 인라인 스타일로 transform을 직접 조작
- `will-change` 속성이 없어 GPU 레이어 생성 지연
- 모달 열림/닫힘 시 부드럽지 않은 애니메이션

### 4. **스크롤바 스타일링의 성능 영향** ⚠️

**문제 코드:**
```css
/* preflight.css */
.scroll::-webkit-scrollbar {
  width: 4px;
  height: 4px;
}

.scroll::-webkit-scrollbar-thumb {
  height: 30%;
  background: grey;
  border-radius: 10px;
}
```

**왜 문제인가?**
- 커스텀 스크롤바는 브라우저 최적화를 우회
- 스크롤 시 추가적인 리페인트 발생
- 특히 대량의 스크롤 요소가 있을 때 성능 저하

### 5. **backdrop-filter 과다 사용** ⚠️

**문제 코드:**
```css
.glass-border {
  backdrop-filter: blur(18px) saturate(160%);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.4), 0 4px 30px rgba(0, 0, 0, 0.1);
}
```

**왜 문제인가?**
- `backdrop-filter`는 GPU 집약적 연산
- 특히 모바일 기기에서 성능 저하 심각
- 여러 요소에 동시 적용 시 프레임 드롭 발생

---

## 최적화 방안

### 1. **GPU 가속 최적화**

**will-change 속성 활용:**
```css
/* 애니메이션 요소에 미리 GPU 레이어 생성 */
.animated-element {
  will-change: transform, opacity;
}

/* 애니메이션 완료 후 will-change 제거 */
.animated-element.animation-complete {
  will-change: auto;
}
```

**transform3d 사용:**
```css
/* 기존 */
transform: translateX(100px);

/* 최적화 */
transform: translate3d(100px, 0, 0);
```

### 2. **애니메이션 속성 최적화**

**단일 속성 transition 사용:**
```css
/* 기존 - 비효율적 */
.transition-all {
  transition: all 0.3s ease;
}

/* 최적화 - 구체적 속성 지정 */
.transition-transform {
  transition: transform 0.3s ease;
}

.transition-opacity {
  transition: opacity 0.3s ease;
}
```

**애니메이션 최적화:**
```css
/* 기존 - 복합 transform */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* 최적화 - 단순화 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### 3. **모달 애니메이션 최적화**

**CSS 클래스 기반 애니메이션:**
```css
.modal-content {
  will-change: transform;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.modal-content.closed {
  transform: translateX(100%);
}

.modal-content.open {
  transform: translateX(0);
}
```

### 4. **스크롤 성능 최적화**

**contain 속성 활용:**
```css
.scroll-container {
  contain: layout style paint;
  overflow: auto;
}

/* 또는 최소한의 스크롤바 스타일링 */
.minimal-scrollbar::-webkit-scrollbar {
  width: 2px;
}

.minimal-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 1px;
}
```

### 5. **backdrop-filter 최적화**

**조건부 적용:**
```css
/* 고성능 기기에서만 적용 */
@media (prefers-reduced-motion: no-preference) {
  .glass-border {
    backdrop-filter: blur(8px); /* blur 값 감소 */
  }
}

/* 저성능 기기에서는 대체 */
@media (prefers-reduced-motion: reduce) {
  .glass-border {
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: none;
  }
}
```

---

## 구현 예시

### 1. **드래그 애니메이션 최적화**

```typescript
// 기존 코드 (문제)
const handleDragMove = (e: MouseEvent) => {
  if (dragCardRef.current) {
    dragCardRef.current.style.transform = `translate(${translateX}px, ${translateY}px)`;
    dragCardRef.current.style.zIndex = '50';
    dragCardRef.current.style.opacity = '0.8';
  }
};

// 최적화된 코드
const useOptimizedDrag = () => {
  const dragElement = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (dragElement.current) {
      // GPU 레이어 미리 생성
      dragElement.current.style.willChange = 'transform';
    }
  }, []);
  
  const handleDragMove = useCallback((e: MouseEvent) => {
    if (dragElement.current) {
      // transform3d로 GPU 가속 활용
      dragElement.current.style.transform = `translate3d(${translateX}px, ${translateY}px, 0)`;
    }
  }, []);
  
  return { dragElement, handleDragMove };
};
```

### 2. **모달 애니메이션 최적화**

```typescript
// 기존 코드 (문제)
<Dialog.Content
  className='... transition-transform duration-300'
  style={{ transform: isOpen ? 'translateX(0)' : 'translateX(100%)' }}
>

// 최적화된 코드
<Dialog.Content
  className={`modal-content ${isOpen ? 'open' : 'closed'}`}
>
```

```css
.modal-content {
  will-change: transform;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.modal-content.closed {
  transform: translateX(100%);
}

.modal-content.open {
  transform: translateX(0);
}
