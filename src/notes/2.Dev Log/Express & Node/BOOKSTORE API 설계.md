# 💡  회원 API
## ✍️ 회원가입

- **Method**: `POST`  
- **URI**: `/users/join`  
- **Status**: `201 OK`

### Request Body
```json
{
  "email": "user@example.com",
  "password": "userpassword"
}
```

### Response Cookie
```json
JWT Token
```


---

## 🔑 로그인

- **Method**: `POST`  
- **URI**: `/users/login`  
- **Status**: `200 OK`

### Request Body
```json
{
  "email": "user@example.com",
  "password": "userpassword"
}
```

### Response Body
```json
{
  "token": "JWT_TOKEN_STRING"
}
```


---

## 🔑 비밀번호 초기화 요청

- **Method**: `POST`  
- **URI**: `/users/reset`  
- **Status**: `200 OK`

### Request Body
```json
{
  "email": "user@example.com"
}
```

### Response Body


---

## ✍️ 비밀번호 초기화 (=수정)

- **Method**: `PUT`  
- **URI**: `/users/reset`  
- **Status**: `200 OK`

### Request Body
```json
{
  "password": "userpassword"
}
```

### Response Body


---

#  💡 도서 API

## 🔑 전체 도서 조회 (Pagination)
이미지 경로, n개씩 보내줘야함 (limit, offset 시작지점을 : req에 담아보낸다.)

| **항목**           | **내용**                                              |
| ---------------- | --------------------------------------------------- |
| **Method**       | GET                                                 |
| **URI**          | /books?limit={page당 도서 수}&currentPage={현재 페이지}      |
| **Status**       | 200 OK                                              |
| **Query Params** | limit, currentPage, category_id(선택), news(선택, true) |


### Response Body
```json
{
  "books": [
    {
      "id": 1,
      "title": "도서 제목",
      "img": 1,
      "summary": "요약 설명",
      "author": "도서 작가",
      "price": 20000,
      "likes": 5,
      "pubDate": "2024-12-10"
    },
    ...
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 5,
    "totalCount": 42
  }
}
```


---

## 🔑 개별 도서 조회

|**항목**|**내용**|
|---|---|
|**Method**|GET|
|**URI**|/books/{bookId}|
|**Status**|200 OK|
|**Request Header**|Authorization (로그인 시)|

### Response Body
```json
{
  "id": 1,
  "title": "도서 제목",
  "img": 1,
  "category": "문학",
  "format": "종이책",
  "isbn": "978-11-123456-7-8",
  "summary": "요약 설명",
  "description": "상세 설명",
  "author": "도서 작가",
  "pages": 250,
  "index": "목차 내용",
  "price": 18000,
  "likes": 3,
  "liked": true,
  "pubDate": "2023-12-10"
}
```
- liked: 로그인한 사용자가 해당 도서를 좋아요 눌렀는지 여부 (true/false)
- 비로그인 시 liked 필드는 포함되지 않음

---

## 🔑 카테고리별 도서 목록 조회
new : true => 신간 조회(기준 : 출간일 1달 이내)

| **항목**     | **내용**                                                         |
| ---------- | -------------------------------------------------------------- |
| **Method** | GET                                                            |
| **URI**    | /books?category_id={카테고리ID}&new=true&limit={n}&currentPage={m} |
| **Status** | 200 OK                                                         |

### Response Body
```json
{
  "books": [
    {
      "id": 1,
      "title": "도서 제목",
      "img": 1,
      "summary": "요약 설명",
      "author": "도서 작가",
      "price": 20000,
      "likes": 3,
      "pubDate": "2023-12-10"
    }
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 2,
    "totalCount": 8
  }
}
```


---

## 🔑 카테고리 전체 조회

- **Method**: `GET`  
- **URI**: `/category`  
- **Status**: `200 OK`

### Response Body
```json
[
  {
    id: 0,
    name: "동화",
  },
{
    id: 2,
    name: "소설",
  }
]
```

---

# 💡  좋아요 API

## ✍️ 좋아요 추가

| **항목**             | **내용**                                   |
| ------------------ | ---------------------------------------- |
| **Method**         | POST                                     |
| **URI**            | /likes/{bookId}                          |
| **HTTP Status**    | 200 OK                                   |
| **Request Header** | Authorization: 로그인할 때 받은 JWT Token (문자열) |



## ✍️ 좋아요 취소
| **항목**             | **내용**                                   |
| ------------------ | ---------------------------------------- |
| **Method**         | DELETE                                   |
| **URI**            | /likes/{bookId}                          |
| **HTTP Status**    | 200 OK                                   |
| **Request Header** | Authorization: 로그인할 때 받은 JWT Token (문자열) |




---

## ✍️ 장바구니 담기

- **Method**: `POST`  
- **URI**: `/cart`  
- **Status**: `201 OK`
- **Request Header** : Authorization: 로그인할 때 받은 JWT Token (문자열)

### Request Body
```json
{
  "bookId": "도서 ID",
  "quantity": 수량,
  userId: "회원 ID"
}
```


---

#   💡 장바구니 API

## 🔑 장바구니 조회

- **Method**: `GET`  
- **URI**: `/cart`  
- **Status**: `200 OK`

### Response Body
```json
[
  {
    "cartItemId": "장바구니 도서 ID",
    "bookId": "도서 ID",
    "title": "도서 제목",
    "summary": "도서 요약",
    "quantity": 수량,
    "price": 가격
  }
]
```


---

## 🔑 장바구니 삭제

- **Method**: `DELETE`  
- **URI**: `/cart/{bookId}`  
- **Status**: `200 OK`


---

# 💡주문 API

## ✍️ 선택한 장바구니 상품 목록 조회

- **Method**: `GET`  
- **URI**: `/cartItems`  
- **Status**: `200 OK`

### Request Body
```json
{
	userId: 회원 id,
	selected: [cartItemId, cartItemId, cartItemId ...]
}
```

### Response Body
```json
[
  {
    "cartItemId": "장바구니 도서 ID",
    "bookId": "도서 ID",
    "title": "도서 제목",
    "summary": "도서 요약",
    "quantity": 수량,
    "price": 가격
  },
 {
    "cartItemId": "장바구니 도서 ID",
    "bookId": "도서 ID",
    "title": "도서 제목",
    "summary": "도서 요약",
    "quantity": 수량,
    "price": 가격
  },
]
```

---

## ✍️ 결제하기 
= 주문하기 = 주문등록 = 데이터베이스 주문 insert = 장바구니에서 주문된 상품은 delete

| **항목**             | **내용**                                               |
| ------------------ | ---------------------------------------------------- |
| **별칭**             | 주문하기, 주문 등록, 결제하기                                    |
| **Method**         | POST                                                 |
| **URI**            | /orders                                              |
| **HTTP Status**    | 200 OK                                               |
| **Request Header** | Authorization: Bearer <JWT Token> (로그인 시 발급된 토큰 사용)  |
| **Request Body**   | 주문 정보 + 배송지 정보 + 장바구니 정보 포함                          |
| **Response Body**  | { "success": true, "orderId": 1, "deliveryId": 3 } 등 |

---

## ✍️ 주문 목록(내역) 조회 

- **Method**: `POST`  
- **URI**: `/orders`  
- **Status**: `200 OK`

### Request Body
```json

```

### Response Body
```json
[
  {
    "id": 1,
    "created_at": "2023-12-13 08:23:00",
    "address": "서울시 서초구",
    "receiver": "영희",
    "contact": "010-1111-1111",
    "book_title": "홍길동전",
    "total_quantity": 2,
    "total_price": 10000
  },
  {
    "id": 2,
    "created_at": "2023-12-13 12:30:00",
    "address": "서울시 강남구",
    "receiver": "철수",
    "contact": "010-2222-2222",
    "book_title": "백설공주",
    "total_quantity": 1,
    "total_price": 5000
  }
]
```

---

## ✍️ 주문 상세 상품 조회 

- **Method**: `POST`  
- **URI**: `/orders/{orderid}`  
- **Status**: `200 OK`

### Request Body
```json

```

### Response Body
```json
[
  {
    "book_id": 3,
    "title": "홍길동전",
    "author": "허균",
    "price": 5000,
    "quantity": 1
  },
  {
    "book_id": 4,
    "title": "심청전",
    "author": "미상",
    "price": 5000,
    "quantity": 1
  }
]
```