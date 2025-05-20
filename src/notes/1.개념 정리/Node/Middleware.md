
## 📘 Express 미들웨어 완전정복 — MVC에서의 위치와 역할까지

## ✅ 미들웨어란?

미들웨어(Middleware)는 클라이언트의 요청(req)이 Controller에 도달하기 전 또는 응답(res)을 보내기 전에
공통적으로 처리해야 할 작업을 가로채서 수행하는 함수입니다.


## 🔄 미들웨어의 실제 예
| **미들웨어 목적** | **예시 함수 이름** | **설명**          |
| ----------- | ------------ | --------------- |
| 인증          | auth.js      | JWT 토큰이 유효한지 검사 |
| 유효성 검사      | validator.js | req.body의 값 체크  |
| 에러 핸들링      | error.js     | 모든 오류를 한 곳에서 처리 |
| 로깅          | logger.js    | 요청 시간, 경로 기록    |



## 💡 미들웨어는 MVC 어디에 속할까?

✅ 미들웨어는 MVC의 핵심 계층(Model, View, Controller) 안에는 속하지 않지만,
Controller 앞단에서 동작하는 보조 계층 (Pre-controller Layer) 역할을 합니다.

❗ Controller 앞단에서 요청을 걸러내거나 가공하는 역할
❗ View나 Model과는 직접적인 관계는 없음

```bash
[사용자 요청]
   ⬇
[🛡 미들웨어 (Middleware)]
   ⬇
[🧠 컨트롤러 (Controller)]
   ⬇
[💾 모델 (Model)]
   ⬇
[응답 반환]
```



## 📁 Express MVC + Middleware 폴더 구조

```bash
book-shop/
├── app.js
├── routes/
│   └── users.js        // URL 정의 + 미들웨어 연결
├── controllers/
│   └── UserController.js
├── middlewares/
│   ├── auth.js         // JWT 인증
│   ├── validator.js    // 유효성 검사
│   └── errorHandler.js // 공통 에러 처리
├── models/
│   └── User.js
```



## 🧪 미들웨어 코드 예시
#### routes/users.js
```js
const express = require('express');
const router = express.Router();
const auth = require('../middlewares/auth');
const UserController = require('../controllers/UserController');

router.get('/mypage', auth, UserController.getMyPage);
```

#### middlewares/auth.js
```js
module.exports = (req, res, next) => {
  const token = req.cookies.token;
  if (!token) return res.status(401).json({ message: '로그인 필요' });

  try {
    const decoded = jwt.verify(token, process.env.PRIVATE_KEY);
    req.user = decoded;
    next(); // 다음으로
  } catch (err) {
    res.status(403).json({ message: '유효하지 않은 토큰' });
  }
};
```


## 🚨 주의할 점
• 미들웨어는 반드시 next()를 호출해야 다음 미들웨어나 컨트롤러로 넘어감
• express.json()처럼 전역에서 가장 먼저 적용되어야 하는 미들웨어도 존재


## ✳️ Custom Middleware 만들기
```js
// middlewares/logger.js
module.exports = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.originalUrl}`);
  next();
};
```

#### 사용 예시 (app.js):
```js
const express = require('express');
const logger = require('./middlewares/logger');

const app = express();
app.use(logger); // 전역으로 적용
```


## 🧭 전역 vs 개별 미들웨어
```js
// 전역 적용
app.use(express.json());

// 개별 적용
router.post('/join', validateUser, UserController.join);
```


## 🚨 에러 핸들링 미들웨어
```js
// middlewares/errorHandler.js
module.exports = (err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: '서버 내부 오류 발생' });
};
```

#### 사용 예시 (app.js):
```js
app.use(require('./middlewares/errorHandler'));
```
- 마지막 app.use()로 등록해야 함 (기본 라우터 이후)


## ✅ 정리

| **항목**      | **설명**                             |
| ----------- | ---------------------------------- |
| 역할          | Controller 전·후 공통 작업 처리 (전처리기)     |
| 전역 vs 개별 구분 | 모든 라우터에 적용 vs 일부 라우터에만 적용          |
| 예시          | 인증, 유효성 체크, 로그, 에러 핸들링 등           |
| MVC 내 위치    | Controller 계층의 보조 계층으로, 구조 외곽에서 동작 |


미들웨어를 잘 활용하면 복잡한 로직을 Controller에서 분리하여
**요청 흐름을 깨끗하게 정리하고, 중복을 줄이는 핵심 요소**입니다.
잘 쪼개서 적용하면 코드 가독성, 유지보수성, 확장성 모두 좋아집니다.
