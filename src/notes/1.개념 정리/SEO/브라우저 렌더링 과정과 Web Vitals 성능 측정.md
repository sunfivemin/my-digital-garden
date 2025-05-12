 
## 브라우저는 어떻게 화면을 그릴까? 
### – 렌더링 원리와 Web Vitals로 성능 측정까지

## 1. 도입: 웹사이트가 느릴 때, 어디서 문제가 생길까?

많은 사람들이 서버 문제만 생각하지만,
실제로는 **브라우저 내부의 화면 그리기(렌더링) 과정**에서 지연이 발생하는 경우가 많다.
 이때 ‘Web Vitals’ 지표가 **사용자가 답답함을 느끼는 순간을 숫자로 보여주는 진단서** 역할을 한다.

## 2. 브라우저 렌더링 과정 요약

**HTML → DOM → CSSOM → Render Tree → Layout → Paint → Composite → 화면 출력**
- DOM: HTML 구조
- CSSOM: CSS 스타일 정보
- Render Tree: 시각적으로 보이는 요소만 추림
- Layout: 위치와 크기 계산
- Paint: 픽셀로 그림
- Composite: 여러 레이어를 합쳐서 화면에 표시

> 이 과정 속에서 **LCP, CLS, FID 문제가 발생**한다.
```bash
[HTML 파싱] → [DOM/CSSOM 생성] → [Render Tree] → [Layout] → [Paint] → [Composite]
       ↓                    ↓                     ↓           ↓
     LCP 발생 가능     CLS 발생 가능     CLS 발생 가능      FID 발생 가능 (JS Block)
```
- Web Vitals는 이 렌더링 파이프라인 속에 **사용자가 체감하는 문제를 시간순으로 수치화한 지표**
- 지표를 보는 건 **렌더링 전체 흐름에서 어디가 문제인지 빠르게 찾게 해주는 지도**


## 3. 렌더링 과정에서 자주 발생하는 성능 문제들

| **문제**           | **설명**                    | **사용자 체감  |
| ---------------- | ------------------------- | --------- |
| Reflow (레이아웃 변경) | 요소 하나 변경해도 다른 요소들 다시 계산됨  | 화면이 덜컹거림  |
| Render blocking  | JS 때문에 HTML/CSS 파싱 멈춤     | 멈춘 느낌     |
| Layout shift     | 이미지 크기나 폰트 로딩이 늦으면 화면이 밀림 | 글씨/버튼 움직임 |


## 4. Web Vitals 3대 지표
📊 Web Vitals = 사용자가 느끼는 불편을 ‘숫자’로 보여주는 브라우저 성능 진단서

| **지표**  | **브라우저 내부 단계**      | **사용자가 느끼는 불편**   |
| ------- | ------------------- | ----------------- |
| **LCP** | DOM + CSSOM + Paint | 중요한 콘텐츠가 늦게 떠서 답답 |
| **CLS** | Layout → Paint      | 페이지가 덜컹거려서 실수 유발  |
| **FID** | JS 실행 (Main Thread) | 클릭했는데 멈춤          |


### **LCP (Largest Contentful Paint)**
- **사용자 입장**: 제일 중요한 이미지나 타이틀이 언제 보이는지. (느리면 답답)
- **브라우저 내부**: DOM 생성 → CSSOM → Render Tree → Layout → Paint
- **최적화**: 핵심 이미지는 **크기 명시 + lazy loading**, CSS는 **최적화해서 빠르게 로드**

#### LCP 개선 예시
```html
<!-- BEFORE -->
<img src="/hero.png" />

<!-- AFTER -->
<img src="/hero.png" loading="lazy" width="600" height="400" />
```
- 핵심 콘텐츠는 **미리 크기 명시**, **필요하면 lazy**

### **CLS (Cumulative Layout Shift)**
- **사용자 입장**: 페이지가 덜컹거리면 사용자가 불편하고 실수할 위험
- **브라우저 내부**: Layout → Paint 과정에서 **이미지 크기 미지정**, **폰트 늦게 로딩** 시 발생
- **최적화**: **고정 크기 명시 + Skeleton UI + 폰트 swap**

### **CLS 개선 예시**
```html
<!-- BEFORE -->
<img src="/product.jpg" />

<!-- AFTER -->
<img src="/product.jpg" style="width:300px; height:300px;" />
```
- 크기 지정으로 화면 밀림 방지

### **FID (First Input Delay)**
- **사용자 입장**: 클릭했는데 멈추면 열받음
- **브라우저 내부**: JS가 메인 쓰레드를 독점할 때 발생 (Main Thread Block)
- **최적화**: **defer, async**, React **Virtual DOM 사용해서 DOM 변경 효율적으로**

### **FID 개선 예시**
```html
<!-- BEFORE -->
<script src="/heavy-lib.js"></script>

<!-- AFTER -->
<script src="/heavy-lib.js" defer></script>
```
- **defer**로 JS는 나중에 실행 → UI 먼저 반응

#### **💡 Virtual DOM 역할 (FID 지표와 연결)
Virtual DOM은 실제 DOM을 직접 건드리지 않고, ‘가상으로 바뀐 부분만 계산’해서 빠르게 반영한다.
그래서 JS의 무거운 DOM 업데이트를 줄여 FID 지표를 개선하는 데 효과적이다.
특히 SPA(싱글 페이지 앱) 구조에서 React의 Virtual DOM이 없으면 클릭 시 UI가 멈추는 일이 더 자주 발생한다.
> Virtual DOM은 변경된 부분만 계산 → DOM 조작 최적화 → FID 개선 효과


## 6. 측정 도구 소개

|**도구**|**기능**|**특징**|
|---|---|---|
|Lighthouse|자동 점수 + 문제 분석|Chrome DevTools 내장|
|WebPageTest|실제 환경 테스트|핸드폰, 느린 네트워크 등 조건 선택 가능|
|Performance 탭|프레임 단위 분석|개발자용 고급 도구|

## 7. 최적화 방법 예시

|**문제**|**해결책**|
|---|---|
|이미지 늦게 뜸|width/height 지정 + lazy|
|글씨 로딩 느림|font-display: swap|
|JS 늦게 로딩|defer 또는 async|
|화면이 밀림|Skeleton UI + 고정 크기 지정|
|느린 렌더링|React + Virtual DOM 구조 활용|


## 8.Before / After 성능 비교 (실험 결과 예시)

| **구분**  | **Before (느림/불안정)**  | **After (최적화/빠름)** |
| ------- | -------------------- | ------------------ |
| LCP     | 이미지 늦게 Paint (3.18s) | 빠른 Paint (0.52s)   |
| CLS     | 화면 덜컹 (0.20)         | 안정 (0)             |
| FID/INP | JS block으로 클릭 응답 불량  | 즉시 반응 (8ms)        |

## **✅ LCP 차이 (3.18s ➡ 0.52s)**

**Before**
- 이미지 크기 미지정 + 즉시 로딩
- 브라우저가 레이아웃, Paint 늦게 시작 → 메인 콘텐츠 늦게 뜸

**After**
- 크기 명시 + lazy loading 적용
- 브라우저가 Paint 빠르게 진행 → LCP 대폭 개선

---

## **✅ CLS 차이 (0.20 ➡ 0)**

**Before**
- 이미지 크기 미지정 → 로딩 중 레이아웃 확정 불가능
- 이미지 로딩 후 갑자기 레이아웃 밀림 → CLS 발생

**After**
- 크기 명시 → 처음부터 자리 확보
- 화면 밀림 없이 안정적 렌더링 → CLS 0

---

## **✅ INP 차이 (32ms ➡ 64ms)**

**Before**
- 메인 스레드 block(3초)이었지만, Lighthouse에서 INP는 32ms로 나옴
- 실제로는 JS block 영향 때문에 사용자 경험은 훨씬 느렸을 가능성 높음
- INP 수치는 낮지만 신뢰성 낮음 (측정 조건상 block 발생 구간 무시될 수 있음)

**After**
- JS defer 적용 → DOM, Paint 먼저 완료 후 JS 실행
- 클릭 시 안정적이고 부드럽게 반응 (64ms → 여전히 사용자 관점 Good 등급)

---

## 9. 개선 팁 3가지

1. 이미지엔 **크기 명시 + lazy**
2. JS는 **defer로 렌더링 차단 최소화**
3. **Skeleton UI**로 로딩 시 안정화


## **10. 결론**
> Web Vitals는 **사용자 짜증을 수치로 보여주는 지표**다.
> 브라우저 렌더링 흐름을 이해하면,
> 속도가 아니라 **사용자 경험 중심 성능 개선이 가능하다.**


