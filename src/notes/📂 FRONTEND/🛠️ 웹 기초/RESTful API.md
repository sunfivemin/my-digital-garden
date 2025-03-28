
# 서버와 유저가 통신하는 법 / RESTful API

서버는 단순한 프로그램으로, **요청을 받아 처리하는 역할**을 한다.  
예를 들어,  

- 사용자가 웹툰을 요청하면 웹툰 데이터를 전송  
- DB 데이터를 요청하면 해당 데이터를 반환  
- 반대로 데이터를 저장해달라고 하면 저장  

즉, 서버는 사용자의 요청을 **정확한 방식으로 전달받아야** 원하는 데이터를 반환할 수 있다.  
마음대로 요청하면 서버는 데이터를 주지 않는다.

## 1. 서버에 요청하는 방법 (HTTP 요청)
서버에게 데이터를 요청하려면 **두 가지 요소**를 정확히 입력해야 한다.  

1. **Method** (어떤 작업을 수행할지)  
2. **URL** (어떤 데이터를 요청할지)  

### 주요 HTTP Methods
- **GET** → 데이터를 요청할 때  
- **POST** → 데이터를 서버에 전송할 때  
- **PUT / UPDATE** → 기존 데이터를 수정할 때  
- **DELETE** → 데이터를 삭제할 때  

> 📌 URL도 정확하게 작성해야 한다. 이를 **엔드포인트(Endpoint)** 라고 부른다.  
> 정확한 method와 URL을 사용해야 서버가 요청을 정상적으로 처리한다.

---

## 2. RESTful API란?
REST(Representational State Transfer)는 **좋은 API를 설계하는 원칙**을 정의한 개념이다.  
RESTful API는 이 원칙을 따르는 API를 의미하며, 다음과 같은 특징을 가진다.

### REST의 6가지 원칙
1. **Uniform Interface (일관성 있는 인터페이스)**  
   - 하나의 URL은 **하나의 데이터**만 다뤄야 한다.  
   - URL과 Method는 **일관성 있고 예측 가능**해야 한다.  

2. **Client-Server 역할 분리**  
   - 클라이언트(사용자)와 서버의 역할을 명확히 구분해야 한다.  
   - 서버의 DB를 직접 조작하지 않도록 설계해야 한다.  

3. **Stateless (무상태성)**  
   - 각 요청은 **독립적**이어야 하며, 이전 요청의 정보를 유지하지 않아야 한다.  
   - 즉, 서버는 클라이언트의 상태를 저장하지 않는다.  

4. **Cacheable (캐싱 가능성)**  
   - 서버의 응답 데이터는 캐싱이 가능해야 한다.  
   - 자주 요청되는 데이터는 브라우저나 CDN에서 캐싱할 수 있도록 설정해야 한다.  

5. **Layered System (계층 구조)**  
   - API 서버는 보안, 로드밸런싱 등을 위해 **여러 개의 계층을 가질 수 있다**.  

6. **Code on demand (선택적 실행 코드 지원)**  
   - 필요할 경우 서버는 클라이언트에게 실행 가능한 코드를 제공할 수도 있다.  

> 🎯 REST 원칙을 **완벽하게 따르는 서버는 거의 없으며**, 권장사항으로 참고하는 개념이다.  
> 대부분은 **Method와 URL을 명확하게 구성하는 것**만으로도 RESTful API라고 부른다.

---

## 3. RESTful API에서 URL 설계 원칙
RESTful API를 만들 때 URL을 보기 쉽게 설계하는 것이 중요하다.

✔ **URL은 동사가 아니라 명사로 구성**  
✔ **언더바 `_` 대신 대시 `-` 사용**  
✔ **파일 확장자 (`.html` 등) 제거**  
✔ **하위 경로는 `/` 기호로 구분**

### 예시
- ✅ `facebook.com/bbc/photos`  
  → BBC 뉴스 계정의 사진첩  
- ✅ `instagram.com/explore/tags/food`  
  → `#food` 태그가 포함된 게시물  

위처럼 URL만 봐도 **어떤 데이터가 반환될지 이해할 수 있도록** 설계하는 것이 RESTful API의 핵심이다.

---

📌 **정리하면...**
- 서버에 요청하려면 **Method + URL**이 정확해야 한다.  
- REST는 API 설계의 권장 원칙이며, **완벽하게 지키지 않아도 된다**.  
- **URL은 직관적이고 명확하게 설계**하는 것이 중요하다.

🚀 RESTful API를 적용할 때 위 원칙들을 참고하자!  

---

