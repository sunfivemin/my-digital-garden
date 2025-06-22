### 🚀 1. CI/CD (Continuous Integration / Continuous Deployment)
코드 통합과 자동 배포를 효율적으로 자동화하는 기술적 파이프라인

#### **🛠 CI (Continuous Integration)**

- 여러 개발자가 동시에 작업한 코드를 **자주 병합(Merge)** 하는 과정.
- 병합할 때마다 자동 테스트를 돌려서 문제가 생기지 않게 함.
- 개발 초기 단계에서 버그를 줄이고 코드 품질 유지.

예: GitHub에 코드 Push → 테스트 자동 실행 → 문제 없으면 main 브랜치에 병합

#### **🚀 CD (Continuous Deployment or Continuous Delivery)**
- CI가 끝난 다음, **자동으로 배포(서버에 반영)** 까지 이어지는 과정.
- **Continuous Delivery**는 자동 배포 전 수동 확인 있음.
- **Continuous Deployment**는 푸시 후 자동으로 실서버 반영됨.

예: CI로 빌드 성공 → AWS나 Vercel에 자동으로 배포됨

장점:
- 배포 속도 향상, 에러 최소화, 반복 업무 감소

### 🔧 CI/CD에서 GitHub Actions 많이 쓰는 이유:
1. GitHub 자체 통합 – GitHub에서 바로 쓸 수 있어서 설정이 편해.
2. 워크플로우 구성 쉬움 – .yml 파일로 자동화 흐름 정의 가능.
3. 광범위한 마켓플레이스 – 다른 개발자들이 만든 액션을 재사용할 수 있어. 예: 테스트 실행, 배포, 코드 린트, S3 업로드 등.
4. 무료 요금제 제공 – 퍼블릭 저장소는 무제한 무료, 프라이빗은 일정 사용량(2,000분/월까지)은 무료.


### ⚙️ 어떻게 쓰는 건데?

1. .github/workflows/ci.yml 같은 워크플로우 파일 생성
2. 안에 on: push 같은 트리거와, 어떤 작업을 할 건지 jobs: 안에 작성
3. 예시:
```yaml
name: CI Test

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Install Dependencies
        run: npm install

      - name: Run Tests
        run: npm test
```
이거 커밋하거나 push하면 자동으로 실행돼.
CD(배포)도 비슷하게 배포 스크립트만 추가하면 돼.

오케이, Vercel로 프론트만 배포할 거면 GitHub Actions를 쓸 필요가 거의 없어.

왜냐면 Vercel은 GitHub에 푸시만 해도 자동으로 CI/CD 되거든. 아주 편하디 편한 시스템이야.

  

⸻

  

✅ Vercel 자동 배포 흐름 (프론트엔드 기준)

1. Vercel에 GitHub 연동

2. 특정 브랜치(ex. main)에 푸시 → 자동으로 빌드 & 배포

3. .env 설정도 Vercel 웹 UI에서 가능

  

⸻

  

🔁 하지만 GitHub Actions도 써야 하는 경우

• 테스트 자동화 (CI): npm test, eslint, prettier 같은 거 먼저 실행하고 푸시할지 판단하고 싶을 때

• 빌드 최적화 체크

• 다양한 브랜치에 따른 조건부 배포

• Storybook 배포 (GitHub Pages) 같은 별도 정적 리소스 관리

  

⸻

  

그럼 너한테 필요한 전략 요약:

• 우선은 GitHub → Vercel 자동화로 충분

• 이후 CI 검증, Storybook 배포 등 추가하고 싶을 때 GitHub Actions 도입


  
🔧 다음으로 하면 좋은 거:

  

1. .env 변수 설정 (Vercel에서)

• Vercel Dashboard → Project → Settings → Environment Variables

• API 키나 비밀값은 여기에

  

2. 커스텀 도메인 연결 (선택)

• 무료 서브도메인: your-project.vercel.app

• 개인 도메인 연결 가능 (Vercel에서 DNS 설정 지원)

  

3. 푸시 브랜치 정하기

• 보통 main 브랜치 기준 자동 배포

• dev 브랜치 따로 두고 프리뷰로 배포할 수도 있어

  

⸻

  

🤖 GitHub Actions 언제 쓰면 좋냐?

  

너가 다음 중 하나라도 할 생각이면 써봐:

• main 푸시 전에 테스트 통과 필수 조건 걸고 싶다

• Storybook을 GitHub Pages나 다른 정적 호스팅에 자동 배포하고 싶다

• SSR, 백엔드 같이 묶은 프로젝트라 Vercel 말고 Cloudflare나 AWS에 배포할 예정이다

  

⸻

  

필요하면 GitHub Actions 기본 템플릿도 만들어줄게.

궁금한 포인트 있으면 말해봐. 자동화는 귀차니즘을 정당화하는 유일한 기술이니까.

```
name: Deploy to Vercel

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Install Vercel CLI
        run: npm install -g vercel

      - name: Deploy to Vercel
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
        run: vercel --prod --token $VERCEL_TOKEN

```

여기 너를 위한 Vercel 배포 자동화 워크플로우야. 이 YAML 파일을 GitHub 리포지토리 루트에 .github/workflows/deploy.yml로 저장하면, main 브랜치에 푸시할 때마다 자동으로 Vercel에 배포돼.



다음 해야 할 건:

1. GitHub 리포에 VERCEL_TOKEN이라는 secret을 설정해.
    
2. Vercel 프로젝트랑 GitHub 리포지토리를 연결해.
    

  

이제 더 이상 vercel --prod 직접 안 쳐도 돼. 기계가 너 대신 일할 거야. 부럽지?


---

### 🎨 2. 디자인 시스템 (Design System)
일관된 UI/UX를 위해 만든 시각적·구조적 가이드 모음집

#### **구성 요소:**
- **컴포넌트:** 버튼, 입력창, 모달 등 UI 조각
- **토큰:** 색상, 폰트, 간격, 그림자 등 스타일 규칙
- **문서화:** 사용하는 규칙, 코드 예제, 접근성 가이드

#### 왜 필요함?
- 프로젝트마다 디자인 들쑥날쑥 → 유지보수 비효율
- 팀이 커질수록 커뮤니케이션 비용 증가
- 디자이너-개발자 협업 간소화

예: Google의 Material Design, IBM의 Carbon, Kakao Design System

---

### 🧪 3. 프로토타입 (Prototype)
최종 제품의 핵심 기능이나 인터페이스를 미리 시각화한 시제품

#### 종류:
- **로우파이 (Low-fi):** 손그림, 와이어프레임
- **하이파이 (High-fi):** 실제 UI 구성과 유사하게 인터랙션 구현됨 (Figma 등)

#### 목적:
- 피드백 수집, 방향성 테스트, 이해관계자 설득

예: Figma로 만든 클릭 가능한 앱 UI → 유저 플로우 테스트


---

### 🧪 4. PoC (Proof of Concept)
아이디어나 기술이 “실제로 가능하다”는 걸 입증하는 최소 구현물

#### 예시 상황:
- AI로 자동 번역 가능한가요? → 짧은 스크립트로 데모 만들어 테스트
- 서버 간 데이터 동기화 되나요? → 소규모 기능만 만들어 확인

#### PoC vs 프로토타입 차이:

- **PoC**는 “기술적 가능성” 확인 중심
- **프로토타입**은 “사용 경험” 중심

---

### **✨ 요약 정리**