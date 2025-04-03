---
tags:
  - TIL
  - HTML
  - CSS
  - Javascript
  - 웹
created: 2025-04-03
---

# 📘 2025-04-03 TIL

## 📌 오늘 배운 핵심 요약
- 

## 🧠 상세 학습 내용

### 📍 주제 1: 웹이란?

 ✅  웹의 이해

인터넷 : international Network의 약자로 전 세계의 모든 컴퓨터를 하나의 통신망 안에서 연결한다는 의미이다.
웹: www(world wide web), 인터넷에 연결된 컴퓨터를 통해 사람들이 정보를 공유할 수 있는 공간

웹 페이지 링크를 타고 다른 웹 페이지로 이동하는 것을 '웹 서핑을 한다', '웹 브라우징을 한다'라고 한다.

웹브라우저: 웹 페이지 혹은 웹 상의 데이터를 찾거나 읽을때 사용하는 것(사파리, 크롬 등)

 ✅  웹의 구조
클라이언트와 서버
Client : 서비스를 요청하는 컴퓨터
Server: 서비스를 제공하는 컴퓨터

웹 프로토콜인 HTTP(HyperText Transfer Protocol)를 사용하여 데이터를 주고 받는다.
* 프로토콜: 클라이언트와 서버 간에 정보를 주고 받을때 약속을 지켜서 통신하는 것

 ✅  웹 개발 직무
 프론트엔드 : 화면 구현
 백엔드 : 로직 구현
 풀스택 : 프론트엔드 + 백엔드 


---

### 📍 주제 2:  HTML, CSS, Javascript

####  ✅ 1. 프론트엔드 3대장
HTML : 
- 하이퍼텍스트 즉, 웹 페이지를 연결하는 기능을 가진 텍스트이자 웹 페이지의 구조를 명시하는 언어
- HTML은 <> 괄호를 사용한다. 이를 태그라고 부른다. 
- 태그는 웹페이지의 구성 요소 하나하나로 역할을 하게 된다.
CSS :

Javascript : 




```bash
git switch -c feature/login  # 기능 개발을 위한 브랜치 생성

# 파일 수정 후 커밋
git add .
git commit -m "login branch commit"
```

➡ 이 시점까지는 **로컬에서만 존재하는 브랜치**다.
GitHub에는 아직 올라가지 않았기 때문에 다른 사람은 이 브랜치를 볼 수 없다.


####  ✅ 2. 

---

#### 💡 fetch vs pull

| 구분    | 설명                                | **동작 범위** |
| ----- | --------------------------------- | --------- |
| fetch | 원격 브랜치의 최신 상태를 가져오기만 함 (병합 X)     | 안전하게 확인용  |
| pull  | fetch + merge 자동 수행 (가져오고 적용까지 함) | 즉시 코드에 영향 |

#### **🧭  git branch -r

- -r 옵션은 **remote(원격) 브랜치 목록만 출력**
- 원격에 브랜치가 잘 반영됐는지 바로 확인할 수 있어 유용하다.


![브랜치병렬개발](https://seonohblog.netlify.app/assets/브랜치병렬개발.png)

#### 📎 실습 결과

- 👉 로컬에서 브랜치를 생성하고 커밋한 뒤, 다른 브랜치로 전환하고 원격 저장소에 push하는 전체 흐름을 실습했다.
- 👉 기능 단위로 브랜치를 나눈 후, Git 히스토리를 통해 **커밋이 포함된 브랜치만 실제로 분기된 상태**라는 것을 시각적으로 확인할 수 있다.

---

## 📍 주제 3: 브랜치 전략 (Git Flow) / 브랜치 병합 방법 ( Merge 방식 - fast-forward, 3-way merge)

#### ✅ 브랜치 전략 (Git Flow)

Git Flow는 기능, 배포, 수정 등 작업의 목적에 따라 **브랜치를 체계적으로 나누는 전략**이다.
협업이 많거나 버전 관리를 체계적으로 하고 싶을 때 특히 유용하다.

| **브랜치**   | 설명                            |
| --------- | ----------------------------- |
| main      | 실제 운영(배포)용 브랜치                |
| develop   | 기능 통합 및 테스트용 브랜치              |
| feature/* | 배포 전 QA용 브랜치 (release/1.2.0)  |
| hotfix/*  | 운영 중 긴급 수정 브랜치 (hotfix/1.2.1) |
|           |                               |

```bash
# 기능 개발용 브랜치 생성
git switch -c feature/login

# 작업 후 커밋
git add .
git commit -m "feat: 로그인 기능 개발"

# 원격 저장소에 push
git push origin feature/login
```

---

### 🔧 Git 브랜치 명령어 요약
```bash
# 원격 저장소의 최신 상태 동기화 
git fetch -p

# 로컬 브랜치 feature/1 생성  
# 원격 브랜치 origin/feature/1을 **추적(tracking)** 으로 설정
# 그 브랜치로 이동
git checkout -t origin/feature/1
==
# 원격 브랜치 추적하며 전환 (최신스타일) 
git switch --track -c feature/1 origin/feature/1 

# 병합
git merge feature/login

# GitHub PR 병합 후 동기화
git pull origin main

# 병합 후 브랜치 삭제
git branch -d feature/login
```


---

#### 💬 실습하면서 느낀 점

- 

---

## 💭 회고

• **새롭게 알게 된 점**
 → 

• **어렵게 느껴졌던 부분**
→ 

• **다음에 학습할 주제**
→ 

## 🔗 참고 자료
