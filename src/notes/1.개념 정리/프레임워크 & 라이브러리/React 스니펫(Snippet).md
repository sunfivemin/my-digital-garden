“짧은 약어를 입력하면, 미리 정의된 코드 조각을 자동 완성해주는 기능”

예를 들어, rafce 라는 스니펫을 입력하면 아래와 같이 **React 화살표 함수 컴포넌트 기본 코드**가 자동으로 생성됩니다:
```tsx
import React from 'react';

const MyComponent = () => {
  return <div>MyComponent</div>;
};

export default MyComponent;
```

## 🔧 어떻게 작동하나요?

- 대부분의 React 스니펫 기능은 **VS Code 확장 프로그램**으로 동작합니다.
- 대표적인 확장:
    👉 **ES7+ React/Redux/React-Native snippets**
    설치 링크: [https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets)

---

## 💡 자주 쓰이는 React 스니펫 단축키

|**스니펫**|**설명**|
|---|---|
|rafce|화살표 함수 기반 컴포넌트 (export 포함)|
|rfce|일반 함수형 컴포넌트 (export 포함)|
|usee|useEffect 기본 형태|
|usf|useState 훅 기본 선언|
|clg|console.log() 자동완성|
|imp|import 문 자동 생성|

---

## ✨ 장점
- ✅ **타이핑 시간 절약** (수동으로 타이핑할 필요 없음)
- ✅ **오타 방지**
- ✅ **일관된 코드 스타일 유지**
- ✅ **컴포넌트 구조를 빠르게 셋업**

---

## 📌 사용 팁
1. VS Code에서 .js, .tsx 파일 열기
2. 예: rafce 입력 후 Tab 또는 Enter 누르기
3. 자동 완성된 템플릿에서 필요한 부분만 수정

---

## 🔍 커스텀 스니펫 만들기
원한다면 .code-snippets 파일을 만들어서 자신만의 스니펫도 등록할 수 있습니다:
```json
"React Function Component": {
  "prefix": "rfc",
  "body": [
    "import React from 'react';",
    "",
    "function $1() {",
    "  return <div>$1</div>;",
    "}",
    "",
    "export default $1;"
  ],
  "description": "React Function Component Template"
}
```