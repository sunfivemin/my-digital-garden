## 1. 서버 사이드 렌더링(SSR) vs 싱글 페이지 애플리케이션(SPA) 차이점

### 1) 서버 사이드 렌더링(Server Side Rendering) 
   - Next.js, Nuxt.js 등
   - 사용자가 페이지를 요청할 때마다 서버에서 완성된 HTML을 만들어 내려보내는 방식

→ 브라우저가 요청하면, 서버가 HTML을 “완성된 상태”로 만들어 보내준다.
JS 없이도 바로 읽히기 때문에 사용자는 바로 완성된 페이지를 볼 수 있다.
초기 로딩이 빠르고 검색엔진이 바로 인식, 인덱싱되므로 SEO에 유리하다.

### 2) 싱글 페이지 애플리케이션(Single Page Application) 
- React, Vue, Angular 등
- 페이지 전체를 새로고침하지 않고 자바스크립트로 컴포넌트만 동적으로 교체하는 방식의 웹앱

→ 최초 요청 시 “빈 HTML”에 루트 div만 있어서 자바스크립트가 실행되어야 브라우저에서 렌더링된다.
검색엔진이 내용 인식에 어려워 SEO에 불리하다.
초기 로딩은 느릴 수 있으나, 페이지 이동 시 새로고침 없이 화면 일부만 바꿔서 동적이고 빠르다.


## 2. SSR(서버 사이드 렌더링)과 SEO(검색엔진 최적화)
### ✅ SSR은 완성된 HTML을 서버에서 미리 만들어서 브라우저에 내려보내줌
사용자가 페이지에 접속하거나,
구글, 네이버 같은 검색엔진 크롤러가 내 사이트에 들어오면
SSR은 완성된 HTML을 서버에서 미리 만들어서 브라우저에 내려보내준다.

### ✅ 검색엔진은 바로 내용을 읽고 인덱싱 가능
- SPA(리액트)에서는 크롤러가 방문하면 빈 HTML(루트 div만 있음)  →  JS가 실행되어야 내용이 생성된다  →  검색엔진이 내용을 잘 못 읽는다.
- SSR(Next.js)은 최초 HTML에 데이터, 메타태그, 본문 등 모든 정보가 이미 있다.  →  크롤러가 JS 없이도 모든 컨텐츠와 구조를 읽을 수 있다.

#### 📌 SEO(검색엔진 최적화) 
= 검색엔진 + SNS + 각 플랫폼 요구사항을 한 번에 충족하는 메타태그 구성하는 것
```ts
// app/ssr/page.tsx (Next.js 13+ App Router)
export const metadata = {
	title: "SSR SEO 예제 | 내 블로그",
	openGraph: {
	description: "이 페이지는 SSR로 만들어져 검색엔진 및 SNS 공유에 최적화됩니다.",
	url: "https://your-domain.com/ssr", // 본인 실제 배포 주소로 교체!
	siteName: "내 블로그",
	images: [
		{
		url: "https://images.unsplash.com/photo-1519125323398-675f0ddb6308", 
		width: 800,
		height: 600,
		alt: "SSR SEO 예제 미리보기 썸네일"
		}
	],
	type: "website"
	},
	twitter: {}
};

export default async function SSRPage() {
	const now = new Date().toLocaleString(); // 서버에서 현재 시간 생성
	return (
		<div>
			<h1>SSR SEO 예제</h1>
			<p>이 페이지는 서버에서 미리 렌더링되어 HTML에 데이터가 들어있어요.</p>
			<p>서버 시간: <b>{now}</b></p>
		</div>
	);
}
```
- 각 페이지별 메타태그(title, description) 동적으로 생성
- 서버에서 최신 데이터로 본문과 구조까지 미리 렌더링
- 검색 결과, SNS/메신저, 모든 플랫폼에서 썸네일·타이틀·설명·키워드가 예쁘게 뜸
- 구글, 네이버, 카카오, 트위터, 페이스북, 네이버블로그, 슬랙 등 모든 채널에서 브랜딩 효과와 노출 극대화
- 리치 검색, 브랜드 인지도, 사이트 방문자 증가

### ✅ 1) 기본 SEO 메타태그
검색엔진(구글, 네이버 등)이 사이트 정보/키워드/요약을 읽는 표준
```html
<title>나의 멋진 블로그</title>
<meta name="description" content="웹개발, Next.js, 프론트엔드 경험 공유 블로그">
<meta name="keywords" content="프론트엔드, Next.js, SEO, 개발자">
```
- 검색결과에 보이는 제목/설명/키워드 최적화
- 클릭률(CTR)과 노출 순위에 영향

### ✅ 2) Open Graph (OG) 태그
페이스북, 카카오톡, 네이버블로그, 슬랙, 디스코드 등 “SNS/메신저” 미리보기 이미지·제목·설명 담당
```html
<meta property="og:type" content="website">
<meta property="og:title" content="SSR SEO 예제 | 내 블로그">
<meta property="og:description" content="SSR로 완성된 정보를 SNS에 멋지게 노출!">
<meta property="og:image" content="https://your-domain.com/og-image.jpg">
<meta property="og:url" content="https://your-domain.com/ssr">
<meta property="og:site_name" content="내 블로그">
```
- URL 공유 시 썸네일/설명/제목 자동 생성
- 브랜딩/홍보 효과, SNS에서 더 잘 보임


### ✅ 3) 구조화 데이터 (Schema.org) 
-  `JSON-LD 형식`으로 `<head>` 안에 삽입
- 눈에 확 띄는 부가정보(FAQ 등) 를 구조화 데이터로 넣으면 구글 검색 결과에서 내 사이트 아래 질문/답변 접혀서 나온다.

```js
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "Next.js에서 구조화 데이터란 무엇인가요?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "구조화 데이터는 검색엔진이 페이지 내용을 쉽게 해석할 수 있게 만드는 JSON-LD 형식의 데이터"
      }
    },
    {
      "@type": "Question",
      "name": "구조화 데이터의 효과는?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "구글에서 내 사이트가 리치 스니펫(FAQ, 별점 등)으로 강조 노출될 수 있습니다."
      }
    }
  ]
}
</script>
```
- 구글 검색에서 리치스니펫(평점, FAQ, 프로필, 썸네일 등)로 강조 노출
- 더 많은 클릭 유도, 정보 신뢰성 ↑


## 3. Next.js vs React
#### 📌  React
- 리액트같은 경우는 뷰(View)만 담당, 기본은 SPA 
-  보통 src/ 안에 컴포넌트/페이지/유틸/스타일 등 “내가 원하는 대로” 폴더를 나눈다.
- 정해진 규칙이 거의 없어서 자유도가 높다.

#### 📌  Next.js
- next.js 같은 경우는 React 기반 프레임워크, SSR/SSG/라우팅/SEO 지원, 완성도 높은 웹앱에 적합
- 페이지와 라우팅 구조가 “폴더 구조 자체”에 강하게 연결되어있다.
- SSR/SSG, API Route, 메타태그 등 프레임워크가 모든 구조를 관리한다.

### 📁 Next.js의 디렉터리 구조 예시

```
my-next-app/
├─ app/
│  ├─ page.tsx           // 루트(/) 메인 페이지
│  ├─ about/
│  │  ├─ page.tsx        // /about
│  │  └─ head.tsx        // /about의 head (메타태그 설정)
│  ├─ contact/
│  │  ├─ page.tsx        // /contact
│  │  └─ head.tsx        // /contact의 head (메타태그 설정)
│  ├─ products/
│  │  ├─ [id]/
│  │  │  └─ page.tsx     // /products/123 (동적 라우트)
│  │  └─ layout.tsx      // products 하위 공통 레이아웃
│  └─ api/
│     └─ hello/route.ts  // /api/hello (API 라우트)
├─ public/
├─ next.config.js
└─ ...
```
- /about, /contact 등 각 페이지(주소)마다 모든 메타태그를 원하는 대로 다르게 넣을 수 있음
- 검색엔진, SNS, 카카오톡 등에서 각각의 링크를 공유해도 미리보기/타이틀/설명이 “페이지별로” 정확하게 나옴


## 4. 서버 사이드 렌더링(SSR) 단점 과 싱글 페이지 애플리케이션(SPA) 장점

Next.js에서는 기본적으로 SSR로 SEO와 퍼포먼스를 챙기되, 동적 UI나 복잡한 상태 처리가 필요한 부분에만 ‘use client’를 선언해 SSR과 CSR을 효과적으로 하이브리드로 조합해 사용

초기 화면과 SEO는 SSR로 최적화, 동적인 대시보드·마이페이지·애니메이션 등은 CSR로 자유롭게 구현
→ 실무에서 가장 많이 쓰는 패턴!

#### 🟥 SSR(Next.js)의 단점 정리
- **서버 부하** : 
  모든 요청마다 서버가 실시간 렌더링 → 트래픽 많아지면 비용/부하 ↑
- **응답 속도** : 
  DB/API 느리면 페이지 로딩도 느려질 수 있음 (SSG, CDN 조합 필요)
- **인프라 복잡** : 
  정적배포(SSG): 빌드 → 파일만 올리면 끝, 서버 운영 無, 장애 걱정 적음 
  동적배포(SSR):서버 프로세스가 항상 돌아야 하고, 유지보수/보안/스케일링 등 “서버 관리”가 꼭 필요
- **페이지 전환 느림** : 
  첫 화면은 빠르지만, JS 번들 크면 전환은 SPA보다 느릴 수 있음
- **상태/비동기 처리 고민** : 
  SSR은 “서버에서 HTML을 렌더링”하기 때문에 서버가 사용자별 데이터(로그인 정보, 인증, 쿠키 등) 혹은 실시간 상태(예: 장바구니, 유저 정보, 알림 등)를 클라이언트(브라우저)와 주고받는 추가 작업 인증 등 추가 구현이 필요하다.

#### 🟢 SPA(React)가 페이지 전환/UX에 강한 이유
- **빠른 페이지 전환**
최초 한 번만 리소스 다운, 이후에는 새로고침 없이 즉시 화면 전환
서버에 다시 요청하지 않고 브라우저 내부에서만 화면 전환 → “앱처럼 빠른 느낌”, 즉시 반응
대규모 동적 UI, 대시보드, 복잡한 사용자 상호작용에 적합

- **풍부한 사용자 경험(UX)**
상태 관리, 애니메이션, 복잡한 동작을 브라우저에서 바로 처리
비동기 데이터 갱신, 무한 스크롤, 모달 등 앱스러운 인터랙션 구현에 유리
“단일 페이지 앱” 특성상 URL은 안 바뀌지만 실제 앱처럼 부드럽게 작동
