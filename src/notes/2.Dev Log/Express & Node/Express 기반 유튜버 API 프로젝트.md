
# 📘 Node.js + Express + React + Storybook: 유튜버 관리 프로젝트

유튜버 정보를 등록하고, 수정하며, 삭제할 수 있는 **풀스택 CRUD 프로젝트**입니다.
**Express.js + React + Tailwind CSS + Storybook** 기반으로 직접 API 설계부터 UI 구현과 컴포넌트 문서화하였습니다.

## **📌 목차**

1. 프로젝트 소개 및 목적
2. 기술 스택 소개
3. 백엔드: Node.js + Express
4. 프론트엔드: React + Tailwind CSS
5. 공통 컴포넌트 & 디자인 시스템
6. Storybook으로 UI 문서화
7. 최종 회고

---
## **1️⃣ 프로젝트 목적**

- CRUD 구조 이해 및 직접 구현
- RESTful API 설계 경험
- 프론트/백 협업 구조 익히기
- 디자인 시스템과 Storybook 적용

---

## **2️⃣ 기술 스택**

| 영역                                      | 기술                                                                                                                                           |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Backend (Node.js + Express)             | - 유튜버 등록, 조회, 수정, 삭제 기능을 포함한 **RESTful API 구현**<br>- **개별 유튜버 관리용 API**를 직접 설계 및 라우팅 처리<br>- 데이터는 Map() 객체를 사용하여 **간단한 인메모리 DB 구조**로 관리      |
| Database (db.json)                      | - JSON 파일 기반 가상 DB                                                                                                                           |
| Frontend (React + Tailwind CSS + Axios) | - React와 Tailwind를 활용해 **깔끔하고 모던한 UI 구현**<br>- API 호출은 Axios로 관리하며 **GET / POST / PUT / DELETE** 요청 수행<br>- 전체 코드는 **간결한 구조**로 유지하면서도 확장성 고려 |
| UI 문서화 (Storybook)                      | Button, Header, Input, YoutuberCard 등 공통 UI 컴포넌트 분리                                                                                          |

---

## **3️⃣ 백엔드: Node.js + Express**

- /youtubers 경로로 RESTful API 제공
- db.json 파일 기반으로 CRUD 작동
- readData, writeData, getNextId 유틸 함수로 깔끔하게 관리

- **GET** /youtubers → 유튜버 전체 목록 조회
- **GET** /youtubers/:id → 유튜버 조회 개별 조회
- **POST** /youtubers → 유튜버 등록
- **PUT** /youtubers/:id →  유튜버 수정
- **DELETE** /youtubers/:id → 유튜버 삭제

### ✅ Express로 API 만들기

#### 📁 디렉토리 구조 예시

```
backend/
├── app.js
├── routes/
│   └── youtubers.js
├── data/
│   └── db.js
└── package.json
```


#### 📌 youtubers.js (라우팅)
```js
const express = require("express");
const fs = require("fs");
const path = require("path");
const router = express.Router();

// 경로 및 유틸 함수 정의
const DB_PATH = path.join(__dirname, "../data/db.json");

// 유틸 함수
const readData = () => {
  const raw = fs.readFileSync(DB_PATH, "utf-8");
  return JSON.parse(raw).youtubers;
};

// db.json 파일에 저장하는 쓰기 함수
const writeData = (list) => {
  fs.writeFileSync(DB_PATH, JSON.stringify({ youtubers: list }, null, 2));
};
  
// 다음 등록할 유튜버의 id를 자동으로 계산해주는 함수
const getNextId = (list) => {
  const ids = list.map((yt) => yt.id);
  return ids.length === 0 ? 1 : Math.max(...ids) + 1; // 가장 큰 id 찾아서 +1
};

// 전체 조회
router.get("/", (req, res) => {
  const list = readData();
  res.json(list);
});

// 개별 조회
router.get("/:id", (req, res) => {
  const list = readData();
  const numericId = parseInt(req.params.id);
  const youtuber = list.find((yt) => yt.id === numericId);
  if (!youtuber) {
    return res.status(404).json({ message: "없는 youtuber입니다." });
  }
  res.json(youtuber);
});

// 등록
router.post("/", (req, res) => {
  const list = readData(); // db.json에서 유튜버 데이터 불러오기
  const newId = getNextId(list); // list 데이터를 매개변수로 받아서 id 계산
  const newYoutuber = { id: newId, ...req.body }; // 새 유튜버 객체 생성
  
  list.push(newYoutuber); // 배열에 추가
  writeData(list); // 변경된 list를 db.json에 다시 저장
  
  res.json({
    message: `${req.body.channelTitle} 유튜버님 생활을 응원합니다!`,
  id: newId,
  });
});

// 수정
router.put("/:id", (req, res) => {
  const list = readData();
  const numericId = parseInt(req.params.id);
  const index = list.findIndex((yt) => yt.id === numericId);

  if (index === -1) {
    return res.status(404).json({ message: "없는 youtuber입니다." });
  }

  list[index] = { id: numericId, ...req.body };
  writeData(list);
  res.json({ message: "수정 완료", id: numericId });
});

// 삭제
router.delete("/:id", (req, res) => {
  const list = readData();
  const numericId = parseInt(req.params.id);
  const newList = list.filter((yt) => yt.id !== numericId);

  if (list.length === newList.length) {
    return res.status(404).json({ message: "없는 youtuber입니다." });
  }

  writeData(newList);
  res.json({ message: `id ${numericId} 유튜버 삭제 완료` });
});

module.exports = router;
```

#### 📌 app.js
```js
const express = require("express");
const app = express();
const youtuberRouter = require("./routes/youtubers");

app.use(express.json());
app.use("/youtubers", youtuberRouter);

app.listen(1234, () => {
  console.log("🚀 서버 실행: http://localhost:1234");
});
```


---
### ✅ Postman으로 API 테스트하기

|기능|Method|Endpoint|Body|
|---|---|---|---|
|전체 조회|GET|`/youtubers`|-|
|개별 조회|GET|`/youtubers/1`|-|
|등록|POST|`/youtubers`|`{ "channelTitle": "브이로그", "sub": "10만명", "videoNum": "77개" }`|
|수정|PUT|`/youtubers/2`|`{ "channelTitle": "수정한내용", ... }`|
|삭제|DELETE|`/youtubers/2`|-|

--- 

## **4️⃣ 프론트엔드: React + Tailwind**

- Axios로 API 연동
- Tailwind로 반응형 UI 구성
- `Component` → `API` → `Page` 구조로 유지복수와 확장성 고려

### ✅  프론트엔드: React UI 구조

#### 📁 React 구조 예시
```bash
frontend/
├── src/
│   ├── pages/            # 페이지 단위 컴포넌트
│   │   ├── YoutuberList.jsx      # 유튜버 전체 목록 페이지
│   │   ├── YoutuberDetail.jsx    # 유튜버 상세 및 수정 페이지
│   │   ├── YoutuberForm.jsx      # 유튜버 등록 페이지
│   ├── api/              # API 호출 함수 모음
│   │   └── youtuber.js
│   ├── components/       # 공통 UI 컴포넌트 (Button, Header, Input 등)
│   └── App.jsx           # 라우팅 및 전역 설정
```

#### 📌  api/youtuber.js (Axios로 API 호출 관리)

```js
import axios from "axios";
const BASE_URL = "http://localhost:1234/youtubers";

export const getAllYoutubers = () => axios.get(BASE_URL);
export const getYoutuber = (id) => axios.get(`${BASE_URL}/${id}`);
export const createYoutuber = (data) => axios.post(BASE_URL, data);
export const updateYoutuber = (id, data) => axios.put(`${BASE_URL}/${id}`, data);
export const deleteYoutuber = (id) => axios.delete(`${BASE_URL}/${id}`);
```

#### 📌 YoutuberList.jsx (목록 렌더링 + API 연동 예시)

유튜버 목록을 가져오는 API를 호출하고, 상태로 관리하는 기본적인 useEffect 흐름입니다.

```js
import { useEffect, useState } from "react";
import { getAllYoutubers } from "../api/youtuber";

export default function YoutuberList() {
  const [list, setList] = useState([]);

  useEffect(() => {
    getAllYoutubers().then((res) => setList(res.data));
  }, []);

  return (
    <div>
      {list.map((yt) => (
        <div key={yt.id}>
          <h2>{yt.channelTitle}</h2>
          <p>{yt.sub}</p>
        </div>
      ))}
    </div>
  );
}
```

![node_frontend_1](https://seonohblog.netlify.app/assets/node_frontend_1.png)

![node_frontend_2](https://seonohblog.netlify.app/assets/node_frontend_2.png)

![node_frontend_3](https://seonohblog.netlify.app/assets/node_frontend_3.png)

----

## **5️⃣ 공통 컴포넌트 설계 (디자인 시스템 기반)**

- `Button`, `Input`, `Header`, `YoutuberCard`, `Loader` 등 재사용 가능한 UI 컴포넌트로 구성
- variant / icon / label 등 props과 함께 재사용 가능
- TailwindCSS class 조합으로 UI 구현

```bash
components/
├── Button.jsx       # variant, icon, label props과 함께 가능
├── Input.jsx        # placeholder, name, value, onChange
├── Header.jsx       # title, showBack, rightElement
├── YoutuberCard.jsx # 정보 + 수정/삭제 Button 포함
└── Loader.jsx       # 로딩 UI
```


---

## **6️⃣ Storybook 적용**

Storybook을 활용해 UI 컴포넌트를 **독립적으로 개발하고 시각적으로 테스트 및 문서화**
- Button, Input, Header, Card 등은 components/에 따로 모아두고 variant, icon 등 props로 확장 가능하게 개발
- 스타일은 Tailwind 기반이지만 디자인 토큰으로 추상화된 부분은 따로 분리
- Storybook을 통해 디자이너와 바로 UI 상태를 공유하거나 컴포넌트 테스트 가능
- 로딩, 오류 처리 등 UX를 위한 기본 요소도 Loader 컴포넌트로 통일

### **✅ 작성 방식 (Storybook)**

- 파일명은 컴포넌트명 + `.stories.jsx` (예: `Button.stories.jsx`)
- 각 컴포넌트 상태(variant)별로 export const Primary, Secondary, WithIcon, Disabled 등 args를 통해 props를 쉽게 테스트하고 문서화 가능
- Storybook 내에서 다양한 상태의 UI를 직접 시각적으로 확인 가능

### **✅ Storybook 사용 목적**

- **Docs 탭** : 컴포넌트 설명과 사용 예제 + 코드 시각화 확인 가능
- **Controls 탭** : props 값을 직접 조절해 UI 상태 실시간 테스트

![node_storybook](https://seonohblog.netlify.app/assets/node_storybook.png)

----

## 파이널 회고

- 이번 프로젝트를 통해 **RESTful API 구조를 직접 설계**해보며, 서버와 클라이언트가 어떻게 통신하고 흐름이 이어지는지 더 체감할 수 있었다. 단순히 axios로 요청 보내는 걸 넘어서, 라우팅과 응답 구조까지 전부 내가 짜니까 더 이해가 깊어졌다.
- 그리고 무엇보다도, **컴포넌트를 어떻게 잘 나누고 독립적으로 개발할 수 있을지** 고민하게 됐다. 그 과정에서 Storybook을 처음 써봤는데, UI 컴포넌트를 따로 문서화해서 테스트하고 관리할 수 있다는 게 너무 좋았다. 앞으로 실무에서도 꼭 쓰고 싶을 만큼 강력한 도구라는 걸 알게 됐다.
- **디자인 시스템의 기반**을 잡으려고 노력했다. 아직 완성은 아니지만, 버튼, 인풋, 카드 컴포넌트를 재사용 가능하게 만들고, props로 확장할 수 있게 구성하면서 확장 가능한 구조에 한 발짝 다가간 느낌이었다.
- 특히 **파일 구조나 역할별 컴포넌트, API 호출 분리**를 깔끔하게 나누니까 유지보수나 가독성이 확실히 좋아졌고, “아, 이렇게 짜야 나중에 팀 프로젝트 할 때도 좋겠다”는 감이 생겼다.
- 마지막으로는 **기능 구현뿐 아니라 문서화**도 직접 챙겨보며, 진짜 하나의 작은 서비스가 돌아가는 느낌을 받았다. 이번 프로젝트를 통해 단순한 학습을 넘어서, 실제 현업 흐름에 가까운 개발 경험을 할 수 있었던 것 같아 굉장히 뿌듯했다.

