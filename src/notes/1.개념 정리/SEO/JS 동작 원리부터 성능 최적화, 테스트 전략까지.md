# “React 앱은 어떻게 빨라지고, 정확해지는가?”
## – JS 동작 원리부터 성능 최적화, 테스트 전략까지

프론트엔드 개발을 하다 보면 느린 앱, 의도치 않은 리렌더링, 테스트되지 않은 코드 등 여러 실무 이슈를 만나게 됩니다. 이 글에서는 다음과 같은 내용을 중심으로 정리해 봅니다:
1. 자바스크립트 코드가 실행되기 전에 어떤 일이 일어나는지
2. 스코프, 호이스팅, 클로저가 버그를 줄이는 데 어떤 도움을 주는지
3. React 앱의 성능 병목을 어떻게 해결할 수 있는지
4. 리렌더링 최적화와 코드 스플리팅을 실제로 어떻게 적용할 수 있는지
5. 테스트와 디버깅은 어떻게 하면 되는지


### 1. JS 실행 흐름 – 실행 컨텍스트란?
- 코드가 어떻게 실행되고 메모리에 올라가는가

### 실행 컨텍스트 생성 단계
자바스크립트 코드는 그냥 위에서 아래로 실행되는 것처럼 보이지만, 사실 먼저 메모리 할당이 먼저 일어나는 단계가 있습니다. 이를 실행 컨텍스트라고 합니다.

예를 들어 다음과 같은 코드가 있을 때:
```js
console.log(a);  // 👉 undefined
var a = 10;
```
- 먼저 a는 메모리에 올라가고 undefined가 들어감 (Creation Phase)
- console.log(a)를 만나서 undefined 출력
-  그 다음 a = 10 실행됨

코드는 두 단계로 처리됩니다:

• Creation Phase (생성 단계): 자바스크립트가 코드를 실행하기 전에, 먼저 변수나 함수 선언만 따로 기억해두는 단계
• Execution Phase (실행 단계): 이후에 10이라는 값이 대입됩니다.



let과 const는 이 과정에서 TDZ(Temporal Dead Zone)에 들어가서, 선언 전에 접근하면 오류가 납니다. 
이 차이를 알고 있으면 디버깅할 때 훨씬 유리합니다.

```js
console.log(b);  // ❌ ReferenceError
let b = 10;
```

- b는 메모리에 **올라가긴 하지만 초기화되지 않음**
- 초기화되기 전에 console.log(b)로 접근하면 **에러 발생**
- 이 에러 구간이 바로 **TDZ (Temporal Dead Zone)**


✅ TDZ
Temporal Dead Zone (일시적 사각지대)는 자바스크립트에서 변수가 선언되었지만 초기화되기 전까지 접근할 수 없는 상태


---

### 2. 스코프 + 호이스팅 + 클로저 - 왜 중요한가?
- 내부 구조 이해 → 버그 줄이기

### 클로저 예제
```jsx
function outer() {
  const name = "sun";
  return function inner() {
    console.log(name);
  };
}
const say = outer();
say(); // "sun"
```

이 코드에서 inner 함수는 outer 함수가 끝난 뒤에도 name 변수를 기억하고 있습니다. 이게 바로 **클로저(Closure)** 입니다. 이 구조는 React의 상태 관리(setState)나 이벤트 핸들러처럼 “과거의 값을 기억해야 하는 상황”에 자주 쓰입니다.

### 디버깅 팁
console.dir(say)을 브라우저에서 실행하면, 함수 내부의 `Scopes`를 통해 실제 클로저 환경을 확인할 수 있습니다.


---


### 3. React 리렌더링 최적화 
- React 성능 병목 제거하기

컴포넌트가 불필요하게 리렌더링되면 사용자 체감 속도가 느려지고 성능이 저하됩니다. 
이걸 방지하는 대표적인 방법이 **React.memo**, **useCallback**, **useMemo**입니다.
### 성능 병목 사례
```jsx
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>+</button>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  const clickHandler = useCallback(() => {
    console.log('clicked');
  }, []);
  return (
    <>
      <p>현재 카운트: {count}</p>
      <button onClick={() => setCount(count + 1)}>카운트+</button>
      <Child onClick={clickHandler} />
    </>
  );
};
```
### 디버깅 방법
- React DevTools의 **Profiler 탭**을 사용하면 어떤 컴포넌트가 리렌더링됐는지 쉽게 확인할 수 있습니다.
- console.log를 사용해 렌더링 타이밍을 비교해보는 것도 좋은 방법입니다.

---

### 4. 코드 스플리팅 & Lazy Loading

초기 번들 크기가 크면 LCP(Largest Contentful Paint) 지표가 나빠집니다. 
이걸 해결하는 대표적인 방법이 **React.lazy + Suspense**입니다.

### Lazy Loading
```tsx
const Chart = React.lazy(() => import('./Chart'));

function Dashboard() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <Chart />
    </Suspense>
  );
}
```

이렇게 하면 Chart 컴포넌트는 처음부터 로딩되지 않고, Dashboard에서 호출될 때만 다운로드되기 때문에 초기 로딩 속도가 빨라집니다.

### 디버깅 방법
- Chrome DevTools → Network 탭에서 Chart.js가 처음에 로드되지 않고, 클릭하거나 렌더링 시점에 로드되는 것을 확인할 수 있습니다.
- npm run build 후 static/js 디렉토리에서 번들이 분리된 것도 확인 가능합니다.


---


### 5. 테스트와 디버깅 – 정확성을 위한 습관
-  어디가 느린지 눈으로 보기 (Lighthouse, Profiler)

React에서는 @testing-library/react + Jest 조합으로 단위 테스트를 작성합니다. 
예를 들어 카운터 테스트는 이렇게 작성할 수 있습니다:
```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('버튼을 누르면 카운트가 1 증가해야 한다', () => {
  render(<Counter />);
  expect(screen.getByTestId('count')).toHaveTextContent('현재 숫자: 0');
  fireEvent.click(screen.getByText('+'));
  expect(screen.getByTestId('count')).toHaveTextContent('현재 숫자: 1');
});
```

### 디버깅 시 주의할 점

- 버튼 클릭 후 텍스트를 검증하려면 screen.getByTestId(...)를 **다시 호출해야 최신 상태를 반영합니다.**
- console.log, debugger, 그리고 React Profiler는 가장 강력한 실시간 분석 도구입니다.



### 6. 테스트 전략
-  코드 품질 보장: 단위~E2E까지
### 테스트 계층

|**유형**|**목적**|**대표 도구**|
|---|---|---|
|Unit Test|함수/컴포넌트 단위 테스트|Jest|
|Integration|컴포넌트 간 상호작용 테스트|React Testing Library|
|E2E (End-to-End)|전체 흐름, 브라우저 테스트|Cypress, Playwright|

### 예시 코드 (Testing Library)
```js
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button with text', () => {
  render(<Button text="Click me" />);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});
```





### 7. 결론
성능은 눈에 보이는 속도, 테스트는 눈에 보이지 않는 정확성입니다.
이 둘을 모두 잡기 위해서는 JS의 실행 구조를 이해하고, React 최적화를 알고, 테스트를 습관처럼 작성해야 합니다.
그 시작은 콘솔에 console.dir(fn) 찍어보는 것부터입니다.








## 🔗 참고 자료
