Success
Retry
Archive

- **“DayzLog”** (매일 기록)
### **👤** 개인 모드

- 나만 로그인해서 사용
- Success / Retry / Archive 
- 우선순위 라벨 설정
- 월별 통계 보기 가능 
- 다이어트/공부/자기관리용으로 최적

- 각 투두는 생성 시 상태: Pending

### **🗓️** 날짜별 캘린더 연결

- 하루 단위 투두 + 리뷰 연결
- 각 날짜 클릭 시:
    - 해당일 **투두 목록**
    - 리뷰 메모
    - 상태 통계 (4 Success / 2 Retry)
    - 무드 이모지 모음

### **🔄** 데일리 갱신

- 매일 자정에 자동 초기화
- 전날 미완료 항목은 Retry로 자동 라벨링 가능 (설정에 따라)

### ✍️ 하루 마감 루틴: Today’s Review

- **Success / Retry 분류 체크리스트**
- 한 줄 **메모 회고** 입력:
    "오늘은 날씨처럼 미적지근했음. 하지만 요가는 했다."
- **기분 태그** 붙이기 (예: 🧘‍♀️ 평온 / 😩 지침 / 😤 폭주 / 😊 좋음 / 😞 나쁨 / 😶 그냥 그럼 / 😔 속상함 / 😢 울컥 / 😌 감사함
/  😵 혼란 / 🤩 뿌듯함 )
- 전날과 비교 리포트: “어제보다 오늘은 더 나았다” 느낌 가능

-  Archive는 월말에 돌아보면서 _오, 나 이거 포기했네_ 감상회 가능
- 우선순위별로 색 필터도 적용 가능: 빨강(1), 주황(2), 노랑(3)


### 🏷️ 우선순위 라벨 (색 or 숫자)

- 1: 긴급! 
- 2: 중요하긴 한데 오늘 아니어도 됨 
- 3: 있으면 좋은 거 

```bash
🟢 Success   🔄 Retry   📁 Archive

[ ] (2) 깃허브 레이아웃 정리       🔄
[✔] (1) 요가 수업 참여              ✅
[ ] (3) 테일윈드 config 블로그 작성 🔄
[ ] (2) 친구 답장 보내기            📁
```


### **🗓 어떻게 적용하면 좋냐면:**

- 오늘 하루 끝에 Today’s Review에
    → 체크리스트 남기고
    → 기분 태그 하나 선택
    → 한 줄 느낌 적기  

예: ✔️ SUCCESS | 😶 그냥 그럼 | "일은 다 했는데 뭔가 공허했음."



 **React + Tailwind**

## **📚 페이지 구성 (기본 구조)**
### 🔹 1. Main 페이지 (오늘의 할 일)
- 투두 리스트 (일 별 리스트)
    - TodoList (Success / Retry / Archive 상태 표시) - 수시로 변경 가능
    - ArchivePanel (선택 시만 열림) - “📂 Archive” 버튼 클릭
    - CalendarView (선택 시만 열림) - 📅 Calendar 버튼 클릭

### 🔹 2. History 페이지 (캘린더 뷰)
[CalendarView]
→ 클릭시 해당 투두로 가기
 → DailyReview (한 줄 회고) - EmotionTag (예: 😊, 😑, 😫 등) 선택, 글쓰기(CRUD)

### 🔹 2. Archive 페이지
- Archive 로 옮겨진 리스트 보임(날짜 표기)


## **🧩 컴포넌트 설계**

### ✅ TodoCard
- props: title, priority, status, onStatusChange
- 내부 버튼: ✔ Success, 🔁 Retry, 📦 Archive
- 스타일: Tailwind로 border-l-priorityColor 라벨 표시

---

### 🧘 EmotionSelector
- props: onSelect(emotion)
- 🙂😐😣😌😑 등 감정 아이콘 클릭
- Tailwind로 선택 강조

---

### 📝 DailyReview
- input: 회고 작성 textarea
- 저장 버튼 or 자동 저장

---

### 📅 CalendarView
- props: todosByDate
- 날짜별 컬러 표시 (Success: green, Retry: yellow, Archive: gray)

---

### 🏷️ PriorityBadge
- props: level (1|2|3)
- Tailwind 스타일: bg-red-500, bg-yellow-400, bg-blue-300 등

---

## 🔄 상태 관리 구조 

- **Recoil** or 그냥 useState + useEffect

```jsx
const [todos, setTodos] = useState<TodoType[]>([]);
const [selectedDate, setSelectedDate] = useState(today);
const [emotion, setEmotion] = useState(null);
const [review, setReview] = useState("");
```

```
📁 src
│
├── 📁 components
│   ├── CalendarView.jsx         # 캘린더 및 날짜 클릭 핸들링
│   ├── TodoList.jsx             # 체크리스트 렌더링
│   ├── EmotionTag.jsx           # 감정 태그 UI
│   ├── DailyReview.jsx          # 하루 회고 텍스트 UI
│   └── ArchivePanel.jsx         # 아카이빙된 항목 모아보기

├── 📁 pages
│   └── Home.jsx                 # 메인 페이지

├── 📁 services
│   └── api.js                   # API 함수들 (axios 요청 모음)

├── 📁 hooks
│   └── useDailyData.js         # 날짜별 데이터 fetch hook

├── 📁 utils
│   └── dateFormat.js           # 날짜 포맷 헬퍼

├── 📁 constants
│   └── moodTags.js             # 감정 태그 preset

├── App.jsx
└── index.js
```




## 📦 백엔드 연동을 위한 구성

### 🔌 API 기본 구성 (예상)
```
GET    /todos?date=YYYY-MM-DD       → 특정 날짜 투두 리스트 조회
POST   /todos                       → 새 투두 생성
PATCH  /todos/:id                  → 투두 상태(Success, Retry 등) 변경
DELETE /todos/:id                  → 투두 삭제

GET    /reviews?date=YYYY-MM-DD    → 감정 및 회고 조회
POST   /reviews                    → 회고 등록 또는 수정

GET    /user                       → 로그인 시 사용자 정보
```

## 🧱 ERD 설계 예시 (가볍게 그림)
```
User
- id
- email
- nickname

Todo
- id
- user_id (FK)
- title
- date
- priority (1,2,3)
- status (Success, Retry, Archive)

Review
- id
- user_id (FK)
- date
- emotion (string or enum)
- content (text)
```

**회원가입 + 로그인 기능**
“내 감정, 회고, 계획”을 혼자만 보고 싶음? (보안 필요)
여러 디바이스에서 기록하고 싶음? (동기화)
기록을 지속적으로 저장하고 싶음? (DB 연결)
진짜 “서비스”처럼 보이는 포트폴리오 느낌이 필요함?
다른 사람과 공유되는 팀 프로젝트에 확장하기 좋음

## **🔐 인증방식 추천**

- MVP라면 그냥 **JWT** 쓰면 되고, 쿠키 or localStorage 저장
- 프론트에서는 axios 인스턴스 만들고 헤더에 토큰 자동 포함시키면 됨

---

## **🪝 React + Express 연동 구조 (Frontend 기준)**
```ts
// services/api.ts
import axios from 'axios';

const instance = axios.create({
  baseURL: 'http://localhost:4000/api',
  headers: {
    'Content-Type': 'application/json',
  },
});

export default instance;
```

✅ Vercel에 프론트 (React) 배포하고
• 로그인 페이지, 캘린더, 회고 다 여기


✅ 백엔드 (Express, JWT) 는 따로
- 백엔드: Render or Railway (Express + MongoDB or PostgreSQL)
• 여기서 JWT 발급해주고, DB랑 통신하고 등등

🤖 그래서 연결은?
1. 사용자가 로그인하면
2. 프론트에서 백엔드로 ID/PW 전송
3. 백엔드가 JWT 만들어서 응답
4. 프론트는 그 JWT를 localStorage나 cookie에 저장
5. 이후 요청마다 JWT를 헤더에 실어서 백엔드로 전송



