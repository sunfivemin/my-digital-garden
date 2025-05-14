---
tags:
  - TIL
created: 2025-05-13
---

# 📘 2025-05-13 TIL

## 📌 오늘 배운 핵심 요약
- HTTP 상태 코드의 역할과 의미를 학습
- Express 서버에서 http-status-codes 라이브러리를 사용해 상태 코드를 더 명확하고 가독성 좋게 관리하는 방법을 실습

## 🧠 상세 학습 내용

## 📍 주제 1: http-status-codes
http-status-codes는 Node.js, Express 같은 서버에서 응답할 때 사용하는 **HTTP 상태 코드를 직관적이고 가독성 있게 관리**해주는 라이브러리다.

### ✅ 왜 사용하는가?

- 상태 코드를 숫자(200, 201, 400, 500) 대신 명확한 의미를 가진 상수(StatusCodes.OK, StatusCodes.CREATED)로 표현.
- 코드 가독성이 높아지고, 실수나 오타를 줄일 수 있음.

### ✅ 설치 방법
```bash
npm install http-status-codes
```

### ✅ 기본 사용법
```js
const { StatusCodes } = require('http-status-codes');

res.status(StatusCodes.OK).json({ message: '성공' });
res.status(StatusCodes.BAD_REQUEST).json({ message: '잘못된 요청' });
res.status(StatusCodes.CREATED).json({ message: '생성 완료' });
```

### ✅ 주요 코드 예시
| **의미**                | **코드 숫자** | **상수 사용**                         |
| --------------------- | --------- | --------------------------------- |
| OK (정상 처리)            | 200       | StatusCodes.OK                    |
| Created (생성 성공)       | 201       | StatusCodes.CREATED               |
| Bad Request (잘못된 요청)  | 400       | StatusCodes.BAD_REQUEST           |
| Unauthorized (인증 필요)  | 401       | StatusCodes.UNAUTHORIZED          |
| Internal Server Error | 500       | StatusCodes.INTERNAL_SERVER_ERROR |

## 📍 주제 2:  회원가입 API에 적용하기
#### 기존 코드 (숫자 직접 사용)
```js
res.status(400).end();
res.status(201).json(results);
```

#### 개선된 코드 (http-status-codes 사용)
```js
const { StatusCodes } = require('http-status-codes');

conn.query(sql, values, (err, results) => {
  if (err) {
    console.log(err);
    return res.status(StatusCodes.BAD_REQUEST).end();
  }
  return res.status(StatusCodes.CREATED).json(results);
});
```

![bookshopJoin](https://seonohblog.netlify.app/assets/bookshopJoin.png)

## 📍 주제 3:  node.js 패키지 구조 (feat. 컨트롤러)
- app.js : 메인 라우터 역할
- /routes : URL(경로) 찾아주는 역할
→ users.js, books.js 등 하위 라우터 존재

---
#### ✔ 라우터가 로직까지 다 하면 생기는 문제
- 코드가 길어지고 복잡해짐
- 가독성, 유지보수성 최악
- 트러블 슈팅 힘듦

#### 💡 해결 방법
라우터는 경로만 찾고, 비즈니스 로직은 콜백 함수(컨트롤러)로 분리

#### 컨트롤러의 역할 (매니저)
- 클라이언트가 요청(req) → router → 컨트롤러가 로직 처리 → 클라이언트에게 응답(res)
- 라우터는 “길 찾기” 역할만, 컨트롤러는 “처리 매니저” 역할
- 컨트롤러는 직접 처리 안 하고, DB(알바생)에게 시켜서 결과만 받아 응답

## 모듈화
#### ✔ routes/users.js (라우터 역할만)
```js
const express = require('express');
const router = express.Router();
const join = require('../controller/UserController');
const { StatusCodes } = require('http-status-codes');

router.use(express.json());
router.post('/join', join);

module.exports = router;
```

#### ✔ controller/UserController.js (컨트롤러 역할)
```js
const conn = require('../mariadb');
const { StatusCodes } = require('http-status-codes');

const join = (req, res) => {
  const { email, password } = req.body;

  let sql = 'INSERT INTO users (email, password) VALUES (?,?)';
  let values = [email, password];

  conn.query(sql, values, (err, results) => {
    if (err) {
      console.log(err);
      return res.status(StatusCodes.BAD_REQUEST).end();
    }
    return res.status(StatusCodes.CREATED).json(results);
  });
};

module.exports = join;
```


#### **💡 이렇게 분리하면?**

- 라우터 → **URL만 관리**
- 컨트롤러 → **로직만 관리**
- **깨끗한 구조 → 유지보수, 트러블 슈팅 편해짐**




![bookshopLogin](https://seonohblog.netlify.app/assets/bookshopLogin.png)


---

## **💭 회고**

### • 새롭게 알게 된 점
- 상태 코드를 숫자가 아닌 상수로 관리하면 협업 시 코드 가독성도 좋아지고 실수도 줄어든다는 걸 느낌.
- 특히 400, 500 같은 에러 코드일수록 숫자만 보면 헷갈릴 수 있는데, 상수를 쓰니까 정확히 어떤 의미인지 명확하게 알 수 있어서 좋았음.
- Express 라우터-컨트롤러 분리 원칙 이해: URL 찾는 역할과 비즈니스 로직은 무조건 나눠야 한다.

### • 어렵게 느껴졌던 부분
- http-status-codes가 처음에는 불필요하게 느껴졌지만, 실제 실습하면서 코드를 다시 볼 때 편하고 가독성이 좋아진 걸 느꼈음.
- Express에서 응답은 무조건 한 번만 할 수 있다는 규칙(중복 응답 금지)을 또 실습에서 체감.

### • 다음에 학습할 주제
- 


## 🔗 참고 자료
[http-status-codes](https://www.npmjs.com/package/http-status-codes)