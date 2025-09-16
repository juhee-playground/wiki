
---

## ✅ 해결 효과

### 1. **문제 해결**
- ✅ FileAttachmentsPopover가 다른 TaskCard 밑으로 깔리지 않음
- ✅ 드래그 중인 카드가 있어도 popover가 정상 표시
- ✅ z-index 충돌 완전 해결

### 2. **성능 개선**
- **렌더링 최적화**: 불필요한 stacking context 중첩 방지
- **메모리 효율성**: 브라우저가 레이어를 효율적으로 관리

### 3. **개발 경험 개선**
- **예측 가능성**: popover가 항상 최상위에 표시됨
- **유지보수성**: stacking context 문제로 인한 디버깅 시간 단축

---

## �� Stacking Context 관련 질문 답변

### Q: z-index는 어떻게 동작하나요?
**A:** z-index는 같은 stacking context 내에서만 상대적으로 비교됩니다. 서로 다른 stacking context 간에는 z-index 값이 직접적으로 비교되지 않습니다.

### Q: 새로운 stacking context가 생성되는 경우는 언제인가요?
**A:** 
- `position: relative/absolute/fixed/sticky` + `z-index` 값
- `opacity < 1`
- `transform` 속성 (none 제외)
- `filter` 속성 (none 제외)
- `isolation: isolate`
- `will-change` 속성
- `contain: layout/style/paint`

### Q: stacking context 때문에 발생할 수 있는 UI 문제 사례는?
**A:** 
1. **Popover가 다른 요소 밑으로 깔림**: 각 컴포넌트가 독립적인 stacking context를 가질 때
2. **z-index가 예상대로 동작하지 않음**: 서로 다른 stacking context 간의 z-index 비교 불가
3. **드래그 중 요소가 다른 요소에 가려짐**: 드래그 요소와 다른 요소가 다른 stacking context에 있을 때
4. **모달이 다른 모달에 가려짐**: 중첩된 모달들이 각각 다른 stacking context를 가질 때

---

## 🎯 핵심 원칙

### 1. **Stacking Context 인식**
- 각 컴포넌트가 독립적인 stacking context를 가질 수 있음을 인식
- z-index 문제 발생 시 stacking context 구조를 먼저 확인

### 2. **Portal 활용**
- 다른 stacking context 간의 문제는 Portal로 해결
- popover, tooltip, modal 등은 상위 context로 이동 고려

### 3. **체계적 z-index 관리**
- 중앙 집중식 z-index 관리 시스템 구축
- CSS 변수와 상수를 활용한 일관성 유지

---

## 📝 향후 개선 방안

### 1. **Stacking Context 모니터링**
```typescript
// 개발 환경에서 stacking context 생성 감지
const detectStackingContext = (element: HTMLElement) => {
  const style = getComputedStyle(element);
  const hasStackingContext = 
    (style.position !== 'static' && style.zIndex !== 'auto') ||
    parseFloat(style.opacity) < 1 ||
    style.transform !== 'none' ||
    style.filter !== 'none';
    
  if (hasStackingContext) {
    console.warn('Stacking context created:', element);
  }
};
```

### 2. **Portal 자동화**
```typescript
// 자동으로 적절한 portal container 찾기
const useAutoPortal = (ref: RefObject<HTMLElement>) => {
  const [portalContainer, setPortalContainer] = useState<HTMLElement | null>(null);
  
  useEffect(() => {
    if (ref.current) {
      const container = ref.current.closest('.scroll') || document.body;
      setPortalContainer(container as HTMLElement);
    }
  }, [ref]);
  
  return portalContainer;
};
```

---

## 결론

Stacking Context 문제는 단순히 z-index를 높이는 것으로 해결되지 않습니다. **Portal을 사용해서 다른 stacking context로 이동**하는 것이 근본적인 해결책입니다.

**주요 성과:**
- ✅ TaskCard file popover z-index 충돌 완전 해결
- ✅ Stacking Context 구조 이해 및 활용
- ✅ Portal 패턴을 통한 UI 문제 해결 방법 습득
- ✅ 복잡한 UI에서도 예측 가능한 레이어 구조 유지

이러한 경험을 통해 복잡한 UI 컴포넌트에서 발생할 수 있는 stacking context 문제를 효과적으로 해결할 수 있게 되었습니다.
