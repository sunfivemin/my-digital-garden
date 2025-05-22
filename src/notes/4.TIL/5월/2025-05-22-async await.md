---
tags:
  - TIL
created: 2025-05-22
---

# 📘 2025-05-22 TIL

## 📌 오늘 배운 핵심 요약
- **Promise**는 비동기 작업을 다룰 수 있게 해주는 JS 객체.
- **then() 체이닝**과 **async/await 문법**을 사용하면 복잡한 비동기 흐름을 순차적으로 제어할 수 있다.
- 실제 Node.js API에서 DB 작업을 순서대로 처리하기 위해 **insertId**를 활용해 연결했다.


## 🧠 상세 학습 내용

## 📍 주제 1: Promise 란?
```js
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve("완료!"), 3000);
});
```
- 비동기 작업을 감싸고, 완료되면 resolve, 실패하면 reject 호출
- .then()을 붙여서 완료 후 실행할 작업을 연결할 수 있음

----

## 📍 주제 2:  then() 체이닝
```js
promise
  .then(function(result) {
    console.log(result); // 완료!
    return result + "!!!!!";
  })
  .then(function(result) {
    console.log(result); // 완료!!!!!
    return result + "!!!!!";
  })
  .then(function(result) {
    console.log(result); // 완료!!!!!!!!! 
  });
```
- .then()은 **Promise가 resolve된 값**을 다음 then에 넘긴다.
- 이렇게 체이닝하면 **동기적으로 코드를 작성한 것처럼** 비동기 흐름을 제어할 수 있다.


---

## 📍 주제 3: async/await 문법
```js
async function f() {
  let promise = new Promise(function(resolve, reject) {
    setTimeout(() => resolve("완료!"), 3000);
  });

  let result = await promise; // 비동기 작업이 끝날 때까지 기다림
  console.log(result); // 완료!
}
f();
```
- async 함수 안에서만 await 사용 가능
- await은 Promise가 끝날 때까지 기다림
- **then보다 코드가 깔끔하고 읽기 쉬움**


---


## 📍 주제 4: async/await로 순차 실행하기
```js
let promise1 = new Promise((resolve) =>
  setTimeout(() => resolve("첫번째 쿼리"), 3000)
);
let result1 = await promise1;
console.log(result1);

let promise2 = new Promise((resolve) =>
  setTimeout(() => resolve("두번째 쿼리 with " + result1), 3000)
);
let result2 = await promise2;
console.log(result2);

let promise3 = new Promise((resolve) =>
  setTimeout(() => resolve("세번째 쿼리 with " + result2), 3000)
);
let result3 = await promise3;
console.log(result3);
```
- await을 통해 Promise들을 **순서대로** 실행 가능
- 이전 결과값을 다음 Promise에 넘겨줄 수 있음


---

## 📍 주제 5: query 순서대로 실행하기
Node.js에서 MySQL에 데이터를 여러 테이블에 순차적으로 저장할 때는 **쿼리 실행 순서**가 매우 중요하다. 특히 INSERT 후 생성된 insertId를 다음 쿼리의 외래키(FK)로 사용할 경우, **순서 보장과 에러 처리**가 핵심이다.
Node.js에서 MySQL로 데이터를 insert할 때, insert된 **id 값**을 다음 쿼리에서 사용하는 경우가 많다.

이번 주문 처리 로직에서는 다음과 같은 흐름으로 쿼리를 실행해야 했다:
### ✅ 1단계: 배송 정보 등록 → delivery_id 확보
```sql
INSERT INTO delivery (address, receiver, contact)
VALUES (?, ?, ?)
```

이 쿼리를 실행한 후, 해당 요청으로 생성된 배송지의 ID를 다음과 같이 확보:
```js
const [deliveryResults] = await connection.query(deliverySql, deliveryValues);
const delivery_id = deliveryResults.insertId;
```


### ✅ 2단계: 주문 정보 등록 → order_id 확보
이제 위에서 얻은 delivery_id를 활용해 주문 정보를 저장한다.
```sql
INSERT INTO orders (book_title, total_quantity, total_price, user_id, delivery_id)
VALUES (?, ?, ?, ?, ?)
```

Node.js 코드에서는 다음과 같이 실행:
```js
const [orderResults] = await connection.query(orderSql, orderValues);
const order_id = orderResults.insertId;
```


### ✅ 3단계: 주문 상세 정보(orderedBook) 다건 등록
앞서 얻은 order_id를 활용해 주문된 도서 정보를 한 번에 insert 한다.
```sql
INSERT INTO orderedBook (order_id, book_id, quantity) VALUES ?
```

Node.js에서는 map을 활용해 2차원 배열로 데이터를 구성한 후:
```js
const orderedBookValues = items.map((item) => [
  order_id,
  item.book_id,
  item.quantity,
]);

await connection.query(orderedBookSql, [orderedBookValues]);
```

### ✨ 왜 이런 순서가 중요한가?

- orderedBook에 먼저 데이터를 넣으면 order_id가 아직 없어서 **외래키 오류(FK error)** 발생
- 각 insert의 **결과값(insertId)** 는 해당 쿼리가 끝난 다음에만 얻을 수 있음
- 순차 실행이 안 되면 관계형 DB에서는 참조 무결성이 깨짐

### ✅ 순차 실행을 위해 async/await 사용
이전에는 콜백(callback) 지옥이나 promise chaining으로 처리했지만, async/await을 쓰면 다음처럼 깔끔하게 순서를 보장할 수 있다.

```js
const [result1] = await connection.query(sql1, values1);
const [result2] = await connection.query(sql2, values2); // result1.insertId 사용
await connection.query(sql3, [data.map(...)]);
```


![promise](https://seonohblog.netlify.app/assets/promise.png)


## 💭 회고

### • 새롭게 알게 된 점
- async/await을 사용하면 마치 동기 코드처럼 순서를 자연스럽게 유지할 수 있다.
- DB에 데이터를 순차적으로 insert하고, 이전 단계에서 생성된 insertId를 다음 쿼리에 바로 활용하는 흐름이 실무에서 자주 쓰인다는 걸 체감했다.

### • 어렵게 느껴졌던 부분
- then 체이닝은 구조는 이해되지만, 반환값을 어떻게 다음으로 넘기는지 감이 잘 안 잡혔다.
- async 함수는 자동으로 Promise를 반환한다는 점, 그리고 그 안에서만 await을 쓸 수 있다는 규칙이 처음엔 헷갈렸다.

### • 다음에 학습할 주제
- conn.query와 async 함수의 결합 방식
- 주문 정보 insert 이후 장바구니(cartItem) 자동 삭제 흐름 만들기
- 주문 내역 전체 조회 API 구현

