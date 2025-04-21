# 🎨 실무에서 통하는 디자인 시스템 설계와 Storybook 활용법

React 프로젝트를 진행하다 보면 자연스럽게 드는 생각이 하나 있어요.
**“공통 UI 요소가 자꾸 반복되는데, 좀 더 체계적으로 정리할 수 없을까?”**
이때 필요한 게 바로 디자인 시스템입니다. 그리고 이를 문서화하고 테스트할 수 있는 최고의 도구는 바로 Storybook이죠.

이번 글에서는 디자인 시스템을 왜 쓰는지부터,
어떻게 구성하면 실무에 유리한지, 그리고 Storybook을 통해 어떻게 문서화하고 협업까지 연결할 수 있는지를 정리해봤어요.

---

## 🧩 디자인 시스템이란?

디자인 시스템은 **일관된 UI/UX를 제공하기 위한 설계 기준**이에요.
예를 들어 팀원들이 만든 버튼이 다 제각각이면 유지보수도 어렵고 사용자 경험도 들쑥날쑥하죠.
이런 문제를 막기 위해 **공통 컴포넌트, 스타일 가이드, 디자인 토큰** 등을 하나로 정리한 시스템이 필요합니다.

- 버튼, 인풋, 카드 같은 공통 컴포넌트
- 컬러, 폰트, 여백 같은 스타일 기준값 (디자인 토큰)
- 사용법과 예제, 문서화

이런 것들을 미리 정리해두면 **개발도 빨라지고, 유지보수도 쉬워지고, 협업도 편해집니다.**

#### 실무에서 디자인 시스템은 이렇게 사용돼요

- 디자이너와 협업 시 Figma 토큰을 기준으로 개발자가 변환해서 코드에 반영
- 컴포넌트가 중복되지 않도록 공유된 **디자인 토큰 기반으로 개발**
- 새로운 페이지가 생겨도 **공통 컴포넌트 조합**만으로 빠르게 개발 가능
- 팀 규모가 커질수록 Storybook을 **내부 개발자용 UI 가이드로 배포**

---

## 🛠 실무에서 사용하는 구조

**토큰과 컴포넌트를 분리**해두면 규모가 커져도 정리가 잘 돼요.
```bash
src/
├── components/         # Button, Input, Card 등 공통 UI 컴포넌트
├── designSystem/
│   ├── tokens/         # 색상, 타이포, 여백 등 디자인 토큰
│   │   ├── colors.js
│   │   ├── spacing.js
│   │   └── typography.js
│   └── index.js        # 모든 토큰 모듈화
├── pages/              # 실제 페이지 단위 컴포넌트
└── App.jsx
```


---

## 🎨 디자인 토큰 작성 예시

디자인 토큰은 **스타일 기준값을 변수처럼 관리하는 것**이에요.
한 번만 정의해두면, 나중에 전체 테마 바꾸기도 편하고, 디자이너 피드백 반영도 훨씬 수월해요.

```js
// designSystem/tokens/colors.js
export const colors = {
  primary: "#3B82F6", // Tailwind blue-500
  secondary: "#F3F4F6",
  danger: "#EF4444",
  text: "#1F2937",
};

// designSystem/tokens/typography.js
export const fontSize = {
  sm: "0.875rem",
  base: "1rem",
  xl: "1.25rem",
};

```


---

## 📦 공통 컴포넌트 작성 방식

variant, icon, disabled 같은 props도 추가해두면 더 유연한 컴포넌트가 됩니다.

```js
// components/Button.jsx
import { colors } from "../designSystem/tokens/colors";

export default function Button({ label, onClick, variant = "primary" }) {
  const style = {
    backgroundColor: colors[variant],
    color: "#fff",
    padding: "8px 16px",
    borderRadius: "8px",
    fontSize: "0.875rem",
  };

  return <button onClick={onClick} style={style}>{label}</button>;
}
```


---

## 🧪 Storybook으로 컴포넌트 문서화

Storybook은 컴포넌트를 독립적으로 개발하고 테스트할 수 있도록 해주는 도구예요.
디자이너, 기획자, QA가 **개발 서버를 보지 않고도 UI 상태를 확인**할 수 있게 해주죠.
Docs 탭, Controls, Canvas를 통해 컴포넌트의 상태와 props를 바로 확인하고 테스트할 수 있어요.

Storybook에서는
- 컴포넌트 상태별 stories 작성 (Primary, Disabled, WithIcon 등)
- **Docs 탭**: 컴포넌트 사용 방법과  UI 상태 등을 문서처럼 자동 정리
- **Controls 탭**: props 값을 직접 조절해보면서 테스트 가능

```bash
npx storybook init
npm run storybook
```

```js
// Button.stories.jsx
import Button from "./Button";

export default {
  title: "Components/Button",
  component: Button,
};

export const Primary = {
  args: {
    label: "확인",
    variant: "primary",
  },
};

```


---

## 💼 실무에서 어떻게 사용하나요?

- 디자인 시스템이 있는 회사에서는 **Figma 디자인 → 토큰으로 변환 → 컴포넌트에 적용**
- Storybook을 통해 팀원들에게 **공통 UI를 보여주고 가이드라인처럼 활용**
- 규모가 클수록, 팀원이 많을수록 **디자인 시스템과 Storybook은 필수**예요
- 배포된 Storybook은 퍼블릭 문서처럼 외부에 공개할 수도 있고, 디자인 QA에도 유용해요.


---

## 🚀 Storybook 배포 및 디자인 토큰 자동화는?

### **1. Storybook 배포**
- GitHub Pages / Vercel / Netlify에 정적 사이트로 배포 가능
- storybook-static 폴더를 GitHub Actions로 자동 배포
- 디자이너나 팀원에게 배포된 링크로 컴포넌트 가이드를 제공
- .storybook 폴더 내 설정을 정리해 build-storybook 명령어로 HTML 생성

```bash
npm run build-storybook
```

### **2. 디자인 토큰 자동화**

- Figma에서 디자인 토큰을 export → style-dictionary로 변환 → 코드 자동 생성 가능
- CSS 변수 → JS 모듈로 변환
- 혹은 Tailwind의 theme.extend에 직접 바인딩
- 테마를 여러 개로 나눠도 유지보수가 쉬워짐

이렇게 하면 디자이너가 디자인을 바꾸면, 바로 코드에 반영되도록 자동화할 수 있어요.

---

## **✍️ 마무리하며**

처음엔 ‘왜 굳이 이렇게까지 해야 해?’ 싶을 수 있지만,
컴포넌트가 많아지고, 팀원이 늘어나고, 유지보수를 하게 될수록 디자인 시스템의 필요성을 절실히 느낍니다.

작은 프로젝트부터라도 **토큰 하나, 버튼 하나부터** 시작해보세요.
다음 글에서는 실제로 디자인 토큰을 자동 변환하고, Storybook을 GitHub Pages에 배포하는 과정을 공유할게요!
