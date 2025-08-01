# 1. 게스트 모드 구현 
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



# 2. 구글 로그인 구현
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

## 💡 개발 팁

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





#### 깃허브 : 
https://github.com/sunfivemin/DOT-DAILY

#### 배포사이트 :
[dot-daily.vercel.app](https://dot-daily.vercel.app/ "https://dot-daily.vercel.app/")
