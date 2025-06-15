
# dot.daily 버튼 시스템 설계부터 적용, 문서화까지

Next.js 기반 사이드 프로젝트 **dot.daily**에서 공통 UI 컴포넌트를 설계하며 class-variance-authority (cva) 기반의 버튼 시스템을 구현했습니다. 또한 Tailwind의 사용자 정의 색상과 함께 Storybook으로 문서화하여 협업 가능성과 UI 일관성을 높였습니다. 이 글은 그 흐름과 실습 내용을 정리한 기술 회고입니다.

---

## 🎯 목표
- ✅ 다양한 버튼 스타일을 하나의 컴포넌트로 통합
- ✅ `variant`, `size`, `rounded`, `fullWidth` 등 옵션화하여 선언형 사용 가능
- ✅ Figma 디자인 토큰 (primary blue, text-strong 등) 반영
- ✅ Storybook을 활용한 UI 상태 시각화 및 문서화
- ✅ Tailwind v4 purge 이슈 해결 경험 공유

---

## 📦 사용 기술 스택

- Next.js (App Router)
- Tailwind CSS (v3)
- class-variance-authority (cva)
- Storybook
```bash
# 설치
npm install class-variance-authority clsx
npm install -D @storybook/nextjs
```

---

## ⚠️ 이슈: Tailwind v4에서 Storybook 사용자 정의 색상 미반영

처음에는 Tailwind v4 환경에서 진행했지만 다음과 같은 문제가 발생:
- `theme.extend.colors`에 정의한 사용자 색상이 purge로 인해 적용되지 않음
- `.storybook/preview.ts`에 글로벌 CSS가 정상 반영되지 않음

#### ✅ 해결 방법
Tailwind를 v3로 다운그레이드:
```bash
npm uninstall tailwindcss
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

📍 `.storybook/preview.ts`에 글로벌 CSS 적용 필수:
```bash
import '../src/app/globals.css';
```

#### 🎨 디자인 토큰 적용
Tailwind 색상은 tailwind.config.ts에서 의미 기반으로 정의:
```ts
import colors from 'tailwindcss/colors';

theme: {
  extend: {
    colors: {
      brand: {
		primary: colors.blue[500], // 로고 강조색
		secondary: colors.indigo[500], // 보조 강조 텍스트
      },
      text: {
        strong: '#1F2937',
        weak: '#6B7280',
      },
      border: {
        default: '#E5E7EB',
      },
    },
  },
},
```


### 💡 class-variance-authority (CVA)란?
CVA는  Tailwind CSS 환경에서 컴포넌트의 스타일 변형(variants) 관리를 돕는 라이브러리입니다. 
Tailwind Variants는 Stitches에서 영감을 받아, variants, slots, compoundVariants 등 더 복잡한 컴포넌트 구조와 반응형 스타일링을 손쉽게 관리할 수 있습니다. 반면, CVA는 단일 컴포넌트의 변형 관리에 특화되어 있으며, TypeScript와의 통합으로 타입 안전성이 뛰어납니다.

•	Tailwind Variants는 더 복잡한 컴포넌트(슬롯, 컴파운드 variants 등)가 필요할 때 강점이 있지만,
•	dot.daily처럼 버튼, 인풋, 모달 등 단일 UI 컴포넌트 중심의 프로젝트에는 CVA가 더 심플하고 빠르게 적용할 수 있습니다.
•	특히 Storybook과의 연동, 타입스크립트 기반의 안전성, 팀 협업 측면에서 CVA가 더 널리 쓰이고 있습니다.

✅ Storybook과 CVA의 조합은 타입 안전한 스타일 선언과 시각적 문서화를 결합해, 디자인 시스템 기반 UI 개발에 최적화된 워크플로우를 제공합니다.
✅ 디자인 시스템 기반 폴더 구조는 Storybook 자체를 의미하는 것이 아니라, 컴포넌트, 스타일, 문서화 파일을 역할별로 분리해 관리하는 구조입니다.
✅ Storybook은 이런 구조에서 컴포넌트의 다양한 상태와 옵션을 문서화하는 도구로 활용됩니다.

#### 🧱 버튼 스타일 정의 (buttonVariants.ts)
```ts
import { cva } from 'class-variance-authority';

export const buttonVariants = cva(
  'inline-flex items-center justify-center font-semibold transition whitespace-nowrap disabled:opacity-50 disabled:pointer-events-none',
  {
    variants: {
      variant: {
        /** 강조 버튼: ex) 오늘 회고 작성하기 */
        primary: 'bg-brand-primary text-white hover:bg-brand-primary/90',

        /** 보조 버튼: ex) 선택한 날짜로 이동 */
        secondary: 'bg-white text-brand-primary border border-brand-primary hover:bg-brand-primary/10',

        /** 파괴적 액션: ex) 삭제하기 */
        danger: 'bg-red-500 text-white hover:bg-red-600',

        /** 텍스트 버튼: 배경 없음 */
        ghost: 'bg-transparent text-brand-primary hover:bg-brand-primary/10',
      },
      size: {
        /** 기본 모바일 버튼 (대부분의 경우) */
        sm: 'h-9 px-3 text-sm',

        /** 피그마 기준: 보통 버튼 (예: 회고 등록하기) */
        md: 'h-10 px-4 text-base',

        /** 큰 버튼이 필요할 경우 (거의 없음) */
        lg: 'h-12 px-6 text-lg',
      },
      rounded: {
        none: '',
        md: 'rounded-md',
        full: 'rounded-full',
      },
      fullWidth: {
        true: 'w-full',
        false: '',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
      rounded: 'full',
      fullWidth: false,
    },
  }
);
```

---

### ✍️ 사용 예시
```tsx
<Button label="오늘 회고 작성하기" variant="primary" size="md" fullWidth />
<Button label="선택한 날짜로 이동" variant="secondary" size="md" fullWidth />
<Button label="삭제" variant="danger" size="sm" />
<Button label="로그아웃" variant="ghost" size="sm" />
```

---

## 🧩 버튼 컴포넌트 구현 (Button.tsx)

```tsx
import { clsx } from 'clsx';
import { buttonVariants } from '@/lib/styles/buttonVariants';
import type { VariantProps } from 'class-variance-authority';

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  label: string;
}

export const Button = ({
  label,
  variant,
  size,
  rounded,
  fullWidth,
  className,
  ...props
}: ButtonProps) => {
  return (
    <button
      className={clsx(buttonVariants({ variant, size, rounded, fullWidth }), className)}
      {...props}
    >
      {label}
    </button>
  );
};
```

---

## 📘 Storybook 설정 & 문서화

### 1. `.storybook/main.ts`
```ts
export default {
  stories: ['../src/**/*.stories.@(js|ts|jsx|tsx)'],
  addons: ['@storybook/addon-links', '@storybook/addon-essentials'],
  framework: '@storybook/nextjs',
};
```

### 2. `.storybook/preview.ts`
```ts
import '../src/app/globals.css';
```

### 3. 버튼 스토리 예시
```tsx
import { Button } from './Button';

export default {
  title: 'UI/Button',
  component: Button,
};

export const Primary = {
  args: {
    label: '오늘 회고 작성하기',
    variant: 'primary',
    size: 'md',
  },
};

export const Outline = {
  args: {
    label: '선택한 날짜로 이동',
    variant: 'outline',
  },
};
```

---

## 🗂️ 디렉토리 구조

```bash
dot-daily/
├── public/                            # 정적 리소스 (로고, 이모지, SVG 아이콘 등)
│   └── logo.svg
│   └── icons/                         # Footer 아이콘 on/off
│
├── src/
│   ├── app/                           # Next.js App Router 기반 페이지 라우팅
│   │   ├── page.tsx                   # / (MyDay 홈)
│   │   ├── retrospect/page.tsx        # /retrospect (회고 탭)
│   │   ├── archive/page.tsx           # /archive (보류함 탭)
│   │   └── profile/page.tsx           # /profile (마이페이지 탭)
│   │
│   ├── components/                    # 공통 UI 컴포넌트 (디자인 시스템 기반)
│   │   ├── ui/
│   │   │   ├── Button.tsx             # 버튼 컴포넌트
│   │   │   ├── Button.stories.tsx     # Storybook 문서화
│   │   │   └── ...                    # Input, Modal 등 기타 공통 컴포넌트 예정
│   │   ├── layout/
│   │   │   ├── Header.tsx             # 공통 헤더 (title prop)
│   │   │   └── Footer.tsx             # 공통 푸터 (탭 이동)
│
│   ├── features/                      # 도메인 기능별 구조화
│   │   ├── myday/                     # 투두 기능 (홈)
│   │   │   ├── components/            # 도메인 전용 UI 컴포넌트
│   │   │   │   ├── TaskItem.tsx
│   │   │   │   ├── TaskList.tsx
│   │   │   │   ├── TaskInput.tsx
│   │   │   │   ├── TaskSection.tsx
│   │   │   │   └── index.ts           # barrel export
│   │   │   ├── api.ts                 # /api/tasks 관련 fetch 함수
│   │   │   ├── store.ts               # useMydayStore (Zustand 상태)
│   │   │   └── utils.ts               # 날짜 정렬/필터 등 유틸 함수
│   │   └── retrospect/                # 회고 기능 (예정)
│   │       ├── components/
│   │       ├── api.ts
│   │       ├── store.ts
│   │       └── utils.ts
│
│   ├── constants/                     # 우선순위/이모지/상태 라벨 등 공통 상수
│   │   ├── priority.ts                # PRIORITY_OPTIONS
│   │   ├── emotion.ts                 # EMOTION_TAGS
│   │   └── ...
│
│   ├── lib/
│   │   └── styles/
│   │       └── buttonVariants.ts      # class-variance-authority 기반 버튼 스타일
│
│   ├── hooks/                         # 공통 커스텀 훅 저장소 (예: useModal, useOutsideClick)
│
│   ├── styles/                        # 글로벌 CSS 및 Tailwind 설정
│   │   ├── globals.css                # tailwind base/components/utilities 포함
│
│   └── types/                         # 전역 타입 정의 (필요 시 추가)
│
├── .storybook/                       # Storybook 설정
│   ├── main.ts                        # stories 경로, addons, 프레임워크 설정
│   └── preview.ts                     # 글로벌 CSS 포함 등 환경 설정
│
├── tailwind.config.ts                # 디자인 토큰 (colors 등) 포함
├── tsconfig.json                     # 타입스크립트 설정
├── package.json
└── README.md                         # 프로젝트 소개 및 실행 방법
```

- **src/app/**: 라우팅 전용 (App Router 기준)
- **components/**: 공통 UI 컴포넌트 (Button, Header 등)
- **features/**: 도메인별 상태/로직/컴포넌트 분리
- **lib/styles/buttonVariants.ts**: class-variance-authority로 버튼 스타일 선언
- **storybook/**: Storybook 설정과 preview 연동
- **constants/**: 우선순위/이모지 등의 상수 정의
- **Tailwind + 글로벌 스타일 통합**: purge 문제 없이 정상 작동
  
---

## 🔚 마무리
버튼 하나라도 디자인 시스템과 cva, Storybook을 활용해 설계하면 다음과 같은 이점이 있습니다:
- 일관된 UI/UX 유지
- 유지보수 용이성 향상
- 팀 간 커뮤니케이션 최소화
- 상태 시각화 → QA/디자이너 확인 편의성

향후에는 모달, 토글, 인풋 등도 이 구조로 확장 예정입니다.


