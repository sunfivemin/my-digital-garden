---
title: 상태 관리
tags:
  - frontend
  - html
---
  - Redux,Zustand

## Pure Redux
### 플럭스 패턴 
: action -> dispatcher -> store -> view -> action -> dispatcher

### 리덕스
: action -> reducer -> store -> view
 플럭스 패턴으로 리덕스를 만들었다.

slice : 도메인단위(기능단위)로 분류하여 리듀서들 모아둔것.
reducer : 리듀서를 하나로 합쳐서 index.ts에서 store에 리듀서 전달 

## 리덕스 툴킷
계속 반복되는 보일러플레이트를 최소화하는 장점이 있어서 쓴다.

### hooks 
redux.ts : useDispatch를 만들기위한 훅, dispatch라는 이름으로 담아주고 액션을 전달해주는 역할이다.
useAuth.ts : 

아무나 바꾸면 안되니까 디스패치나 리듀서를 통해서만 쓸수잇다는 개념.

스토어라는 하나의 객체에서 관리가 되어있고 리듀서를 거친다.

state는 전역상태로 관리할것만 저장해야한다.
Redux,Zustand 모두 전역상태 조작 도구들이다. 
전역은 useState로 관리하면된다.
리덕스는 플러스패턴 , 리코일은 아토믹 패션

Store를 구독하는 모든 컴포넌트가 리랜더링 되면 리덕스가 안좋은거 아닌가요? 
-> 안정성이 있다. 



npm install redux
node index.js
```js
const { createStore } = require('redux');

// 초기 state 를 정의
const initState = {
	name: '김코딩',
	post: []
}

// action 
// 객체. 액션 객체를 리턴하는 Action creator 함수를 작성
// 그 액션에 필요한 데이터를 넘기는 역할, 어떤 액션을 할 것인지 타입 지정
const changeUsername = (name) => {
	return {
		type: "CHANGE_NAME",
		name
	}
}

const addPost = (post) => {
	return {
		type: "ADD_POST",
		post
	}
}

// reducer
// 액션의 타입에 따라서 새로운 State를 생성해내는 순수 함수
const reducer = (prevState, action) => {
	switch (action.type) {
		case "CHANGE_NAME":
			return {
				...prevState,
				name: action.name
			}
		case "ADD_POST":
			return {
				...prevState,
				name: [...prevState.posts, action.post]
			}
		default:
			return prevState;
	}
}

// store
const store = createStore(reducer, initState) 


// dispatch
store.dispatch(changeUsername('steve'))
store.dispatch(addPost('post 1'))
store.dispatch(addPost('post 2'))

console.log(store.getState())
```


### **✅ 페이로드(payload)란?**
Redux나 API에서 **“내가 지금 무슨 데이터를 보내고 싶어”** 라고 전달하는 **실제 데이터**입니다.

---
### **✅ Redux에서 payload의 의미**

Redux에서는 액션 객체가 이렇게 생겼어요:
```ts
{
  type: 'UPDATE_TASK',   // 어떤 액션인지 식별
  payload: {             // 그 액션에 필요한 데이터
    taskId: 'task-1',
    taskName: '새로운 이름',
    ...
  }
}
```
즉, payload는 내가 하고 싶은 액션에 필요한 구체적인 정보(데이터)를 담고 있어요.

---

### **✅ 그때그때 바뀌는 이유?**
사용자의 행동에 따라 계속 바뀝니다.

|**상황**|**발생하는 액션**|**payload 내용**|
|---|---|---|
|Task 제목 수정|updateTask()|수정된 task 객체|
|Task 삭제|deleteTask()|삭제할 task의 ID|
|모달 열기|setModalData()|선택한 task의 boardId, listId, task 전체|
이렇게 매 순간 **그 시점에서 필요한 데이터만 payload에 담아** dispatch합니다.

---

### **✅ 업데이트는 왜 필요할까?**
Redux는 “불변성”을 유지하면서 상태를 업데이트해야 하므로,
**기존 상태는 건드리지 않고 새로운 상태를 만들어주는 역할**을 payload가 하게 됩니다.

예를 들어:
```ts
dispatch(updateTask({ boardId, listId, task }))
```
→ 이 payload 덕분에 reducers에서 정확히 어떤 task를 바꿀지 판단할 수 있어요.

- payload는 택배 상자의 **내용물**이고,
- action type은 상자에 붙은 **스티커**(‘업데이트’, ‘삭제’ 등)입니다.






프론트엔드 과제 전형 기출 문제 리스트
 퍼널 구조 / useFunnel 
 디바운싱과 쓰로틀링 - 요청 받은것에 대해 최적화 
  살펴보면 퀄리티 향상에 도움이 된다.

[https://m.blog.naver.com/topblade71/222964598090](https://m.blog.naver.com/topblade71/222964598090)
[https://m.blog.naver.com/topblade71/222964598090](https://m.blog.naver.com/topblade71/222964598090)




