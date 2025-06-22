---
tags:
  - TIL
  - React
created: 2025-06-19
---
# 📘 2025-06-19 TIL

## 📌 오늘 배운 핵심 요약
- Drag And Drop 기능
- Firebase 연결
- Login, LogOut 기능 구현
- GitHub Actions로 자동으로 Firebase에 배포

## 🧠 상세 학습 내용
## 📍 주제 1: Drag And Drop 기능

### ✅ 사용 라이브러리
- [@hello-pangea/dnd](https://github.com/hello-pangea/dnd)
→ react-beautiful-dnd의 커뮤니티 포크로 유지보수가 활발함
```bash
npm install @hello-pangea/dnd
```

### ✅ 필수 컴포넌트
| **컴포넌트**        | **설명**                      |
| --------------- | --------------------------- |
| DragDropContext | 전체 Drag-and-Drop 범위를 감싼다    |
| Droppable       | 드롭 가능한 영역 설정 (ex. 리스트)      |
| Draggable       | 드래그 가능한 항목 설정 (ex. 카드, 태스크) |

```tsx
<DragDropContext onDragEnd={handleDragEnd}>
  <Droppable droppableId="list-1">
    {(provided) => (
      <div ref={provided.innerRef} {...provided.droppableProps}>
        {tasks.map((task, index) => (
          <Draggable key={task.id} draggableId={task.id} index={index}>
            {(provided) => (
              <div
                ref={provided.innerRef}
                {...provided.draggableProps}
                {...provided.dragHandleProps}
              >
                {task.name}
              </div>
            )}
          </Draggable>
        ))}
        {provided.placeholder}
      </div>
    )}
  </Droppable>
</DragDropContext>
```


---

### ✅ 드래그 상태를 관리하는 onDragEnd 함수
```tsx
const handleDragEnd = (result: DropResult) => {
  const { destination, source, draggableId } = result;

  // destination이 없으면 drop하지 않은 상태 → 무시
  if (!destination) return;

  dispatch(
    sort({
      boardIndex: boards.findIndex(board => board.boardId === activeBoardId),
      droppableIdStart: source.droppableId,      // 시작 리스트 ID
      droppableIdEnd: destination.droppableId,   // 도착 리스트 ID
      droppableIndexStart: source.index,         // 시작 위치
      droppableIndexEnd: destination.index,      // 도착 위치
      draggableId,
    })
  );
};
```




### ✅ sort 리듀서로 Drag & Drop 상태 반영
드래그로 위치를 바꾸는 것만으로는 UI에만 반영되고, 실제 데이터는 변하지 않는다.
이를 위해 redux 리듀서에서 상태를 갱신해줘야 한다.
이를 위해 우리는 onDragEnd에서 dispatch(sort(...))를 호출하고, 그 내부에서 tasks 배열을 직접 재배치한다.

#### 🔹 Payload 타입 정의
```ts
type TSortAction = {
  boardIndex: number;
  droppableIdStart: string;
  droppableIdEnd: string;
  droppableIndexStart: number;
  droppableIndexEnd: number;
  draggableId: string;
};
```



#### 🔹 boardsSlice.ts 내 sort 리듀서
```tsx
sort: (state, { payload }: PayloadAction<TSortAction>) => {
  const board = state.boardArray[payload.boardIndex];

  if (payload.droppableIdStart === payload.droppableIdEnd) {
    // 같은 리스트 내 이동
    const list = board.lists.find(list => list.listId === payload.droppableIdStart);
    if (list) {
      const dragged = list.tasks.splice(payload.droppableIndexStart, 1)[0];
      list.tasks.splice(payload.droppableIndexEnd, 0, dragged);
    }
  } else {
    // 다른 리스트 간 이동
    const sourceList = board.lists.find(list => list.listId === payload.droppableIdStart);
    const destList = board.lists.find(list => list.listId === payload.droppableIdEnd);

    if (sourceList && destList) {
      const dragged = sourceList.tasks.splice(payload.droppableIndexStart, 1)[0];
      destList.tasks.splice(payload.droppableIndexEnd, 0, dragged);
    }
  }
}
```


---

## 📍 주제 2: Firebase 연결하기
### ✅ Firebase란?
Firebase는 Google에서 제공하는 BaaS(Backend-as-a-Service)로, 인증, DB, 호스팅, 알림 등 다양한 기능을 제공함. 우리는 이 중 **Authentication(인증)** 기능을 사용해서 로그인 기능을 구현함.

### ✅ Firebase 설정 절차
1. [Firebase 콘솔](https://console.firebase.google.com/) 접속 → **프로젝트 생성**
2. 왼쪽 사이드바 → **Authentication** → 로그인 제공업체 → **Google 활성화**
3. 프로젝트 설정 → **웹 앱 등록** → config 객체 복사

### ✅ 프로젝트에 적용
```bash
npm install firebase
```

```ts
// src/firebase.ts
import { initializeApp } from 'firebase/app';

const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  authDomain: 'YOUR_PROJECT_ID.firebaseapp.com',
  projectId: 'YOUR_PROJECT_ID',
  storageBucket: 'YOUR_PROJECT_ID.appspot.com',
  messagingSenderId: 'SENDER_ID',
  appId: 'APP_ID',
};

export const app = initializeApp(firebaseConfig);
```
이제부터 이 app 객체를 통해 Firebase 기능을 사용할 수 있ek.

---

## 📍 주제 3: Login 기능 구현하기
### ✅ 사용하는 기능
- getAuth : 인증 인스턴스를 가져오는 함수
- GoogleAuthProvider : 구글 로그인 제공자 생성
- signInWithPopup : 팝업으로 로그인 창 열기

### ✅ 전체 흐름
1. 사용자가 로그인 버튼 클릭
2. 구글 팝업이 뜨고 로그인
3. 로그인 성공 시 유저 객체 반환 → 이메일, uid 등 포함
4. Redux에 저장

### ✅ 코드
```ts
import { getAuth, GoogleAuthProvider, signInWithPopup } from 'firebase/auth';
import { app } from './firebase';

const auth = getAuth(app);
const provider = new GoogleAuthProvider();

signInWithPopup(auth, provider)
  .then(result => {
    const user = result.user;
    console.log('유저 이메일:', user.email);
    console.log('유저 uid:', user.uid);
  })
  .catch(error => {
    console.error('로그인 실패:', error.message);
  });
```
실무에서는 로그인 실패 시 Toast로 안내하거나, 에러 코드를 별도로 처리하기도 한다.


---

## 📍 주제 4: Redux Store에 유저 데이터 넣기
### ✅ 왜 Redux에 저장하는가?
- 로그인 상태를 유지하려면 앱 전역에서 유저 정보를 쉽게 접근할 수 있어야 함
- 이후 게시판 생성, 태스크 수정 등에도 유저 ID가 필요할 수 있음

### ✅ slice 만들기
```ts
// store/slices/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

type TUserState = {
  id: string;
  email: string;
};

const initialState: TUserState = {
  id: '',
  email: '',
};

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, { payload }: PayloadAction<TUserState>) => {
      state.id = payload.id;
      state.email = payload.email;
    },
    removeUser: state => {
      state.id = '';
      state.email = '';
    },
  },
});

export const { setUser, removeUser } = userSlice.actions;
export const userReducer = userSlice.reducer;
```

### ✅ 유저 정보 저장하기
```ts
dispatch(
  setUser({
    email: user.email ?? '',
    id: user.uid,
  })
);
```
Firebase는 종종 email이 undefined인 경우가 있기 때문에 null 체크 필요하다.


---

## 📍 주제 5: LogOut 기능 구현하기
### ✅ 로그아웃 흐름
1. 로그아웃 버튼 클릭
2. Firebase의 signOut()으로 서버 로그아웃
3. Redux의 user slice에서 유저 정보 제거

### ✅ 코드
```ts
import { signOut, getAuth } from 'firebase/auth';
import { removeUser } from '../store/slices/userSlice';

const auth = getAuth();

signOut(auth)
  .then(() => {
    dispatch(removeUser());
    console.log('로그아웃 완료');
  })
  .catch(error => {
    console.error('로그아웃 실패:', error.message);
  });
```

### ✅ 로그아웃 이후 처리할 수 있는 것
- 로그인 화면으로 리다이렉트
- Toast 알림 띄우기
- BoardList UI에서 Google 아이콘 다시 보이게 하기

---

## ✅ 정리
|**기능**|**역할**|
|---|---|
|Firebase|로그인 백엔드 역할|
|GoogleAuthProvider|구글 인증을 위한 provider|
|Redux user slice|로그인한 유저 상태 전역 저장|
|signInWithPopup / signOut|로그인 / 로그아웃 API 실행|
|useAuth|로그인 여부를 컴포넌트에서 쉽게 확인|


---

## 📍 주제 6: 배포하기
### ✅ GitHub Actions + Firebase Hosting
- main 브랜치에 코드를 푸시하면 GitHub Actions가 자동으로 Firebase에 배포를 실행함
- .github/workflows/firebase-hosting-merge.yml 파일로 배포 자동화 설정

### ✅ 설정 필수 사항
- firebase.json: public 경로로 react-task-app/dist 지정
- .firebaserc: 프로젝트 ID react-task-app-ae979 포함
- GitHub Secrets에 FIREBASE_SERVICE_ACCOUNT_REACT_TASK_APP 등록

### ✅ 실행 흐름
1. npm run build로 dist 디렉터리 생성
2. GitHub Actions가 dist를 Firebase Hosting에 업로드
3. PR에서는 Preview 채널로 임시 배포 진행됨

### 🔗 배포 URL

- [https://react-task-app-ae979.web.app](https://react-task-app-ae979.web.app)


![boardLogin](https://seonohblog.netlify.app/assets/boardLogin.png)

---

## 💭 회고
• **새롭게 알게 된 점**
- Firebase Hosting과 GitHub Actions를 연동해 CI/CD 환경을 손쉽게 구성할 수 있다는 점
- Redux를 통해 전역 상태를 깔끔하게 관리하는 방법
- Drag-and-Drop 라이브러리의 실제 사용 방식

• **어렵게 느껴졌던 부분**
- Firebase 배포 시 dist 폴더 경로 문제로 에러가 반복되었음
- GitHub Actions의 설정 위치(working-directory, Secret 키 등)를 올바르게 작성하는 것이 까다로웠음
- Drag 위치 상태를 Redux로 일치시키는 로직 설계
  
• **다음에 학습할 주제**
- Book Store React(TypeScript) 동적 UI 개발

### 🔗 참고자료
- [@hello-pangea/dnd 공식 문서](https://github.com/hello-pangea/dnd)
- [Firebase Hosting 배포 문서](https://firebase.google.com/docs/hosting)
- [Redux Toolkit 공식 문서](https://redux-toolkit.js.org/)
- [GitHub Actions Docs](https://docs.github.com/en/actions)