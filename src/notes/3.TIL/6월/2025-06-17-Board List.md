---
tags:
  - TIL
  - React
created: 2025-06-17
---
# 📘 2025-06-17 TIL

## 📌 오늘 배운 핵심 요약
- 게시판(Board) 목록 렌더링과 스타일 구성
- + 버튼을 통한 게시판 추가 입력 폼 구현(SideForm)
- Redux 연결을 통한 게시판 상태 관리
- 리스트 & 태스크 컴포넌트 구조 이해 및 로그 기록 처리
- ActionButton + DropdownForm 컴포넌트를 통한 리스트/일(Task) 추가 흐름 구현


## 🧠 상세 학습 내용

## 📍 주제 1: Board List 생성 및 스타일 적용
### ✅ 1. BoardList 컴포넌트 생성
- 게시판 목록을 상단에 렌더링하고, 현재 선택된 게시판을 강조 표시한다.
- `boardArray`는 Redux 상태에서 가져오며, `useTypedSelector` 훅을 사용해 불러온다.
- 선택된 게시판 ID는 `activeBoardId`로 관리되며, 클릭 시 `setActiveBoardId`로 갱신한다.

```ts
const boardArray = useTypedSelector((state) => state.boards.boardArray);
```

```tsx
{boardArray.map((board: IBoard) => (
  <li
    key={board.boardId}
    onClick={() => setActiveBoardId(board.boardId)}
    className={clsx(boardItem, {
      [boardItemActive]: board.boardId === activeBoardId,
    })}
  >
    {board.boardName}
  </li>
))}
```
- clsx를 통해 선택된 항목에만 boardItemActive 클래스를 동적으로 추가한다.

---

### ✅ 2. BoardList 스타일 구성 (BoardList.css.ts)
- Vanilla Extract와 글로벌 디자인 토큰(vars)을 활용해 일관된 스타일을 정의한다.
```ts
import { style } from '@vanilla-extract/css';
import { vars } from '../../App.css';

export const container = style({
  display: 'flex',
  flexDirection: 'row',
  alignItems: 'center',
  flexWrap: 'wrap',
  rowGap: 15,
  padding: vars.spacing.big2,
  backgroundColor: vars.color.mainDarker,
});

export const title = style({
  color: vars.color.brightText,
  fontSize: vars.fontSizing.T2,
  marginRight: vars.spacing.big1,
});

export const boardItem = style({
  color: vars.color.brightText,
  fontSize: vars.fontSizing.T3,
  backgroundColor: vars.color.mainFaded,
  padding: vars.spacing.medium,
  borderRadius: 10,
  marginRight: vars.spacing.big1,
  cursor: 'pointer',
  ':hover': {
    opacity: 0.8,
    transform: 'scale(1.03)',
  },
});

export const boardItemActive = style({
  backgroundColor: vars.color.selectedTab,
});
```
- boardItem: 기본 카드 스타일
- boardItemActive: 선택된 항목 강조 색상

---

### ✅ 3. + 버튼으로 SideForm 열기

- 게시판 추가 폼을 + 버튼 클릭으로 토글하여 보여준다.
- useState로 isFormOpen 상태를 관리한다.

```ts
const [isFormOpen, setIsFormOpen] = useState(false);
```

```tsx
<div className={addSection}>
  {isFormOpen ? (
    <SideForm setIsFormOpen={setIsFormOpen} />
  ) : (
    <FiPlusCircle
      size={24}
      style={{ cursor: 'pointer' }}
      onClick={() => setIsFormOpen(true)}
    />
  )}
</div>
```

---

## 📍 주제 2: SideForm 생성 및 Redux 연결

### ✅ SideForm의 역할
- + 아이콘 클릭 시 나타나는 입력 폼
- 새 게시판 이름을 입력받고 Redux 상태에 추가함
- 입력창 자동 포커스, 외부 클릭 시 닫힘, Enter 입력 또는 체크 버튼 클릭으로 등록 가능

### ✅ Redux 연결
- addBoard 액션으로 board 추가
- uuid로 고유 ID 생성

### ✅ 타입 정의: IBoard (📂 types/index.ts)
```ts
export interface ITask {
  taskId: string;
  taskName: string;
  taskDescription: string;
  taskOwner: string;
}

export interface IList {
  listId: string;
  listName: string;
  tasks: ITask[];
}

export interface IBoard {
  boardId: string;
  boardName: string;
  boardDescription: string; 
  lists: IList[];
}
```


### ✅ Redux 슬라이스: boardsSlice.ts (📂 store/slices)
```ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit';
import type { IBoard } from '../../types';

type TAddBoardAction = {
  board: IBoard;
};

type TBoardState = {
  boardArray: IBoard[];
};

const initialState: TBoardState = {
  boardArray: [],
};

const boardsSlice = createSlice({
  name: 'boards',
  initialState,
  reducers: {
    addBoard: (state, { payload }: PayloadAction<TAddBoardAction>) => {
      state.boardArray.push(payload.board);
    },
  },
});

export const { addBoard } = boardsSlice.actions;
export const boardsReducer = boardsSlice.reducer;
```


### ✅ 입력 컴포넌트: SideForm.tsx
```ts
import React, {
  useEffect,
  useRef,
  useState,
  type ChangeEvent,
} from 'react';
import { useDispatch } from 'react-redux';
import { FiCheck } from 'react-icons/fi';
import { addBoard } from '@/store/slices/boardsSlice';
import { v4 as uuidv4 } from 'uuid';

type TSideFormProps = {
  setIsFormOpen: React.Dispatch<React.SetStateAction<boolean>>;
};

const SideForm: React.FC<TSideFormProps> = ({ setIsFormOpen }) => {
  const dispatch = useDispatch();
  const [inputText, setInputText] = useState('');
  const wrapperRef = useRef<HTMLDivElement>(null);
  const inputRef = useRef<HTMLInputElement>(null);

  // ✅ 입력창 자동 포커스
  useEffect(() => {
    setTimeout(() => {
      inputRef.current?.focus();
    }, 0);
  }, []);

  // ✅ 외부 클릭 시 닫힘
  useEffect(() => {
    const handleClickOutside = (e: MouseEvent) => {
      if (
        wrapperRef.current &&
        !wrapperRef.current.contains(e.target as Node)
      ) {
        setIsFormOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [setIsFormOpen]);

  // ✅ 입력값 업데이트
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setInputText(e.target.value);
  };

  // ✅ 등록 버튼 클릭 or Enter 시 dispatch
  const handleSubmit = () => {
    const trimmed = inputText.trim();
    if (!trimmed) return;

    dispatch(
      addBoard({
        board: {
          boardId: uuidv4(),
          boardName: trimmed,
          boardDescription: '', 
          lists: [],
        },
      })
    );
    setInputText('');
    setIsFormOpen(false);
  };

  return (
    <div
      ref={wrapperRef}
      style={{ display: 'flex', alignItems: 'center', gap: 8 }}
    >
      <input
        ref={inputRef}
        type="text"
        placeholder="새로운 게시판 등록하기"
        value={inputText}
        onChange={handleChange}
        onKeyDown={(e) => {
          if (e.key === 'Enter') handleSubmit();
        }}
      />
      <FiCheck
        size={20}
        style={{ cursor: 'pointer' }}
        onClick={handleSubmit}
        title="등록"
      />
    </div>
  );
};

export default SideForm;
```


---

## 📍 주제 3: List& Task 컴포넌트 구조 

Do 앱에서 `List` 컴포넌트는 하나의 작업 리스트(예: "할 일", "진행 중")를 표현하고, `Task` 컴포넌트는 그 리스트 안에 포함된 개별 작업 항목을 표현합니다. 이 두 컴포넌트는 Redux와 연결되어 삭제, 편집, 모달 동작 등을 수행합니다.

### ✅ List 컴포넌트 역할 및 구조

#### 🔹 props 정의
```ts
type TListProps = {
  boardId: string;
  list: IList;
};
```
- boardId: 현재 보드의 식별자
- list: 리스트 데이터 (listId, listName, tasks 포함)

#### 🔹 주요 기능
- 리스트 헤더에 이름과 삭제 버튼 렌더링
- 리스트에 속한 Task 목록을 map으로 출력
- Task 클릭 시 모달 열기
- 리스트 삭제 시 로그 기록

```ts
const List: FC<TListProps> = ({ list, boardId }) => {
  const dispatch = useTypedDispatch();

  const handleListDelete = (listId: string) => {
    dispatch(deleteList({ boardId, listId }));
    dispatch(addLog({
      logId: v4(),
      logMessage: `리스트 삭제하기 : ${list.listName}`,
      logAuthor: 'User',
      logTimestamp: String(Date.now()),
    }));
  };

  const handleTaskChange = (boardId: string, listId: string, task: ITask) => {
    dispatch(setModalData({ boardId, listId, task }));
    dispatch(setModalActive(true));
  };

  return (
    <div className={listWrapper}>
      <div className={header}>
        <div className={names}>{list.listName}</div>
        <FiTrash
          className={deleteButton}
          onClick={() => handleListDelete(list.listId)}
        />
      </div>

      {list.tasks.map((task, index) => (
        <div
          onClick={() => handleTaskChange(boardId, list.listId, task)}
          key={task.taskId}
        >
          <Task
            id={task.taskId}
            taskName={task.taskName}
            taskDescription={task.taskDescription}
            boardId={boardId}
            index={index}
          />
        </div>
      ))}

      <ActionButton />
    </div>
  );
};
```


---

### ✅ Task 컴포넌트 역할 및 구조

#### 🔹 props 정의
```ts
type TTaskProps = {
  index: number;
  id: string;
  boardId: string;
  taskName: string;
  taskDescription: string;
};
```
- 현재는 index, id, boardId는 UI 렌더링에서는 사용되지 않지만, 추후 드래그 정렬 등에서 활용 가능

#### 🔹 렌더링 구조
```tsx
const Task: FC<TTaskProps> = ({
  taskName,
  taskDescription,
}) => {
  return (
    <div className={container}>
      <div className={title}>{taskName}</div>
      <div className={description}>{taskDescription}</div>
    </div>
  );
};
```


### ✅ Redux와의 연결 요약
| **액션**    | **디스패치 함수**                             | **설명**               |
| --------- | --------------------------------------- | -------------------- |
| 리스트 삭제    | deleteList                              | boardsSlice에서 리스트 제거 |
| 로그 기록     | addLog                                  | loggerSlice에 로그 저장   |
| 모달 활성화    | setModalActive(true)                    | 모달을 열도록 설정           |
| 모달 데이터 설정 | setModalData({ boardId, listId, task }) | 클릭된 Task 데이터를 저장     |

---

## 📍 주제 4: ActionButton + DropdownForm 컴포넌트

### ✅ ActionButton 컴포넌트
- 리스트 또는 일(Task) 추가를 위한 버튼 컴포넌트
- 클릭 시 DropdownForm 오픈
- list prop 여부에 따라 기능 분기 (리스트 추가 vs 작업 추가)

#### 🔹 props 타입 정의
```ts
type TActionButtonProps = {
  boardId: string;
  listId: string;
  list?: boolean; // true면 리스트 추가 버튼
};
```
- boardId: 현재 보드 ID
- listId: 리스트가 추가될 위치 지정 (리스트 등록 시에도 필요)
- list: 리스트 등록 버튼인지(Task 등록이 아닌지) 여부 판단

#### 🔹 핵심 로직
```ts
const [isFormOpen, setIsFormOpen] = useState(false);

if (isFormOpen) {
  return (
    <DropdownForm
      setIsFormOpen={setIsFormOpen}
      list={!!list}
      boardId={boardId}
      listId={listId}
    />
  );
}

return (
  <div onClick={() => setIsFormOpen(true)}>
    <IoIosAdd />
    <p>{list ? '새로운 리스트 등록' : '새로운 일 등록'}</p>
  </div>
);
```
- isFormOpen이 true일 때 DropdownForm 표시
- 버튼 클릭 → DropdownForm 렌더링 → 텍스트 입력 후 dispatch


### ✅ DropDownForm 컴포넌트
DropdownForm은 실제 입력 폼입니다.
사용자가 리스트 또는 작업의 제목을 입력하면 Redux 상태에 추가됩니다.

#### 🔹 props 타입 정의
```ts
type TDropDownFormProps = {
  boardId: string;
  listId: string;
  setIsFormOpen: React.Dispatch<React.SetStateAction<boolean>>;
  list?: boolean;
};
```

#### 🔹 핵심 로직
```ts
const handleButtonClick = () => {
  if (!text) return;

  if (list) {
    dispatch(addList({ boardId, list: { listId: v4(), listName: text, tasks: [] } }));
    dispatch(addLog({ logId: v4(), logMessage: `로그 생성하기: ${text}`, ... }));
  } else {
    dispatch(addTask({ boardId, listId, task: { taskId: v4(), taskName: text, ... } }));
    dispatch(addLog({ logId: v4(), logMessage: `일 생성하기: ${text}`, ... }));
  }
};
```
- 리스트 추가 시: addList + addLog
- 작업 추가 시: addTask + addLog
- 입력 창(textarea)은 onBlur 시 자동 닫힘
- 버튼 클릭은 onMouseDown으로 onBlur보다 먼저 처리

### ✅ 동작 흐름 요약
```tsx
[ActionButton 클릭]
       ↓
[isFormOpen → true]
       ↓
[DropdownForm 렌더링] // ActionButton은 UI의 진입점, 내부에서 DropdownForm을 조건부로 렌더링
       ↓            // DropdownForm은 입력 폼 역할을 하며, Redux 상태를 직접 업데이트
[텍스트 입력 후 버튼 클릭] 
       ↓
[Redux dispatch → addList or addTask]
       ↓
[addLog로 로그까지 기록]
```


![board](https://seonohblog.netlify.app/assets/board.png)


---


## 💭 회고
• **새롭게 알게 된 점**
- useRef를 사용한 외부 클릭 감지 및 자동 포커스 처리
- Redux 상태 업데이트와 동시에 로그까지 남기는 구조 (addLog 병행)
- 컴포넌트 내에서 조건부 렌더링으로 폼 토글을 자연스럽게 처리하는 방식

• **어렵게 느껴졌던 부분**
- 외부 클릭 감지 구현 시 ref와 eventListener 해제 타이밍 조절
- 상태에 따라 다양한 컴포넌트를 조건부로 렌더링할 때 props 분기 처리 로직
  
• **다음에 학습할 주제**
- Modal Edit 컴포넌트
- LogItem 컴포넌트
- 게시판 삭제 기능

### 🔗 참고자료
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Vanilla Extract Docs](https://vanilla-extract.style/)