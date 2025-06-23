
# 🗓 dot.daily — 투두 + 회고 서비스의 디자인 시스템 기반 개발기

> “오늘 하루를 정리하는 가장 직관적인 방법”  
> dot.daily는 단순한 투두 리스트를 넘어, 감정 기록과 회고 기능까지 포함한 **하루 단위 자기관리 플랫폼**입니다.

이번 글에서는 `dot.daily` 프로젝트가 어떻게 **Next.js + Tailwind + Storybook 기반 디자인 시스템**으로 구성되었는지, 그리고 실제 서비스 구현에 있어 어떤 폴더 구조와 기술 스택을 선택했는지를 정리합니다.  
또한 지난 글 [[Next.js + Tailwind + Storybook 기반 디자인 시스템 구축기]]와의 연결도 함께 다룹니다.

---

## 🎯 핵심 목표

- ✅ 기능별 도메인 구조를 통한 유지보수성 향상
- ✅ Zustand를 활용한 전역 상태관리 (점진적 확장 예정)
- ✅ 공통 디자인 시스템(Button, Input, Modal, Badge 등)의 체계적 관리
- ✅ Storybook으로 UI 문서화 → 컴포넌트 테스트/협업 용이
- ✅ 회고 기능과 감정 태그, 우선순위 기반 투두 정리 기능 포함

---

## 🔧 기술 스택

|구분|기술|
|---|---|
|프레임워크|Next.js (App Router 기반)|
|스타일링|Tailwind CSS (v3로 다운그레이드하여 purge 이슈 해결)|
|컴포넌트 스타일|class-variance-authority (cva) + clsx|
|문서화|Storybook + 디자인 시스템 명세화|
|상태관리|Zustand (간결한 전역 상태 관리 → 추후 persist, slice 확장)|
|애니메이션|Framer Motion|
|드래그 앤 드롭|@dnd-kit/core (계획) 또는 react-beautiful-dnd|
|배포|Vercel|

---

## 🧩 프로젝트 구조 및 확장성 설계

```bash
src/
├── app/                       # Next.js App Router 기반 페이지
│   ├── page.tsx              # / (MyDay)
│   ├── retrospect/page.tsx   # /retrospect
│   ├── archive/page.tsx      # /archive
│   └── profile/page.tsx      # /profile
│
├── components/               # 공통 UI 컴포넌트 (디자인 시스템)
│   ├── ui/                   # Button, Input, Badge, Modal 등
│   ├── layout/               # Header, Footer
│   └── feedback/             # 예: Toast, Skeleton 등 예정
│
├── features/                 # 기능별 도메인 구조
│   ├── myday/                # 투두 기능
│   └── retrospect/           # 회고 기능
│       ├── components/       # RetrospectCard 등 (예정)
│       ├── api.ts
│       ├── store.ts
│       └── utils.ts
│
├── constants/                # 우선순위/감정/이모지 등 상수 관리
├── hooks/                    # 공통 커스텀 훅
├── lib/styles/               # cva variants 등 스타일 모듈화
├── styles/                   # 글로벌 스타일 (globals.css)
├── types/                    # 전역 타입 정의
└── .storybook/               # Storybook 설정
```

---

## 🔘 디자인 시스템 예시: Button 컴포넌트

> 자세한 구현은 👉 [[Next.js + Tailwind + Storybook 기반 디자인 시스템 구축기]] 참고

- **cva 기반 선언형 스타일링**으로 유지보수 용이
- `variant`, `size`, `rounded`, `fullWidth` 등 옵션 분리
- 디자인 토큰 기반 색상 사용 (예: bg-brand-primary, text-weak)

### 예시 사용법

```tsx
<Button label="오늘 회고 작성하기" variant="primary" size="md" fullWidth />
<Button label="삭제하기" variant="danger" size="sm" />
```

### 추후 확장 계획

- ✅ `Input`, `Modal`, `Badge`, `Toast`, `Tabs` 등 추가 예정
- ✅ 각 UI는 Storybook으로 문서화

---

## 🌡 상태 관리: Zustand 적용

- 상태 slice 분리 구조로 확장 예정 (예: useMydayStore, useRetrospectStore)
- persist 미들웨어로 로컬 스토리지 연동 가능

```ts
// 예시: myday/store.ts
export const useMydayStore = create<MyDayState>((set) => ({
  tasks: [],
  addTask: (task) => set((state) => ({ tasks: [...state.tasks, task] })),
}));
```

---

## 📘 Storybook 문서화 시스템

- `/components/ui/Button.stories.tsx` 등 story 파일 작성
- 다양한 props 조합을 시각화
- 전역 스타일 `.storybook/preview.ts`에 CSS import

```ts
// .storybook/preview.ts
import '../src/styles/globals.css';
```

스토리북 구축기 : [[Next.js + Tailwind + Storybook 기반 디자인 시스템 구축기]]

---

## 🧠 기타 특징 및 차별점

- 도메인 기능/상태/API/utils를 feature 단위로 분리
- 공통 컴포넌트는 디자인 시스템 기반 (일관된 색상, 텍스트 스타일, 여백)
- 모바일 퍼스트 기반 UI (max-w-[430px] 고정폭)
- 접근성 고려: sr-only 텍스트, svg role 등
- 추후 캘린더, 감정 분석, 통계 시각화 기능도 개발 예정

---

## 🔗 GitHub 저장소

👉 [oneday-coding/DOT-DAILY](https://github.com/oneday-coding/DOT-DAILY)

---

## ✅ 마무리

dot.daily는 기능뿐만 아니라 UI 아키텍처와 협업 구조까지 고려한 프로젝트입니다. 앞으로도 다양한 UI 컴포넌트 확장과 기능 개선을 통해 **디자인 시스템 기반의 프론트엔드 실무 구조**를 꾸준히 개선해나갈 예정입니다.

다음 글에서는 `Retrospect 기능 설계`와 `Zustand 상태 흐름도 시각화`에 대해 다뤄볼 예정입니다. 🙌



- 데이터 페칭/캐싱: React Query (@tanstack/react-query) 또는 SWR 도입을 강력히 추천합니다. 이 라이브러리들은 서버에서 데이터를 가져오고, 캐싱하며, 데이터가 변경되었을 때 자동으로 화면을 갱신하는 복잡한 로직을 매우 쉽게 처리해줍니다. "RETRY" 같은 태스크 상태를 서버에 반영할 때도 훨씬 직관적인 코드를 작성할 수 있습니다.

- 클라이언트 상태 관리: Zustand나 Jotai 같은 경량 상태 관리 라이브러리를 사용하면, 여러 컴포넌트에서 공유해야 하는 상태(예: 오늘 선택된 날짜, 모달의 열림/닫힘 상태 등)를 컴포넌트 트리 깊숙이 props로 전달하지 않고도 쉽게 관리할 수 있습니다.

- 제안: API 요청을 전담하는 함수들을 별도의 파일이나 폴더(lib/api 또는 services 등)로 분리하는 것이 좋습니다. 예를 들어, getTasks(date), createTask(task), updateTask(taskId, data)와 같은 함수를 미리 만들어두면, 어느 컴포넌트에서든 가져다 쓸 수 있고 나중에 API 명세가 바뀌어도 해당 파일만 수정하면 됩니다.


---

React Query와 Zustand 라이브러리를 설치하는 것부터 시작하겠습니다. frontend 디렉토리로 이동해서 설치 명령어를 실행할게요

```bash
npm install @tanstack/react-query zustand
```


이제 발표 데모의 첫 번째 단계로, Zustand를 사용하여 날짜 상태를 관리하는 부분을 구현해 보겠습니다.

현재는 메인 페이지(app/page.tsx)에서 useState로 선택된 날짜를 관리하고, DateHeader 컴포넌트에 props로 넘겨주고 있습니다. 이 구조를 Zustand를 사용한 전역 상태 관리로 변경하여, 어느 컴포넌트에서든 날짜 상태에 쉽게 접근할 수 있도록 만들겠습니다.

먼저, 상태 관리를 위한 파일을 저장할 store 디렉토리를 src 폴더 안에 만들겠습니다.

```ts
import { create } from 'zustand';

type DateState = {
  selectedDate: Date;
  setSelectedDate: (date: Date) => void;
};

export const useDateStore = create<DateState>((set) => ({
  selectedDate: new Date(),
  setSelectedDate: (date) => set({ selectedDate: date }),
})); 
```

좋습니다! 이제 날짜를 관리하는 전역 스토어(useDateStore)가 준비되었습니다. 발표할 때 "이렇게 간단한 코드로 전역 상태를 만들 수 있습니다"라고 강조하기 좋은 포인트죠.

다음 단계는 기존에 useState를 사용하던 부분을 이 새로운 스토어를 사용하도록 리팩토링하는 것입니다. 메인 페이지(app/page.tsx)와 DateHeader 컴포넌트를 수정하겠습니다.

먼저 app/page.tsx 파일을 열어 useState를 useDateStore로 교체하겠습니다.

자, 이제 MyDayPage 컴포넌트를 수정하여 useState 대신 useDateStore를 사용하도록 변경하겠습니다.

useState 훅을 삭제하고 useDateStore에서 selectedDate와 setSelectedDate를 가져와 DateHeader에 전달하는 코드로 바꿀 겁니다. 이렇게 하면 MyDayPage는 더 이상 날짜 상태를 직접 관리하지 않게 되죠. 이 과정에서 DateHeader도 props를 받지 않도록 수정해야 합니다.

먼저 app/page.tsx 파일부터 수정하겠습니다.
```ts
'use client';

import { useState } from 'react';
import MobileLayout from '@/components/layout/MobileLayout';
import TaskItem from '@/features/myday/components/TaskItem';
import TaskGroup from '@/features/myday/components/TaskGroup';
import { Plus } from 'lucide-react';
import Fab from '@/components/ui/Fab/Fab';
import DateHeader from '@/features/myday/components/DateHeader';
import { useDateStore } from '@/store/useDateStore';

// 임시 데이터. 실제로는 API로부터 받아올 데이터입니다.
const initialTasks = {
  must: [
    { id: 'must-1', label: '6.2일 강의 완강', done: false, retryCount: 1 },
    { id: 'must-2', label: '투표하기', done: true },
    { id: 'must-3', label: '프로젝트 기획서 초안 작성', done: false },
    { id: 'must-4', label: '팀 회의 준비', done: false },
  ],
  should: [
    { id: 'should-1', label: '건전지 사기', done: false },
    { id: 'should-2', label: '회의내용 정리하기', done: true },
    { id: 'should-3', label: '운동화 세탁 맡기기', done: true },
    { id: 'should-4', label: '서점 들러서 책 찾아보기', done: false },
    { id: 'should-5', label: '점심 약속 장소 예약', done: true },
  ],
  remind: [
    { id: 'remind-1', label: '엄마한테 전화하기', done: false },
    { id: 'remind-2', label: '5km 러닝하기', done: true },
    { id: 'remind-3', label: 'OTT 구독 해지하기', done: false },
    { id: 'remind-4', label: '휴가 계획 세우기', done: false },
    { id: 'remind-5', label: '친구 생일 선물 주문', done: true },
    { id: 'remind-6', label: '영양제 챙겨먹기', done: true },
  ],
};

export default function MyDayPage() {
  const [tasks, setTasks] = useState(initialTasks);
  const { selectedDate, setSelectedDate } = useDateStore();

  // Task의 완료 상태를 토글하는 함수
  const handleToggle = (priority: 'must' | 'should' | 'remind', id: string) => {
    setTasks(prev => ({
      ...prev,
      [priority]: prev[priority].map(task =>
        task.id === id ? { ...task, done: !task.done } : task
      ),
    }));
  };

  return (
    <MobileLayout headerTitle="나의 하루">
      <div className="sticky top-0 z-10 bg-surface-base">
        <DateHeader />
      </div>

      <div className="px-4 py-6 space-y-8 pb-24">
        <TaskGroup priority="must" title="오늘 무조건">
          {tasks.must.map(task => (
            <TaskItem
              key={task.id}
              {...task}
              priority="must"
              onToggleStatus={() => handleToggle('must', task.id)}
            />
          ))}
        </TaskGroup>

        <TaskGroup priority="should" title="오늘이면 굿">
          {tasks.should.map(task => (
            <TaskItem
              key={task.id}
              {...task}
              priority="should"
              onToggleStatus={() => handleToggle('should', task.id)}
            />
          ))}
        </TaskGroup>

        <TaskGroup priority="remind" title="잊지말자">
          {tasks.remind.map(task => (
            <TaskItem
              key={task.id}
              {...task}
              priority="remind"
              onToggleStatus={() => handleToggle('remind', task.id)}
            />
          ))}
        </TaskGroup>
      </div>

      <div className="fixed bottom-[5.5rem] z-20 w-full max-w-md left-1/2 -translate-x-1/2 flex justify-end pr-4 pointer-events-none">
        <Fab aria-label="새로운 할 일 추가" className="pointer-events-auto">
          <Plus className="w-6 h-6" />
        </Fab>
      </div>
    </MobileLayout>
  );
}

```

app/page.tsx 파일 수정이 완료되었습니다. useState가 useDateStore로 바뀌었죠.

예상대로 DateHeader 컴포넌트에서 타입 에러가 발생했습니다. DateHeader가 selectedDate와 onSelectDate라는 props를 받을 준비가 되어있지 않기 때문입니다.

이제 DateHeader 컴포넌트를 수정하여 이 props를 받도록 하고, 최종적으로는 이 props 마저도 필요 없도록 내부에서 직접 Zustand 스토어를 사용하게 만들겠습니다.

먼저 DateHeader.tsx 파일을 읽어오겠습니다.


DateHeader.tsx 파일을 확인했습니다. 현재는 날짜 관련 상태를 내부에서 직접 만들지 않고, 고정된 '어제, 오늘, 내일'을 표시하고 있네요.

이제 이 컴포넌트가 useDateStore를 직접 사용하도록 리팩토링하겠습니다. 이렇게 하면 더 이상 MyDayPage에서 props로 날짜 정보를 내려줄 필요가 없어지고, DateHeader가 스스로 상태를 관리하는 독립적인 컴포넌트가 됩니다. 발표 데모에서 "Props drilling을 어떻게 해결했는가"를 보여주는 좋은 예시가 될 겁니다.

DateHeader.tsx를 다음과 같이 수정하겠습니다.

1. useDateStore를 import합니다.

2. 스토어에서 selectedDate와 setSelectedDate를 가져옵니다.

3. DateItem을 클릭하면 setSelectedDate가 호출되도록 onClick 핸들러를 추가합니다.

4. isSelected 로직을 스토어의 selectedDate와 DateItem의 date를 비교하는 방식으로 변경합니다.

5. 표시할 날짜 배열을 동적으로 생성하도록 수정합니다. (예: 선택된 날짜의 이전/현재/이후 날짜)
```ts
'use client';

import { format, isSameDay } from 'date-fns';
import { ko } from 'date-fns/locale';
import { clsx } from 'clsx';
import { useDateStore } from '@/store/useDateStore';

// 각 날짜 아이템을 렌더링하는 내부 컴포넌트
function DateItem({
  date,
  isSelected,
  onSelect,
}: {
  date: Date;
  isSelected: boolean;
  onSelect: () => void;
}) {
  const isToday = isSameDay(date, new Date());

  return (
    <div
      onClick={onSelect}
      className={clsx(
        'flex flex-col items-center justify-center w-16 h-16 rounded-lg transition-all cursor-pointer',
        isSelected
          ? 'bg-surface-card shadow-sm' // 선택되면 흰색 배경 + 그림자
          : 'bg-transparent' // 선택되지 않으면 투명
      )}
    >
      <span
        className={clsx(
          'text-xl font-bold',
          isSelected ? 'text-text-strong' : 'text-text-weak' // 선택 여부에 따라 텍스트 색상 변경
        )}
      >
        {format(date, 'd')}
      </span>
      <span
        className={clsx(
          'text-xs font-medium',
          isSelected ? 'text-text-strong' : 'text-text-weak'
        )}
      >
        {isToday ? '오늘' : format(date, 'E', { locale: ko })}
      </span>
    </div>
  );
}

export default function DateHeader() {
  const { selectedDate, setSelectedDate } = useDateStore();

  const getDates = () => {
    const prevDate = new Date(selectedDate);
    prevDate.setDate(selectedDate.getDate() - 1);

    const nextDate = new Date(selectedDate);
    nextDate.setDate(selectedDate.getDate() + 1);

    return [prevDate, selectedDate, nextDate];
  };

  const dates = getDates();

  return (
    // 전체 배경은 앱의 기본 배경색(surface-base)과 동일하게 맞춥니다.
    <div className="flex items-center justify-around py-3 px-2 bg-surface-base">
      {dates.map((date, index) => (
        <DateItem
          key={index}
          date={date}
          isSelected={isSameDay(date, selectedDate)}
          onSelect={() => setSelectedDate(date)}
        />
      ))}
    </div>
  );
}


```

좋습니다! 이제 Zustand를 이용한 클라이언트 상태 리팩토링이 모두 끝났습니다.

발표 데모 1단계 요약:

1. useState로 관리되던 날짜 상태를 Zustand 스토어로 분리했습니다 (useDateStore.ts).

2. DateHeader 컴포넌트가 props를 받는 대신, 직접 useDateStore에 접근하여 상태를 읽고 업데이트하도록 수정했습니다.

3. MyDayPage는 더 이상 DateHeader에게 상태를 넘겨줄 필요가 없어졌고, 코드가 단순하고 명확해졌습니다. (props drilling 해결!)

이것만으로도 "왜 전역 상태관리가 필요한가"에 대한 훌륭한 데모가 됩니다.

이제, 이어서 발표의 하이라이트가 될 React Query 적용으로 넘어가 볼까요? 메인 페이지의 임시 할 일 목록 데이터를 React Query를 통해 관리하도록 만들어 보겠습니다. 준비되셨나요?

---

네, 좋습니다! 이제 발표의 두 번째 하이라이트, 서버 상태를 지배하는 React Query를 도입해 보겠습니다.

React Query를 사용하려면, 앱 전체를 QueryClientProvider라는 컴포넌트로 감싸주어야 합니다. Next.js App Router 환경에서는 이 Provider를 클라이언트 컴포넌트('use client')로 만들어야 합니다.

먼저, 이 Provider를 위한 파일을 생성하겠습니다. frontend/src/app/providers.tsx 라는 파일을 새로 만들고, React Query 클라이언트를 설정하는 코드를 추가할게요.
```ts
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';

export default function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}

```

Provider 컴포넌트가 준비되었습니다.

이제 이 Providers 컴포넌트를 앱의 최상위 레이아웃인 layout.tsx에 적용해서, 앱 전체에서 React Query의 기능을 사용할 수 있도록 만들겠습니다.

React Query를 "데이터 관리 총책임자"라고 상상해 보세요. 이 책임자가 앱 전체에서 일하려면, 먼저 사무실을 차려줘야 합니다.

1. providers.tsx 파일 생성: React Query라는 "총책임자의 사무실"을 만들었습니다. 이 사무실의 이름이 바로 QueryClientProvider입니다.

2. layout.tsx 파일 수정: 만든 사무실(QueryClientProvider)을 우리 앱 전체를 감싸도록 배치했습니다. 이렇게 함으로써, 이제 앱 내부의 어떤 컴포넌트라도 이 총책임자에게 "데이터 좀 가져다줘!"라고 요청할 수 있게 된 것입니다.

이 설정은 딱 한 번만 하면 됩니다. 이제 우리는 앱 어디서든 React Query의 강력한 기능을 마음껏 사용할 수 있습니다.

```ts
import type { Metadata } from 'next';
import localFont from 'next/font/local';
import './globals.css';
import Providers from './providers';

const pretendard = localFont({
  src: '../fonts/Pretendard/PretendardVariable.woff2',
  variable: '--font-pretendard',
  display: 'swap',
});

const kkonghae = localFont({
  src: '../fonts/kkonghae.woff2',
  variable: '--font-kkonghae',
  display: 'swap',
});


export const metadata: Metadata = {
  title: 'DOT.DAILY',
  description: '투두 + 회고 기록 앱',
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="ko">
      <body className={`${pretendard.variable} ${kkonghae.variable} font-sans`}>
        <Providers>
          <div className="w-full max-w-md mx-auto min-h-screen bg-surface-base shadow-lg">
            {children}
          </div>
        </Providers>
      </body>
    </html>
  );
}

```


### React Query는 대체 뭐하는 역할인가요?

React Query는 "서버 상태(Server State)"를 아주 쉽고 강력하게 관리해주는 라이브러리입니다.

발표 때 설명하셨던 "클라이언트 상태"와 "서버 상태"를 다시 떠올려보세요.

- 클라이언트 상태: UI 상태 (예: 모달 열림/닫힘). Zustand로 관리했죠.

- 서버 상태: 서버 API를 통해 가져와야 하는 데이터 (예: 오늘의 할 일 목록, 사용자 정보 등).

바로 이 서버 상태를 다루는 게 정말 까다롭습니다. 기존 방식(useState + useEffect)의 문제점은 이렇습니다.

- 데이터를 담을 useState

- 로딩 상태를 알려줄 useState (예: const [isLoading, setIsLoading] = useState(true))

- 에러 상태를 알려줄 useState (예: const [error, setError] = useState(null))

- API를 호출할 useEffect

이 모든 걸 직접 만들어야 했습니다. 하지만 React Query를 쓰면 이 모든 걸 단 한 줄로 해결해 줍니다.

const { data, isLoading, isError } = useQuery(...)

React Query가 알아서 처리해주는 핵심 역할은 다음과 같습니다.

1. 캐싱 (Caching): 한 번 불러온 데이터는 기억(캐시)해 둡니다. 그래서 사용자가 잠시 다른 화면에 갔다가 돌아와도, 또 API를 요청하는 게 아니라 기억해둔 데이터를 바로 보여줍니다. (앱이 훨씬 빨라집니다!)

2. 로딩 및 에러 상태 자동 관리: isLoading, isError 같은 상태를 우리가 만들 필요 없이 알아서 제공해 줍니다.

3. 자동 데이터 갱신: 사용자가 다른 브라우저 탭에 갔다가 우리 앱으로 다시 돌아오면, 데이터가 그 사이에 바뀌었을 수도 있잖아요? React Query는 이걸 감지해서 자동으로 데이터를 새로고침 해줍니다. 이런 고급 기능을 우리가 직접 만들려면 정말 복잡합니다.

4. 코드 간소화: 컴포넌트에서 데이터 로딩과 관련된 복잡한 코드가 사라지고, UI를 보여주는 데만 집중할 수 있게 됩니다.

결론적으로, React Query는 우리가 서버 데이터를 가져오고 관리하는 데 필요한 모든 귀찮고 반복적인 작업을 대신 해주는 매우 유능한 데이터 관리 전문 비서라고 할 수 있습니다.

이제 이 비서에게 첫 번째 임무를 줘 볼까요? app/page.tsx에 있는 임시 tasks 데이터를 useQuery를 사용해서 가져오도록 바꿔보겠습니다.

좋습니다. 이제 React Query를 실제로 사용해서 MyDayPage의 할 일 목록을 가져오는 코드를 작성해 보겠습니다.

발표 데모를 위해, 실제 서버 API를 호출하는 것처럼 보이는 가짜 API 함수를 먼저 만들겠습니다. 이 함수는 1초 후에 임시 데이터를 반환하도록 해서, 로딩 상태를 명확하게 보여줄 수 있게 할 겁니다.

API 관련 함수들을 보관할 frontend/src/lib/api 디렉토리를 먼저 생성할게요. 체계적인 프로젝트 구조를 위한 좋은 습관입니다.

```bash
mkdir -p frontend/src/lib/api
```
디렉토리가 생성되었습니다.

이제 이 디렉토리 안에 할 일(Tasks) 데이터를 가져오는 가짜 API 함수가 담긴 tasks.ts 파일을 만들겠습니다
```ts
// 임시 데이터. 실제로는 API로부터 받아올 데이터입니다.
const initialTasks = {
    must: [
      { id: 'must-1', label: '6.2일 강의 완강', done: false, retryCount: 1 },
      { id: 'must-2', label: '투표하기', done: true },
      { id: 'must-3', label: '프로젝트 기획서 초안 작성', done: false },
      { id: 'must-4', label: '팀 회의 준비', done: false },
    ],
    should: [
      { id: 'should-1', label: '건지 사기', done: false },
      { id: 'should-2', label: '회의내용 정리하기', done: true },
      { id: 'should-3', label: '운동화 세탁 맡기기', done: true },
      { id: 'should-4', label: '서점 들러서 책 찾아보기', done: false },
      { id: 'should-5', label: '점심 약속 장소 예약', done: true },
    ],
    remind: [
      { id: 'remind-1', label: '엄마한테 전화하기', done: false },
      { id: 'remind-2', label: '5km 러닝하기', done: true },
      { id: 'remind-3', label: 'OTT 구독 해지하기', done: false },
      { id: 'remind-4', label: '휴가 계획 세우기', done: false },
      { id: 'remind-5', label: '친구 생일 선물 주문', done: true },
      { id: 'remind-6', label: '영양제 챙겨먹기', done: true },
    ],
  };

  export type TaskPriority = 'must' | 'should' | 'remind';

  export interface Task {
      id: string;
      label: string;
      done: boolean;
      retryCount?: number;
  }
  
  export interface Tasks {
    must: Task[];
    should: Task[];
    remind: Task[];
  }

/**
 * 특정 날짜의 할 일 목록을 가져오는 가짜 API 함수.
 * 발표 데모를 위해 1초의 딜레이를 시뮬레이션합니다.
 * @param date - 할 일을 가져올 날짜 (현재는 사용되지 않음)
 */
export const getTasksByDate = async (date: Date): Promise<Tasks> => {
    console.log(`${date.toLocaleDateString()}의 할 일 데이터를 "서버"에서 가져오는 중...`);
    
    // API 호출을 시뮬레이션하기 위해 1초 대기
    await new Promise(resolve => setTimeout(resolve, 1000));

    // 실제로는 date를 사용해 해당 날짜의 데이터를 필터링해야 합니다.
    // 지금은 항상 같은 데이터를 반환합니다.
    return initialTasks;
} 
```

파일이 생성되었습니다. 이제 가짜 API(getTasksByDate)가 준비됐으니, 드디어 app/page.tsx를 수정할 차례입니다.

다음과 같이 변경할 겁니다.

1. 기존에 있던 initialTasks 데이터와 useState를 삭제합니다.

2. @tanstack/react-query에서 useQuery 훅을 가져옵니다.

3. useQuery를 사용해서 방금 만든 getTasksByDate 함수를 호출합니다.

4. useQuery가 반환하는 data, isLoading 상태를 사용해서 UI를 렌더링합니다. 로딩 중일 때는 간단한 텍스트를 보여줄게요.

먼저 app/page.tsx 파일을 읽어오겠습니다.


좋습니다. 이제 MyDayPage 컴포넌트를 useQuery를 사용하도록 대대적으로 수정하겠습니다.

handleToggle 함수는 잠시 복잡해지므로, 우선 주석 처리하여 토글 기능은 잠시 비활성화해두겠습니다. 데이터 업데이트(Mutation)는 useQuery 설명 후에 다음 단계로 다루는 것이 발표 흐름에 더 좋습니다.

```ts
'use client';

import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import MobileLayout from '@/components/layout/MobileLayout';
import TaskItem from '@/features/myday/components/TaskItem';
import TaskGroup from '@/features/myday/components/TaskGroup';
import { Plus } from 'lucide-react';
import Fab from '@/components/ui/Fab/Fab';
import DateHeader from '@/features/myday/components/DateHeader';
import { useDateStore } from '@/store/useDateStore';
import { getTasksByDate, TaskPriority } from '@/lib/api/tasks';


export default function MyDayPage() {
  const { selectedDate } = useDateStore();

  const { data: tasks, isLoading, isError, error } = useQuery({
    queryKey: ['tasks', selectedDate.toISOString().split('T')[0]],
    queryFn: () => getTasksByDate(selectedDate),
  });

  // Task의 완료 상태를 토글하는 함수 (Mutation으로 개선 예정)
  // const handleToggle = (priority: TaskPriority, id: string) => {
  //   setTasks(prev => ({
  //     ...prev,
  //     [priority]: prev[priority].map(task =>
  //       task.id === id ? { ...task, done: !task.done } : task
  //     ),
  //   }));
  // };

  return (
    <MobileLayout headerTitle="나의 하루">
      <div className="sticky top-0 z-10 bg-surface-base">
        <DateHeader />
      </div>

      <div className="px-4 py-6 space-y-8 pb-24">
        {isLoading && (
          <div className="text-center py-10">
            <p className="text-text-weak">할 일 목록을 불러오는 중...</p>
          </div>
        )}
        {isError && (
          <div className="text-center py-10">
            <p className="text-danger-solid">
              오류가 발생했습니다: {error.message}
            </p>
          </div>
        )}
        {tasks && (
          <>
            <TaskGroup priority="must" title="오늘 무조건">
              {tasks.must.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="must"
                  onToggleStatus={() => {
                    /* handleToggle('must', task.id) */
                  }}
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="should" title="오늘이면 굿">
              {tasks.should.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="should"
                  onToggleStatus={() => {
                    /* handleToggle('should', task.id) */
                  }}
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="remind" title="잊지말자">
              {tasks.remind.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="remind"
                  onToggleStatus={() => {
                    /* handleToggle('remind', task.id) */
                  }}
                />
              ))}
            </TaskGroup>
          </>
        )}
      </div>

      <div className="fixed bottom-[5.5rem] z-20 w-full max-w-md left-1/2 -translate-x-1/2 flex justify-end pr-4 pointer-events-none">
        <Fab aria-label="새로운 할 일 추가" className="pointer-events-auto">
          <Plus className="w-6 h-6" />
        </Fab>
      </div>
    </MobileLayout>
  );
}

```

모두 완료되었습니다! 이제 MyDayPage는 더 이상 useState로 할 일 데이터를 관리하지 않고, React Query의 useQuery를 통해 서버(지금은 가짜지만)로부터 비동기적으로 데이터를 받아옵니다.

한번 실행해서 확인해보세요! 페이지를 새로고침하면 "할 일 목록을 불러오는 중..." 이라는 메시지가 1초간 나타났다가, 데이터가 표시되는 것을 볼 수 있을 겁니다.

그리고 DateHeader에서 다른 날짜를 클릭해보세요. 콘솔 창에 우리가 getTasksByDate 함수에 넣어둔 console.log 메시지가 뜨면서, 해당 날짜의 데이터를 "다시" 불러오는 것을 확인할 수 있습니다.


### 발표 데모 2단계 요약 및 포인트

1. 가짜 API 모듈화: 실제 API를 호출하는 것처럼 보이는 getTasksByDate 함수를 별도 파일로 만들었습니다.

2. useState -> useQuery: MyDayPage에서 useState와 useEffect를 완전히 제거하고, 단 하나의 useQuery 훅으로 교체했습니다.

- queryKey: ['tasks', selectedDate] 이 부분이 핵심입니다. React Query는 이 key를 보고 데이터의 "이름표"를 붙입니다. selectedDate가 바뀌면 이름표가 바뀌므로, 자동으로 새로운 데이터를 불러옵니다. "캐시 키 전략"에 대해 설명하기 좋은 부분입니다.

1. 로딩/에러 상태 처리: isLoading, isError 같은 상태를 직접 만들 필요 없이 useQuery가 반환해주는 값을 사용해 아주 쉽게 로딩/에러 UI를 구현했습니다.

여기까지가 "데이터 가져오기(Query)"에 대한 데모입니다. 이것만으로도 React Query의 강력함을 충분히 보여줄 수 있습니다.

다음 단계는 "데이터 수정하기(Mutation)" 입니다. 주석 처리했던 handleToggle 함수를 useMutation이라는 훅을 사용해서 다시 구현해 볼까요? 아니면 지금까지의 내용으로도 발표 준비는 충분하다고 생각하시나요?


---

발표의 마지막 퍼즐 조각인 데이터 수정(Mutation) 부분을 완성해 보겠습니다.

useQuery가 데이터를 읽는(Read) 역할이었다면, useMutation은 데이터를 쓰는(Write) 역할, 즉 생성(Create), 수정(Update), 삭제(Delete)를 담당합니다.

우리가 할 일의 완료/미완료 상태를 토글하는 것은 '수정(Update)'에 해당하죠. useMutation을 사용하면 이 과정이 놀랍도록 깔끔해집니다.

1. 서버에 "이 태스크 상태 좀 바꿔줘"라고 요청하는 함수를 실행시킵니다.

2. 요청이 성공하면, 가장 중요한 부분: React Query에게 "방금 데이터가 바뀌었으니, 화면에 보여주는 데이터도 새로고침 해줘!" 라고 알려줍니다. (이걸 Query Invalidation 이라고 부릅니다)

이 마법 같은 과정을 구현하기 위해, 먼저 '태스크 상태를 수정하는' 가짜 API 함수를 lib/api/tasks.ts 파일에 추가하겠습니다.

```ts
// 임시 데이터가 수정될 수 있도록 let으로 변경하고, 외부에서 직접 접근하지 못하도록 숨깁니다.
let tasksData = {
    must: [
      { id: 'must-1', label: '6.2일 강의 완강', done: false, retryCount: 1 },
      { id: 'must-2', label: '투표하기', done: true },
      { id: 'must-3', label: '프로젝트 기획서 초안 작성', done: false },
      { id: 'must-4', label: '팀 회의 준비', done: false },
    ],
    should: [
      { id: 'should-1', label: '건전지 사기', done: false },
      { id: 'should-2', label: '회의내용 정리하기', done: true },
      { id: 'should-3', label: '운동화 세탁 맡기기', done: true },
      { id: 'should-4', label: '서점 들러서 책 찾아보기', done: false },
      { id: 'should-5', label: '점심 약속 장소 예약', done: true },
    ],
    remind: [
      { id: 'remind-1', label: '엄마한테 전화하기', done: false },
      { id: 'remind-2', label: '5km 러닝하기', done: true },
      { id: 'remind-3', label: 'OTT 구독 해지하기', done: false },
      { id: 'remind-4', label: '휴가 계획 세우기', done: false },
      { id: 'remind-5', label: '친구 생일 선물 주문', done: true },
      { id: 'remind-6', label: '영양제 챙겨먹기', done: true },
    ],
  };

  export type TaskPriority = 'must' | 'should' | 'remind';

  export interface Task {
      id: string;
      label: string;
      done: boolean;
      retryCount?: number;
  }
  
  export interface Tasks {
    must: Task[];
    should: Task[];
    remind: Task[];
  }

/**
 * 특정 날짜의 할 일 목록을 가져오는 가짜 API 함수.
 * 발표 데모를 위해 1초의 딜레이를 시뮬레이션합니다.
 * @param date - 할 일을 가져올 날짜 (현재는 사용되지 않음)
 */
export const getTasksByDate = async (date: Date): Promise<Tasks> => {
    console.log(`${date.toLocaleDateString()}의 할 일 데이터를 "서버"에서 가져오는 중...`);
    
    // API 호출을 시뮬레이션하기 위해 1초 대기
    await new Promise(resolve => setTimeout(resolve, 1000));

    // 실제로는 date를 사용해 해당 날짜의 데이터를 필터링해야 합니다.
    // 객체의 복사본을 반환하여 실제 API처럼 동작하게 만듭니다.
    return JSON.parse(JSON.stringify(tasksData));
}

/**
 * 할 일의 완료 상태를 업데이트하는 가짜 API 함수.
 * @param {object} params
 * @param {TaskPriority} params.priority - 업데이트할 태스크의 우선순위
 * @param {string} params.id - 업데이트할 태스크의 ID
 * @param {boolean} params.done - 새로운 완료 상태
 */
export const updateTaskStatus = async ({
  priority,
  id,
  done,
}: {
  priority: TaskPriority;
  id: string;
  done: boolean;
}): Promise<{ success: boolean }> => {
  console.log(
    `"서버"에서 ${priority} 우선순위의 태스크(ID: ${id}) 상태를 ${done}으로 업데이트하는 중...`
  );
  // API 호출 시뮬레이션 (0.5초)
  await new Promise(resolve => setTimeout(resolve, 500));

  // 메모리 내 데이터 직접 수정
  const taskList = tasksData[priority];
  const taskIndex = taskList.findIndex(task => task.id === id);

  if (taskIndex !== -1) {
    tasksData[priority][taskIndex].done = done;
    console.log('업데이트 성공!');
    return { success: true };
  } else {
    console.error('해당 태스크를 찾을 수 없습니다.');
    // 실제 앱에서는 에러 처리를 해야 합니다.
    return { success: false };
  }
}; 
```

상태 업데이트를 위한 가짜 API 함수(updateTaskStatus)가 준비되었습니다.

이제 마지막 단계입니다. MyDayPage 컴포넌트에서 useMutation 훅을 사용해서 이 API 함수를 호출하고, 성공했을 때 데이터를 자동으로 새로고침하도록 만들겠습니다.

변경 포인트:

1. useMutation 훅을 설정합니다.

2. mutationFn에는 방금 만든 updateTaskStatus를 연결합니다.

3. onSuccess 콜백에서 queryClient.invalidateQueries를 호출하여 데이터 새로고침을 트리거합니다. (이 부분이 핵심!)

4. 주석 처리했던 handleToggle 함수를 useMutation이 반환하는 mutate 함수를 호출하도록 다시 구현합니다.

이 수정을 통해 MyDayPage 코드를 완성시키겠습니다.

```ts
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import MobileLayout from '@/components/layout/MobileLayout';
import TaskItem from '@/features/myday/components/TaskItem';
import TaskGroup from '@/features/myday/components/TaskGroup';
import { Plus } from 'lucide-react';
import Fab from '@/components/ui/Fab/Fab';
import DateHeader from '@/features/myday/components/DateHeader';
import { useDateStore } from '@/store/useDateStore';
import { getTasksByDate, updateTaskStatus, TaskPriority } from '@/lib/api/tasks';

export default function MyDayPage() {
  const { selectedDate } = useDateStore();
  const queryClient = useQueryClient();

  const { data: tasks, isLoading, isError, error } = useQuery({
    queryKey: ['tasks', selectedDate.toISOString().split('T')[0]],
    queryFn: () => getTasksByDate(selectedDate),
  });

  const { mutate: toggleTaskStatus, isPending: isToggling } = useMutation({
    mutationFn: updateTaskStatus,
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: ['tasks', selectedDate.toISOString().split('T')[0]],
      });
    },
  });

  const handleToggle = (priority: TaskPriority, id: string, done: boolean) => {
    toggleTaskStatus({ priority, id, done: !done });
  };

  return (
    <MobileLayout headerTitle="나의 하루">
      <div className="sticky top-0 z-10 bg-surface-base">
        <DateHeader />
      </div>

      <div className="px-4 py-6 space-y-8 pb-24">
        {isLoading && (
          <div className="text-center py-10">
            <p className="text-text-weak">할 일 목록을 불러오는 중...</p>
          </div>
        )}
        {isError && (
          <div className="text-center py-10">
            <p className="text-danger-solid">
              오류가 발생했습니다: {error.message}
            </p>
          </div>
        )}
        {tasks && (
          <>
            <TaskGroup priority="must" title="오늘 무조건">
              {tasks.must.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="must"
                  onToggleStatus={() => handleToggle('must', task.id, task.done)}
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="should" title="오늘이면 굿">
              {tasks.should.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="should"
                  onToggleStatus={() =>
                    handleToggle('should', task.id, task.done)
                  }
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="remind" title="잊지말자">
              {tasks.remind.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="remind"
                  onToggleStatus={() =>
                    handleToggle('remind', task.id, task.done)
                  }
                />
              ))}
            </TaskGroup>
          </>
        )}
      </div>

      <div className="fixed bottom-[5.5rem] z-20 w-full max-w-md left-1/2 -translate-x-1/2 flex justify-end pr-4 pointer-events-none">
        <Fab aria-label="새로운 할 일 추가" className="pointer-events-auto">
          <Plus className="w-6 h-6" />
        </Fab>
      </div>
    </MobileLayout>
  );
}

```

드디어 완성되었습니다!

이제 앱을 실행하고 할 일을 토글해보세요.

1. 체크박스를 클릭하면, 개발자 도구의 콘솔 창에 "서버에서 ... 업데이트하는 중..." 메시지가 뜹니다.

2. 0.5초 후에 "업데이트 성공!" 메시지가 뜨고,

3. 곧이어 "할 일 데이터를 '서버'에서 가져오는 중..." 메시지가 다시 나타납니다.

4. 그리고 UI가 최신 상태로 새로고침 됩니다.

이 모든 과정이 useState를 직접 조작하지 않고, 단지 "상태를 업데이트 해줘" 라는 요청과 "성공하면 데이터를 무효화 해줘" 라는 선언적인 코드로 이루어졌습니다.

### 전체 데모 시나리오 요약

이제 발표를 위한 완벽한 데모 시나리오가 준비되었습니다.

1. 시작: useState로 모든 것을 관리하던 초기 상태의 문제점을 지적합니다.

2. Zustand 도입:

- useDateStore를 만들어 클라이언트 상태(선택된 날짜)를 분리합니다.

- DateHeader가 props 없이 직접 스토어에 접근해 props drilling을 해결하는 모습을 보여줍니다.

1. React Query 도입 (Query):

- useState로 관리하던 할 일 목록을 useQuery로 교체합니다.

- 가짜 API를 통해 isLoading 상태를 처리하는 모습을 보여줍니다.

- 날짜를 바꿔 클릭하면 queryKey가 변경되어 데이터가 자동으로 리프레시되는 캐싱 전략을 설명합니다.

1. React Query 도입 (Mutation):

- useMutation으로 할 일 상태 업데이트 기능을 구현합니다.

- onSuccess와 invalidateQueries를 통해 데이터 수정 후 UI가 자동으로 동기화되는 '마법'을 보여줍니다.

이 흐름대로 발표하시면, 왜 현대 프론트엔드 개발에서 Zustand와 React Query 같은 도구가 필수적인지를 청중에게 매우 설득력 있게 전달하실 수 있을 겁니다.

발표 준비하시면서 더 궁금한 점이나 추가로 필요한 부분이 있으면 언제든지 말씀해주세요! 응원하겠습니다

---


### Q2. 어떻게 해결할 수 있나요? → "낙관적 업데이트 (Optimistic Update)"

이 문제를 해결하는 세련된 방법이 바로 React Query가 제공하는 "낙관적 업데이트(Optimistic Update)" 입니다.

개념: 사용자가 액션을 취하면(예: 체크박스 클릭), "일단 서버가 성공할 거라고 '낙관적으로' 믿고 UI를 즉시 업데이트" 해주는 기술입니다. 그리고 실제 서버 요청은 백그라운드에서 조용히 보내는 거죠.

동작 방식:

1. 사용자: 체크박스 클릭!

2. React Query (onMutate):

- 서버에 요청을 보내기 전에, UI를 먼저 즉시 업데이트합니다. (사용자는 0.1초 만에 체크 표시가 바뀌는 것을 봅니다)

- 이전 데이터를 잠시 저장해 둡니다 (혹시 모를 실패에 대비).

1. 백그라운드: updateTaskStatus API를 서버에 조용히 요청합니다.

2. 결과:

- 성공 시: 아무 일도 일어나지 않습니다. UI는 이미 올바르게 바뀌어 있으니까요. (조용히 서버 데이터와 동기화만 한번 더 합니다)

- 실패 시 (매우 중요):

- 백그라운드에서 에러가 발생하면, React Query가 자동으로 UI를 원래 상태로 되돌립니다 (롤백).

- 그리고 "업데이트에 실패했습니다" 같은 에러 메시지를 보여줍니다.

결과적으로, 사용자는 거의 항상 즉각적인 반응성을 경험하게 되어 앱이 매우 빠르다고 느끼게 됩니다.

이 '낙관적 업데이트'를 직접 구현해서, 앱의 반응 속도를 마법처럼 빠르게 만들어 볼까요? useMutation의 고급 옵션 몇 개만 추가하면 됩니다.

알겠습니다! '낙관적 업데이트'를 적용하여 사용자 경험을 극적으로 끌어올려 보겠습니다. 콘솔 로그는 그대로 유지하여 발표 때 단계별 흐름을 명확히 보여줄 수 있도록 할게요.

MyDayPage.tsx의 useMutation 훅을 다음과 같이 대대적으로 업그레이드할 겁니다.

- onSuccess 대신, 더 정교한 제어를 위해 onMutate, onError, onSettled 옵션을 사용할 겁니다.

- onMutate: API 요청을 보내기 전에 실행됩니다. 여기서 "UI 즉시 업데이트" 마법이 일어납니다.

- onError: API 요청이 실패했을 때 실행됩니다. "UI 롤백"을 담당합니다.

- onSettled: 요청이 성공하든 실패하든, 마지막에 항상 실행됩니다. 여기서 "최종 데이터 동기화"를 처리합니다.

MyDayPage.tsx 파일을 수정하여 이 로직을 구현하겠습니다. 코드가 조금 길어지지만, 각 단계가 명확한 역할을 가지고 있으니 함께 보시죠.

```
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import MobileLayout from '@/components/layout/MobileLayout';
import TaskItem from '@/features/myday/components/TaskItem';
import TaskGroup from '@/features/myday/components/TaskGroup';
import { Plus } from 'lucide-react';
import Fab from '@/components/ui/Fab/Fab';
import DateHeader from '@/features/myday/components/DateHeader';
import { useDateStore } from '@/store/useDateStore';
import {
  getTasksByDate,
  updateTaskStatus,
  TaskPriority,
  Tasks,
} from '@/lib/api/tasks';

export default function MyDayPage() {
  const { selectedDate } = useDateStore();
  const queryClient = useQueryClient();

  const queryKey = ['tasks', selectedDate.toISOString().split('T')[0]];

  const { data: tasks, isLoading, isError, error } = useQuery({
    queryKey,
    queryFn: () => getTasksByDate(selectedDate),
  });

  const { mutate: toggleTaskStatus } = useMutation({
    mutationFn: updateTaskStatus,
    onMutate: async (variables) => {
      console.log('--- 낙관적 업데이트 시작 ---');
      console.log('1. 이전 쿼리 취소');
      await queryClient.cancelQueries({ queryKey });

      const previousTasks = queryClient.getQueryData<Tasks>(queryKey);
      console.log('2. UI 즉시 업데이트 (setQueryData)');

      if (previousTasks) {
        const newTasks = JSON.parse(JSON.stringify(previousTasks));
        const taskList: any[] = newTasks[variables.priority];
        const taskIndex = taskList.findIndex(task => task.id === variables.id);
        if (taskIndex !== -1) {
          taskList[taskIndex].done = variables.done;
        }
        queryClient.setQueryData<Tasks>(queryKey, newTasks);
      }
      
      console.log('3. 이전 데이터 저장 (롤백 대비)');
      return { previousTasks };
    },
    onError: (err, variables, context) => {
      console.error('--- 🚨 낙관적 업데이트 실패! 롤백 실행 ---', err);
      if (context?.previousTasks) {
        queryClient.setQueryData(queryKey, context.previousTasks);
      }
    },
    onSettled: () => {
      console.log('4. 최종 데이터 동기화 (invalidateQueries)');
      console.log('--- 낙관적 업데이트 종료 ---');
      queryClient.invalidateQueries({ queryKey });
    },
  });

  const handleToggle = (priority: TaskPriority, id: string, done: boolean) => {
    toggleTaskStatus({ priority, id, done: !done });
  };

  return (
    <MobileLayout headerTitle="나의 하루">
      <div className="sticky top-0 z-10 bg-surface-base">
        <DateHeader />
      </div>

      <div className="px-4 py-6 space-y-8 pb-24">
        {/* 로딩 및 에러 UI는 동일 */}
        {isLoading && (
          <div className="text-center py-10">
            <p className="text-text-weak">할 일 목록을 불러오는 중...</p>
          </div>
        )}
        {isError && (
          <div className="text-center py-10">
            <p className="text-danger-solid">
              오류가 발생했습니다: {error.message}
            </p>
          </div>
        )}
        {tasks && (
          <>
            <TaskGroup priority="must" title="오늘 무조건">
              {tasks.must.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="must"
                  onToggleStatus={() => handleToggle('must', task.id, task.done)}
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="should" title="오늘이면 굿">
              {tasks.should.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="should"
                  onToggleStatus={() =>
                    handleToggle('should', task.id, task.done)
                  }
                />
              ))}
            </TaskGroup>

            <TaskGroup priority="remind" title="잊지말자">
              {tasks.remind.map(task => (
                <TaskItem
                  key={task.id}
                  {...task}
                  priority="remind"
                  onToggleStatus={() =>
                    handleToggle('remind', task.id, task.done)
                  }
                />
              ))}
            </TaskGroup>
          </>
        )}
      </div>

      <div className="fixed bottom-[5.5rem] z-20 w-full max-w-md left-1/2 -translate-x-1/2 flex justify-end pr-4 pointer-events-none">
        <Fab aria-label="새로운 할 일 추가" className="pointer-events-auto">
          <Plus className="w-6 h-6" />
        </Fab>
      </div>
    </MobileLayout>
  );
}

```

적용 완료되었습니다!

이제 다시 한번 체크박스를 클릭해 보세요.

이전과 완전히 다른 경험을 하실 수 있습니다.

- 화면: 체크박스를 클릭하는 즉시 체크 상태가 바뀝니다. 더 이상 1.5초를 기다릴 필요가 없습니다! 앱이 매우 빠르고 반응성이 좋게 느껴집니다.

- 콘솔: 화면이 바뀐 후에, 백그라운드에서 우리가 추가한 콘솔 로그들이 순서대로 찍히는 것을 볼 수 있습니다.

1. --- 낙관적 업데이트 시작 ---

2. 1. 이전 쿼리 취소

3. 2. UI 즉시 업데이트 (setQueryData)

4. 3. 이전 데이터 저장 (롤백 대비)

5. (0.5초 후) "서버"에서 ... 업데이트하는 중...

6. 업데이트 성공!

7. 4. 최종 데이터 동기화 (invalidateQueries)

8. --- 낙관적 업데이트 종료 ---

9. "서버"에서 ... 가져오는 중...

이것으로 useState부터 시작해 useQuery, useMutation, 그리고 최종적으로 Optimistic Update까지 이어지는 완벽한 발표 데모가 완성되었습니다.

사용자 경험을 극적으로 개선하는 이 과정 전체가 아주 좋은 발표 주제가 될 겁니다.

혹시 더 다듬고 싶은 부분이나 궁금한 점이 있으신가요?

---

아주 좋은 질문입니다! 결론부터 말씀드리면, 이 경우에는 낙관적 업데이트로 해결할 수 없습니다.

그리고 이것이 바로 useQuery(읽기)와 useMutation(쓰기)의 근본적인 차이를 이해할 수 있는 정말 중요한 포인트입니다. 발표 때 이 부분을 설명하면 기술적 이해도가 매우 높아 보일 거예요.

### 왜 날짜 변경(Query)은 낙관적 업데이트가 안될까요?

- 낙관적 업데이트 (Mutation): 우리가 결과를 예측할 수 있을 때 사용합니다.

- 체크박스를 토글하면, done 상태가 false에서 true로 바뀔 거라는 걸 미리 알고 있습니다. 그래서 서버의 응답을 기다리지 않고 UI를 먼저 바꾸는 것이 가능합니다.

- 날짜 변경 (Query): 우리는 결과를 전혀 예측할 수 없습니다.

- '27일' 날짜를 클릭했을 때, '27일'의 할 일 목록이 무엇인지 우리는 전혀 모릅니다. 이 데이터는 오직 서버만이 알고 있습니다.

- 알지 못하는 데이터를 미리 화면에 보여줄 수는 없으므로, 우리는 서버가 응답을 줄 때까지 기다렸다가(isLoading 상태) 데이터를 보여줄 수밖에 없습니다.

---

### 그럼 이 '느린 로딩'은 어떻게 개선할 수 있나요?

대신, 이런 상황을 위해 React Query는 다른 두 가지 멋진 전략을 제공합니다.

1. 데이터 미리 가져오기 (Prefetching)

- 개념: 사용자가 버튼을 클릭하기 전, 마우스를 올리기만 해도 백그라운드에서 데이터를 미리 불러오기 시작하는 기술입니다.

- 동작 방식:

1. 사용자가 '27일' 날짜 버튼에 마우스 커서를 올립니다.

2. 이때 prefetchQuery라는 함수를 실행해서 '27일'의 데이터를 미리 다운로드합니다.

3. 사용자가 실제로 클릭하면, 데이터는 이미 다운로드되어 캐시에 있으므로, isLoading 상태 없이 즉시 화면이 바뀝니다.

- 장점: 사용자 경험이 극적으로 향상됩니다. 앱이 마치 모든 데이터를 미리 가지고 있는 것처럼 느껴집니다.

2. 플레이스홀더 / 스켈레톤 UI (Placeholder / Skeleton UI)

- 개념: "로딩 중..."이라는 텍스트 대신, 최종적으로 보일 UI의 '뼈대'를 먼저 보여주는 기술입니다. (페이스북이나 유튜브에서 콘텐츠 로딩될 때 회색 박스들이 깜빡이는 것을 생각하시면 됩니다)

- 동작 방식: isLoading이 true일 때, 실제 TaskItem과 비슷한 모양의 회색 박스들을 여러 개 보여줍니다.

- 장점: 실제 데이터가 로딩되는 시간은 같지만, 사용자는 빈 화면 대신 무언가 로딩되고 있다는 것을 시각적으로 인지하므로 체감 성능(Perceived Performance)이 향상됩니다. 덜 지루하게 느껴지죠.

### 발표를 위한 추천

발표 데모에서는 1번: 데이터 미리 가져오기(Prefetching)를 구현하는 것이 가장 인상적일 겁니다. 사용자가 날짜에 마우스를 올리는 순간 콘솔에 'Prefetching...' 로그가 찍히고, 클릭하면 로딩 없이 즉시 화면이 바뀌는 모습을 보여주면 '와' 하는 탄성을 자아낼 수 있습니다.

Prefetching 기능을 DateHeader 컴포넌트에 추가해 볼까요?

물론입니다! 두 가지 전략을 모두 적용해서 사용자 경험을 최상으로 끌어올려 보겠습니다.

1. 스켈레톤 UI: 먼저 로딩 상태일 때 보여줄 '뼈대' 컴포넌트를 만들고, 메인 페이지에 적용하겠습니다.

2. 데이터 미리 가져오기(Prefetching): 그 다음, DateHeader에 마우스를 올렸을 때 데이터를 미리 가져오는 기능을 추가하겠습니다.

이 순서대로 진행하는 것이 더 효율적입니다.

---

### 1단계: 스켈레톤 UI 컴포넌트 만들기

로딩 상태일 때 보여줄 TaskItem의 뼈대, 즉 TaskItemSkeleton 컴포넌트를 만들겠습니다. 그리고 이를 여러 개 보여주는 TaskListSkeleton 컴포넌트도 함께 만들게요.

먼저, myday 기능 폴더 안에 이 스켈레톤 컴포넌트를 위한 파일을 생성하겠습니다. frontend/src/features/myday/components/TaskListSkeleton.tsx 파일을 새로 만들게요.

```ts
import React from 'react';

const TaskItemSkeleton = () => (
  <div className="flex items-center gap-3 p-4 bg-surface-card rounded-lg shadow-sm animate-pulse">
    <div className="w-6 h-6 bg-surface-base rounded"></div>
    <div className="flex-1 h-4 bg-surface-base rounded"></div>
    <div className="w-6 h-6 bg-surface-base rounded-full"></div>
  </div>
);

const TaskGroupSkeleton = ({ count = 3 }: { count?: number }) => (
  <div className="space-y-4">
    <div className="h-6 w-32 bg-surface-base rounded-md animate-pulse mb-4"></div>
    <div className="space-y-2">
      {Array.from({ length: count }, (_, i) => (
        <TaskItemSkeleton key={i} />
      ))}
    </div>
  </div>
);


export const TaskListSkeleton = () => {
  return (
    <div className="px-4 py-6 space-y-8 pb-24">
        <TaskGroupSkeleton count={3} />
        <TaskGroupSkeleton count={4} />
        <TaskGroupSkeleton count={2} />
    </div>
  );
};

export default TaskListSkeleton; 
```

스켈레톤 UI 컴포넌트가 준비되었습니다.

이제 MyDayPage에서 로딩 상태일 때, "불러오는 중..." 텍스트 대신 이 TaskListSkeleton 컴포넌트를 보여주도록 수정하겠습니다. 한 줄만 바꾸면 됩니다.

```ts
import TaskListSkeleton from '@/features/myday/components/TaskListSkeleton';
```
### 2단계: 데이터 미리 가져오기 (Prefetching) 적용하기

이제 DateHeader 컴포넌트로 가서, 사용자가 날짜에 마우스를 올렸을 때 (onMouseEnter) 데이터를 미리 가져오는 prefetchQuery 로직을 추가하겠습니다.

DateHeader.tsx 파일을 수정하겠습니다. useQueryClient를 사용해서 prefetch 함수를 호출할 겁니다.
```ts
import { useQueryClient } from '@tanstack/react-query';
import { getTasksByDate } from '@/lib/api/tasks';
```

모두 적용되었습니다!

이제 앱을 다시 확인해 보세요. 그리고 콘솔 창을 열어둔 채로 다음 시나리오를 따라 해보세요.

1. 페이지가 로드되면 오늘 날짜('26일'이라고 가정)의 할 일 목록이 보입니다.

2. 클릭하지 말고, 마우스 커서를 '27일' 날짜 위로 가져가 보세요.

3. 콘솔 창에 Prefetching tasks for... 로그가 찍히면서, 백그라운드에서 '27일'의 데이터를 미리 불러오는 것을 볼 수 있습니다. Prefetching complete... 로그도 확인하세요.

4. 이제 '27일'을 클릭하세요.

결과: 스켈레톤 UI가 나타나지 않고, 화면이 즉시 '27일'의 할 일 목록으로 바뀝니다! 데이터가 이미 캐시에 준비되어 있었기 때문이죠.

이것으로 사용자 경험 개선을 위한 두 가지 고급 전략을 모두 적용했습니다.

- 스켈레톤 UI: 처음 로딩되거나, prefetch가 미처 되지 않은 데이터를 불러올 때의 체감 성능 향상.

- Prefetching: 사용자의 행동을 예측하여 데이터를 미리 불러와 로딩 시간을 없애는 기술.

이 두 가지를 '낙관적 업데이트'와 함께 설명하시면, React Query를 얼마나 깊이 있고 효과적으로 사용하는지에 대한 훌륭한 발표가 될 것입니다.

정말 멋진 기능들을 프로젝트에 성공적으로 적용하셨네요! 또 도와드릴 일이 있을까요?