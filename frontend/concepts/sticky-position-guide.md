# Sticky Position 문제 해결 가이드

## 목차
1. [Sticky가 동작하지 않는 경우들](#sticky가-동작하지-않는-경우들)
2. [현재 프로젝트의 문제점](#현재-프로젝트의-문제점)
3. [해결 방안](#해결-방안)
4. [대안 구현 방법](#대안-구현-방법)
5. [성능 고려사항](#성능-고려사항)

---

## Sticky가 동작하지 않는 경우들

### 1. **부모 요소의 overflow 속성** ⚠️

**문제:**
```css
/* 부모에 overflow가 있으면 sticky 무효화 */
.parent {
  overflow: auto; /* sticky 동작 안함 */
  height: 300px;
}

.child {
  position: sticky;
  top: 0;
}
```

**왜 문제인가?**
- `overflow: auto`, `overflow: hidden`, `overflow: scroll` 등이 있으면 sticky가 동작하지 않음
- 브라우저가 새로운 스크롤 컨테이너를 생성하여 sticky의 기준점이 변경됨

### 2. **부모 요소의 height 제한** ⚠️

**문제:**
```css
/* 부모 높이가 제한되면 sticky 영역이 없어짐 */
.parent {
  height: 200px; /* sticky가 붙을 공간이 없음 */
  overflow: auto;
}

.sticky-element {
  position: sticky;
  top: 0;
}
```

**왜 문제인가?**
- sticky는 스크롤 가능한 영역이 있어야 동작
- 부모 높이가 고정되면 sticky가 붙을 공간이 부족

### 3. **Flexbox나 Grid 컨테이너** ⚠️

**문제:**
```css
/* flex/grid 컨테이너에서 sticky 예상과 다르게 동작 */
.flex-container {
  display: flex;
  flex-direction: column;
  height: 400px;
  overflow: auto;
}

.sticky-item {
  position: sticky; /* flex 컨테이너에서 예상과 다르게 동작 */
  top: 0;
}
```

**왜 문제인가?**
- Flexbox와 Grid는 새로운 formatting context를 생성
- sticky의 동작 방식이 일반적인 block 요소와 다름

### 4. **Transform이 적용된 부모** ⚠️

**문제:**
```css
/* 부모에 transform이 있으면 새로운 stacking context 생성 */
.parent {
  transform: translateZ(0); /* sticky 무효화 */
  overflow: auto;
}

.child {
  position: sticky; /* 동작 안함 */
  top: 0;
}
```

**왜 문제인가?**
- `transform`, `filter`, `perspective` 등이 있으면 새로운 stacking context 생성
- sticky의 기준점이 변경되어 예상과 다르게 동작

### 5. **조건부 overflow 적용** ⚠️

**문제:**
```typescript
// 조건에 따라 overflow가 변경되면 sticky 불안정
<div
  style={{
    overflowY: itemCount > 10 ? 'auto' : 'visible', // 조건부 overflow
  }}
>
  <thead className='sticky top-0'> {/* 불안정한 동작 */}
```

**왜 문제인가?**
- 동적으로 overflow가 변경되면 sticky의 기준점이 계속 바뀜
- 브라우저가 sticky 상태를 재계산하면서 깜빡임 현상 발생

---

## 현재 프로젝트의 문제점

### **MembersTable의 Sticky 헤더 문제** ⚠️

**문제 코드:**
```typescript
// src/domains/setting/components/MembersTable.tsx
<div
  className='border border-gray-700 rounded-lg bg-gray-800/50'
  style={{
    maxHeight: memberCount > MAX_VISIBLE_ROWS ? `${maxHeight}px` : 'auto',
    overflowY: memberCount > MAX_VISIBLE_ROWS ? 'auto' : 'visible', // 조건부 overflow
  }}
>
  <table className='w-full border-collapse'>
    <thead className='sticky top-0 bg-gray-800 border-b border-gray-600 z-[60]'>
      {/* sticky가 불안정하게 동작 */}
```

**문제점:**
1. **조건부 overflow**: 멤버 수에 따라 `overflow: visible` ↔ `overflow: auto` 변경
2. **동적 maxHeight**: 높이가 동적으로 변경되면서 sticky 기준점 변경
3. **불안정한 동작**: 멤버 수가 경계값 근처에서 sticky가 깜빡임

**사용자 경험 문제:**
- 멤버 수가 적을 때: sticky 헤더가 사라짐
- 멤버 수가 많을 때: sticky 헤더가 불안정하게 동작
- 스크롤 시: 헤더가 깜빡이거나 사라지는 현상

---

## 해결 방안

### 1. **스크롤 컨테이너 분리** ✅

**수정 전 (문제):**
```typescript
<div
  style={{
    maxHeight: memberCount > MAX_VISIBLE_ROWS ? `${maxHeight}px` : 'auto',
    overflowY: memberCount > MAX_VISIBLE_ROWS ? 'auto' : 'visible',
  }}
>
  <table>
    <thead className='sticky top-0'>
```

**수정 후 (해결):**
```typescript
<div className="table-container">
  <div className="table-wrapper">
    <table>
      <thead className='sticky top-0 bg-gray-800 z-[60]'>
```

```css
.table-container {
  height: 400px; /* 고정 높이 */
  overflow: hidden; /* overflow를 부모로 이동 */
  border: 1px solid #374151;
  border-radius: 0.5rem;
  background-color: rgba(31, 41, 55, 0.5);
}

.table-wrapper {
  height: 100%;
  overflow-y: auto; /* 스크롤을 별도 컨테이너로 */
}

.table-wrapper thead {
  position: sticky;
  top: 0;
  z-index: 60;
  background-color: #1f2937;
  border-bottom: 1px solid #4b5563;
}
