---
tags:
  - TIL
  - React
created: 2025-06-23
---
# 📘 2025-06-23 TIL

## 📌 오늘 배운 핵심 요약



## 🧠 상세 학습 내용
## 📍 주제 1: React 컴포넌트의 children 타입: JSX.Element, ReactElement, ReactNode 차이

React에서 컴포넌트의 자식(children) 요소를 어떻게 타입 지정해야 하는지 혼란스러울 수 있다. 특히 TypeScript를 사용할 때 JSX.Element, ReactElement, ReactNode 중 무엇을 사용해야 할지 헷갈린다.

아래 그림처럼 세 타입은 포함 관계를 가진다:

| **타입**       | **설명**                                       |
| ------------ | -------------------------------------------- |
| JSX.Element  | return `<div>hi</div>` 처럼 JSX 문법으로 생성된 단일 요소 |
| ReactElement | JSX.Element과 사실상 동일하지만 React 내부에서 구분됨        |
| ReactNode    | **모든 렌더링 가능한 타입** 포함 (문자열, 숫자, null 등도 포함)   |

### ✅  실무에서 가장 많이 쓰는 건 ReactNode
```tsx
interface Props {
  children: React.ReactNode;
}
```
이렇게 하면 문자열, 숫자, JSX, 컴포넌트, null 등 다양한 타입을 허용한다.

---

## 🧩 실전 예시: Layout 컴포넌트
App 전체의 구조를 담당하는 Layout 컴포넌트를 구현해보자.
### 🗂 폴더 구조
```bash
src/
  ├── components/
  │   ├── common/
  │   │   ├── Header.tsx
  │   │   └── Footer.tsx
  │   └── Layout.tsx
  ├── pages/
  │   └── Home.tsx
  └── App.tsx
```

### 📄 Layout.tsx
```tsx
import Footer from '../common/Footer';
import Header from '../common/Header';

interface LayoutProps {
  children: React.ReactNode;
}

function Layout({ children }: LayoutProps) {
  return (
    <>
      <Header />
	      <main>{children}</main>
      <Footer />
    </>
  );
}

export default Layout;
```

### 📄 App.tsx
```tsx
import Layout from './components/Layout/Layout';
import Home from './pages/Home';

function App() {
  return (
    <Layout children={<Home />} />
  );
}

export default App;
```
- Layout 컴포넌트가 children이라는 **props**를 받음
- <Home /> 컴포넌트가 그 children으로 **전달됨**


| **타입**       | **사용 예시**             | **특징**                |
| ------------ | --------------------- | --------------------- |
| JSX.Element  | 함수형 컴포넌트의 반환 타입       | 단일 JSX 요소에만 해당        |
| ReactElement | createElement로 생성된 요소 | JSX.Element와 거의 동일    |
| ReactNode    | props.children 타입     | 가장 범용적이며, 문자열/숫자 등 포함 |

#### 🔹 





---

## 📍 주제 2: 테마스위처 contextAPI




---

## 📍 주제 3: 






![redux-toolkit](https://seonohblog.netlify.app/assets/redux-toolkit.png)

---

## 💭 회고
• **새롭게 알게 된 점**
- 

• **어렵게 느껴졌던 부분**
-  
  
• **다음에 학습할 주제**


### 🔗 참고자료


```ts
// styles/font.css.ts
import { style } from '@vanilla-extract/css';

export const kkokghaeFont = style({
  fontFamily: '"kkokghae", cursive',
});
```

```ts
// 예: TestComponent.tsx
import { kkokghaeFont } from '@/styles/font.css';

export const TestComponent = () => (
  <div>
    <p>일반 텍스트</p>
    <p className={kkokghaeFont}>이건 꼬깃체(Kkokghae) 폰트</p>
    <p className="font-kkokghae text-xl">Tailwind 방식으로도 적용 가능</p>
  </div>
);
```