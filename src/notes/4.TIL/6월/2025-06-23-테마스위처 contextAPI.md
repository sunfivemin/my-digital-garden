---
tags:
  - TIL
  - React
created: 2025-06-23
---
# 📘 2025-06-23 TIL

## 📌 오늘 배운 핵심 요약
- ReactNode의 활용 이유와 범위
- ThemeProvider + useTheme 훅 구현하여 전역 다크모드 상태 관리
- Vanilla Extract 스타일에 classList 방식 적용
- DarkModeToggle 컴포넌트 Context 방식으로 리팩토링


## 🧠 상세 학습 내용
## 📍 주제 1: children 타입 정의 – JSX.Element vs ReactElement vs ReactNode

React에서 컴포넌트의 자식(children) 요소를 어떻게 타입 지정해야 하는지 혼란스러울 수 있다. 특히 TypeScript를 사용할 때 JSX.Element, ReactElement, ReactNode 중 무엇을 사용해야 할지 헷갈린다.

| **타입**       | **설명**                                         |
| ------------ | ---------------------------------------------- |
| JSX.Element  | `<div>hi</div>`와 같이 JSX 문법으로 생성된 단일 요소         |
| ReactElement | React 내부에서 사용하는 JSX 표현                         |
| ReactNode    | **모든 렌더링 가능한 타입** 포함 (JSX, 문자열, 숫자, null 등 포함) |


### ✅  실무에서는 가장 범용성 있는 ReactNode를 주로 사용한다.
```tsx
interface Props {
  children: React.ReactNode;
}
```
이렇게 하면 문자열, 숫자, JSX, 컴포넌트, null 등 다양한 타입을 허용한다.

---

## 🧩 실전 예시: Layout 컴포넌트
App 전체의 구조를 담당하는 Layout 컴포넌트를 구현해보자.
#### 🗂 폴더 구조
```bash
src/
├── components/
│   └── ui/
│       ├── Button/
│       └── DarkModeToggle.tsx
├── contexts/
│   └── ThemeContext.ts
├── hooks/
│   └── useTheme.ts
├── providers/
│   └── ThemeProvider.tsx
├── pages/
│   ├── Home.tsx
│   └── Detail.tsx
├── styles/
│   ├── App.css.ts
│   └── theme.css.ts
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


---

## 📍 주제 2: 테마 스위처 (Theme Switcher) with Context API
#### 💡 styled-components 기반 테마 시스템 + ContextAPI 개념과 Vanilla Extract 적용 비교 분석

#### ✅ 1. 왜 테마 시스템이 필요한가?
테마 시스템은 디자인 시스템의 일관성을 유지하고, 사용자 경험(UX)을 개선하며, 유지보수성을 높이기 위한 핵심 설계 방식이다.

#### 📌 테마 시스템의 주요 이점
1. **UI, UX의 일관성 유지**
2. **유지보수성 향상** - 색상 하나 바꾸면 전체 반영 가능
3. **확장성** - 다크모드, 하이콘트라스트 모드 등 유연하게 확장 가능
4. **재사용성** - 중복 없이 스타일 공통화
5. **사용자 정의 가능성** - 사용자가 테마 선택 가능

---

### 🎨 2. styled-components + ThemeProvider 구조 분석

#### 2-1. 테마 정의
```ts
// theme.ts
export const light = {
  name: 'light',
  color: {
    primary: 'brown',
    background: 'lightgray',
    secondary: 'blue',
    third: 'green',
  },
};

export const dark = {
  name: 'dark',
  color: {
    primary: 'coral',
    background: 'midnightblue',
    secondary: 'darkblue',
    third: 'darkgreen',
  },
};
```

#### 2-2. ThemeProvider로 감싸기
```tsx
// App.tsx
import { ThemeProvider } from 'styled-components';
import { dark, light } from './theme';

function App() {
  return (
    <ThemeProvider theme={light}>
      <AppLayout />
    </ThemeProvider>
  );
}
```

#### 2-3. styled-components 내 테마 사용
```tsx
const HeaderStyle = styled.header`
  background-color: ${({ theme }) => theme.color.background};
  h1 {
    color: ${({ theme }) => theme.color.primary};
  }
`;
```

---

### 🔄 3. Context API를 통한 테마 전환 구현

#### 3-1. ThemeContext 생성
```tsx
export const ThemeContext = createContext();

export const ThemeProviderCustom = ({ children }) => {
  const [theme, setTheme] = useState(light);

  const toggleTheme = () => {
    setTheme(theme.name === 'light' ? dark : light);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <ThemeProvider theme={theme}>
        {children}
      </ThemeProvider>
    </ThemeContext.Provider>
  );
};
```

#### 3-2. 사용 예시
```tsx
const { theme, toggleTheme } = useContext(ThemeContext);
<button onClick={toggleTheme}>다크모드 전환</button>
```

---

### 🧁 Vanilla Extract + classList 적용 방식

#### 4-1. 테마 설정 방식
```ts
// theme.css.ts
export const vars = createGlobalTheme(':root', {
  color: {
    background: '#ffffff',
    text: '#111111',
    primary: '#0070f3',
  },
  font: {
    family: 'Pretendard, system-ui, sans-serif',
  },
});

export const darkThemeClass = createTheme(vars, {
  color: {
    background: '#111111',
    text: '#ffffff',
    primary: '#1e90ff',
  },
});
```

#### 4-2. 전역 테마 전환 (useEffect)
```tsx
useEffect(() => {
  document.documentElement.classList.toggle(darkThemeClass, isDark);
}, [isDark]);
```

#### ✅ 핵심 차이점
|항목|styled-components|Vanilla Extract|
|---|---|---|
|테마 적용 방식|ThemeProvider + context|`classList`로 테마 클래스 적용|
|런타임 스타일 접근|`theme.color.primary` 등 props 통해 접근|정적인 변수로만 구성됨|
|퍼포먼스|JS 런타임 계산 필요|정적 CSS로 최적화 됨|
|타입 안전성|유연하지만 제한적|`createThemeContract`로 강력한 타입 제공|

- **styled-components**는 `ThemeProvider`로 감싸고, `theme` prop을 통해 동적으로 스타일을 계산하는 방식
- **Vanilla Extract**는 CSS 클래스 기반이기 때문에 테마 클래스를 HTML에 직접 토글해줘야 함 (`classList.add/remove`)
- Vanilla Extract에서는 “스타일 자체의 토글”은 classList 전환으로 하되, Context API는 “전역 상태(예: 다크모드 여부)“를 앱 전역에서 공유하는 용도로는 매우 유용하고 권장됨.


#### 💡 Context API가 Vanilla Extract에서 “권장되는 영역”
|**영역**|**설명**|**예시**|
|---|---|---|
|✅ 테마 상태 전역 공유|isDarkMode 상태를 Context로 관리하고, 다크모드 버튼이나 컴포넌트에서 참조|useTheme() 훅으로 접근|
|✅ 토글 핸들러 공유|토글 함수를 context로 공유해서 어디서든 테마 전환 가능|theme.toggle()|
|✅ 퍼시스턴스 저장소 연동|localStorage 등과 연계하여 사용자 설정 기억|최초 mount 시 useEffect에서 적용|
|✅ UI 동기화 (icon, text 등)|🌙 / ☀️ 아이콘이나 “다크모드 켜기/끄기” 텍스트 제어|isDark ? "Light" : "Dark"|


---


### 🌗 Vanilla Extract + Context API로 구현하는 다크모드 전환

#### ✅ 왜 Context API를 써야 할까?
처음엔 이렇게 했을 수도 있다:
```tsx
import { useEffect, useState } from 'react';
import { darkThemeClass } from '@/styles/theme.css';

const DarkModeToggle = () => {
  const [isDark, setIsDark] = useState(false); // 다크모드 여부 상태

  useEffect(() => {
    const root = document.documentElement; // <html> 요소 선택
    // 상태에 따라 class를 토글한다
    root.classList.toggle('dark', isDark);
    root.classList.toggle(darkThemeClass, isDark);
  }, [isDark]);

  return (
    <button onClick={() => setIsDark(prev => !prev)}>
      {isDark ? '🌙 다크모드 해제' : '🌞 다크모드 적용'}
    </button>
  );
};

export default DarkModeToggle;
```

### ❗ 이 방식의 한계
- 여러 컴포넌트가 이 isDark 상태를 공유할 수 없음
- 다른 컴포넌트에서 테마 상태를 알 수 없음
- 다크모드 상태를 Context로 분리하지 않으면 **재사용 불가**

그래서 우리는 **Context API를 사용해서 전역적으로 테마 상태를 관리**하도록 개선하는 것이 좋다.

#### 🛠 ThemeProvider 구현 (전역 상태 관리)
```tsx
// src/providers/ThemeProvider.tsx
import { useEffect, useState } from 'react';
import { darkThemeClass } from '@/styles/theme.css';
import { ThemeContext } from '@/contexts/ThemeContext';

export const ThemeProvider = ({ children }: { children: React.ReactNode }) => {
  const [isDark, setIsDark] = useState(() => {
    if (typeof window !== 'undefined') {
      return document.documentElement.classList.contains('dark');
    }
    return false;
  });

  useEffect(() => {
    const root = document.documentElement;
    root.classList.toggle('dark', isDark);
    root.classList.toggle(darkThemeClass, isDark);
  }, [isDark]);

  const toggleTheme = () => setIsDark(prev => !prev);

  return (
    <ThemeContext.Provider value={{ isDark, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

---

#### 💡 다크모드 토글 버튼: DarkModeToggle
```tsx
// src/components/ui/DarkModeToggle.tsx
import { useTheme } from '@/hooks/useTheme';

const DarkModeToggle = () => {
  const { isDark, toggleTheme } = useTheme();

  return (
    <button onClick={toggleTheme}>
      {isDark ? '🌙 다크모드 해제' : '🌞 다크모드 적용'}
    </button>
  );
};

export default DarkModeToggle;
```

---

#### 🔧 커스텀 훅 정의
```ts
// src/hooks/useTheme.ts
import { ThemeContext } from '@/contexts/ThemeContext';
import { useContext } from 'react';

export const useTheme = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
};
```


---

#### 🧱 ThemeContext 정의
```ts
// src/contexts/ThemeContext.ts
import { createContext } from 'react';

export interface ThemeContextType {
  isDark: boolean;
  toggleTheme: () => void;
}

export const ThemeContext = createContext<ThemeContextType | undefined>(
  undefined
);
```


---

#### 🧪 App.tsx에서 적용
```tsx
// src/App.tsx
import { app } from '@/styles/App.css';
import Layout from '@/components/layout/Layout';
import { Button } from '@/components/ui/Button/Button';
import Home from '@/pages/Home';
import Detail from '@/pages/Detail';
import DarkModeToggle from '@/components/ui/DarkModeToggle';
import { ThemeProvider } from '@/providers/ThemeProvider';

function App() {
  return (
    <ThemeProvider>
      <div className={app}>
        <h1>Hello, Vanilla Extract + Tailwind Variants!</h1>
        <Layout>
          <Home />
          <Detail />
          <DarkModeToggle />
          <Button intent="primary" size="sm">Primary Small</Button>
          <Button intent="danger" size="md">Danger Medium</Button>
          <Button intent="ghost" size="lg">Ghost Large</Button>
          <p className="font-kkokghae text-lg">이건 꽃게체입니다</p>
          <p className="font-pretendard text-base">이건 프리텐다드입니다</p>
          <p className="font-serifk italic">이건 세리프체입니다</p>
          <div className="bg-white text-black dark:bg-black dark:text-white">
            다크모드 테스트
          </div>
        </Layout>
      </div>
    </ThemeProvider>
  );
}

export default App;
```

---

#### 🪄 정리

| **장점** | **설명**                                     |
| ------ | ------------------------------------------ |
| 상태 공유  | 모든 컴포넌트에서 테마 상태 접근 가능                      |
| 스타일 분리 | theme.css.ts로 전역 테마 정의 가능                  |
| 유지보수   | 테마 로직 중앙 집중, 컴포넌트별 깔끔                      |
| 타입 안전성 | Vanilla Extract와 Context 모두 TypeScript 친화적 |

---

#### 📌 결론
Vanilla Extract는 스타일을 타입 안전하게 구성하고 테마 분기를 CSS 변수로 제어할 수 있는 장점이 있다. 여기에 **Context API를 이용한 전역 상태 관리**를 함께 적용하면, **실무에 강한 테마 시스템을 구성**할 수 있다.

🤝 Vanilla Extract + Context API = 확장성과 유지보수성이 뛰어난 테마 아키텍처


![lightmode](https://seonohblog.netlify.app/assets/lightmode.png)

![darkmode](https://seonohblog.netlify.app/assets/darkmode.png)


---

## 💭 회고
• **새롭게 알게 된 점**
- Vanilla Extract에서는 classList 전환 방식으로 테마를 적용하며, 전역 상태는 Context로 따로 관리해야 한다는 점

• **어렵게 느껴졌던 부분**
- darkThemeClass를 classList에 적용할 때 초기 상태와 동기화하는 부분

• **다음에 학습할 주제**
- localStorage에 사용자 테마 저장 및 적용

### **🔗 참고자료**
- [Vanilla Extract 공식 문서](https://vanilla-extract.style/)
    - createGlobalTheme, createTheme, createThemeContract의 사용법
    - classList 기반 테마 전환 예시와 권장 방식 포함
- [React 공식 문서 - Context API](https://reactjs.org/docs/context.html)
    - 전역 상태 관리 방식, ThemeContext 작성 방법
- [TypeScript 공식 문서 - JSX 타입](https://www.typescriptlang.org/docs/handbook/jsx.html)
	- ReactNode, JSX.Element, ReactElement의 타입 차이 명확히 설명