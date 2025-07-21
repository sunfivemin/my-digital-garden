## 🎯 VanillaWrapper의 목적

### 핵심 목적: React 안에서 순수 JavaScript를 안전하게 실행하기

```ts
// ❌ 문제 상황: React에서 직접 DOM 조작하면 위험
const BadExample = () => {
  // React가 아직 렌더링하지 않았는데 DOM에 접근하려고 함
  document.getElementById('my-div').innerHTML = 'Hello'; // 에러!
  
  return <div id="my-div"></div>;
};
```

```ts
// ✅ 해결책: VanillaWrapper가 안전한 환경 제공
const GoodExample = () => {
  return (
    <VanillaWrapper
      initiator={(element) => {
        // React가 렌더링 완료 후 실행됨
        element.innerHTML = 'Hello'; // 안전!
      }}
    />
  );
};
```


## 🔍 initiator 함수란?

### initiator = "초기화하는 함수"
```ts
// initiator 함수의 기본 형태
const initiator = (element: HTMLDivElement) => {
  // element는 실제 DOM 요소
  // 이 안에서 순수 JavaScript로 뭔가를 만들어줌
};
```

### 실제 사용 예시들:

#### 1. 간단한 텍스트 표시
```ts
const simpleInitiator = (element) => {
  element.innerHTML = '<p>안녕하세요!</p>';
};
```

#### 2. 버튼 만들기
```ts
const buttonInitiator = (element) => {
  element.innerHTML = '<button>클릭하세요</button>';
  
  const button = element.querySelector('button');
  button.onclick = () => alert('클릭됨!');
};
```

#### 3. 차트 만들기 (D3.js)
```ts
const chartInitiator = (element) => {
  // D3.js로 차트 그리기
  const svg = d3.select(element)
    .append('svg')
    .attr('width', 400)
    .attr('height', 200);
  
  // 차트 내용...
};
```


##  왜 VanillaWrapper가 필요한가?

### 1. React의 한계
```ts
// React는 선언적(declarative) 방식
const ReactComponent = () => {
  return <div>Hello</div>; // JSX로 선언
};
```

```ts
// 일부 라이브러리는 명령적(imperative) 방식
// DOM에 직접 접근해야 함
const vanillaJS = () => {
  const element = document.getElementById('my-element');
  element.innerHTML = 'Hello'; // 직접 DOM 조작
};
```

### 3. VanillaWrapper의 역할
```ts
// React와 Vanilla JS를 연결하는 브릿지
<VanillaWrapper
  initiator={(element) => {
    // React가 준비한 DOM 요소를 받아서
    // Vanilla JS로 뭔가를 만들어줌
    element.innerHTML = 'Hello';
  }}
/>
```



## VanillaAccordion에서 initiator 함수 분석
### initiator 함수가 하는 일:

1. DOM 요소 생성
```ts
const li = document.createElement("li");
const tab = document.createElement("div");
const description = document.createElement("div");
```

2. 이벤트 리스너 등록
```ts
ul.addEventListener("click", handleClick);
ul.addEventListener("keydown", handleKeyDown);
```

3. 상태 관리
```ts
let currentId: string | null = null;
// 클릭할 때마다 상태 변경
```

4. cleanup 함수 반환
```ts
return () => {
  ul.removeEventListener("click", handleClick);
  ul.removeEventListener("keydown", handleKeyDown);
};
```

## 💡 정리

### VanillaWrapper의 목적:
- React 안에서 순수 JavaScript를 안전하게 실행하기
- DOM 직접 조작이 필요한 라이브러리들을 React와 연결하기

### initiator 함수:
- VanillaWrapper가 제공하는 DOM 요소를 받아서 뭔가를 만들어주는 함수
- 이벤트 리스너, 상태 관리, cleanup 등을 담당

### 사용 현황:
- 모든 컴포넌트에 사용되지 않음
- VanillaAccordion과 Chart 컴포넌트에서만 사용
- React 기본 방식으로 해결할 수 있는 경우에는 사용하지 않음
