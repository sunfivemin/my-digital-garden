---
tags:
  - TIL
created: 2025-05-14
---

# 📘 2025-05-14 TIL

## 📌 오늘 배운 핵심 요약
- HTTP 상태 코드의 역할과 의미를 학습
- Express 서버에서 http-status-codes 라이브러리를 사용해 상태 코드를 더 명확하고 가독성 좋게 관리하는 방법을 실습
- 회원가입 시 비밀번호를 안전하게 암호화해 DB에 저장하는 실습 (crypto, pbkdf2 사용)

## 🧠 상세 학습 내용

## 📍 주제 1: http-status-codes
http-status-codes는 Node.js, Express 등 서버 개발 시 HTTP 상태 코드를 **숫자 대신 의미 있는 상수로 관리**할 수 있게 해주는 라이브러리.

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

---

## 📍 주제 2:  회원가입 API에 적용하기 (비밀번호 암호화)

#### 기존 코드 (숫자 직접 사용)
```js
res.status(400).end();
res.status(201).json(results);
```

#### 개선된 코드 (http-status-codes 사용)
```js
const conn = require('../mariadb');
const { StatusCodes } = require('http-status-codes');
const crypto = require('crypto');

const join = (req, res) => {
  const { email, password } = req.body;

  // salt 생성 (64바이트 → base64)
  const salt = crypto.randomBytes(64).toString('base64');

  // pbkdf2 암호화 (10000회 해싱, 64바이트, sha512 → base64 인코딩)
  const hashPassword = crypto
    .pbkdf2Sync(password, salt, 10000, 64, 'sha512')
    .toString('base64');

  // DB에 저장 (주의: password, salt 컬럼은 VARCHAR(255) 이상이어야 함)
  let sql = 'INSERT INTO users (email, password, salt) VALUES (?, ?, ?)';
  let values = [email, hashPassword, salt];

  conn.query(sql, values, (err, results) => {
    if (err) {
      console.log(err);
      return res.status(StatusCodes.BAD_REQUEST).end();
    }
    return res.status(StatusCodes.CREATED).json(results);
  });
};
```

![bookshopJoin](https://seonohblog.netlify.app/assets/bookshopJoin.png)


---

## 📍 주제 3:  node.js 패키지 구조 (feat. 컨트롤러)
### **✅ 기본 구조**
- app.js : 메인 서버, 라우터 등록
- /routes : URL 관리 (users.js, books.js)
- /controller : 비즈니스 로직 담당 (UserController.js)

#### ✔ 라우터가 로직까지 다 하면 생기는 문제
- 코드가 장황해지고 관리 어려움
- 가독성, 유지보수 최악
- 에러 발생 시 원인 파악 힘듦

#### 💡 해결 방법
- **라우터 → 경로 관리 (최소한의 역할)**
- **컨트롤러 → 비즈니스 로직만 관리**
- 컨트롤러 내부에서도 DB 로직과 분리 권장 (DAO 패턴 등 활용 가능)

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


🚀 **장점 요약**
- **라우터 → URL 관리 집중**
- **컨트롤러 → 로직 분리로 가독성↑, 유지보수 쉬움**
- **에러 발생 시 디버깅, 테스트도 수월**


---

## 📍 주제 4: StatusCodes.UNAUTHORIZED (401)

### ✅ 401 Unauthorized의 의미 (공식 표준)
- 401 Unauthorized
    - **“클라이언트가 인증되지 않았음”**
    - 서버가 요청을 이해했으나, **인증 자격 증명(토큰, 세션, 쿠키 등)이 없거나 유효하지 않아 요청이 거부됨**
    - 대표적인 사용 상황: **로그인 실패, 잘못된 토큰, 토큰 없음, 토큰 만료 등**

💡 참고: “Unauthorized”는 ‘인증되지 않음’이라는 의미이며, ‘권한 없음’과는 다름

### ❗️ 401과 403의 혼동 주의
| **코드** | **공식 명칭 (영어)** | **의미 (한글)**           | **사용 상황 예시**                      |
| ------ | -------------- | --------------------- | --------------------------------- |
| 401    | Unauthorized   | 인증되지 않은 상태 (로그인 실패 등) | 로그인 실패, 잘못된 토큰, 토큰 없음             |
| 403    | Forbidden      | 인증은 되었지만 권한이 없는 상태    | 로그인은 했지만 권한 부족 (예: 관리자 전용 페이지 접근) |
- **401 Unauthorized**
    → **“너 누구인지 모름. 인증하고 와라”**

- **403 Forbidden**
    → **“누구인지는 알지만, 넌 이 리소스에 접근할 권한이 없다”**

![bookshopLogin](https://seonohblog.netlify.app/assets/bookshopLogin.png)


---

## 📍 주제 5: 회원가입 시 비밀번호 암호화 (pbkdf2 + salt)

### ✅ 왜 비밀번호를 암호화해야 하나?
- 사용자의 비밀번호를 평문(Plain text)으로 저장하면 **치명적인 보안 리스크 발생**.
- 해킹 시 사용자 비밀번호가 그대로 유출됨.
- 그래서 **암호화 + salt** 방식을 사용하여 해커의 공격(무차별 대입, rainbow table 등)을 어렵게 만듦.

---

### 🔐 pbkdf2 + salt 개념 정리

| **개념**     | **설명**                                                                                       |
| ---------- | -------------------------------------------------------------------------------------------- |
| **Salt**   | 비밀번호에 **랜덤 문자열(Salt)를 추가해서 같은 비밀번호도 다른 암호화 결과를 만듦**                           |
| **PBKDF2** | 비밀번호 + salt를 **수천 번(예: 10,000회) 반복해서 해싱 → 해커가 brute-force 시도 시 시간 오래 걸림** |

---

### 📌 주요 보안 이슈와 해결
| **문제**                         | **해결 방법**                           |
|------------------------------|------------------------------------|
| 비밀번호만 단순 해싱 시, 같은 비밀번호 → 같은 해시 발생 | **Salt 추가 → 같은 비밀번호도 다른 결과**   |
| 해커가 rainbow table 공격 시도 가능            | **PBKDF2 반복 해싱 → 공격 시 계산 시간 증가** |

---

### ✅ 회원가입 시 암호화 적용 시 주의할 점
- 비밀번호는 **단순 해시가 아니라 pbkdf2 + salt로 암호화 후 저장**.
- salt도 함께 DB에 저장
- 로그인 시 **입력 비밀번호를 동일하게 암호화하여 비교**.
- 암호화 결과는 길이가 길기 때문에 **DB 칼럼은 VARCHAR(255) 이상 사용 권장**.

---

### ✔ 비밀번호 검증 흐름 (로그인 시)
```js
// 로그인 시 입력된 비밀번호 + 저장된 salt로 pbkdf2 해싱 후 DB 값과 비교
const hashInputPassword = crypto.pbkdf2Sync(inputPassword, storedSalt, 10000, 64, 'sha512').toString('base64');

if (hashInputPassword === storedPassword) {
  // 로그인 성공
} else {
  // 비밀번호 불일치
}
```



![bookshopCrypto](https://seonohblog.netlify.app/assets/bookshopCrypto.png)


### ✔  비밀번호 초기화 API 개선
```js
const passwordReset = (req, res) => {
  const { email, password } = req.body;

  let sql = 'UPDATE users SET password=?, salt=? WHERE email = ?';

  const salt = crypto.randomBytes(64).toString('base64');
  const hashPassword = crypto
    .pbkdf2Sync(password, salt, 10000, 64, 'sha512')
    .toString('base64');

  let values = [hashPassword, salt, email];

  conn.query(sql, values, (err, results) => {
    if (err) {
      console.log(err);
      return res.status(StatusCodes.BAD_REQUEST).end();
    }
    if (results.affectedRows == 0) {
      console.log('해당 이메일 없음');
      return res.status(StatusCodes.BAD_REQUEST).end();
    }
    return res.status(StatusCodes.CREATED).json(results);
  });
};
```



---

## **💭 회고**

### • 새롭게 알게 된 점
- 상태 코드를 숫자가 아닌 상수로 관리하면 협업 시 코드 가독성도 좋아지고 실수도 줄어든다는 걸 느낌.
- 특히 400, 500 같은 에러 코드일수록 숫자만 보면 헷갈릴 수 있는데, 상수를 쓰니까 정확히 어떤 의미인지 명확하게 알 수 있어서 좋았음.
- Express 라우터-컨트롤러 분리 원칙 이해: URL 찾는 역할과 비즈니스 로직은 무조건 나눠야 한다.
- pbkdf2 + salt = 비밀번호에 랜덤 추가하고 수천 번 암호화 → 해커가 깨기 어렵게 만든다.

### • 어렵게 느껴졌던 부분
- http-status-codes가 처음에는 불필요하게 느껴졌지만, 실제 실습하면서 코드를 다시 볼 때 편하고 가독성이 좋아진 걸 느꼈음.
- Express에서 응답은 무조건 한 번만 할 수 있다는 규칙(중복 응답 금지)을 또 실습에서 체감.
- pbkdf2 + salt 사용으로 비밀번호 암호화 과정을 안전하게 처리하는 흐름을 실습하면서 암호화 결과가 길어지므로 DB 컬럼 크기도 고려해야 한다는 점을 새롭게 인지.
- 비밀번호 초기화(Reset) API에서도 동일하게 암호화 로직 적용하고, email이 존재하지 않을 경우 `affectedRows`를 통해 정확히 에러를 핸들링하도록 개선.

### • 다음에 학습할 주제
- 도서 전체 조회, 상세 도서 조회
- 카테고리 전체 목록 조회 API 구현


## 🔗 참고 자료
[http-status-codes](https://www.npmjs.com/package/http-status-codes)