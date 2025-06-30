---
tags:
  - TIL
  - React
created: 2025-06-16
---
# 📘 2025-06-16 TIL

## 📌 오늘 배운 핵심 요약
- React 18 기반으로 안정적인 프로젝트 환경 설정
- 기능 단위로 나눈 컴포넌트 및 Redux 구조 설계 이해
- 필수 라이브러리 도입과 각 라이브러리의 역할 정리
- clsx를 활용한 조건부 className 처리 방식 학습
- Redux Toolkit을 기반으로 한 상태 관리 구조 학습
- Vanilla Extract + clsx 연계 스타일링 방식 실습


## 🧠 상세 학습 내용

## 📍 주제 1: react-task-app 초기 세팅 및 구조 구성  

### ✅ React 18 버전으로 고정한 이유
현재 시점에서 일부 라이브러리(dnd-kit, redux 관련 등)는 React 19를 정식 지원하지 않기 때문에, 안정적인 개발을 위해 React 18로 맞추었다.
```bash
npm uninstall react react-dom
npm install react@18 react-dom@18
```

### ✅ 패키지 설치 목록 및 설명
```bash
npm install @reduxjs/toolkit redux react-redux clsx @vanilla-extract/css @vanilla-extract/css-utils @vanilla-extract/vite-plugin react-icons uuid react-beautiful-dnd
```

### 📦 사용한 주요 패키지 및 역할 요약
| **패키지명**                | **역할 요약**                        |
| ----------------------- | -------------------------------- |
| @reduxjs/toolkit, redux | 전역 상태 관리 (slice, reducer 포함)     |
| react-redux             | React와 Redux 연결                  |
| clsx                    | 조건부 className 처리 유틸리티            |
| @vanilla-extract/*      | 타입 안전한 CSS-in-TypeScript 스타일 시스템 |
| react-icons             | 다양한 아이콘 제공                       |
| uuid                    | 고유 ID 생성 (예: task, board 등 식별자)  |
| react-beautiful-dnd     | 리스트 및 카드 드래그 앤 드롭 UX 구현          |

### ✅ 폴더 구조

```bash
src/
├── components/         # 기능 단위로 컴포넌트 분리
│   ├── BoardList/
│   ├── SideForm/
│   ├── EditModal/
│   ├── List/
│   ├── ListsContainer/
│   ├── LoggerModal/
│   ├── LogItem/
│   └── Task/
├── hooks/
│   └── redux.ts        # useSelector, useDispatch 커스텀 훅 정의
├── store/
│   ├── reducer/
│   │   └── reducer.ts
│   └── slices/         # Redux Toolkit slice 파일 분리
│       ├── boardsSlice.ts
│       ├── loggerSlice.ts
│       └── modalSlice.ts
├── types/              # 전역 타입 정의
├── App.tsx
├── main.tsx
└── index.html
```

- 컴포넌트 단위로 폴더 나누고, 각각 `컴포넌트명.tsx + 컴포넌트명.css.ts` 구성
- store 하위에 slice 별로 관리되며, redux hook은 분리

---


## 📍 주제 2: clsx로 className 조건 처리

### ✅ 정의
clsx는 JavaScript에서 조건부로 className을 쉽게 조합할 수 있도록 도와주는 유틸리티 함수이다. classnames 라이브러리와 거의 동일한 API를 제공하며, 더 가볍고 빠르다.

📦 설치 명령어
```bash
npm install clsx
```

 📥 import 
```ts
import clsx from 'clsx';
```

### ✅ 기본 사용법
```tsx
<div className={clsx('btn', isActive && 'btn-primary')} />
```
- isActive가 true면 → `btn btn-primary`
- false면 → `btn`

|**전통 방식**|**clsx 방식**|
|---|---|
|isActive ? 'btn btn-primary' : 'btn'|clsx('btn', isActive && 'btn-primary')|
 false, null, undefined 값은 자동으로 무시되므로, 조건 분기 처리 시 더욱 간결한 코드를 작성할 수 있다.
 
### ✅ 다양한 사용 예시
```tsx
clsx('foo', true && 'bar', 'baz');
// 결과: 'foo bar baz'

const isDark = false;
clsx('base', isDark && 'dark');
// 결과: 'base'

clsx({ 'active': true, 'hidden': false })
// 결과: 'active'
```

---

## 📍 주제 3: Redux 구조 및 흐름 이해
Redux는 단방향 데이터 흐름을 따르는 전역 상태 관리 도구로, 앱 규모가 커질수록 유용해짐. Redux Toolkit을 사용하면 slice 단위로 구조화가 쉬워지고 코드도 간결해진다.

#### ✅ Redux Flow
1. **Action**: 상태 변경을 요청하는 객체
2. **Dispatch**: Action을 스토어에 전달하는 함수
3. **Reducer**: Action의 type에 따라 상태를 변경하여 새 state 반환
4. **Store**: 전체 앱의 상태를 보관하는 전역 저장소

리덕스를 사용할 때 일반적으로 Toolkit을 함께 사용하는 것이 보편적이다. 이유는 slice 단위 관리, immer 기반 불변성 유지, 코드 간결화 때문이다.

#### ✅ Redux Toolkit 구성 요소

- **slice**: 상태 + 리듀서 + 액션을 묶은 단위 (createSlice로 생성)
- **configureStore**: 스토어 생성기 (Redux DevTool 자동 연동 포함)
- **useSelector, useDispatch**: 상태 조회 및 액션 디스패치용 hook

#### ✅ 예시 흐름

```ts
// modalSlice.ts
const modalSlice = createSlice({
  name: 'modal',
  initialState: { isOpen: false },
  reducers: {
    openModal: (state) => { state.isOpen = true },
    closeModal: (state) => { state.isOpen = false },
  }
});
```

```ts
// 사용 예시
const dispatch = useAppDispatch();
dispatch(openModal());
```


---

## 📍 주제 4: Redux Toolkit 실습 및 slice 구조

React 프로젝트에서 Redux Toolkit을 활용하면 slice 단위로 상태를 모듈화하고 유지보수가 쉬운 상태 관리 구조를 만들 수 있다.

### ✅ boardsSlice.ts
```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Board {
  id: string;
  title: string;
}

interface BoardsState {
  items: Board[];
}

const initialState: BoardsState = {
  items: [],
};

const boardsSlice = createSlice({
  name: 'boards',
  initialState,
  reducers: {
    addBoard: (state, action: PayloadAction<Board>) => {
      state.items.push(action.payload);
    },
    removeBoard: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter((board) => board.id !== action.payload);
    },
  },
});

export const { addBoard, removeBoard } = boardsSlice.actions;
export default boardsSlice.reducer;
```

### ✅ store 설정
```ts
import { configureStore } from '@reduxjs/toolkit';
import boardsReducer from './slices/boardsSlice';

export const store = configureStore({
  reducer: {
    boards: boardsReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### ✅ 커스텀 훅으로 추상화
```ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

이렇게 하면 컴포넌트에서는 복잡한 타입 선언 없이 다음과 같이 간결하게 사용 가능합니다:
```ts
const boards = useAppSelector((state) => state.boards.items);
const dispatch = useAppDispatch();
```

----

## 📍 주제 5: Vanilla Extract + clsx 연계
Tailwind 없이 Vanilla Extract만 사용하는 경우, className의 조건 분기는 clsx와 함께 처리하는 것이 가장 직관적이고 유지보수에 유리하다.

### ✅ 예시 코드: ListItem.tsx
```ts
import { listItem, selectedItem } from './ListItem.css';
import clsx from 'clsx';

type Props = {
  isSelected: boolean;
  title: string;
};

export const ListItem = ({ isSelected, title }: Props) => {
  return (
    <div className={clsx(listItem, isSelected && selectedItem)}>
      {title}
    </div>
  );
};
```

### ✅ 관련 스타일: ListItem.css.ts
```ts
import { style } from '@vanilla-extract/css';

export const listItem = style({
  padding: '10px',
  borderRadius: '8px',
  backgroundColor: 'var(--color-task)',
  transition: 'all 0.2s ease',
});

export const selectedItem = style({
  backgroundColor: 'var(--color-selectedTab)',
});
```


![redux-toolkit](https://seonohblog.netlify.app/assets/redux-toolkit.png)

---

## 💭 회고
• **새롭게 알게 된 점**
- 기존에 className={isActive ? 'a' : 'b'} 식으로 조건 분기를 쓰던 방식보다 clsx()가 훨씬 명료하고 깔끔하다는 것을 알게 되었다.
- Redux Flow에서 Dispatch → Reducer → Store 업데이트 → UI 반영의 단방향 흐름이 인상 깊었고, slice 단위로 나누니 관리가 확실히 쉬웠다.

• **어렵게 느껴졌던 부분**
-  slice를 어떻게 기능 단위로 나눌지, 상태 범위를 어디까지 책임지게 할지에 대한 기준이 아직 명확하지 않다.
  
• **다음에 학습할 주제**
- Board List
- SideForm
- List 컴포넌트
- Task 컴포넌트
- Action Button
- DropDownForm 컴포넌트 

### 🔗 참고자료
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Vanilla Extract Docs](https://vanilla-extract.style/)
- [clsx 공식 문서](https://github.com/lukeed/clsx)