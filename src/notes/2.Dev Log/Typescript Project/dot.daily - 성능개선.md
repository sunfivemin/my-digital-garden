# 🗓 dot.daily — 성능개선

## 성능 병목과 Next.js + React 최적화 전략

**dot.daily** 프로젝트에서 직접 개선하며 얻은 인사이트 기반

---

## 🔍 1. 왜 성능 개선이 중요한가?

- 빠른 UI 반응은 곧 **사용자 만족도**로 연결됩니다.
- 느린 UI → 이탈률 상승, 전환율 저하
- 크롬 개발자 도구의 **Web Vitals 지표**는 이런 문제를 수치로 알려줍니다.

---

### 🚫 문제 1: 인위적인 렌더 지연 (mock 딜레이)

### ❗ 현상
- setTimeout으로 의도적으로 1초 지연
- 실제 배포에선 UX 저해 요소

mock API나 테스트 코드에 남아있는 setTimeout, setInterval 등으로 인해 불필요하게 렌더링이 지연되는 경우가 많습니다.
이런 인위적 딜레이는 실제 서비스에서는 즉각적인 반응을 방해하므로 반드시 제거해야 합니다.

🚫 개선 전
```ts
// mock API에서 인위적 딜레이
export const getTasksByDate = async (date: Date): Promise<Task[]> => {
  await new Promise(resolve => setTimeout(resolve, 1000)); // 실제 서비스에는 불필요
  return tasksData[dateString] || [];
}
```

🛠 개선 후
```ts
export const getTasksByDate = async (date: Date): Promise<Task[]> => {
  // setTimeout 제거 → 즉각적인 데이터 반환
  return tasksData[dateString] || [];
}
```

> 개발 중에는 네트워크 환경을 시뮬레이션할 수 있지만, 실제 배포 전에는 반드시 제거해야 합니다.

### ✅ 효과
- 실시간 반응으로 사용자 만족도 증가
- Lighthouse 지표 중 **INP** 개선

---

## 🚫 문제 2: Next.js `<Image />` 경고 & UI 깨짐

### ❗ 현상
- Next.js  컴포넌트에 width/height 누락 → 레이아웃 시프트
- 모바일에서 비율 깨짐

Next.js의 <Image /> 컴포넌트는 width/height를 명확히 지정하지 않으면 경고가 발생합니다.
SVG 등 벡터 이미지의 크기가 일관되지 않으면 UI가 깨지고, CSS로만 크기를 조정하면 레이아웃 시프트(LCP, CLS) 문제가 생길 수 있습니다.

🚫 개선 전
```tsx
<Image src="/dropdown.svg" alt="달력 선택" width={24} />
```
<Image src="/dropdown.svg" alt="달력 선택" width={24} />

🛠 개선 후
```tsx
<Image
  src="/dropdown.svg"
  alt="달력 선택"
  width={20}
  height={20}
  style={{ width: 20, height: 20 }}
/>
```

> SVG 내부에도 width/height 속성이 있으면 더욱 좋고, style로 명확히 크기를 지정하면 모바일/웹에서 일관된 UI를 보장할 수 있습니다.

### ✅ 효과

- CLS (Cumulative Layout Shift) 개선
- 다양한 기기에서도 **일관된 UI 유지**

---

## 🚫 문제 3: React Query 사용 미흡 (새로고침 필요)

### ❗ 현상
- 데이터 이동 후 새로고침해야 반영됨
- 사용자는 반응 없는 UI에 답답함

CRUD 후 새로고침해야만 UI가 갱신되는 경우가 많습니다.
서버 round-trip이 많으면 UX가 느려지고, 캐시와 실제 데이터가 불일치하면 데이터 일관성 문제가 발생합니다.

🚫 개선 전
```ts
const handleMoveToToday = (id: string) => {
  // TODO: 오늘 할 일로 이동
}
```

🛠 개선 후
```ts
const handleMoveToToday = async (id: string) => {
  try {
    // 1. 실제 데이터 이동 (archive → 오늘 할 일)
    const movedTask = await moveToTodayFromArchive(Number(id));
    // 2. 오늘 날짜 key 생성
    const todayKey = format(selectedDate, 'yyyy-MM-dd');
    // 3. React Query 캐시 즉시 업데이트 (UI 즉시 반영)
    queryClient.setQueryData(['tasks', todayKey], (old: any[] = []) => {
      return [...old, movedTask];
    });
  } catch (error) {
    alert('오늘 할 일로 이동에 실패했습니다.');
  }
};
```

> React Query의 setQueryData를 활용하면 서버에 재요청하지 않고도 프론트엔드 캐시(메모리)에서 바로 UI를 갱신할 수 있습니다.

### ✅ 효과
• 즉각적인 화면 반영 (캐시 갱신)
• 서버 요청 줄고, 사용자 경험 대폭 향상

---

## 🚫 문제 4: 미사용 코드/변수 방치

### ❗ 현상
• 사용되지 않는 변수, 함수가 코드 곳곳에
• 유지보수 난이도 상승, 빌드 크기 증가

미사용 변수, 함수, import가 많으면 코드 가독성이 떨어지고, 중복 로직이 많으면 유지보수가 어려워집니다.

🚫 개선 전
```ts
const priorities = [ ... ]; // 사용하지 않음
const handlePostpone = (task: Task) => { /* ... */ }; // 미사용
```

🛠 개선 후
```ts
// 미사용 변수/함수 완전 제거 → 코드 가독성 및 번들 크기 감소
```

> 코드 리뷰 시 미사용 코드, 중복 로직, 불필요한 상태 변경을 적극적으로 제거하세요.

---

## 🚫 문제 5: 빌드/런타임 경고 무시

### ❗ 현상

- “Warning은 괜찮겠지…” → 배포 후 장애 유발

빌드 경고/에러가 남아있으면 배포 시 장애가 발생할 수 있습니다.
타입 에러, lint 에러가 누적되면 코드 품질이 저하됩니다.
작은 경고 하나도 무시하지 마세요. 프로덕션에서는 큰 문제가 됩니다.

🛠 개선 후
- 모든 lint/type 에러, 빌드 경고 제거

- 예시:
```
  ✓ Compiled successfully
  ✓ Linting and checking validity of types
```

![BuildSuccess.png](https://seonohblog.netlify.app/assets/BuildSuccess.png)


---

## 🚫 문제 6: SVG/이미지 불일치

### ❗ 현상
- SVG에 width/height 누락 → 크기/비율 깨짐
- CSS만으로 크기 조정 시 기기별 다름

SVG에 viewBox만 있고 width/height가 없으면 크기/비율이 깨지고,
여러 곳에서 아이콘 크기가 다르면 UI 일관성이 떨어집니다.

### ✅ 개선
- 모든 SVG 파일에 width, height, viewBox 명시
- Next.js <Image />에서도 사이즈 고정

---

## Web Vitals 3대 지표와 실무 성능 개선 사례

### 🌐 Web Vitals란?

Web Vitals는 사용자가 실제로 느끼는 웹사이트의 속도와 안정성을 수치로 보여주는 브라우저 성능 진단서입니다.
LCP, CLS, FID(INP) 세 가지가 대표적입니다.

| **지표**   | 사용자가 느끼는 불편       | **실무 문제**        | **개선**                              | 브라우저 내부 단계          |
| -------- | ----------------- | ---------------- | ----------------------------------- | ------------------- |
| LCP      | 중요한 콘텐츠가 늦게 떠서 답답 | 이미지 크기 없음, 폰트 지연 | <Image width height loading="lazy"> | DOM + CSSOM + Paint |
| CLS      | 페이지가 덜컹거려서 실수 유발  | 이미지/SVG 크기 없음    | 스타일 고정                              | Layout → Paint      |
| FID(INP) | 클릭했는데 멈춤          | 무거운 JS, 이벤트 지연   | Virtual DOM + defer                 | JS 실행(Main Thread)  |

### LCP (Largest Contentful Paint)
- 사용자 입장: 제일 중요한 이미지나 타이틀이 언제 보이는지. (느리면 답답)
- 실무 문제: 이미지 크기 미지정, CSS/JS 번들 지연, 폰트/스타일 늦게 로딩
- 최적화: 핵심 이미지는 width/height 명시 + lazy loading, CSS/폰트는 최적화해서 빠르게 로드

🌐 LCP 개선 예시
```tsx
<!-- BEFORE -->
<img src="/hero.png" />

<!-- AFTER -->
<img src="/hero.png" loading="lazy" width="600" height="400" />
```

> 크기 지정으로 화면 밀림 방지

---

### FID (First Input Delay) / INP (Interaction to Next Paint)

- 사용자 입장: 클릭했는데 멈추면 열받음
- 실무 문제: 무거운 JS 번들, 대용량 데이터 파싱, 이벤트 핸들러에서 비동기 처리 누락
- 최적화: defer, async로 JS는 렌더링 후 실행, React Virtual DOM으로 DOM 변경 최소화

🌐 FID/INP 개선 예시
```ts
<!-- BEFORE -->
<script src="/heavy-lib.js"></script>

<!-- AFTER -->
<script src="/heavy-lib.js" defer></script>
```

 defer로 JS는 나중에 실행 → UI 먼저 반응

### 💡 Virtual DOM 역할 (FID/INP 지표와 연결)
- Virtual DOM은 실제 DOM을 직접 건드리지 않고, ‘가상으로 바뀐 부분만 계산’해서 빠르게 반영한다.
- JS의 무거운 DOM 업데이트를 줄여 FID/INP 지표를 개선하는 데 효과적이다.
- SPA(싱글 페이지 앱) 구조에서 React의 Virtual DOM이 없으면 클릭 시 UI가 멈추는 일이 더 자주 발생한다.

---

### 실무에서 자주 겪는 Web Vitals 문제와 개선 팁

#### 1. 이미지엔 크기 명시 + lazy
- `<img src="..." width="..." height="..." loading="lazy" />`
- Next.js <Image />는 width/height/style 모두 명시

#### 2. JS는 defer로 렌더링 차단 최소화
- `<script src="..." defer></script>`
- 번들 분할, 코드 스플리팅 적극 활용

#### 3. Skeleton UI로 로딩 시 안정화
- 데이터 로딩 중에는 Skeleton 컴포넌트로 자리 확보
- 레이아웃 시프트 방지

#### 4. 폰트는 font-display: swap
- 폰트가 늦게 로딩되어도 기본 폰트로 먼저 보여주고, 나중에 교체

#### 5. React/SPA에서는 불필요한 re-render, useEffect 남발 주의
- React.memo, useMemo, useCallback 적극 활용
- 상태/props 최소화, context 남용 주의

---

### Before / After 성능 비교 (실험 결과 예시)

### ✅ 결론
Web Vitals는 단순히 점수만 올리는 것이 아니라
실제 사용자가 느끼는 “빠르고 안정적인 경험”을 만드는 데 핵심입니다.
이미지, 폰트, JS, 레이아웃, 이벤트 처리 등 모든 레이어에서 최적화가 필요합니다. 

DOT.DAILY 프로젝트에서도
- ✅ 불필요 렌더/딜레이 제거
- ✅ 이미지 최적화 및 레이아웃 안정화
- ✅ React Query 캐시로 즉각 반영
- ✅ 코드 정리 및 타입 안정성 확보
- ✅ Web Vitals 지표로 성능 수치화 및 개선

등으로 Web Vitals 지표가 크게 개선되었습니다.

---

- Lighthouse, WebPageTest, Performance 탭을 적극 활용해 병목 구간을 찾아내세요.
- “내가 직접 써보면서 답답한 부분”이 있다면, Web Vitals 지표로 수치화해서 개선하세요.
- 경고/에러는 무시하지 말고, 근본적으로 해결하는 습관을 들이세요.
