## 📘 REST API 완전 정복 — 구조, 원칙, 특징까지 한번에 정리!


## ✅ REST API란?
 **REST**(Representational State Transfer)는 웹의 기본 개념인 **HTTP를 가장 잘 활용하는 아키텍처 스타일**입니다.
 REST API는 이 REST 원칙을 따르며 만든 **HTTP 기반의 통신 방식**을 말합니다.

---

## ✨ REST API는 어떤 API인가요?
| **항목**                | **설명**                              |
| --------------------- | ----------------------------------- |
| ✅ **HTTP 표준을 충실히 따름** | GET, POST, PUT, DELETE 등 메서드 명확히 사용 |
| ✅ **Stateless(무상태)**  | 서버는 클라이언트의 이전 요청 상태를 기억하지 않음        |
| ✅ **자원 중심 설계**        | URL은 명사(Resource), 동사는 HTTP 메서드에 맡김 |
| ✅ **클라이언트-서버 분리**     | 프론트와 백은 완전히 분리되어 독립적으로 개발 가능        |
| ✅ **계층화 구조**          | 인증, 캐시, 라우터 등의 계층적 설계가 가능함          |

---

## **📐 Uniform Interface란?**

REST에서 중요한 설계 원칙 중 하나는 바로 Uniform Interface (일관된 인터페이스)입니다.
즉, API가 일관된 규칙으로 동작해야 한다는 뜻입니다.

### **주요 원칙**
1. **자원의 URI 명확성** → /users, /products/1
2. **표준 HTTP 메서드 사용** → GET, POST, PUT, DELETE
3. **자기서술적 메시지** → 요청만 보면 어떤 동작인지 알 수 있음
4. **표준 상태코드와 응답 포맷(JSON)** 사용


---


## 🧠 REST API의 핵심 구성요소
| **구성 요소**          | **예시**                                    |
| ------------------ | ----------------------------------------- |
| **HTTP Method**    | GET, POST, PUT, DELETE 등                  |
| **URL (자원 주소)**    | /users, /books/123 등 명확한 자원 표현            |
| **Headers**        | 요청/응답의 메타데이터 (ex: 인증 토큰, 콘텐츠 타입 등)        |
| **Body (Payload)** | 요청 시 전달할 데이터 (ex: JSON)                   |
| **Status Code**    | 응답 결과 표현 (200, 201, 400, 401, 404, 500 등) |


---


## 🛠 RESTful URL 설계 예시
| **동작**    | **HTTP Method** | **URL 예시** | **설명**               |
| --------- | --------------- | ---------- | -------------------- |
| 전체 사용자 조회 | GET             | /users     | 모든 사용자 데이터 반환        |
| 특정 사용자 조회 | GET             | /users/:id | ID에 해당하는 사용자 반환      |
| 사용자 생성    | POST            | /users     | Body에 담긴 데이터로 사용자 생성 |
| 사용자 수정    | PUT             | /users/:id | 해당 사용자의 전체 정보 수정     |
| 사용자 삭제    | DELETE          | /users/:id | 해당 사용자 삭제            |

✅ URL은 자원을 명사로 표현,
행위는 HTTP 메서드로 구분하는 것이 RESTful 설계입니다.


---


## 🚫 REST API에서 지켜야 할 규칙들

### 🔹 1. URL에 동사를 쓰지 말 것
- ❌ /getUserList, /deleteUser
- ✅ /users, /users/:id

### 🔹 2. 상태를 서버가 기억하지 않는다 (Stateless)
• 요청마다 필요한 모든 정보를 포함해야 한다.
• 세션 없이 토큰 기반 인증(JWT 등) 사용

### 🔹 3. 응답에는 적절한 HTTP 상태 코드를 포함해야 한다.

| **상태 코드**                 | **의미**    |
| ------------------------- | --------- |
| 200 OK                    | 요청 성공     |
| 201 Created               | 리소스 생성 성공 |
| 400 Bad Request           | 잘못된 요청    |
| 401 Unauthorized          | 인증 안 됨    |
| 403 Forbidden             | 권한 없음     |
| 404 Not Found             | 리소스 없음    |
| 500 Internal Server Error | 서버 에러     |


---


## 🧩 Express에서 REST API 구성 예시
```js
const express = require('express');
const router = express.Router();
const UserController = require('../controllers/UserController');

// 사용자 CRUD
router.get('/users', UserController.getAll);
router.get('/users/:id', UserController.getOne);
router.post('/users', UserController.create);
router.put('/users/:id', UserController.update);
router.delete('/users/:id', UserController.remove);
```



---


## 🌐 REST API vs 일반 API
| **항목**   | **일반 API 예시**     | **REST API 예시**        |
| -------- | ----------------- | ---------------------- |
| URL 설계   | /getUserList      | /users                 |
| Method   | 모두 POST로 처리       | GET, POST, PUT, DELETE |
| 상태 코드 사용 | 거의 없음 (성공 시 200만) | 상황별로 정확한 코드 반환         |
| 명확성      | 기능 중심으로 혼란        | 자원 중심으로 직관적            |
|          |                   |                        |
|          |                   |                        |
## 🚧 REST API 설계 시 자주 하는 실수
| **실수**     | **설명**                 | **개선 방법**              |
| ---------- | ---------------------- | ---------------------- |
| URL에 동사 사용 | /getUsers, /createUser | /users                 |
| 자원 이름 혼용   | /user/1, /users 혼용     | 복수형(users)로 통일         |
| 상태 코드 무시   | 모든 응답을 200으로 처리        | 400, 404, 500 등 정확히 구분 |
| 메서드 남용     | POST로 모든 동작 처리         | HTTP 메서드에 따라 구분        |

---

## 🔍 REST vs GraphQL 간단 비교
| **항목**    | **REST API**     | **GraphQL**            |
| --------- | ---------------- | ---------------------- |
| 요청 구조     | 여러 URL           | 하나의 /graphql           |
| 응답 필드     | 고정된 구조           | 클라이언트가 원하는 필드만 요청      |
| 오버페치/언더페치 | 발생 가능            | 없음                     |
| 학습 난이도    | 쉬움               | 복잡함                    |
| 사용 시점     | CRUD, 정형 데이터 API | 복잡한 연관 관계, 동적 쿼리 필요할 때 |


---


## 💬 마무리: REST API는 ‘규약’을 잘 지키는 API

• REST는 새로운 기술이 아니라, HTTP를 어떻게 잘 쓸 것인가에 대한 철학입니다.
• RESTful한 API는 명확한 규칙을 기반으로 하기에 유지보수, 협업, 확장성이 뛰어납니다.
• Express, NestJS, Fastify 등 대부분의 프레임워크가 REST 구조를 기본으로 제공합니다.
