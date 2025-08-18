
#### 깃허브 : 
https://github.com/oneday-coding/DOT-DAILY
#### 배포사이트 :
[dot-daily.vercel.app](https://dot-daily.vercel.app/ "https://dot-daily.vercel.app/")

# 1. 날짜 키 처리 방식 통일로 React Query 캐시 동기화 문제 해결

**DOT.DAILY 프로젝트**를 진행하며 겪었던 가장 골치 아픈 버그 중 하나는,
바로 **오늘 등록한 할 일이 내일로 저장되거나**,
**할 일을 등록/삭제/이동했는데 실시간으로 UI에 반영되지 않는** 문제였습니다.
API는 분명 정상 응답을 주고 있음에도…
👀 **UI는 새로고침(F5)을 눌러야만 갱신**됐습니다.

## 🚨 문제 상황

### **1. 등록한 날짜가 하루 밀린다?**
- 8월 4일에 등록했는데 → 데이터는 8월 5일로 저장됨
- 같은 날짜인데도 다른 항목처럼 분리되어 보임

### **2. 할 일 등록/삭제/이동 후에도 UI 무반응**
- 삭제해도 화면에 계속 남아 있음
- “재시도”, “보류함” 이동 후에도 반영되지 않음
- 등록했는데 아무 변화 없음

### 네트워크 로그 분석
```bash
# 콘솔에서는 API 요청이 성공적으로 처리됨
✅ DELETE /api/v1/todos/139 - 200 OK
✅ PUT /api/v1/todos/140/retry - 200 OK  
✅ POST /api/v1/todos - 201 Created
```
서버에서는 정상 응답을 주고 있었고, DB에도 반영되고 있었습니다.

---

### 🔍 원인 1: 날짜가 하루 밀리는 이유는 UTC 때문

#### ❌ 1. tasks.ts (API 호출)
```typescript
export const getTasksByDate = async (date: Date | string): Promise<Task[]> => {
  // ❌ 문제: toISOString() 사용
  const dateStr = typeof date === "string" ? date : date.toISOString().split("T")[0];
  const response = await httpClient.get(`/todos?date=${dateStr}`);
  return response.data.data;
};
```
- toISOString()은 **UTC 기준 날짜**를 반환
- 한국(KST) 기준으로 2025-08-04일지라도 → 2025-08-03T15:00:00.000Z
- 결국 "2025-08-03"으로 잘림 → **하루 전 날짜로 저장**

#### ✅ 해결 방법: toLocaleDateString("en-CA") 사용
```typescript
const dateStr = date.toLocaleDateString("en-CA");  // "2025-08-04"
```
- 로컬 시간(KST) 기준으로 날짜를 문자열로 변환
- "en-CA"는 YYYY-MM-DD 포맷 제공
- 날짜 밀림 문제 해결!

| **항목**   | toISOString()            | toLocaleDateString()  |
| -------- | ------------------------ | --------------------- |
| 기준 시간대   | UTC 기준                   | 로컬 시간대 (예: 한국은 UTC+9) |
| 출력 형식    | YYYY-MM-DDTHH:mm:ss.sssZ | 국가별 읽기 쉬운 날짜 형식       |
| 용도       | API 전송, DB 저장 등에 적합      | UI 표시용, 사용자 친화적 포맷    |
| 시간 포함 여부 | 날짜 + 시간                  | 날짜만 (기본적으로)           |

---

### 🔍 원인 2: React Query 캐시 키 불일치

| **위치**            | **키 생성 방식**                      |
| ----------------- | -------------------------------- |
| tasks.ts          | date.toISOString().split("T")[0] |
| MyDayPage.tsx     | ["tasks", date]                  |
| TaskFormModal.tsx | format(date, "yyyy-MM-dd")       |
| TaskItem.tsx      | toISOString()                    |

→ 날짜 표현이 제각각이라 React Query가 **다른 데이터로 인식**
→ invalidateQueries()가 작동하지 않음 → **UI 미갱신**

#### 2. TaskFormModal.tsx (할 일 등록/수정)
```typescript
const handleSubmit = async () => {
  const taskData = {
    title: label.trim(),
    priority,
    // ❌ 문제: format() 사용  
    date: format(date, "yyyy-MM-dd"),
  };
  
  // ❌ 문제: 다른 방식으로 캐시 키 생성
  const dateKey = format(date, "yyyy-MM-dd");
  queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
};
```

#### 3. MyDayPage.tsx (메인 페이지)
```typescript
const { data: tasks = [] } = useQuery({
  // ❌ 문제: 또 다른 방식으로 키 생성
  queryKey: ["tasks", selectedDate.toISOString().split("T")[0]],
  queryFn: () => getTasksByDate(selectedDate),
});
```

#### 4. TaskItem.tsx (개별 할 일 컴포넌트)
```typescript
const handleDelete = async () => {
  await deleteTask(task.id as number);
  
  // ❌ 문제: 또 다른 방식
  const dateKey = selectedDate.toISOString().split("T")[0];
  queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
};
```

**결과**: 캐시 키가 `"2025-01-15"`와 `"2025-01-16"`으로 달라져서 캐시 무효화가 제대로 동작하지 않았습니다.

```typescript
// 방식 1: toISOString() - UTC 기준
"2025-01-15T15:00:00.000Z" → "2025-01-15"

// 방식 2: format() - 로컬 시간대 기준  
format(new Date(), "yyyy-MM-dd") → "2025-01-16"

// 방식 3: toLocaleDateString() - 로컬 시간대 기준
date.toLocaleDateString("en-CA") → "2025-01-16"
```

---

## 🛠️ 해결 방법

### **✅ 1단계: 날짜 처리 방식 통일 + KST 기준 정규화**

**모든 날짜 관련 로직을 getTodayInKorea() 유틸 함수로 정리**하고,
toLocaleDateString("en-CA")를 통해 YYYY-MM-DD 형식으로 통일했습니다.
#### 📁 utils/dateUtils.ts
```typescript
/**
 * 한국 시간 기준으로 오늘 날짜를 가져오는 함수
 */
export const getTodayInKorea = (): Date => {
  const now = new Date();
  const koreaTime = new Date(
    now.toLocaleString("en-US", { timeZone: "Asia/Seoul" })
  );
  koreaTime.setHours(0, 0, 0, 0); // 00:00:00 정규화
  return koreaTime;
};
```
 ✅ UTC 기준으로 하루 밀리는 버그는 대부분 Date 객체가 생성 시 시차(타임존)를 고려하지 않기 때문에 발생합니다.
 이를 Asia/Seoul 기준으로 맞추고, 시각도 자정으로 맞춰야 버그 없이 날짜 비교가 가능합니다.

### **✅ 2단계: 상태 관리 스토어 정규화 적용**
#### 📁 store/useDateStore.ts
```typescript
import { getTodayInKorea } from "@/utils/dateUtils";

export const useDateStore = create<DateState>()((set) => ({
  selectedDate: getTodayInKorea(),

  setSelectedDate: (date) => {
    const normalizedDate = new Date(date);
    normalizedDate.setHours(0, 0, 0, 0); // 항상 자정으로 맞춤
    set({ selectedDate: normalizedDate });
  },
}));

// reset 기능도 유틸화
export const resetDateStore = () => {
  useDateStore.setState({ selectedDate: getTodayInKorea() });
};
```

### **✅ 3단계: 클라이언트 전역에서 날짜 포맷 통일**
#### A. tasks.ts 수정
```typescript
export const getTasksByDate = async (date: Date | string): Promise<Task[]> => {
  // ✅ 수정: 일관된 날짜 처리
  const dateStr = typeof date === "string" ? date : date.toLocaleDateString("en-CA");
  const response = await httpClient.get(`/todos?date=${dateStr}`);
  return response.data.data;
};
```

#### B. TaskFormModal.tsx 수정
```typescript
const handleSubmit = async () => {
  const taskData = {
    title: label.trim(),
    priority,
    // ✅ 수정: 통일된 방식
    date: date.toLocaleDateString("en-CA"),
  };

  // ✅ 수정: 캐시 무효화 강화
  const dateKey = date.toLocaleDateString("en-CA");
  await queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
  await queryClient.refetchQueries({ queryKey: ["tasks", dateKey] });
};
```

#### C. MyDayPage.tsx 수정
```typescript
const { data: tasks = [] } = useQuery({
  // ✅ 수정: 통일된 키 생성
  queryKey: ["tasks", selectedDate.toLocaleDateString("en-CA")],
  queryFn: () => getTasksByDate(selectedDate),
});
```

#### D. TaskItem.tsx 수정
```typescript
const handleDelete = async () => {
  try {
    await deleteTask(task.id as number);

    // ✅ 수정: 통일된 날짜 키 + 강화된 캐시 처리
    const dateKey = selectedDate.toLocaleDateString("en-CA");
    
    // 1. Optimistic Update (즉시 UI에서 제거)
    queryClient.setQueryData(["tasks", dateKey], (old: Task[]) => {
      return old?.filter((t) => t.id !== task.id) || [];
    });

    // 2. 서버와 동기화
    await queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
    
    showToast("할 일이 삭제되었습니다 🗑️");
  } catch (error) {
    // 실패 시 캐시 롤백
    const dateKey = selectedDate.toLocaleDateString("en-CA");
    queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
    showToast("할 일 삭제에 실패했습니다 😞");
  }
};
```

---

## 🔄 캐시 무효화 전략 개선

#### ✅ Optimistic Updates (즉시 UI 반영 → 실패 시 롤백)
```typescript
// 즉시 UI 업데이트 → 서버 요청 → 실패 시 롤백
queryClient.setQueryData(["tasks", dateKey], (old: Task[]) => {
  return old?.filter((t) => t.id !== task.id) || [];
});
```

#### ✅ 무효화 + 리패치로 서버 동기화
```typescript
await queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
await queryClient.refetchQueries({ queryKey: ["tasks", dateKey] });
```

| **함수**            | **설명**                     | **타이밍** |
| ----------------- | -------------------------- | ------- |
| invalidateQueries | 해당 쿼리를 무효화(다음 사용 시 새로 요청됨) | 지연됨     |
| refetchQueries    | 해당 쿼리를 즉시 서버에서 다시 가져옴      | 즉시      |

#### ✅ 에러 핸들링 강화
```typescript
try {
  await deleteTask(task.id);
} catch (error) {
  // 실패 시 캐시 상태 복구
  queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
  showToast("작업에 실패했습니다 😞");
}
```

---

## 🎯 결과

### Before (문제 상황)
```bash
사용자: 할 일 삭제 클릭
→ API 요청 성공 ✅
→ UI 변화 없음 ❌
→ F5 새로고침 필요 😤
```

### After (해결 후)
```bash
사용자: 할 일 삭제 클릭  
→ 즉시 UI에서 제거 ✅
→ API 요청 성공 ✅
→ 서버와 동기화 완료 ✅
→ 새로고침 불필요 🎉
```

### 성능 개선
- **사용자 경험**: 즉시 반응하는 인터페이스
- **네트워크 효율성**: 불필요한 새로고침 제거
- **상태 일관성**: 클라이언트-서버 동기화 보장

---

## 💡 정리

### 1. 날짜는 시간대 고려 + 포맷 통일이 핵심
```typescript
// ✅ 유틸 함수 사용
getTodayInKorea();                        // → Date 객체 자체가 KST 자정 기준
date.toLocaleDateString("en-CA");        // → YYYY-MM-DD 형식 보장

// ❌ 피해야 할 방식
date.toISOString().split("T")[0];        // → UTC 기준, 하루 밀림 발생 가능성
```

### 2. React Query 캐시 키 관리
```typescript
// ✅ 캐시 키 생성을 중앙화
const getCacheKey = (date: Date) => ["tasks", date.toLocaleDateString("en-CA")];

// ✅ 무효화도 재사용 가능하게
const invalidateTaskCache = async (date: Date) => {
  const key = getCacheKey(date);
  await queryClient.invalidateQueries({ queryKey: key });
  await queryClient.refetchQueries({ queryKey: key });
};
```

### 3. Optimistic Updates 활용으로 UX 향상
```typescript
// 사용자 경험을 위한 즉시 UI 업데이트
queryClient.setQueryData(key, optimisticUpdate);
try {
  await apiCall();
} catch {
  queryClient.invalidateQueries({ queryKey: key }); // 롤백
}
```

### 4. **디버깅 팁**
```typescript
// 캐시 상태 확인
console.log(queryClient.getQueryCache().getAll());

// 특정 쿼리 데이터 확인  
console.log(queryClient.getQueryData(['tasks', '2025-01-15']));
```


---

# 2. 게스트 모드 구현 
**문제**: 사용자가 로그인 없이 앱을 체험할 수 없어 이탈률 증가
**해결**: 브라우저 localStorage를 활용한 게스트 모드 구현

## 🔧 핵심 구현

### **1. localStorage 데이터 저장**

```tsx
// 날짜별로 분리된 저장 방식
const dateStr = date.toISOString().split("T")[0];
localStorage.setItem(`guest-tasks-${dateStr}`, JSON.stringify(tasks));

```

### **2. 게스트 모드 전용 API 클라이언트**

```tsx
// lib/api/guestTasks.ts
export const getGuestTasks = (): GuestTask[] => {
  if (typeof window === "undefined") return [];
  try {
    const stored = localStorage.getItem("guest_tasks");
    return stored ? JSON.parse(stored) : [];
  } catch {
    return [];
  }
};

export const createGuestTask = (taskData) => {
  const newTask = {
    id: `guest_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
    // ... task data
  };
  // localStorage에 저장
};
```

### **3. 통합된 API 분기 처리**

```tsx
// MyDayPage.tsx에서 게스트/서버 모드 분기
const { isGuest, isAuthenticated } = useAuthStore();

// 게스트 모드일 때 localStorage 사용
if (isGuest) {
  const stored = localStorage.getItem(`guest-tasks-${dateStr}`);
  // localStorage 데이터 처리
} else {
  // 서버 API 사용
  const { data: tasks } = useQuery({...});
}

```

### **4. 모달에서의 분기 처리**

```tsx
// TaskFormModal.tsx
if (isGuest) {
  // 게스트 모드: 로컬 스토리지에 저장
  if (saveGuestTask(taskData)) {
    showToast("새로운 할 일이 등록되었습니다! ✅");
    if (onSuccess) {
      onSuccess();
    }
  }
} else {
  // 인증된 사용자: 서버 API 사용
  const newTask = await createTask(taskData);
  queryClient.invalidateQueries({ queryKey: ["tasks", dateKey] });
}

```


---


# 3. 구글 로그인 구현
프론트엔드 개발자로서 DOT-DAILY 프로젝트에서 구글 로그인을 구현한 경험을 공유하려고 합니다.
DOT-DAILY는 개인 일정 관리 및 회고 서비스로, 사용자 편의성을 위해 소셜 로그인을 도입했습니다. 특히 구글 로그인은 가장 보편적이고 안정적인 OAuth 제공자 중 하나라서 선택하게 되었습니다.

### 1. **Identity Toolkit API** (Google Cloud)

- **역할**: Google OAuth 인증의 핵심 엔진
- **기능**: 사용자 인증 토큰 검증, Google 계정 정보 제공

### 2. **People API** (Google Cloud)

- **역할**: 사용자 프로필 정보 조회
- **기능**: 이름, 이메일, 프로필 사진 등 제공

### 3. **@react-oauth/google** (프론트엔드)

- **역할**: React에서 Google OAuth 팝업 처리
- **기능**: Google 토큰 받아오기, 사용자 인터랙션 처리
```bash
npm install @react-oauth/google
```

- React 전용: React 생태계에 최적화
- 간단한 API: 복잡한 OAuth 플로우를 단순화
- TypeScript 지원: 완벽한 타입 정의

### 1. 프론트엔드 구현
```tsx
// 사용 라이브러리
import { GoogleOAuthProvider, useGoogleLogin } from "@react-oauth/google";

// 환경변수 설정
const GOOGLE_CLIENT_ID = process.env.NEXT_PUBLIC_GOOGLE_CLIENT_ID;
```

#### B. GoogleOAuthProvider 설정
```tsx
export default function LoginPage() {
  return (
    <GoogleOAuthProvider
      clientId={GOOGLE_CLIENT_ID}
      onScriptLoadError={() => {
        console.warn("Google OAuth 스크립트 로드 실패");
      }}
    >
      <LoginPageContent />
    </GoogleOAuthProvider>
  );
}
```

#### C. Google 로그인 훅
```tsx
const googleLogin = useGoogleLogin({
  onSuccess: async (tokenResponse) => {
    // 1. Google에서 받은 access_token 추출
    const accessToken = tokenResponse.access_token;
    
    // 2. 백엔드로 Google access token 전달
    const response = await axios.post(
      `${API_BASE_URL}/auth/google/login`,
      { accessToken }
    );
    
    // 3. 백엔드에서 받은 JWT 토큰과 사용자 정보 저장
    const jwt = response.data.accessToken;
    const user = response.data.user;
    
    // 4. 로컬 상태 업데이트
    localStorage.setItem("accessToken", jwt);
    login(user, jwt);
  },
  onError: (error) => {
    console.error("Google OAuth SDK 오류:", error);
  },
});
```


#### D. UI 구현
```tsx
<button
  onClick={() => googleLogin()}
  disabled={!GOOGLE_CLIENT_ID}
  className="flex items-center justify-center gap-2 bg-white border hover:bg-gray-100 rounded-full py-3"
>
  <Image src="/google.svg" alt="구글 로그인" width={24} height={24} />
  구글로 로그인
</button>
```


### 2. 백엔드 구현

#### A. 라우터 설정
```tsx
// auth.router.ts
router.post('/google/login', async (req: Request, res: Response) => {
  const { accessToken } = req.body;
  const result = await googleTokenService(accessToken);
  
  res.json({
    success: true,
    accessToken: result.accessToken,
    user: result.user,
    message: 'Google 로그인 성공',
  });
});
```

#### B. Google OAuth 서비스
```tsx
// googleAuth.service.ts
export const googleTokenService = async (accessToken: string) => {
  // 1. Google API로 사용자 정보 조회
  const googleUserResponse = await fetch(
    `https://www.googleapis.com/oauth2/v2/userinfo?access_token=${accessToken}`
  );
  
  const googleUser = await googleUserResponse.json();
  
  // 2. 기존 사용자 확인 또는 새 사용자 생성
  let user = await prisma.user.findUnique({
    where: { email: googleUser.email },
  });
  
  if (!user) {
    user = await prisma.user.create({
      data: {
        email: googleUser.email,
        username: googleUser.name,
        image: googleUser.picture,
      },
    });
  }
  
  // 3. Google 계정 정보 저장/업데이트
  await prisma.account.upsert({
    where: {
      provider_providerAccountId: {
        provider: 'google',
        providerAccountId: googleUser.id,
      },
    },
    update: { access_token: accessToken },
    create: {
      userId: user.id,
      type: 'oauth',
      provider: 'google',
      providerAccountId: googleUser.id,
      access_token: accessToken,
    },
  });
  
  // 4. JWT 토큰 생성
  const token = jwt.sign(
    { id: user.id, email: user.email },
    process.env.JWT_SECRET!,
    { expiresIn: '24h' }
  );
  
  return {
    user: {
      id: user.id,
      email: user.email,
      username: user.username,
      image: user.image,
    },
    accessToken: token,
  };
};
```

### 3. 전체 플로우
```text
1. 사용자가 "구글로 로그인" 버튼 클릭
   ↓
2. @react-oauth/google이 Google OAuth 팝업 열기
   ↓
3. 사용자가 Google 계정으로 로그인 및 권한 승인
   ↓
4. Google이 Authorization Code 반환
   ↓
5. @react-oauth/google이 Authorization Code를 Access Token으로 교환
   ↓
6. 프론트엔드가 Access Token을 백엔드로 전송
   ↓
7. 백엔드에서:
   - Google API로 Access Token 검증
   - 사용자 정보 조회 (이메일, 이름, 프로필 사진)
   - DB에서 기존 사용자 확인 또는 새 사용자 생성
   - Google 계정 정보 저장/업데이트
   - JWT 토큰 생성
   ↓
8. 백엔드가 JWT 토큰과 사용자 정보를 프론트엔드로 응답
   ↓
9. 프론트엔드가 JWT를 localStorage에 저장
   ↓
10. 로그인 완료, 메인 페이지로 이동
```


## ⚠️ 에러 처리

### 1. Toast 알림 시스템
```tsx
// frontend/src/components/ui/Toast/ToastProvider.tsx
import { createContext, useContext, useState } from "react";

interface ToastContextType {
  showToast: (message: string, type?: "success" | "error" | "info") => void;
}

const ToastContext = createContext<ToastContextType | undefined>(undefined);

export const ToastProvider = ({ children }: { children: React.ReactNode }) => {
  const [toasts, setToasts] = useState<ToastItem[]>([]);

  const showToast = (message: string, type: "success" | "error" | "info" = "info") => {
    const id = Date.now();
    const newToast = { id, message, type };
    
    setToasts(prev => [...prev, newToast]);
    
    // 3초 후 자동 제거
    setTimeout(() => {
      setToasts(prev => prev.filter(toast => toast.id !== id));
    }, 3000);
  };

  return (
    <ToastContext.Provider value={{ showToast }}>
      {children}
      <div className="fixed top-4 right-4 z-50 space-y-2">
        {toasts.map(toast => (
          <ToastItem key={toast.id} {...toast} />
        ))}
      </div>
    </ToastContext.Provider>
  );
};
```

### 2. 구글 로그인 에러 처리
```tsx
const googleLogin = useGoogleLogin({
  onSuccess: async (tokenResponse) => {
    try {
      // ... 로그인 로직
    } catch (error) {
      console.error("❌ Google 로그인 실패:", error);

      if (axios.isAxiosError(error)) {
        console.error("Axios 오류 상세:", {
          status: error.response?.status,
          data: error.response?.data,
          message: error.message,
        });
      }

      showToast("Google 로그인에 실패했습니다.");
    }
  },
  onError: (error) => {
    console.error("❌ Google OAuth SDK 오류:", error);
    showToast("Google 로그인 중 오류가 발생했습니다.");
  },
});
```

### 환경변수 검증
```tsx
// 환경변수 체크 및 경고
if (!GOOGLE_CLIENT_ID) {
  console.warn("⚠️ NEXT_PUBLIC_GOOGLE_CLIENT_ID 환경변수가 설정되지 않았습니다.");
  console.warn("Google 로그인이 비활성화됩니다.");
  return <LoginPageContent />;
}

console.log("✅ Google Client ID 설정됨:", GOOGLE_CLIENT_ID.substring(0, 20) + "...");
```

## ⚡ 성능 최적화

### 1. 이미지 최적화
```tsx
// Next.js Image 컴포넌트 활용
<Image
  src="/google.svg"
  alt="구글 로그인"
  width={24}
  height={24}
  priority={false} // 로그인 버튼은 우선순위 낮음
/>

<Image
  src="/logo-vertical.svg"
  alt="dot_daily logo"
  width={60}
  height={60}
  priority // 로고는 우선순위 높음
/>
```

### 2. 코드 스플리팅
```tsx
// 동적 import로 Google OAuth 라이브러리 로딩
const GoogleOAuthProvider = dynamic(
  () => import("@react-oauth/google").then(mod => mod.GoogleOAuthProvider),
  { ssr: false }
);
```

### 3. 메모이제이션
```tsx
// 불필요한 리렌더링 방지
const LoginPageContent = React.memo(() => {
  // 컴포넌트 로직
});

// 콜백 메모이제이션
const handleGoogleLogin = useCallback(() => {
  googleLogin();
}, [googleLogin]);
```

## 💡 정리

### 1. TypeScript 활용
```tsx
// 타입 정의로 런타임 에러 방지
interface GoogleTokenResponse {
  access_token: string;
  token_type: string;
  expires_in: number;
}

interface LoginResponse {
  success: boolean;
  accessToken: string;
  user: User;
  message: string;
}

// API 응답 타입 체크
const response: LoginResponse = await axios.post(/* ... */);
```

### 2. 개발자 도구 활용
```tsx
// 개발 환경에서만 상세 로깅
if (process.env.NODE_ENV === "development") {
  console.log("🔍 Google 토큰 응답:", tokenResponse);
  console.log("🔄 백엔드로 Google 토큰 전송 중...");
}
```

### 3. 환경별 설정
```tsx
// 환경에 따른 API URL 설정
const API_BASE_URL = process.env.NODE_ENV === "production"
  ? process.env.NEXT_PUBLIC_PRODUCTION_API_URL || "https://dot-daily.onrender.com/api/v1"
  : process.env.NEXT_PUBLIC_API_BASE_URL || "http://localhost:3001/api/v1";
```


