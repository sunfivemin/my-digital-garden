
# 🗓 dot.daily — 

> “오늘 하루를 정리하는 가장 직관적인 방법”  
> dot.daily는 단순한 투두 리스트를 넘어, 감정 기록과 회고 기능까지 포함한 **하루 단위 자기관리 플랫폼**입니다.

이번 글에서는 `dot.daily` 프로젝트가 어떻게 **Next.js + Tailwind + Storybook 기반 디자인 시스템**으로 구성되었는지, 그리고 실제 서비스 구현에 있어 어떤 폴더 구조와 기술 스택을 선택했는지를 정리합니다.  
또한 지난 글 [[Next.js + Tailwind + Storybook 기반 디자인 시스템 구축기]]와의 연결도 함께 다룹니다.

---

## 🎯 핵심 목표

- ✅ 기능별 도메인 구조를 통한 유지보수성 향상
- ✅ Zustand를 활용한 전역 상태관리 (점진적 확장 예정)
- ✅ 공통 디자인 시스템(Button, Input, Modal, Badge 등)의 체계적 관리
- ✅ Storybook으로 UI 문서화 → 컴포넌트 테스트/협업 용이
- ✅ 회고 기능과 감정 태그, 우선순위 기반 투두 정리 기능 포함

---

## 🔧 기술 스택

|구분|기술|
|---|---|
|프레임워크|Next.js (App Router 기반)|
|스타일링|Tailwind CSS (v3로 다운그레이드하여 purge 이슈 해결)|
|컴포넌트 스타일|class-variance-authority (cva) + clsx|
|문서화|Storybook + 디자인 시스템 명세화|
|상태관리|Zustand (간결한 전역 상태 관리 → 추후 persist, slice 확장)|
|애니메이션|Framer Motion|
|드래그 앤 드롭|@dnd-kit/core (계획) 또는 react-beautiful-dnd|
|배포|Vercel|

---

## 🧩 프로젝트 구조 및 확장성 설계

```bash
src/
├── app/                       # Next.js App Router 기반 페이지
│   ├── page.tsx              # / (MyDay)
│   ├── retrospect/page.tsx   # /retrospect
│   ├── archive/page.tsx      # /archive
│   └── profile/page.tsx      # /profile
│
├── components/               # 공통 UI 컴포넌트 (디자인 시스템)
│   ├── ui/                   # Button, Input, Badge, Modal 등
│   ├── layout/               # Header, Footer
│   └── feedback/             # 예: Toast, Skeleton 등 예정
│
├── features/                 # 기능별 도메인 구조
│   ├── myday/                # 투두 기능
│   └── retrospect/           # 회고 기능
│       ├── components/       # RetrospectCard 등 (예정)
│       ├── api.ts
│       ├── store.ts
│       └── utils.ts
│
├── constants/                # 우선순위/감정/이모지 등 상수 관리
├── hooks/                    # 공통 커스텀 훅
├── lib/styles/               # cva variants 등 스타일 모듈화
├── styles/                   # 글로벌 스타일 (globals.css)
├── types/                    # 전역 타입 정의
└── .storybook/               # Storybook 설정
```
