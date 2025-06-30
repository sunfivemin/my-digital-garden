---
tags:
  - TIL
created: 2025-04-24
---
 
# 📘 2025-04-24 TIL

## 📌 오늘 배운 핵심 요약
- **Express Router의 역할과 구조**를 직접 실습하고, 코드 분리의 필요성을 체감
- 기능별 라우터 모듈 분리로, **URL 자동 관리와 유지보수의 장점**을 익힘
- **app.use()를 통한 경로 연결**의 동작 원리를 이해함
- 실제 회원-채널 **1:N 관계의 ERD 설계**를 그림과 표로 정리


## 🧠 상세 학습 내용

## 📍 주제 1:  Express Router (라우터)와 URL 구조

### 1. 라우터(router)란?
- Express에서 “경로(주소)와 HTTP 메서드별로 실행될 코드를 나눠 관리”하는 역할을 한다.
- 예를 들어, 회원과 채널 관련 API를 각각 다른 파일(users.js, channels.js)로 분리하면 코드가 훨씬 깔끔해진다.
#### ✅ 왜 라우터를 써야 할까?
• 프로젝트가 커질수록 한 파일에 모든 API 코드를 작성하면 유지보수/확장성이 떨어진다.
• 기능별로 라우터를 분리하면 팀 작업, 버전 관리, 디버깅까지 편리하다.

#### /routes/users.js
```js
const express = require('express');
const router = express.Router();

router.post('/join', ...);   // 회원가입
router.post('/login', ...);  // 로그인
router.get('/users', ...);   // 전체 회원 조회

module.exports = router;
```

#### /routes/channels.js
```js
const express = require('express');
const router = express.Router();

router.post('/', ...);          // 채널 생성
router.get('/', ...);           // 전체/내 채널 목록
router.put('/:id', ...);        // 채널 수정
router.delete('/:id', ...);     // 채널 삭제

module.exports = router;
```

---

#### ✅ app.use()와 URL 구조 연결
```js
const express = require('express');
const app = express();
app.use(express.json());

const userRouter = require('./routes/users');
const channelRouter = require('./routes/channels');

app.use('/', userRouter); // 회원 관련 경로 담당 (/join, /login 등)
app.use('/channels', channelRouter); // 채널 관련 경로 담당 (/channels, /channels/:id 등)

app.listen(7777, () => {
  console.log("서버가 7777번 포트에서 실행 중입니다.");
});
```
- /로 시작하는 모든 회원 관련 API는 **userRouter**가 담당
- /channels로 시작하는 모든 API는 **channelRouter**가 담당

---

#### **🟡** GET 요청에서 쿼리 파라미터 사용하기

**GET 요청에서는 body 대신** URL에 쿼리스트링을 붙여서 파라미터를 전달해야 한다!
- 예시: GET /channels?userId=tester2


---
## 📍 주제 2.  회원마다 채널 가지게 ERD 설계

#### 1. ERD(개체-관계 다이어그램)란?
- **회원(유저)** : 여러 개의 유튜브 채널을 가질 수 있다 (1:N 관계)
- **채널** : 반드시 한 명의 회원(소유자)을 가진다

#### 2. 테이블(모델) 예시

#### 1) 회원(User) 테이블

| **user_id** | **password** | **name** |
| ----------- | ------------ | -------- |
| testId1     | 1234         | tester1  |
| testId2     | 5678         | tester2  |

#### 1) 채널(Channel) 테이블
| **id** | **channel_title** | **user_id** | **sub_num** | **video_num** |
| ------ | ----------------- | ----------- | ----------- | ------------- |
| 1      | 달려라               | testId1     |             |               |
| 2      | 뛰어라               | testId1     |             |               |
| 3      | 걸어라               | testId2     |             |               |
- **user_id**: 회원 테이블의 기본키(PK)이며, 채널 테이블에서는 외래키(FK)로 사용
→ 이 값을 통해 “이 채널의 소유자”를 바로 알 수 있음

### 3. ERD 구조 관계 요약
```bash
[User] 1 ------ N [Channel]
회원(user_id)   <--- FK ---   채널(user_id)
```


![250424](https://seonohblog.netlify.app/assets/250424.png)


---

## **💭 회고**

• **새롭게 알게 된 점**
- GET 요청에서는 body를 사용하지 않고, 반드시 **쿼리 파라미터(query string)**로 데이터를 전달해야 한다는 점
- Express에서 라우터를 분리해서 관리하면, 실제로 협업이나 기능 추가가 훨씬 편리해진다.
- ERD를 직접 그려보니 1:N(회원-채널) 구조와 FK(외래키) 개념이 한눈에 들어옴
  
• **어렵게 느껴졌던 부분**
- 처음엔 POST/GET/PUT/DELETE 등 메서드별로 **파라미터 위치(body, params, query) 차이**가 헷갈렸으나, Postman 실습으로 정리가 잘 됐다.
- 라우터 분리 시 app.use() 경로 관리에서 한번에 개념을 잡기가 조금 헷갈렸다.

• **다음에 학습할 주제**
- DBMS(데이터베이스 관리시스템)란?
- RDBMS 사용하는 이유
- PK, 데이터 중복, 정규화
- 테이블 분리, 장단점
- 관계형 모델(1-N, 관계의 주인)과 예시
- MySQL Workbench 실습/설치 및 유튜브 프로젝트 DB 설계
