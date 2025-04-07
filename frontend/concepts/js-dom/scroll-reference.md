# 📜 스크롤 관련 속성 정리 (React + DOM 기준)

---

## ✅ 1. 기본 개념 요약

| 속성           | 의미                               | 예시               | 설명                                     |
| -------------- | ---------------------------------- | ------------------ | ---------------------------------------- |
| `scrollLeft`   | 현재 수평 스크롤 위치(px)          | `div.scrollLeft`   | 요소의 가장 왼쪽에서 얼마나 스크롤됐는지 |
| `scrollTop`    | 현재 수직 스크롤 위치(px)          | `div.scrollTop`    | 요소의 가장 위에서 얼마나 스크롤됐는지   |
| `scrollWidth`  | 요소의 **전체 스크롤 가능한 너비** | `div.scrollWidth`  | 스크롤 포함한 전체 콘텐츠 너비           |
| `scrollHeight` | 요소의 **전체 스크롤 가능한 높이** | `div.scrollHeight` | 스크롤 포함한 전체 콘텐츠 높이           |
| `clientWidth`  | 현재 보이는 콘텐츠 너비            | `div.clientWidth`  | 패딩 포함, 스크롤바 제외                 |
| `clientHeight` | 현재 보이는 콘텐츠 높이            | `div.clientHeight` | 패딩 포함, 스크롤바 제외                 |
| `offsetWidth`  | 요소의 실제 렌더링된 너비          | `div.offsetWidth`  | 패딩 + 스크롤바 포함 (border 제외)       |

---

## ✅ 2. 스크롤 위치 체크 예시

```ts
const container = gridScrollRef.current;

const { scrollLeft, scrollWidth, clientWidth } = container;

const isScrollRightEnd = scrollLeft + clientWidth >= scrollWidth - 1; // 오른쪽 끝
const isScrollLeftEnd = scrollLeft <= 0; // 왼쪽 끝
```

---

## ✅ 3. 스크롤이 생겼는지 체크

```ts
const hasHorizontalScroll = scrollWidth > clientWidth;
const hasVerticalScroll = scrollHeight > clientHeight;
```

---

## ✅ 4. 스크롤 위치 강제로 설정

```ts
container.scrollLeft = 100; // 수평 100px로 이동
container.scrollTop = 0; // 수직 맨 위로 이동
```

---

## ✅ 5. React에서 ref로 접근하는 패턴

```ts
const containerRef = useRef<HTMLDivElement>(null);

useEffect(() => {
  const container = containerRef.current;
  if (!container) return;

  console.log("스크롤 위치:", container.scrollLeft);
  console.log("전체 콘텐츠 너비:", container.scrollWidth);
  console.log("보이는 너비:", container.clientWidth);
}, []);
```

---

## ✅ 6. 헷갈릴 수 있는 비교 예시

| 비교                                      | 예시                           | 설명                          |
| ----------------------------------------- | ------------------------------ | ----------------------------- |
| `scrollWidth > clientWidth`               | 가로 스크롤 생김               | 전체 콘텐츠가 더 넓음         |
| `scrollHeight > clientHeight`             | 세로 스크롤 생김               | 전체 콘텐츠가 더 높음         |
| `offsetWidth` vs `clientWidth`            | 보더나 스크롤바 포함 여부 다름 | `offsetWidth`는 스크롤바 포함 |
| `scrollLeft = 0`                          | 스크롤 맨 왼쪽                 |                               |
| `scrollLeft + clientWidth >= scrollWidth` | 스크롤 맨 오른쪽               |                               |

---

## ✅ 7. 실제 프로젝트에서 자주 쓰는 패턴

```ts
const isAtRightEnd = scrollLeft + clientWidth >= scrollWidth - 1;
const visibleScrollRatio = scrollLeft / (scrollWidth - clientWidth);
```

---

## ✅ 8. 스크롤바 너비 계산 (선택)

```ts
const scrollbarWidth = container.offsetWidth - container.clientWidth;
```

> 💡 보통 macOS는 `scrollbarWidth = 0`,  
> Windows는 `4~17px` 정도
