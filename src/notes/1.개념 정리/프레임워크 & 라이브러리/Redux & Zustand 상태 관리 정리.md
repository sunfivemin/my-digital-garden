---
title: 상태 관리
tags:
  - frontend
---
# 상태 관리 (Redux & Zustand)

## 🧭 상태 관리란?
상태 관리는 **애플리케이션의 UI와 데이터 간의 동기화**를 유지하는 핵심 역할입니다. 특히 전역적으로 공유해야 할 데이터(로그인 정보, 장바구니, 모달 상태 등)를 다룰 때 필요합니다.

## ✅ 상태 관리 전략 개요
- **Redux**: 구조적이고 명확한 상태 관리, 복잡한 상태 트리와 미들웨어 처리가 필요한 경우에 적합
- **Zustand**: 간단한 API와 빠른 설정, 로직 분리가 자유롭고 보일러플레이트가 없음
- **useState**: 컴포넌트 내부의 간단한 상태 관리

---

### 🟩 Redux 사용 프로젝트

https://react-task-app-ae979.web.app/
- 🔗 [jslearner](https://github.com/sunfivemin/jslearner)
- 복잡한 전역 상태 트리와 액션 구조가 필요한 경우 적용

### 🟦 Zustand 사용 프로젝트

- 🔗 [book-store](https://github.com/sunfivemin/book-store)
- 장바구니, 모달, Toast 등 간단한 전역 상태 공유가 핵심일 때 적용

---

## ⚙️ Redux: Flux 아키텍처 기반의 상태 관리 라이브러리

### 🔁 Flux 패턴
```
action → dispatcher → store → view → action
```

### 🔁 Redux 흐름
```
action → reducer → store → view
```

 Flux 패턴을 단순화하여 Redux가 만들어졌습니다.

### 🔸 핵심 개념 정리

- **Store**: 상태를 담고 있는 객체
- **Action**: 상태에 어떤 변화가 필요한지 설명하는 객체
- **Reducer**: 액션을 처리하여 새로운 상태를 반환하는 순수 함수
- **Dispatch**: 액션을 리듀서로 전달하는 메서드
- **Slice**: 리덕스 툴킷에서 도메인 단위로 리듀서를 나누는 단위
- **dispatch**: 액션을 store에 전달하는 함수


### 🔹 Redux Toolkit 
- 반복되는 코드(boilerplate)를 줄이기 위한 Redux 공식 도구
- `createSlice`, `configureStore`, `createAsyncThunk` 등 유틸 API를 통해 보일러플레이트 감소
- Immer를 통해 불변성 유지

```ts
const counterSlice = createSlice({
  name: 'counter',
  initialState: 0,
  reducers: {
    increment: state => state + 1,
    decrement: state => state - 1,
  },
});
```

---

## 💡 Zustand: 초간단 글로벌 상태 관리

Zustand는 Redux보다 훨씬 **가볍고 직관적인 상태 관리 라이브러리**입니다. 특히 React에서 **Context 없이도 전역 상태를 다룰 수 있는** 점이 특징입니다.

### ✅ 기본 사용법

```ts
import { create } from 'zustand';

export const useCartStore = create(set => ({
  items: [],
  addToCart: (item) => set(state => ({ items: [...state.items, item] })),
}));
```

### ✅ 장점

- Redux에 비해 **보일러플레이트가 거의 없음**
- **selector로 리렌더링 최적화** 가능
- React Context API보다 덜 복잡하고 성능 최적화에 유리

---

## 🧠 Redux vs Zustand 비교

|항목|Redux|Zustand|
|---|---|---|
|코드 복잡도|복잡 (툴킷으로 개선 가능)|매우 단순|
|러닝 커브|중간 이상|낮음|
|상태 구조화|명시적 (slice/reducer)|자유도 높음|
|비동기 처리|thunk, saga 등 미들웨어 필요|자체 지원 (비동기 함수에서 set 사용 가능)|
|리렌더링 제어|가능하지만 추가 설정 필요|selector로 간단하게 제어 가능|

---

## 🚀 Redux 작동 예시

```ts
const changeUsername = (name) => ({
  type: "CHANGE_NAME",
  payload: name,
});

const reducer = (state, action) => {
  switch (action.type) {
    case "CHANGE_NAME":
      return { ...state, name: action.payload };
    default:
      return state;
  }
};

const store = createStore(reducer, { name: '김코딩' });

store.dispatch(changeUsername('Steve'));
```

---

## 📦 payload란?

"지금 이 액션을 처리하는 데 필요한 데이터를 담은 객체"

```ts
{
  type: 'UPDATE_TASK',
  payload: {
    taskId: 'task-1',
    taskName: '새로운 이름',
  }
}
```

- **payload는 변화에 필요한 정보를 담은 상자**입니다.
- 매번 사용자의 행동에 따라 다르게 채워짐

---

## 📌 병행 사용 전략: Redux or Zustand + useState

 "전역으로 공유되어야 하는가?"를 기준으로 전역(Zustand/Redux) vs 지역(useState) 상태를 나누면 좋습니다.

```ts
const [loading, setLoading] = useState(false);  // 지역 상태
const { addToCart } = useCartStore();           // 전역 상태
```

---

## 💬 결론

- Redux는 **강력한 구조화, 예측 가능한 상태 관리**에 유리
- Zustand는 **간단하고 빠르게 전역 상태를 공유**할 수 있음
- 두 라이브러리 모두 **적절한 상황**에서 사용하면 좋은 도구입니다.

---

## 🔗 참고자료

- [Redux 공식문서](https://redux.js.org/)
- [Zustand 공식문서](https://docs.pmnd.rs/zustand/getting-started/introduction)
