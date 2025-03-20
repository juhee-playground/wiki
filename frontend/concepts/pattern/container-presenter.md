# Container-Presenter 패턴

## 개요

Container-Presenter 패턴은 React 컴포넌트를 두 가지 역할로 명확하게 분리하여 관리하는 디자인 패턴입니다.  
이 패턴을 사용하면 데이터 처리와 UI 렌더링을 구분하여 유지보수, 재사용성, 테스트 용이성을 크게 향상시킬 수 있습니다.

- **Container (Smart) 컴포넌트**: 데이터 페칭, 상태 관리, 비즈니스 로직 처리 등을 담당합니다.
- **Presenter (Dumb) 컴포넌트**: Container로부터 전달받은 데이터를 기반으로 순수하게 UI를 렌더링합니다.

## 패턴 설명

### Container 컴포넌트 (Smart Component)

- **역할**:
  - 외부 API 호출, 상태 관리, 비즈니스 로직 처리
  - Presenter 컴포넌트에 필요한 데이터를 가공하여 전달
- **특징**:
  - `useState`, `useEffect`, 커스텀 훅 등을 사용해 데이터를 관리
  - UI 스타일링이나 레이아웃은 Presenter에 위임

### Presenter 컴포넌트 (Dumb Component)

- **역할**:
  - Container로부터 전달받은 props를 사용해 UI를 렌더링
- **특징**:
  - 로직이 없고, 주어진 데이터에 따라 항상 동일한 결과를 렌더링
  - 재사용성이 높고, 테스트하기 용이함

## 폴더 구조 예시

아래는 Container-Presenter 패턴을 적용한 React 프로젝트의 폴더 구조 예시입니다:

```
src
├─ pages
│   ├─ CalendarPage.tsx
│   └─ ...
│
├─ components
│   ├─ calendar
│   │   ├─ container
│   │   │   ├─ CalendarContainer.tsx
│   │   │   └─ (추가 Container 컴포넌트들)
│   │   └─ ui
│   │       ├─ CalendarHeader.tsx
│   │       ├─ CalendarTimeLine.tsx
│   │       ├─ CalendarGrid.tsx
│   │       ├─ CalendarEventCard.tsx
│   │       └─ (추가 Presenter 컴포넌트들)
│   └─ ...
│
└─ App.tsx
```

## 코드 예시

### Container 컴포넌트 (CalendarContainer.tsx)

```tsx
import { useState, useEffect } from "react";
import CalendarHeader from "../ui/CalendarHeader";
import CalendarGrid from "../ui/CalendarGrid";

const CalendarContainer = () => {
  const [currentDate, setCurrentDate] = useState(new Date());
  const [events, setEvents] = useState([]);

  // API 호출이나 기타 로직 처리
  useEffect(() => {
    // 예시: 이벤트 데이터를 불러와 setEvents 호출
  }, [currentDate]);

  const handleToday = () => setCurrentDate(new Date());

  return (
    <div className="calendar-container">
      <CalendarHeader currentDate={currentDate} onToday={handleToday} />
      <CalendarGrid events={events} currentDate={currentDate} />
    </div>
  );
};

export default CalendarContainer;
```

### Presenter 컴포넌트 (CalendarHeader.tsx)

```tsx
import { format } from "date-fns";

interface CalendarHeaderProps {
  currentDate: Date;
  onToday: () => void;
}

const CalendarHeader = ({ currentDate, onToday }: CalendarHeaderProps) => {
  return (
    <header className="flex items-center justify-between p-4 bg-gray-800 text-white">
      <h1>{format(currentDate, "yyyy MMMM d")}</h1>
      <button onClick={onToday} className="bg-green-500 rounded px-4 py-2">
        Today
      </button>
    </header>
  );
};

export default CalendarHeader;
```

## 장점

- 책임 분리: 데이터 처리와 UI 렌더링이 명확히 분리되어, 각 컴포넌트가 한 가지 역할에 집중합니다.
- 재사용성: Presenter 컴포넌트는 다양한 Container에서 재사용할 수 있습니다.
- 테스트 용이성: Presenter는 순수한 렌더링 컴포넌트로, 주어진 props에 따라 항상 동일한 결과를 반환합니다.
- 유지보수성: 로직과 UI가 분리되어 있어, 기능 추가나 수정 시 영향을 최소화할 수 있습니다.

## 단점

- 작은 컴포넌트의 경우, 너무 세분화되면 파일 수가 늘어나 관리가 번거로울 수 있습니다.
- 과도한 분리는 오버엔지니어링이 될 위험이 있으므로, 프로젝트 규모와 필요에 맞게 적용해야 합니다.

## 추가 고려사항

- 상태 관리: Context API나 전역 상태 관리 라이브러리(Zustand, Redux 등)와 함께 사용하면 더욱 효율적입니다.
- 테스트: Presenter는 순수 함수 형태로 테스트하기 쉬우므로, 단위 테스트를 적극 활용할 수 있습니다.
- 유연성: Container-Presenter 패턴은 UI 라이브러리나 프레임워크에 의존하지 않으므로, 다양한 환경에 유연하게 적용할 수 있습니다.

## 결론

Container-Presenter 패턴은 React에서 데이터 처리와 UI 렌더링을 분리하여 코드의 가독성, 재사용성, 테스트 용이성을 높이는 강력한 디자인 패턴입니다.
프로젝트의 요구 사항과 팀의 개발 스타일에 맞게 적절히 적용하면 유지보수와 협업 측면에서 큰 장점을 얻을 수 있습니다.
