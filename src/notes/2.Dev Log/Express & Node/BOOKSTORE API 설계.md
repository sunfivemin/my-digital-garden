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

## 🔑 전체 도서 조회
이미지 경로, n개씩 보내줘야함 (limit, offset 시작지점을 : req에 담아보낸다.)

- **Method**: `GET`  
- **URI**: `/books?limit={page당 도서 수}&currentPage={현재 page}`  
- **Status**: `200 OK`

### Response Body
```json
[
  {
    "id": "도서 ID",
    "title": "도서 제목",
    img: 이미지 id,
    "summary": "요약 설명",
    "author": "도서 작가",
    "price": 가격,
    "likes": 좋아요 수,
    "pubDate": "출간일"
  }
]
```


---

## 🔑 개별 도서 조회

- **Method**: `GET`  
- **URI**: `/books/[bookId]`  
- **Status**: `200 OK`

### Response Body
```json
{
  "id": "도서 ID",
  "title": "도서 제목",
  img: 이미지 id,
  "category": "카테고리",
  "format": "포맷",
  "isbn": "isbn",
  "summary": "요약 설명",
  "description": "상세 설명",
  "author": "도서 작가",
  "pages": 쪽 수,
  "index": "목차",
  "price": 가격,
  "likes": 좋아요 수,
  "liked": true,
  "pubDate": "출간일"
}
```


---

## 🔑 카테고리별 도서 목록 조회
new : true => 신간 조회(기준 : 출간일 1달 이내)
- **Method**: `GET`  
- **URI**: `/books/categoryId={categoryId}&new={boolean}`  
- **Status**: `200 OK`

### Response Body
```json
[
  {
    "id": "도서 ID",
    "title": "도서 제목",
    img: 이미지 id,
    "summary": "요약 설명",
    "author": "도서 작가",
    "price": 가격,
    "likes": 좋아요 수,
    "pubDate": "출간일"
  }
]
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

|**항목**|**내용**|
|---|---|
|**Method**|POST|
|**URI**|/likes/{bookId}|
|**HTTP Status**|200 OK|
|**Request Header**|Authorization: Bearer <token> (필요 시)|
|**Request Body**|{ "user_id": number }|
|**Response Body**|{ "message": "좋아요 등록 완료", ... } 또는 SQL 결과|


## ✍️ 좋아요 취소
|**항목**|**내용**|
|---|---|
|**Method**|DELETE|
|**URI**|/likes/{bookId}|
|**HTTP Status**|200 OK|
|**Request Body**|{ "user_id": number }|
|**Response Body**|{ "message": "좋아요 취소 완료", ... } 또는 SQL 결과|



---

## ✍️ 장바구니 담기

- **Method**: `POST`  
- **URI**: `/cart`  
- **Status**: `201 OK`

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
    "count": 수량,
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

## ✍️ 장바구니에서 선택한 주문 예상상품 목록 조회

- **Method**: `GET`  
- **URI**: `/cartItems`  
- **Status**: `200 OK`

### Request Body
```json
{
 cartItemId, cartItemId, cartItemId, 
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
    "count": 수량,
    "price": 가격
  },
 {
    "cartItemId": "장바구니 도서 ID",
    "bookId": "도서 ID",
    "title": "도서 제목",
    "summary": "도서 요약",
    "count": 수량,
    "price": 가격
  },
]
```

---

## ✍️ 결제하기 
= 주문하기 = 주문등록 = 데이터베이스 주문 insert = 장바구니에서 주문된 상품은 delete
 
- **Method**: `POST`  
- **URI**: `/orders`  
- **Status**: `200 OK`

### Request Body
```json
{
	 items: [
		 {cartItemId: 장바구니 도서 id, bookId: 도서 id, count: 수량},
		 {cartItemId: 장바구니 도서 id, bookId: 도서 id, count: 수량}, ...
	 ],
	 delivery: {
		 address: "주소",
		 receiver: "이름",
		 contact: "010-0000-0000"
	 }
	 totalPrice: 총금액
}
```

### Response Body
```json
[

]
```

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
	order_id: 주문 id,
	create_at: "주문일자",
	delivery: {
		address: "주소",
		receiver: "이름",
		contact: "전화번호"
	},
	bookTitle: "대표 책 제목",
	totalPrice: 결제 금액,
	totalCount: 총 수량
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
	book_id: 도서 id,
	create_at: "주문일자",
	bookTitle: "대표 책 제목",
	author: "작가명",
	price: 가격,
	count: 수량
 },
{
	book_id: 도서 id,
	create_at: "주문일자",
	bookTitle: "대표 책 제목",
	author: "작가명",
	price: 가격,
	count: 수량
 }
]
```