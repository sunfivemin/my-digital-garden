
## 📘 MVC 패턴이란?
 MVC는 **Model - View - Controller**의 약자로,
 웹 애플리케이션의 역할을 3가지로 나눠서 유지보수성과 확장성을 높이는 소프트웨어 설계 패턴입니다.

## ✨ 각 구성요소의 역할
| **구성요소**       | **설명**                              |
| -------------- | ----------------------------------- |
| **Model**      | DB와 직접 연결되어 데이터를 저장하거나 가져오는 역할      |
| **View**       | 사용자에게 보여지는 화면 (HTML, EJS, Pug 등)    |
| **Controller** | 요청을 받고 Model에 명령을 내리고, 결과를 View로 전달 |

---


## 📁 Express 기반 MVC 패턴 구조 설명
```bash
book-shop/
├── app.js
├── controllers/     ← 💥 Controller (요청 처리, 로직 흐름 제어)
├── models/          ← 💾 Model (DB와 직접 통신)
├── routes/          ← 🚏 라우터 (요청 URL을 Controller로 연결)
├── views/           ← 👁️ View (화면 반환용 - 주로 템플릿)
├── middlewares/     ← 🔐 Middleware (Controller 전·후 공통 처리)
├── services/        ← 🧠 Service (복잡한 비즈니스 로직 분리 - 선택적)
├── config/, utils/  ← ⚙️ 설정 및 유틸
```
💡 routes, middlewares, services 등은 MVC 외에 **확장된 아키텍처 요소**로 많이 사용됩니다.

---


## 🔎 각 역할을 MVC 기준으로 분해
|**계층**|**폴더**|**역할 설명**|
|---|---|---|
|**Model**|models/|DB 테이블과 매핑되어 실제 데이터를 다룸. 예: SQL 쿼리 수행, 조회/삽입/삭제|
|**View**|views/|클라이언트에게 보여지는 결과 화면. Express에서는 EJS, Pug 등 사용 가능|
|**Controller**|controllers/|클라이언트 요청을 받아 Model 호출, 처리 결과를 View나 JSON으로 반환|
|**+ Router**|routes/|클라이언트 요청 URL을 해당 컨트롤러 함수에 연결 (/users → UserController)|
|**+ Middleware**|middlewares/|Controller 실행 전/후 공통 처리 (인증, 유효성 검사, 에러 핸들링 등)|

---


## 🎯 예시 흐름: /users/join 요청
```bash
[사용자 요청]
   ⬇
[app.js]
   ⬇
📦 [routes/users.js]         → URL "/users/join" 요청 받음
   ⬇
🛡️ [middlewares/validator.js] → 입력값 유효성 검사 
   ⬇
🧠 [controllers/UserController.js] → 로직 처리
   ⬇
💾 [models/User.js]          → DB에 회원정보 저장 (INSERT)
   ⬇
🧭 [controllers] → 응답 JSON 반환 or [views] → HTML 렌더링
   ⬇
[클라이언트 응답 완료]

```


---

## 📌 MVC 패턴의 장점 (Express에서)
| **항목**  | **효과**                                 |
| ------- | -------------------------------------- |
| 관심사 분리  | 라우팅, 로직, DB 분리 → 코드 가독성 향상             |
| 유지보수 쉬움 | Model이나 Controller 수정해도 각각 독립적으로 유지 가능 |
| 테스트 용이  | 각 계층 단위로 유닛 테스트 가능                     |
| 협업에 유리  | 각 역할별로 작업 분담 가능 (ex. 백엔드-프론트 연동 명확)    |

❗ MVC 패턴의 단점: MVC는 구조적으로 명확하지만, 작은 프로젝트에서는 오히려 **폴더가 많고 코드가 분산되어 관리가 불편**할 수 있습니다. 따라서 규모가 커질수록 더욱 빛을 발하는 아키텍처입니다.


---


## 📍 View는 반드시 필요한가요?

아닙니다. API 서버의 경우 View 없이 Controller가 **JSON 응답만 반환**하는 구조로 충분합니다.
SSR(서버 사이드 렌더링) 프로젝트가 아니면 views/ 폴더는 생략 가능합니다.


---

## 📌 Controller가 너무 복잡해진다면?
MVC에서는 Controller가 모든 요청을 처리하지만,
실제 서비스에서는 인증, 결제, 이메일 전송처럼 복잡한 로직이 생깁니다.

이런 경우 services/ 폴더를 만들어 **비즈니스 로직을 별도 계층으로 분리**하면 코드가 훨씬 깔끔해집니다.
```bash
book-shop/
├── services/
│   └── UserService.js  ← 비즈니스 로직 분리
```

 Controller가 너무 비대해질 경우, 실제 로직 처리를 services/ 폴더로 분리하는 경우가 많습니다.
예: 회원가입 → 비밀번호 암호화 + 이메일 중복 확인 + 유저 저장 같은 과정을 UserService.js에 분리해서 처리하면, **Controller는 흐름 제어만 하고 비즈니스 로직은 서비스가 담당하는 깔끔한 구조**가 됩니다.


---


## 💬 마무리
Express에서 MVC 구조를 명확하게 정리하면 협업도 쉬워지고, 프로젝트가 커져도 복잡해지지 않습니다.
여기서 빠질 수 없는 요소가 바로 **Middleware(미들웨어)**입니다.

→ 다음 글에서 **미들웨어는 MVC 구조에서 어떤 위치인지**, **왜 중요한지**, **실제 폴더 구조와 함께** 자세히 다뤄볼게요!

  
