---
title: TypeScript
tags:
  - frontend
  - typescript
  - Interface
  - TypeAlias
  - Class
created: 2025-03-27
---

# 🚀 Type Alias, Interface, Class 

## 📌 Type Alias란?

`Type Alias`는 **기존 타입에 별칭(Alias)** 을 부여하는 기능입니다. 
객체뿐만 아니라 **유니온 타입, 함수 타입, 튜플** 등 다양한 타입을 정의할 때 사용됩니다.

### ✅ 기본 예제

```ts
// 기본 타입에 별칭 부여
type UserId = number;
type UserName = string;
type IsActive = boolean;

let userId: UserId = 12345;
let userName: UserName = "홍길동";
let isActive: IsActive = true;
```

### ✅ 객체 타입 정의

```ts
// 주문 상태 타입 정의
type OrderStatus = "pending" | "processing" | "shipped" | "delivered" | "canceled";

type Order = {
  id: number;
  productId: number;
  customer: string;
  quantity: number;
  status: OrderStatus;
  orderDate: Date;
};

// 사용 예
const order1: Order = {
  id: 1001,
  productId: 5,
  customer: "이지은",
  quantity: 2,
  status: "pending",
  orderDate: new Date(),
};
```

✅ **유니온 타입을 활용하면 제한된 값만 사용할 수 있어 안전한 코드 작성이 가능!**

### ✅ 함수와 메서드에 Type Alias 사용

```ts
// 함수 타입 별칭
type Calculator = (a: number, b: number) => number;

const add: Calculator = (x, y) => x + y;
const multiply: Calculator = (x, y) => x * y;

console.log(add(10, 20));  // 30
console.log(multiply(10, 20));  // 200
```

---

## 📌 객체 readonly 속성

읽기 전용 속성을 사용하면 **객체의 불변성(immutability)** 을 유지할 수 있습니다.

```ts
type User = {
  readonly id: number;
  readonly createdAt: Date;
  name: string;
  age: number;
};

const user: User = {
  id: 12345,
  createdAt: new Date(),
  name: "홍길동",
  age: 30,
};

// 가능: 일반 속성 수정
user.name = "김철수";
user.age = 31;

// 에러: readonly 속성 수정 불가
// user.id = 67890;
// user.createdAt = new Date();
```

✅ **읽기 전용 속성을 활용하면 데이터를 보호할 수 있음!**

---

## 📌 Interface란?

`Interface`는 **객체의 구조를 정의**하는 용도로 사용됩니다. `type alias`와 유사하지만, `extends`를 통해 다른 인터페이스를 확장할 수 있으며, 클래스에서도 `implements` 키워드로 사용할 수 있습니다.

### ✅ 기본 예제

```ts
interface Car {
  model: string;
  price: number;
  tax(): number;
}

const myCar: Car = {
  model: "Tesla",
  price: 5000,
  tax() {
    return this.price * 0.1;
  },
};
```

### ✅ Interface 확장 (extends)

```ts
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const myDog: Dog = {
  name: "바둑이",
  breed: "골든 리트리버",
};
```

✅ `**extends**`**를 사용하여 기존 인터페이스를 확장할 수 있음!**

---

## 📌 Type Alias vs Interface 비교

|비교 항목|Type Alias|Interface|
|---|---|---|
|객체 구조 정의|✅ 가능|✅ 가능|
|확장 방법|`&` 연산자 사용|`extends` 키워드 사용|
|유니온 타입 사용|✅ 가능|❌ 불가능|
|함수 타입 정의|✅ 가능|✅ 가능|
|중복 선언|❌ 불가능|✅ 자동 병합 가능|

---

## 📌 Class (클래스)란?

TypeScript에서 **클래스(Class)** 는 객체를 만들기 위한 **설계도(템플릿)** 입니다. 
코드를 **재사용**하고, **구조화**하여 효율적인 프로그래밍이 가능합니다.

### ✅ 기본 클래스 정의

```ts
class Car {
  model: string;
  price: number;

  constructor(model: string, price: number) {
    this.model = model;
    this.price = price;
  }

  tax(): number {
    return this.price * 0.1;
  }
}
```

### ✅ 클래스 상속 (Extends)

```ts
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  sound() {
    console.log(`${this.name}이(가) 소리를 냅니다.`);
  }
}

class Dog extends Animal {
  bark() {
    console.log(`${this.name}이(가) 멍멍! 짖습니다.`);
  }
}
```

✅ **부모 클래스의 기능을 유지하면서 새로운 기능 추가 가능!**

---

## 🎯 정리: 언제 `type`, `interface`, `class`를 사용할까?

✔ **객체 구조를 정의할 때** → `interface` 사용 (확장 가능) 
✔ **유니온 타입, 튜플, 함수 타입을 정의할 때** → `type alias` 사용
✔ **클래스를 사용할 때, 인터페이스를 활용** → `interface` + `implements` 사용
✔ **클래스에서 공통 속성을 물려받을 때** → `extends` 사용

---

## 🎉 마무리

이번 글에서는 TypeScript에서 자주 사용되는 `type alias`, `interface`, `class`의 개념과 차이점을 살펴보았습니다. 각각의 개념을 잘 활용하면 더 **안전한 타입 기반 코드**를 작성할 수 있습니다. 🎯

