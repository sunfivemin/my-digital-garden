1. Modal 모달창
• ✅ 할 일 등록 모달 (우선순위, 날짜 선택 등)
• ✅ 회고 작성/수정 모달
• ✅ 삭제 확인용 confirm 모달

구성 요소 예시:
• Modal.tsx: isOpen, onClose, children props로 제어
• 내부에 Dialog, Transition 라이브러리 쓰거나 div + z-index 조합



2. Input 인풋 필드
• ✅ 기본 인풋 (할 일 제목, 회고 내용)
• ✅ 날짜 선택 (캘린더 아이콘 포함 가능)

컴포넌트 예시:

• TextInput.tsx

• DateInput.tsx 또는 DatePicker.tsx (react-day-picker나 date-fns 사용 가능)

```
components/
  ui/
    Input/
      ├── Input.tsx            // ⭐️ 공통 스타일 및 base input (styled only)
      ├── TextInput.tsx        // ✅ 텍스트 필드 (label, error, helperText 포함)
      ├── TextareaInput.tsx    // ✅ 회고 입력 등 멀티라인 용
      ├── DateInput.tsx        // ✅ 날짜 선택용 (아이콘 포함)
      ├── SelectInput.tsx      // ✅ 우선순위 선택 등 라디오/셀렉트용
      ├── Input.stories.tsx    // ✅ Storybook 문서화
      └── index.ts             // ✅ barrel export
```

Toast, Dropdown, Modal 상태관리를 useUIStore처럼 zustand로 통합하는 설계 이거를 주스텐드로 설계하는게 낫지? 컴포넌트도 하나씩 만들어줘야해


  

1. ✅ SelectInput

• @headlessui/react 기반으로 구현했었지.

• 현재 라벨 애니메이션 문제 남아 있음 (Input과 동일한 방식으로 수정 필요).

  

2. ✅ DatePicker

• react-day-picker 기반.

• floating label 위치 문제 해결 필요.

  

3. 🔜 Modal, Toast

• 이미 코드만 있는 상태야. 디자인 시스템에 맞춰 다음을 정리해야 해:

• 상태별 색상 (성공/실패/정보 등)

• 애니메이션 (framer-motion 적용 가능)

• 모달은 포커스 트랩, Escape 닫기 기능 등 포함 여부 고려
```ts
const [open, setOpen] = useState(false);

<FullScreenModal isOpen={open} onClose={() => setOpen(false)}>
  <div className="flex items-center justify-between">
    <h2 className="text-lg font-semibold">할일 추가 모달</h2>
    <button onClick={() => setOpen(false)}>닫기</button>
  </div>

  <div className="mt-6 space-y-6">
    <Input label="오늘 할 일" placeholder="할 일 입력" />
    <SelectInput label="우선순위" options={...} />
    <DatePicker label="날짜" />
  </div>

  <Button className="mt-10 w-full rounded-full">할 일 등록하기</Button>
</FullScreenModal>
```



4. 🔜 공통 Alert, Badge, Tag, Chip 등

• 예: StatusBadge, TagLabel, PriorityLabel

• 디자인 토큰 기반으로 status/priority 컬러 매핑 가능

  

5. 🔜 Skeleton, LoadingSpinner, EmptyState

• UX 향상을 위한 시각적 컴포넌트

• Skeleton은 추후 카드나 페이지에서 재사용 가능

⸻
필요하면 DatePicker 커스터마이징(react-day-picker, headlessui, flatpickr 등)이나 Select를 <Listbox>로 바꾸는 것도 가능해요. 원하면 다음 단계로 이어서 도와줄게요.
  

💡 점점 확장하면 좋은 공통 컴포넌트들

3. Dropdown / Popover 메뉴

• ✅ 점 세개 눌렀을 때 메뉴 (수정, 삭제, 보류)

• <DropdownMenu> 컴포넌트로 만들고 children 방식으로 활용

• radix-ui 또는 headless-ui 사용 가능

  

4. Toast 알림

• ✅ “할 일이 등록되었습니다” 등 피드백용

• useToast() 훅 + ToastProvider, ToastItem 컴포넌트 구조 추천

• sonner 추천 (깔끔하고 Tailwind 호환 잘됨)

  

⸻

  

5. Toggle 토글 버튼

• ✅ 예: 알림 켜기/끄기, 체크박스 대체용 등

• Toggle.tsx 또는 Switch.tsx로 구현

• optional: checked, onToggle, disabled 등 prop 포함

  

⸻

  

6. Badge / Tag 컴포넌트

• ✅ “RETRY 1” 같은 태그 (색상/라벨)

• Badge.tsx에 variant 기반 스타일 지정 (ex: retry, success, neutral 등)

  

⸻

  

7. RadioGroup / Selector

• ✅ 우선순위 선택 (1 오늘 무조건, 2 오늘이면 굿, 3 잊지말자)

• RadioGroup.tsx 또는 PrioritySelector.tsx로 분리

• 디자인 시스템에 따라 icon + label 조합도 가능