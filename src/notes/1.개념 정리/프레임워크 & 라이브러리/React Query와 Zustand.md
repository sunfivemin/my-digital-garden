## 발표 자료: useState를 넘어서, React Query와 Zustand로 진화하는 프론트엔드

발표 목표: useState만으로 상태를 관리하던 초기 앱의 문제점을 알아보고, Zustand와 React Query를 도입하여 어떻게 코드 품질과 사용자 경험을 극적으로 개선했는지 과정을 공유합니다.

---

### 1. The "Before" : 우리의 시작점

#### 코드 상황
- 모든 상태를 useState로 관리했습니다.
- 날짜 상태: MyDayPage에서 useState로 생성 후, DateHeader로 props 전달 (props drilling).
- 할 일 목록 상태: MyDayPage에 하드코딩된 임시 데이터를 useState로 관리.

```tsx
// MyDayPage.tsx (개선 전, 요약)
export default function MyDayPage() {
  // 1. 날짜 상태 관리
  const [selectedDate, setSelectedDate] = useState(new Date());

  // 2. 할 일 목록 관리
  const [tasks, setTasks] = useState(initialTasks);

  // 3. 자식에게 props로 전달
  return (
    <MobileLayout>
      <DateHeader
        selectedDate={selectedDate}
        onSelectDate={setSelectedDate}
      />
      {/* tasks.must.map(...) */}
    </MobileLayout>
  );
}
```

#### 문제점
1. 복잡한 상태 로직:
- API 연동 시 isLoading, isError 상태를 위한 useState가 추가로 필요.
- 데이터 동기화를 위한 useEffect 훅이 필수적이며, 코드가 길고 복잡해짐.

2. Props Drilling:
- selectedDate가 필요 없는 중간 컴포넌트를 거쳐야 할 경우, 코드가 지저분해짐.

2. 끔찍한 사용자 경험 (만약 실제 API였다면):
- 느린 데이터 로딩: 날짜 변경 시, 매번 1초 이상 기다려야 화면이 바뀜.
- 느린 데이터 업데이트: 체크박스 클릭 시, 서버와 통신이 끝날 때까지 (약 1.5초) UI가 멈춰있어 매우 답답함.

---

### 2. The "After" : 우리의 해결책

#### 핵심 철학: "상태를 두 종류로 나누어, 각 전문가에게 맡기자!"

① 클라이언트 상태 (Client State): 오직 내 앱(브라우저)에만 존재하는 상태.
- 예시: 다크모드 ON/OFF, 모달창 열림/닫힘, 오늘 내가 선택한 날짜.
- 서버는 이 상태에 대해 전혀 모릅니다. 
	- 해결사 → Zustand

② 서버 상태 (Server State): 서버 데이터베이스에서 가져와야 하는 상태.
- 예시: 오늘의 할 일 목록, 내 프로필 정보, 게시글 목록.
- 특징: 비동기(Asynchronous)이며, 항상 로딩/성공/실패 상태를 가집니다.
	-  해결사 → React Query

---

### 3. Transformation #1: Zustand로 클라이언트 상태 정복하기
#### 개념: Zustand란?
"아주 작고 간단한 전역 상태 보관함" 이라고 생각하시면 됩니다. Redux처럼 복잡한 설정 없이, 어떤 컴포넌트에서든 쉽게 꺼내 쓸 수 있는 공유 상자 같은 것입니다.

#### 적용: useState와 props drilling을 제거하다

#### Before
useState로 날짜를 관리하고, props로 자식에게 전달.

#### After
1. useDateStore 스토어 생성
- 단 몇 줄의 코드로 전역 상태와 상태 변경 함수를 만듦.
-  Zustand의 create 함수로 '날짜 보관함'(useDateStore)을 만듭니다.
```ts
    // store/useDateStore.ts
    export const useDateStore = create<DateState>((set) => ({
      selectedDate: new Date(),
      setSelectedDate: (date) => set({ selectedDate: date }),
    }));
```

2. 컴포넌트에서 직접 사용
- 이제 DateHeader는 부모에게 props를 받을 필요 없이, 필요할 때 직접 useDateStore라는 훅을 호출해서 상태를 가져옵니다.
```ts
    // features/myday/components/DateHeader.tsx
    export default function DateHeader() {
      const { selectedDate, setSelectedDate } = useDateStore();
      // ...
    }
```

#### 결과
- Props Drilling 해결: 더 이상 불필요한 props 전달이 사라짐.
- 컴포넌트 독립성 향상: DateHeader는 이제 스스로 상태를 관리하는 독립적인 부품이 됨.

---

### 4. Transformation #2: React Query로 서버 상태 지배하기
#### 개념: React Query란?

"데이터를 다루는 아주 유능한 개인 비서" 입니다. fetch나 axios처럼 단순히 데이터를 요청만 하는 도구가 아닙니다. 데이터를 가져오고, 캐싱(기억)하고, 로딩/에러 상태를 관리하고, 화면을 자동으로 업데이트하는 등 서버 데이터와 관련된 모든 귀찮은 일을 대신 해줍니다.

#### 적용 1: useQuery - 데이터 가져오기

#### Before
useState와 (상상 속의) useEffect로 API 데이터를 관리. 로딩/에러 처리, 캐싱, 업데이트 모두 수동으로 구현해야 함.
useState 3개(data, loading, error)와 useEffect 1개, 총 4개가 필요했던 데이터 가져오기.

#### After (3단계 개선)
#### 1단계: useQuery로 데이터 읽기
- useState, useEffect를 단 하나의 useQuery 훅으로 대체.
- isLoading, isError 상태를 무료로 제공받음.

```tsx
// MyDayPage.tsx
import { useQuery } from '@tanstack/react-query';
import { getTasksByDate } from '@/lib/api/tasks';

// ...
const { data, isLoading, isError } = useQuery({
  queryKey: ['tasks', selectedDate], // 데이터의 "이름표"
  queryFn: () => getTasksByDate(selectedDate), // 실제 데이터를 가져오는 함수
});
```
- useQuery 훅: "데이터 좀 가져와 줘" 라고 요청하는 훅입니다.
- queryKey: React Query가 데이터를 구별하기 위한 고유한 이름표입니다. selectedDate가 바뀌면 이름표도 바뀌어서, 비서가 새로운 데이터를 가져옵니다. (캐싱의 핵심!)
- queryFn: "어떻게" 데이터를 가져올지 알려주는 실제 비동기 함수입니다.
- 반환값: data(성공한 데이터), isLoading(로딩 중인지?), isError(실패했는지?) 상태를 알아서 제공합니다.

#### 결과: 복잡했던 데이터 로직이 단 한 줄로 정리되었습니다.


#### 2단계: UX 개선 - 로딩 시간을 즐겁게!
- 스켈레톤 UI: "로딩 중..." 텍스트 대신, UI 뼈대를 보여주어 사용자가 느끼는 체감 성능 향상.
- 데이터 미리 가져오기 (Prefetching): 사용자가 날짜에 마우스를 올리는 순간 데이터를 미리 로드. 클릭 시 로딩 없이 즉시 화면 전환!

```tsx
// DateHeader.tsx
    function DateItem({ date, onSelect }) {
      const queryClient = useQueryClient();
      const prefetchTasks = () => {
        queryClient.prefetchQuery(...); // 마우스 올리면 미리 가져오기
      };
      return <div onMouseEnter={prefetchTasks} onClick={onSelect} />;
    }**

```

### 3단계: UX 혁신 - "낙관적 업데이트"
- 문제: 체크박스를 눌러도 1.5초 뒤에 반응하는 끔찍한 경험.
- 해결: useMutation의 onMutate 옵션을 사용.

1. UI 즉시 업데이트: React Query의 onMutate 훅을 사용 UI를 즉시 변경.
2. 백그라운드 통신: UI 변경 후, 서버에 조용히 업데이트 요청.
3. 자동 롤백: 만약 서버 요청이 실패하면, UI를 원래대로 되돌려 데이터 정합성 유지.

```tsx
// MyDayPage.tsx
const queryClient = useQueryClient();
const { mutate } = useMutation({
  mutationFn: updateTaskStatus, // "어떻게" 데이터를 수정할지 알려줌

  // 여기가 마법의 핵심: 낙관적 업데이트(Optimistic Update)
  onMutate: async (variables) => {
    // 1. UI를 즉시 업데이트 (사용자는 바로 반응을 본다)
    queryClient.setQueryData(queryKey, newOptimisticData);
    return { previousTasks }; // 2. 실패 시 되돌릴 원본 데이터 저장
  },

  onError: (err, variables, context) => {
    // 3. 만약 실패하면, 저장해 둔 원본 데이터로 UI를 롤백
    queryClient.setQueryData(queryKey, context.previousTasks);
  },

  onSettled: () => {
    // 4. 성공하든 실패하든, 마지막에 서버와 데이터를 최종 동기화
    queryClient.invalidateQueries({ queryKey });
  },
});
```
- useMutation 훅: "데이터를 수정/생성/삭제해 줘" 라고 요청하는 훅입니다.
- onMutate: "일단 성공할거라 믿고 UI부터 바꿔!" - 사용자 경험의 핵심입니다.
- onError: "앗, 실패했네. 얼른 원래대로 되돌리자!" - 데이터의 안정성을 보장합니다.
- invalidateQueries: "이제 이 데이터는 낡았으니, 새로고침해줘!" - 서버와의 동기화를 담당합니다.

#### 결과
- 혁신적인 사용자 경험: 사용자는 모든 인터랙션에 앱이 즉각적으로 반응한다고 느낌.
- 선언적인 코드: "데이터를 가져와라", "데이터를 수정해라" 라고 선언만 하면, 복잡한 비동기 로직은 React Query가 알아서 처리. 개발자는 UI에만 집중 가능.

### 4단계: prefetchQuery - 로딩 시간 없애기

DateHeader에서 prefetchQuery 메소드를 사용했습니다.
```tsx
// DateHeader.tsx
const queryClient = useQueryClient();
const prefetchTasks = () => {
  queryClient.prefetchQuery({ // 마우스를 올리면
    queryKey: ['tasks', date], // 해당 날짜의 데이터를
    queryFn: () => getTasksByDate(date), // 미리 가져온다!
  });
};
```
- prefetchQuery: 사용자가 클릭하기 전, 마우스를 올리는 것만으로 데이터를 미리 캐시에 저장합니다. 클릭 시에는 캐시된 데이터를 즉시 보여줘 로딩 시간을 0으로 만듭니다.


---

### 5. 결론

|구분|Before (useState)|After (Zustand + React Query)|
|---|---|---|
|코드 복잡도|높음 (수많은 상태, useEffect)|낮음 (선언적, 역할 분리)|
|Props Drilling|문제 발생|해결 (전역 스토어)|
|로딩 처리|수동 구현 (지루한 텍스트)|자동 + 스켈레톤 UI + Prefetching (세련된 경험)|
|업데이트 처리|수동 구현 (느리고 답답함)|자동 + 낙관적 업데이트 (즉각적인 반응)|
|개발자 경험|좋지 않음|매우 좋음|
|사용자 경험|끔찍함|매우 뛰어남|

우리는 단순히 라이브러리 두 개를 추가한 것이 아니라, 애플리케이션을 설계하는 패러다임을 전환했습니다. 이를 통해 사용자와 개발자 모두를 만족시키는 더 나은 웹 애플리케이션을 만들 수 있었습니다.