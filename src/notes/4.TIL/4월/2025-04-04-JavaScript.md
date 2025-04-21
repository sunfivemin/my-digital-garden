---
tags:
  - TIL
  - HTML
  - CSS
  - Javascript
  - 웹
created: 2025-04-04
---

# 📘 2025-04-04 TIL


## 📌 오늘 배운 핵심 요약

- CSS, JavaScript 작성 방식
- 조건문과 변수 사용법
- 함수 실습

## 🧠 상세 학습 내용

## 📍 주제 1: CSS
 
HTML은 웹 페이지의 구조를 정의하는 반면, CSS(Cascading Style Sheets)는 그 구조에 **스타일과 생동감을 부여**하는 역할을 한다. 단순히 태그 단위로 꾸미는 것이 아니라, 페이지 전반에 걸쳐 **일관된 디자인을 적용**할 수 있도록 도와준다.

#### 1. ✅ 인라인 스타일 (Inline Style)

HTML 태그 내부에 직접 style 속성을 사용하여 스타일을 적용하는 방식
🛠  A/B 테스트, JS 동적 스타일 변경, 예외 케이스에서 사용

```html
<p style="color: red; font-weight: bold;">이 문장은 빨간색입니다.</p>
```

- **장점**: 빠르게 테스트할 때 유용하며, 특정 요소 하나에만 스타일을 적용할 수 있음.
- **단점**: 유지보수가 어렵고, HTML과 스타일이 뒤섞여 가독성이 떨어짐. 대규모 프로젝트에 적합하지 않음.




#### 2. ✅  내부 스타일 시트 (Internal Style Sheet)

HTML 문서의 `<head>` 태그 안에 `<style>` 요소로 작성하는 방식
🛠 단일 페이지, 테스트용 프로토타입, 독립적인 문서 스타일링

```html
<head>
  <style>
    p {
      color: blue;
    }
  </style>
</head>
```

- **장점**: 동일 문서 내 여러 요소에 스타일 적용 가능. HTML과 CSS가 같은 파일에 있어 테스트/배포가 간편함.
- **단점**: 여러 페이지에 동일한 스타일을 공유할 수 없음. HTML 문서마다 스타일이 중복될 수 있음.


#### 3. ✅  외부 스타일 시트 (External Style Sheet)

별도의 .css 파일로 작성하고, HTML 문서에서는 `<link>` 태그를 통해 연결하는 방식
🛠 대부분의 프로젝트에서 필수적으로 사용하는 방식

```html
<head>
  <style>
    p {
      color: blue;
    }
  </style>
</head>
```

- **장점**: 하나의 스타일 파일로 여러 HTML 문서에 일관된 스타일 적용 가능. 유지보수에 유리하고, 캐싱을 통해 성능 개선 효과도 큼.
- **단점**: CSS 파일이 외부에 존재하므로, 연결이 끊기면 스타일이 적용되지 않을 수 있음.

---

## 📍 주제 2:  Javascript

웹 브라우저에서 동작하는 대표적인 스크립트 언어로, HTML과 CSS로 만든 정적인 웹페이지에 동적인 기능을 추가해주는 언어이다.
과거에는 자바스크립트가 오직 브라우저 안에서만 작동했지만, 최근에는 런타임 환경(Node.js)의 발달로 **서버 개발, 데스크탑 앱, 모바일 앱, IoT 기기**까지 다양한 곳에서 사용되고 있다.

#### ✅ 1. 인라인 스크립트 (inline script)

 - 태그에 직접 JavaScript 코드를 작성하는 방식
 - 간단한 테스트나 아주 짧은 동작을 처리할 때 유용하지만, 유지보수가 어렵고 가독성이 떨어지기 때문에 실무에서는 자주 사용되지 않는다.
 
```html
<button onclick="alert('안녕하세요!')">button</button>
```

#### ✅ 2. 내부 스크립트 (internal script)

- `<head>`나 `<body>` 하단에 HTML 문서 안의 `<script>` 태그 안에 JavaScript 코드를 작성하는 방식
- 단일 파일로 관리할 수 있어서 소규모 프로젝트에 좋지만, 규모가 커지면 외부 스크립트로 분리하는 것이 좋다.

```html
<script>
  console.log("페이지가 로드되었습니다!");
</script>
```

#### ✅ 3. 외부 스크립트 (external script)

- JavaScript 코드를 별도의 .js 파일로 작성하고, HTML에서 src로 불러오는 방식
- 유지보수에 용이하기 때문에 실무 개발에서는 이 방식을 사용한다.

```html
  <script src="main.js"></script>
```

---

### 🔍  요소를 선택하는 3가지 기본 방법

```js
// 1. ID로 찾기
document.getElementById('아이디');

// 2. class 이름으로 찾기
document.getElementsByClassName('클래스이름');

// 3. 태그 이름으로 찾기
document.getElementsByTagName('태그이름');
```

---

###  📦 자바스크립트 기초 문법

###  ✅ 조건문 (if / else)

```js
if (조건) {
  // 조건이 참(true)일 때 실행되는 코드
} else {
  // 조건이 거짓(false)일 때 실행되는 코드
}

function checkLogin() {
  const id = document.getElementById("txt_id").value;

  if (id === "") {
    alert("아이디를 입력해주세요.");
  } else {
    alert("입력한 아이디는 " + id + "입니다.");
  }
}
```

###  ✅ 변수

 데이터를 저장하고 재사용할 수 있도록 이름을 붙여주는 것
- 자바스크립트에서는 변수를 선언할 때 let, const, var 세 가지 키워드를 사용할 수 있다.
```js
let name = '제임스'; // 대부분의 경우 사용
name = '존';         // 가능 

const birthYear = 2000;
// birthYear = 1995; // const는 재할당 불가

var temp = '안녕';
temp = '잘가';      // 가능은 하지만 추천하지 않음 
```


---
## 🧭 JavaScript 실습 예제

#### 📎 실습 화면

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Login Page</title>
    <link rel="stylesheet" href="./style.css" />
    <script src="./script.js" defer></script>
  </head>
  <body>
    <h1>Login</h1>
    <form action="/login" method="POST">
      <input
        id="txt_id"
        class="login_inputs"
        type="text"
        name="id"
        placeholder="ID"
        autocomplete="username"
        required
      />
      <input
        id="txt_pw"
        class="login_inputs"
        type="password"
        name="password"
        placeholder="Password"
        autocomplete="current-password"
        onclick="myFunction('비밀번호 입력칸 클릭!')"
        required
      />
      <button id="btn" type="button" onclick="checkLogin()">로그인 하기</button>
    </form>
    <a href="/go.html">비밀번호 찾기</a>
  </body>
</html>
```

```js
function checkLogin() {
  const id = document.getElementById("txt_id").value;
  if (id === "") {
    alert("아이디를 입력해주세요.");
  } else {
    alert("입력한 아이디는 " + id + "입니다.");
  }
}

function myFunction(msg) {
  alert("1");
  alert("2");
  alert("3");
  console.log(msg);
}
```

![myFunction](https://seonohblog.netlify.app/assets/myFunction.png)

![loginFunc.png](https://seonohblog.netlify.app/assets/loginFunc.png)

#### 💬 실습하면서 느낀 점
onclick에 값을 넘길 때 함수가 바로 실행되는 경우와 실행만 대기하는 경우가 있어서, 그 차이를 이해하는 게 좀 헷갈렸다.
자바스크립트가 코드를 어떤 순서로 처리하는지, 변수의 범위가 어디까지인지, 화면이 언제 그려지는지도 조금씩 감이 잡히기 시작했다.
console.log만으로는 확인이 어려운 상황이 있어서 크롬 개발자 도구에서 breakpoint를 써봤는데, 생각보다 훨씬 편하게 디버깅할 수 있었다.

---

## 💭 회고

• **새롭게 알게 된 점**
→ 이벤트 핸들러에 함수를 넘길 때, 함수 자체를 넘기는 것과 실행하는 것의 차이를 알게 됐다.
→ let과 const가 블록 단위로 동작한다는 건 알고 있었지만, 변수가 어떻게 끌어올려지는지 직접 확인해보니까 이해가 더 잘 됐다.
→ DOM을 조작할 때는 브라우저가 화면을 다 그린 뒤여야 내가 의도한 대로 잘 작동한다는 걸 느꼈다.

• **어렵게 느껴졌던 부분**
→ undefined가 출력되는 문제가 단순히 값이 없는 게 아니라, 실행 타이밍의 문제일 수 있다는 걸 알게 됐다.
→ 함수 호출 방식에 따라 버그가 생기는데 그 이유를 파악하려면 콜 스택, 실행 컨텍스트 개념을 어느 정도 알아야 해서 어렵게 느껴졌다.
→ 버튼을 클릭했을 때 alert()이 여러 번 겹쳐 뜨는 문제는 단순히 코드 문제만은 아니었고, 브라우저의 렌더링 블로킹 이슈도 얽혀 있어서 신기했다.

• **다음에 학습할 주제**
→ 데이터를 기반으로 개발자 포트폴리오 제작
→ 백엔드 기초: Node.js 기본

## 🔗 참고 자료
[JavaScript 이벤트 전달 방식](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Events)
[렌더링 블로킹](https://developer.chrome.com/docs/lighthouse/performance/render-blocking-resources?hl=ko)
