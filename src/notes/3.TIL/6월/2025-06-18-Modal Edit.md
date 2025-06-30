---
tags:
  - TIL
  - React
created: 2025-06-18
---
# 📘 2025-06-18 TIL

## 📌 오늘 배운 핵심 요약
- Redux 기반 모달 UI(EditModal, LoggerModal) 상태 관리 및 이벤트 처리 흐름 이해
- 로그 기록 등 기능별 액션 연결 방식 숙지
- 게시판 삭제 기능 구현 및 상태 변화 대응 로직 설계


## 🧠 상세 학습 내용

## 📍 주제 1: Modal Edit 컴포넌트

EditModal은 사용자가 특정 Task를 수정하거나 삭제할 수 있도록 지원하는 **모달 다이얼로그 UI**입니다.
Redux의 상태 값을 기반으로 열리고 닫히며, Task 데이터를 편집 후 저장하거나 삭제할 수 있습니다.

#### 🔹 주요 역할
- taskName, taskDescription, taskOwner를 **수정 및 삭제** 가능
- 수정 시 updateTask 액션 디스패치
- 삭제 시 deleteTask 액션 디스패치
- 각 동작 시, addLog 액션으로 로그도 함께 남김
- setModalActive(false)로 모달 닫기


#### 🔹 상태 관리 구조
```ts
const modalActive = useTypedSelector(state => state.boards.modalActive);
{modalActive ? <EditModal /> : null}
```
- Redux store에서 모달이 열려있는지를 modalActive로 확인
- modalActive === true일 경우 <EditModal /> 렌더링

#### 🔹 EditModal에서 사용하는 Slice
1. **boardsSlice**
- modalActive: boolean → 모달 열림/닫힘 여부
- updateTask, deleteTask → Task 수정 및 삭제
2. **modalSlice**
- 현재 수정할 Task의 상세 정보 (boardId, listId, task)

#### 🔹 핵심 코드 요약
```tsx
const modalActive = useTypedSelector(state => state.boards.modalActive);
{modalActive ? <EditModal /> : null}
```

```tsx
const editingState = useTypedSelector(state => state.modal);
const [data, setData] = useState(editingState);
```
- modalActive: 모달 열기 여부
- editingState: 현재 수정 중인 Task 정보
- setData: 로컬 상태로 input 폼과 연동

#### 🔹 주요 이벤트 핸들러
```tsx
// 제목, 설명, 작성자 수정
const handleNameChange = (e) => { ... };
const handleDescriptionChange = (e) => { ... };
const handleOwnerChange = (e) => { ... };

// 수정 완료 시
dispatch(updateTask(...));
dispatch(addLog(...));
dispatch(setModalActive(false));

// 삭제 시
dispatch(deleteTask(...));
dispatch(addLog(...));
dispatch(setModalActive(false));
```
✔️ 수정 및 삭제 후에는 로그가 남고 모달이 자동으로 닫히도록 구성

#### ✅ Redux 상태 흐름 요약
1. dispatch(setModalData({ boardId, listId, task })) → 수정할 Task를 modalSlice에 저장
2. dispatch(setModalActive(true)) → 모달 열림
3. 사용자가 폼에서 수정 후 updateTask() or deleteTask() 실행
4. 로그 기록 (addLog) 후 setModalActive(false)로 모달 종료


---

## 📍 주제 2: LoggerModal & LogItem 컴포넌트
LoggerModal은 유저의 모든 작업 이력을 보여주는 **활동 로그 모달 창**입니다.
보드에서 작업을 수행할 때마다 로그(log)가 저장되며, 이 컴포넌트를 통해 확인할 수 있습니다.

### ✅ LoggerModal 역할
- Redux store에서 logArray를 불러와 활동 로그 표시
- 모달 닫기 버튼 제공
- 로그가 없으면 "활동기록이 없습니다." 메시지 출력

#### ✅ props 타입 정의
```ts
type TLoggerModalProps = {
  setLoggerOpen: React.Dispatch<React.SetStateAction<boolean>>;
};
```
- setLoggerOpen: 부모 컴포넌트(App)에서 isLoggerOpen 상태를 제어하는 함수 전달

#### ✅ 주요 코드 설명
```ts
const logs = useTypedSelector(state => state.logger.logArray);
```
- Redux store에서 로그 배열을 가져옴
- logArray는 사용자가 어떤 행동을 했는지를 기록한 배열

```ts
{logs.length === 0
  ? '활동기록이 없습니다.'
  : logs.map(log => <LogItem key={log.logId} />)}
```
- 로그가 하나도 없으면 안내 메시지 출력
- 로그가 존재하면 LogItem 컴포넌트를 반복 렌더링

```ts
<AiOutlineClose onClick={() => setLoggerOpen(false)} />
```
- 닫기 버튼 클릭 시 setLoggerOpen(false)를 호출하여 모달 비활성화


#### ✅ App 컴포넌트 내에서의 연동
```ts
{isLoggerOpen ? <LoggerModal setLoggerOpen={setLoggerOpen} /> : null}
```
- isLoggerOpen이 true일 때만 LoggerModal이 보임

```ts
<button
  onClick={() => setLoggerOpen(!isLoggerOpen)}
>
  {isLoggerOpen ? '활동 목록 숨기기' : '활동 목록 보이기'}
</button>
```
- 버튼을 클릭하면 활동 목록 보기 ↔ 숨기기 토글됨


### ✅ LogItem 역할
각 로그 항목을 개별적으로 보여주는 **UI 컴포넌트**입니다.
로그 작성자, 메시지, 시간 경과 정보를 표시합니다.
```tsx
type TLogItemProps = {
  logItem: ILogItem;
};
```


#### 🔹 시간 표시 로직
```ts
const timeOffset = new Date(Date.now() - Number(logItem.logTimestamp));

const showOffsetTime =
  `${timeOffset.getMinutes() > 0 ? `${timeOffset.getMinutes()}m ` : ''}` +
  `${timeOffset.getSeconds() > 0 ? `${timeOffset.getSeconds()}s ago` : ''}` +
  `${
    timeOffset.getMinutes() === 0 && timeOffset.getSeconds() === 0
      ? 'just now'
      : ''
  }`;
```
- 현재 시간과 로그 생성 시간의 차이를 계산해서
    - 방금 전, 30초 전, 3분 10초 전과 같은 형태로 보여줌

#### 🔹 전체 구조
```ts
<div className={logItemWrap}>
  <div className={author}>
    <BsFillPersonFill />
    {logItem.logAuthor}
  </div>
  <div className={message}>{logItem.logMessage}</div>
  <div className={dates}>{showOffsetTime}</div>
</div>
```

|**항목**|**설명**|
|---|---|
|logAuthor|로그 작성자|
|logMessage|로그에 담긴 메시지|
|logTimestamp|로그 생성 시간 (시간 차이 계산에 사용)|

- LoggerModal은 로그 배열을 map 돌면서 LogItem을 렌더링
- LogItem은 각 항목을 사용자 이름, 메시지, 시간 정보로 출력


---

## 📍 주제 3: 게시판 삭제 기능 (handleDeleteBoard)
이 기능은 유저가 현재 보고 있는 게시판을 삭제할 수 있도록 하는 핵심 로직입니다.
삭제와 동시에 활동 기록(Log)도 함께 저장되어 히스토리 확인이 가능합니다.

```tsx
const handleDeleteBoard = () => {
  if (boards.length > 1) {
    dispatch(deleteBoard({ boardId: getActiveBoard.boardId }));
    dispatch(
      addLog({
        logId: v4(),
        logMessage: `게시판 지우기: ${getActiveBoard.boardName}`,
        logAuthor: 'User',
        logTimestamp: String(Date.now()),
      })
    );

    const newIndexToSet = () => {
      const indexToDeleted = boards.findIndex(
        board => board.boardId === activeBoardId
      );
      return indexToDeleted === 0 ? indexToDeleted + 1 : indexToDeleted - 1;
    };
    setActiveBoardId(boards[newIndexToSet()].boardId);
  } else {
    alert('최소 게시판 갯수는 한 개입니다.');
  }
};
```

| **단계** | **설명**                             |
| ------ | ---------------------------------- |
| 1      | 현재 게시판이 삭제 가능한지 확인 (최소 1개는 유지해야 함) |
| 2      | dispatch(deleteBoard)로 해당 게시판 삭제   |
| 3      | dispatch(addLog)로 활동 로그 저장         |
| 4      | 삭제 후 포커스를 이전 또는 다음 게시판으로 이동        |
| 5      | 삭제 불가 시 alert 표시                   |

#### **📌** dispatch(deleteBoard({ boardId }))
Redux store의 boardsSlice에서 해당 게시판을 제거합니다.
```ts
deleteBoard: (state, { payload }: PayloadAction<TDeleteBoardAction>) => {
  state.boardArray = state.boardArray.filter(
    board => board.boardId !== payload.boardId
  );
};
```

#### 📌 dispatch(addLog(...))
loggerSlice에 활동 기록을 추가합니다.
uuid.v4()로 고유한 logId를 생성하고, 현재 시간을 타임스탬프로 저장합니다.

#### **📌** setActiveBoardId(...)
삭제 이후, 현재 선택된 게시판이 사라지므로 **이전 게시판 또는 다음 게시판으로 자동 포커스 전환**합니다.


### ✅ 삭제 제한 조건
```ts
if (boards.length > 1) {
  // 삭제 진행
} else {
  alert('최소 게시판 갯수는 한 개입니다.');
}
```
- 게시판이 단 하나만 있을 경우 삭제할 수 없도록 방지합니다.
- UX적으로 실수 방지를 위한 필수 조건입니다.


![LogItem](https://seonohblog.netlify.app/assets/LogItem.png)

---

## 💭 회고
• **새롭게 알게 된 점**
- Redux에서 slice별로 액션을 구조화하고, payload를 통해 데이터 흐름을 관리하는 것이 구조적으로 매우 중요하다는 것을 체감했다.
- 또한, Task/Board/Modal 간 상태 연결을 설계할 때 의도적인 상태 분리(modalSlice vs boardSlice)가 관리 측면에서 유리함을 느꼈다.

• **어렵게 느껴졌던 부분**
- updateTask와 deleteTask 로직에서 taskId, listId, boardId 간 경로 추적이 처음에는 헷갈렸다.
- 중첩된 상태 구조에서 정확히 어떤 레벨에서 task를 찾아야 하는지를 명확히 파악하는 것이 관건이었다.
  
• **다음에 학습할 주제**
- Drag And Drop 
- sort 로직
- Firebase 연결
- Login 기능 구현

### 🔗 참고자료
- 날짜 계산 로직 참고: [MDN Date.now()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now)