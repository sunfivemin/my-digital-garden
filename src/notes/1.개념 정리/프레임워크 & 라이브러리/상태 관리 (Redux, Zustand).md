---
title: 상태 관리
tags:
  - frontend
  - html
---
  - Redux,Zustand
Pure Redux
플럭스 패턴 : action -> dispatcher -> store -> view -> action -> dispatcher

리덕스 : action -> reducer -> store -> view
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

프론트엔드 과제 전형 기출 문제 리스트
 퍼널 구조 / useFunnel 
 디바운싱과 쓰로틀링 - 요청 받은것에 대해 최적화 
  살펴보면 퀄리티 향상에 도움이 된다.

[https://m.blog.naver.com/topblade71/222964598090](https://m.blog.naver.com/topblade71/222964598090)
[https://m.blog.naver.com/topblade71/222964598090](https://m.blog.naver.com/topblade71/222964598090)