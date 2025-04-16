---
title: Node.js 기본 생태계
tags:
  - frontend
  - html
---

# 📘 Node.js 기본 생태계

Node.js는 백엔드 개발을 자바스크립트로 할 수 있게 해주는 런타임이며, 실제 개발에서는 **패키지 매니저(NPM)**, **의존성 관리**, **스크립트 실행** 등 다양한 생태계 도구와 함께 사용된다.

## 🧩 Node.js란?

- Node.js는 브라우저가 아닌 환경에서 **자바스크립트를 실행할 수 있게 해주는 런타임**
- V8 JavaScript 엔진을 기반으로 하고 있으며, **비동기 I/O 처리**에 강함
- 주로 **웹 서버, API 서버, CLI 도구** 등으로 많이 사용됨

## 📦 Node.js 생태계 구성 요소

| 구성 요소        | 설명                        |
| ------------ | ------------------------- |
| Node.js      | 런타임, 자바스크립트를 실행해주는 환경     |
| npm          | 패키지 매니저, 외부 모듈 설치 및 관리 도구 |
| npx          | 설치 없이 즉시 실행 가능한 실행기       |
| package.json | 프로젝트 설정과 의존성 정보를 담은 핵심 파일 |
| node_modules | 설치된 패키지들이 실제로 저장되는 폴더     |

---

## 📍 npm (Node Package Manager)

### ✅ npm이란?
- Node.js 설치 시 함께 설치되는 **기본 패키지 매니저**
- 외부 오픈소스 패키지를 설치/삭제/업데이트 가능

### ✅ 주요 명령어

```bash
npm init          # package.json 생성
npm install       # 의존성 설치
npm install <pkg> # 특정 패키지 설치
npm uninstall <pkg> # 패키지 제거
```

### ✅ 전역 설치 vs 로컬 설치

| 구분        | 명령어 예시                   | 설치 위치                   | 사용 용도                          |
| --------- | ------------------------ | ----------------------- | ------------------------------ |
| **로컬 설치** | `npm install express`    | 현재 프로젝트의 `node_modules` | 해당 프로젝트에서만 사용하는 라이브러리          |
| **전역 설치** | `npm install -g nodemon` | 시스템 전체 (글로벌)            | CLI 툴이나 여러 프로젝트에서 공통으로 사용하는 도구 |

---

## 📍 npx란?

`npx`는 Node.js 5.2 이상 버전부터 함께 제공되는 CLI 도구로,  
**패키지를 설치하지 않고도 한 번만 실행**할 수 있도록 도와준다.

### ✅ 특징 요약

| 항목           | 설명 |
|----------------|------|
| 목적           | 패키지를 설치하지 않고도 실행 가능하게 함 |
| 실행 방식      | 로컬/전역 설치 여부와 상관없이 자동 처리 |
| 사용 예시      | CRA(create-react-app), eslint, cowsay 등 CLI 툴 일회성 실행에 적합 |
| 설치 필요 여부 | 설치 없이 사용 가능 (내부적으로 자동 처리 후 캐시됨) |

### ✅ 사용 예시

```bash
npx create-react-app my-app      # CRA 설치 없이 프로젝트 생성
npx eslint .                     # 로컬 ESLint 설치 없이 코드 검사
npx cowsay "Hello Node.js!"      # CLI 유틸 실행
```

---
## 📍 package.json

package.json은 Node.js 프로젝트의 **메타 정보와 의존성, 실행 스크립트 등을 정의하는 설정 파일**이다.
프로젝트를 클론하거나 배포받았을 때, 이 파일만 있으면 npm install을 통해 동일한 환경 구축이 가능하다.

### ✅ 주요 필드 설명

| 항목              | 설명 |
|-------------------|------|
| `name`            | 프로젝트 이름. npm에 배포 시 고유 식별자로 사용됨 |
| `version`         | 프로젝트 버전 정보 |
| `scripts`         | `npm run` 명령어로 실행할 수 있는 사용자 정의 명령어 목록 |
| `dependencies`    | 실행 시 필요한 외부 라이브러리 목록 |
| `devDependencies` | 개발 환경에서만 필요한 도구 목록 (예: 테스트, 빌드 툴 등) |
| `main`            | 애플리케이션의 진입 파일. CommonJS 방식에서 사용됨 |
| `license`         | 오픈소스 라이선스 정보 (예: MIT, ISC 등) |

### ✅ 예시

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  },
  "main": "index.js",
  "license": "MIT"
}
```


### **💡 사용 팁**

- npm run dev처럼 scripts에 등록한 명령어를 단축 실행 가능
- dependencies는 운영 시 사용, devDependencies는 개발 환경에서만 사용
- 실제 배포 시엔 --production 옵션으로 devDependencies 제외 가능