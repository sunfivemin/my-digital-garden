---
tags:
  - TIL
created: 2025-05-21
---

# 📘 2025-05-21 TIL

## 📌 오늘 배운 핵심 요약
- 주문 기능을 **배송 → 주문 → 주문 상세(orderedBook)** 3단계로 분리해 구현했다.
- 각 단계별 insertId를 활용해 외래키 관계를 연결했다.
- 외래키 제약조건, 인덱스 이름 중복 오류를 해결하며 DB 설계의 중요성을 다시 체감했다.


## 🧠 상세 학습 내용

## 📍 주제 1: FK 제약조건 생성과 네이밍 컨벤션

MySQL에서는 외래키(FK) 설정 시 자동 생성되는 제약조건(CONSTRAINT) 이름이나 인덱스 이름이 중복되면 오류가 발생한다.

예시:
- fk_orders_users_user_id
- fk_orderedBook_books_book_id

이런 네이밍 규칙을 사용하면 나중에 테이블 구조를 이해하거나 디버깅할 때 훨씬 쉽다.

---

## 📍 주제 2: 주문하기 SQL 흐름
주문을 처리할 때는 **배송지 → 주문 → 주문 상세** 순으로 insert가 필요하다.
#### 1. 배송 정보 입력
```sql
INSERT INTO delivery (address, receiver, contact)
VALUES ("서울시", "김진아", "010-1111-2222");
```
✔️ delivery_id = deliveryResults.insertId 또는 SELECT MAX(id) FROM delivery

### **🔍** delivery_id = deliveryResults.insertId vs SELECT MAX(id) FROM delivery 차이
두 방식 모두 INSERT 이후 생성된 레코드의 ID를 가져오는 방법입니다.

### ✅ 1. delivery_id = deliveryResults.insertId (✔️ 권장)
- **설명**: SQL INSERT 실행 직후, 해당 요청으로 새로 생성된 **auto_increment 값**을 바로 반환해주는 Node.js의 기능입니다.
- **실행 위치**: Node.js 코드에서 conn.query()의 결과 객체 내부
- **예시**:
```js
conn.query(sql, values, (err, results) => {
  const delivery_id = results.insertId;
});
```
- **장점**: 다른 사용자가 동시에 INSERT를 해도 내가 넣은 행의 id만 정확히 가져온다.


### ❌ 2. SELECT MAX(id) FROM delivery (⚠️ 비권장)
- **설명**: 테이블에서 가장 큰 ID를 가져옴
- **문제점**: **멀티 유저 환경**에서는 다른 사용자의 INSERT가 먼저 처리되면 엉뚱한 id를 가져오게 됨.
- **예시**:
```js
SELECT MAX(id) FROM delivery;
```

- **사용 이유**:
    - SQL 콘솔에서 테스트할 때 (단일 사용자 환경)
    - insertId를 못 쓰는 상황에서 임시로 조회할 때


--- 

#### 2. 주문 정보 입력
```sql
INSERT INTO orders (book_title, total_quantity, total_price, user_id, delivery_id)
VALUES ("어린왕자들", 3, 60000, 1, delivery_id);
```
✔️ order_id = orderResults.insertId 또는 SELECT MAX(id) FROM orders


--- 

#### 3. 주문 상세(orderedBook) 입력
```sql
INSERT INTO orderedBook (order_id, book_id, quantity)
VALUES (order_id, 1, 1);
```

📸 실제 DB 확인 결과
![deliverydb](https://seonohblog.netlify.app/assets/deliverydb.png)
![ordersdb](https://seonohblog.netlify.app/assets/ordersdb.png)
![orderedbook](https://seonohblog.netlify.app/assets/orderedbook.png)

#### 4. insert된 주문 ID로 주문 상세 목록 입력
```sql
SELECT max(id) FROM orders;
-- 또는 orderResults.insertId

INSERT INTO orderedBook (order_id, book_id, quantity)
VALUES (order_id, 1, 1);
```


----


## 📍 주제 3: Express + MySQL 주문 API 코드
```js
const conn = require('../mariadb');
const { StatusCodes } = require('http-status-codes');

const order = (req, res) => {
  const { items, delivery, totalQuantity, totalPrice, userId, firstBookTitle } = req.body;

  // Step 1: 배송 정보 입력
  const deliverySql = `
    INSERT INTO delivery (address, receiver, contact)
    VALUES (?, ?, ?)
  `;
  const deliveryValues = [delivery.address, delivery.receiver, delivery.contact];

  conn.query(deliverySql, deliveryValues, (err, deliveryResults) => {
    if (err) {
      console.log('Delivery insert error:', err);
      return res.status(StatusCodes.BAD_REQUEST).end();
    }

    const delivery_id = deliveryResults.insertId;

    // Step 2: 주문 정보 입력
    const orderSql = `
      INSERT INTO orders (book_title, total_quantity, total_price, user_id, delivery_id)
      VALUES (?, ?, ?, ?, ?)
    `;
    const orderValues = [
      firstBookTitle,
      totalQuantity,
      totalPrice,
      userId,
      delivery_id,
    ];

    conn.query(orderSql, orderValues, (err, orderResults) => {
      if (err) {
        console.log('Order insert error:', err);
        return res.status(StatusCodes.BAD_REQUEST).end();
      }

      const order_id = orderResults.insertId;

      // Step 3: 주문 상세 목록 입력
      const orderedBookSql = `
        INSERT INTO orderedBook (order_id, book_id, quantity)
        VALUES ?
      `;
      const orderedBookValues = items.map((item) => [
        order_id,
        item.book_id,
        item.quantity,
      ]);

      conn.query(orderedBookSql, [orderedBookValues], (err, result) => {
        if (err) {
          console.log('OrderedBook insert error:', err);
          return res.status(StatusCodes.BAD_REQUEST).end();
        }

        return res.status(StatusCodes.OK).json({
          success: true,
          orderId: order_id,
          deliveryId: delivery_id,
        });
      });
    });
  });
};

module.exports = { order };
```


## 📍 주제 4: API 요청 예시 (Postman)
```json
POST /orders
Content-Type: application/json

{
  "items": [
    { "book_id": 1, "quantity": 1 },
    { "book_id": 2, "quantity": 2 }
  ],
  "delivery": {
    "address": "서울시 중구",
    "receiver": "민선오",
    "contact": "010-3152-3178"
  },
  "firstBookTitle": "어린왕자들",
  "totalQuantity": 3,
  "totalPrice": 60000,
  "userId": 1
}
```



## **💭 회고**

### • 새롭게 알게 된 점
- insertId를 다음 insert의 외래키로 활용하는 흐름을 실무처럼 구현했다.
- MySQL 제약조건 이름을 직접 지정하는 것이 얼마나 중요한지 경험했다.

### • 어렵게 느껴졌던 부분
- FOREIGN KEY 설정 시 발생하는 오류 메시지가 직관적이지 않아서 원인 파악에 시간이 걸렸다.
- insertId는 INSERT 직후 Node.js에서 안전하게 반환되는 **내가 추가한 행의 고유 ID**이며, SELECT MAX(id)는 테스트용으로만 쓰는 조회 방식이다. 실무에서는 동시성 문제가 없도록 항상 insertId를 사용해야 한다.

### • 다음에 학습할 주제
- Node.js 비동기
- promise chaining
- async, await

