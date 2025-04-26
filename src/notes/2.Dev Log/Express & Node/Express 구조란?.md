
# 📘 Express 프로젝트 구조란?

Express는 Node.js에서 **백엔드 API 서버를 만들 때 가장 많이 쓰는 웹 프레임워크**입니다.
프로젝트가 커질수록 “기능별로 코드를 나누어 관리”하는 게 핵심입니다.

### 🗂️ 대표적인 구조
```bash
/project-root
  ├── app.js             # 서버 진입점, 모든 라우터/미들웨어 연결
  ├── routes/            # URL별 기능(회원, 채널 등) 라우터 파일 모음
  │     ├── users.js
  │     └── channels.js
  ├── controllers/       # 실질적 비즈니스 로직 (데이터 검증, 응답 등)
  │     ├── userController.js
  │     └── channelController.js
  ├── models/            # 데이터 관리(임시 DB/Map, 나중엔 DB연동)
  │     ├── user.js
  │     └── channel.js
  └── package.json       # 프로젝트 메타/모듈 정보
```

---

## 2. 각 역할 요약

- **app.js**:
    - 서버 생성, 라우터 연결, 미들웨어 등록, 포트 열기 등 “프로젝트의 뇌/진입점” 역할

- **routes/**:
    - URL과 HTTP 메서드별로 어떤 controller를 실행할지 지정 (주소/기능 분배)

- **controllers/**:
    - 실제로 클라이언트 요청을 처리하는 로직(데이터 체크, 응답 형식, 에러처리 등)

- **models/**:
    - 데이터 저장/조회/수정/삭제 담당 (실습에선 Map, 실무에선 DB와 연결)


----
## 3. 코드 연결 흐름 예시
### [1] app.js
```js
const express = require('express');
const app = express();
app.use(express.json());

const userRouter = require('./routes/users');
const channelRouter = require('./routes/channels');

app.use('/', userRouter);           // 회원 관련 주소(/join, /login, /users)
app.use('/channels', channelRouter); // 채널 관련 주소(/channels, /channels/:id)

app.listen(7777, () => {
  console.log('서버가 7777번 포트에서 실행 중입니다.');
});
```

### [2] routes/users.js
```js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.post('/join', userController.join);
router.post('/login', userController.login);
router.get('/users', userController.getAll);
// ... (개별 조회/수정/삭제는 .route('/users/:id').get(...).put(...).delete(...))

module.exports = router;
```

### [3] controllers/userController.js
```js
// 실질적 처리 (ex: 회원가입)
exports.join = (req, res) => {
  const { userId, password, name } = req.body;
  // (유효성 체크 → models 호출 → 응답)
};
```


### [4] models/user.js
```js
const db = new Map();
function createUser({ userId, password, name }) { ... }
function getUser(id) { ... }
module.exports = { createUser, getUser, ... }
```


---

## 4. Express 구조의 실무적 장점

- ✅ **가독성↑**: 기능별로 코드가 분리되어, 협업/유지보수에 최적
- ✅ **확장성↑**: 회원/채널 외에 “관리자/게시판/결제” 등 새로운 기능 추가도 쉽다
- ✅ **테스트/디버깅**: 특정 라우터·컨트롤러만 독립적으로 테스트 가능
- ✅ **실무와 동일한 설계 패턴**: 실질적인 스타트업/대기업 실무에서도 이 구조를 많이 사용

---

## 5. 실제 URL 흐름

예시로, 회원가입 요청의 경우
**클라이언트 → /join(POST) → userRouter → userController.join → userModel.createUser → 응답 반환**
이런 단계로 흘러간다.

---

## 6. 회고/정리
이번에 Express 구조를 설계하면서,
- 처음에는 모든 API를 한 파일에 몰아쓰는 게 편할 줄 알았지만, 실제로 기능이 조금만 늘어나도 코드가 금방 복잡해졌다는 걸 체감했다.
- **routes / controllers / models**로 분리하는 구조를 적용하니, 어디서 어떤 역할을 담당하는지 바로바로 찾기 쉬워서, 디버깅이나 기능 추가가 정말 빨라졌다.
- 특히 app.use()로 라우터를 경로별로 연결하는 패턴이 익숙해지니, “회원/채널/게시판/관리자”처럼 기능이 늘어나도 일관된 방식으로 확장할 수 있다는 점이 실무적으로 가장 큰 장점이었다.

**개발자 입장에서 배운 점**
- “작은 프로젝트라도 처음부터 구조를 나눠 관리하는 습관”이 중요하다는 걸 느꼈다.
- 실무에서 협업할 때 각자 맡은 도메인(회원/게시판 등)만 집중 관리할 수 있으니, 혼자서 개발하든 팀으로 개발하든 이 구조는 거의 필수라고 생각했다.
- 앞으로 더 복잡한 기능(인증, DB연동, 미들웨어 등)도 이 구조 위에서 얼마든지 유연하게 확장할 수 있을 것 같다.

**TIP.
- 처음에는 routes/controller/model 구분이 익숙하지 않지만,
    직접 실습하고 Postman으로 API 요청 흐름을 따라가보면 구조의 장점이 바로 체감된다!
- “Express 구조 분리”는 결국 코드를 읽고 고치고 확장하기 쉽게 만드는 투자라는 걸 이번에 알게 됐다.