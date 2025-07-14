# 🛠️ Next.js에서 발생하는 플리커(Flicker)와 레이아웃 시프트 문제 해결기


> hydration 지연, 이미지/폰트 로딩, 레이아웃 밀림 현상을 어떻게 개선했는가?

---
## 🧠 플리커와 레이아웃 시프트란?

### ❓ 플리커(Flicker)란?
- 화면이 렌더링된 직후, **일시적으로 레이아웃이 깨졌다가 복구되는 현상**을 의미합니다.
- 주로 **Next.js의 hydration 과정**에서 발생하며, SSR → CSR 전환 시 발생합니다.

### ❓ 레이아웃 시프트(Layout Shift)란?
- **로딩 중 콘텐츠의 위치가 갑자기 바뀌는 현상**입니다.
- 사용자 입장에서는 콘텐츠가 움직이는 듯 보여 불쾌감을 유발합니다.

---

## ⚙️ 원인 분석: 왜 이런 문제가 생길까?

### 1. Next.js의 렌더링 구조
- **SSR (Server Side Rendering)**: 서버에서 HTML을 미리 만들어 클라이언트에 전달
- **Hydration**: 클라이언트에서 React 앱으로 변환하는 과정
- 이때, **스타일/이미지/폰트가 늦게 적용되면 shift 발생**

### 2. CSR(fetch) 로딩 지연
- 클라이언트에서 데이터를 받아오는데 시간이 걸리면,
- Skeleton → 실제 콘텐츠 전환 과정에서 shift 발생

### 3. 폰트/이미지 로딩 지연
- 커스텀 폰트, SVG, 아이콘 등이 늦게 뜨면 요소 크기가 변형

### 4. h-screen / min-h-screen / flex 레이아웃 충돌
- Tailwind의 height 관련 클래스, flex, overflow-hidden 등이 복잡하게 얽히면 레이아웃 문제 발생
  
### 5. 모바일 뷰포트 변화
- 모바일 브라우저의 주소창/툴바 숨김으로 인해 화면 높이가 바뀌며 layout shift 유발

---

## ✅ SSR(Server Components)로 전환한 이유

```tsx
// 기존: 클라이언트 fetch 기반 CSR
const { data, isLoading } = useQuery(...);
```

- CSR 방식은 hydration 지연이 발생하고, 레이아웃이 깜빡이는 문제가 많았음
- SSR(Server Components)로 데이터를 미리 받아서 렌더하면
    - HTML에 데이터가 포함되어 있어 hydration 전에 레이아웃이 안정적으로 출력됨
    - 따라서 “로딩 중 빈 공간 → 데이터 뜨면서 shift” 현상이 줄어듦


---

## 🧠 Next.js의 SSR 전략 정리

Next.js에서는 다음과 같은 **3가지 서버 렌더링 전략**을 제공합니다.

### 1. Static Rendering (SSG, 기본값)
- 빌드 타임에 정적으로 HTML을 생성하거나, revalidate 기준으로 백그라운드 재생성
- 결과는 캐시되어 CDN에 푸시되어 빠른 응답 가능
- 블로그, 마케팅 페이지 등 자주 바뀌지 않는 페이지에 적합

### 2. Dynamic Rendering (전통적인 SSR)
- 요청 시마다 서버에서 HTML 생성 (매 요청마다 렌더링)
- 사용자에 따라 다른 데이터를 표시하는 개인화 영역에 유리
- **주의**: dynamic function이나 비캐시 API를 포함하면 해당 페이지 전체가 dynamic으로 간주됨

### 3. Streaming Rendering (App Router + Suspense 기반)
- React 18의 **Suspense + Server Components** 조합을 활용
- **서버에서 UI를 점진적으로 렌더링**하여 첫 화면 응답 속도를 향상시킴
- loading.tsx + suspense 컴포넌트 구조로 구현

기본적으로 Next.js에서는 use client를 명시하지 않으면 모든 컴포넌트는 **서버 컴포넌트**로 렌더링됩니다.

---

## 🔄 Next.js의 Server Components와 Hydration 이해하기

### ✅ Server Components란?
- React 18에서 도입된 기능으로, **컴포넌트 자체가 서버에서 렌더링**됩니다.
- Next.js의 App Router에서는 기본적으로 서버 컴포넌트가 사용됩니다.
- 기존 Pages Router에서는 페이지 단위로 SSR/SSG를 구분했지만,
    → App Router에서는 **컴포넌트 단위로 서버/클라이언트 렌더링을 제어**할 수 있습니다.
  
### ❓근데 서버에서 렌더링된 HTML이 React 컴포넌트는 아니잖아?
→ 맞습니다. 그래서 **Hydration**이라는 과정이 필요합니다.
  
### ✅ Hydration이란?
- 서버에서 렌더링된 HTML을 클라이언트에서 React가 제어 가능한 상태로 만드는 과정
- 서버는 React 컴포넌트를 **RSC Payload**로 직렬화하여 전달
- 클라이언트는 이를 **하이드레이션하여 React 앱으로 복원**

### ❗ Hydration 에러가 발생하는 경우

#### 1. 상태 불일치
- 클라이언트와 서버가 참조하는 상태가 다르면 에러 발생
- 예: localStorage, window 등 클라이언트 전용 API를 서버에서 호출할 경우

#### 2. 비동기 데이터 불일치
- 서버에서 렌더된 데이터와 클라이언트에서 fetch한 데이터가 다를 경우
- 해결: React Query의 Hydration API 등 사용하여 서버 데이터를 클라이언트에 동일하게 전달

---

## ❗하지만, 인증이 필요한 API는 SSR에서 호출 불가
- SSR은 서버에서 실행되므로 **쿠키/토큰이 없으면 401 에러 발생**
- 따라서 로그인한 유저의 데이터는 **클라이언트에서 fetch** 필요

---

## 🧩 클라이언트 fetch 시 플리커를 최소화하는 실무 방법

### 1. Skeleton/Placeholder 적극 사용

```tsx
if (isLoading) return <TaskListSkeleton />;
```
- 로딩 중에도 실제 콘텐츠와 **동일한 크기의 Skeleton 컴포넌트** 렌더링
- 공간을 미리 확보해 shift 방지

### 2. 이미지/아이콘/폰트는 크기 명시
- next/image → width / height 필수
- SVG 아이콘도 viewBox, width, height 명시
- 폰트는 font-display: swap, preload 적용

### 3. 레이아웃 구조 단순화
- min-h-screen만 최상위에서 사용
- h-screen, flex, overflow-hidden 최소화
- main 태그에 pt-20, pb-20 등 여백 명확히 지정

### 4. Suspense, dynamic import 시 fallback 필수

```tsx
<Suspense fallback={<Skeleton />}>
  <HeavyComponent />
</Suspense>
```

### 5. SSR/CSR 결과를 일치시키기
- hydration mismatch 경고가 없도록 조건부 렌더 최소화

---

## 💡 실무적으로 레이아웃 시프트 최소화 전략 정리

- ✅ **Skeleton/Placeholder**로 공간 확보 → 콘텐츠가 떠도 밀림 없음
- ✅ 이미지/폰트는 **크기 명시** → 리플로우 방지
- ✅ **레이아웃 구조 단순화** → Tailwind height/overflow 속성 최소화
- ✅ **Suspense fallback 필수**, **dynamic import 시 loading 처리**
- ✅ API는 CSR로 fetch하되, 로딩 상태 구간을 명확히 컨트롤

---

## 📱 모바일 레이아웃 실전 팁 (Footer 고정 문제)

#### 1. Footer는 항상 최상위에서 `fixed bottom-0`
- 내부 nav만 `max-w-md mx-auto`로 감싸기

#### 2. `main`에 `pt-20`, `pb-20` 추가
- Header/Footer와 콘텐츠 겹침 방지

#### 3. `h-screen`, `flex`, `overflow-hidden` 조합 제거
- 모바일 주소창/툴바 변화 대응 → `min-h-screen`만 사용

---

## ⚡ LCP, INP, CLS 측면에서의 최적화 전략

### ✅ LCP (Largest Contentful Paint)
- **원인**: API 응답 지연, 이미지/폰트 늦게 로드, skeleton 오래 노출
- **해결**:
    - 서버/DB 속도 개선 (200~500ms 내 응답)
    - 이미지 preload, 폰트 preload
    - skeleton 최소화: 데이터 오자마자 전환

### ✅ INP (Interaction to Next Paint)
- **원인**: 불필요한 렌더링, 이벤트 처리 지연
- **해결**:
    - `React.memo`, `useCallback`, `useMemo` 활용
    - DnD/Modal에서 상태 업데이트 최소화

### ✅ CLS (Cumulative Layout Shift)
- **플리커 대응책으로 개선된 지표**
- Skeleton, 고정 크기 요소, layout.tsx 구조 안정화로 0.00 유지

---

## 🔍 결론

|**항목**|**최적화 전략**|
|---|---|
|플리커/시프트|Skeleton + 고정 크기 + 단순 레이아웃|
|SSR/CSR 전환|로그인 상태 분기 처리, useQuery + fallback|
|모바일 대응|Footer fixed, padding 확보, min-h-screen 사용|
|성능 최적화|preload, memo, 코드 분리, API 속도 개선|

✅ Next.js에서의 hydration 구조를 이해하고, 레이아웃 구조와 loading 처리를 명확히 하면 실무에서도 flicker-free한 UX를 구현할 수 있습니다!