
## ✅ 목표
- 디자인 시스템에 맞는 재사용 가능한 UI 컴포넌트를 구축
- CVA (Class Variance Authority)를 활용해 Tailwind 기반의 유연한 스타일 구성
- **Storybook**으로 문서화 및 UI 테스트 가능하게 구성
- **Zustand**를 통해 UI 상태 관리 통합 (예: 모달, 토스트 등)

---

## 🔗 컴포넌트 실사용 예시

이 글에서 소개한 Modal, Input, DatePicker, SelectInput, Toast 등의 UI 컴포넌트는 실제로 프로젝트에 구현되어 있으며, 아래 GitHub 저장소에서 확인하실 수 있습니다:
👉 [oneday-coding/DOT-DAILY - components/ui](https://github.com/oneday-coding/DOT-DAILY/tree/main/frontend/src/components/ui)

- 컴포넌트 단위로 분리된 구조
- CVA(Class Variance Authority) 기반의 variant 설계
- Storybook으로 문서화된 UI 구성
- Zustand를 활용한 전역 UI 상태관리 (Modal, Toast 등)

---

## 1. Modal 모달창 설계

### ✅ 사용 예시
- 할 일 등록 모달
- 회고 작성/수정 모달
- 삭제 확인(confirm) 모달

### 📦 구성
```tsx
<Modal isOpen={open} onClose={() => setOpen(false)}>
  <h2>할 일 등록</h2>
  <Input label="제목" />
  <SelectInput label="우선순위" />
  <DatePicker label="날짜" />
</Modal>
```

- **Modal.tsx**: isOpen, onClose, children props
- **내부 구현**: Dialog, Transition (headlessui) 또는 z-index 기반 커스텀 가능
- **상태 제어**: useUIStore().openModal('task')

---

## 2. Input 인풋 계열

### 🔧 공통 구조
```bash
components/ui/Input/
├── Input.tsx            // 기본 인풋 (스타일)
├── TextInput.tsx        // 텍스트 필드
├── TextareaInput.tsx    // 멀티라인 입력
├── DateInput.tsx        // 날짜 선택
├── SelectInput.tsx      // 셀렉트 필드
├── Input.stories.tsx    // 📕 Storybook 문서
└── index.ts             // barrel export
```

### 🗓 DatePicker.tsx
- react-day-picker 기반
- floating label, 포맷 변경 등 커스터마이징 가능

---

## 3. Zustand 상태 관리 통합

Toast, Modal 등 UI 상태는 컴포넌트 간 공유되기 때문에 전역 관리가 필요합니다.
```ts
export const useUIStore = create(set => ({
  modal: null,
  openModal: (name: string) => set({ modal: name }),
  closeModal: () => set({ modal: null }),
  toast: { message: '', type: 'success' },
  showToast: (msg, type) => set({ toast: { message: msg, type } }),
}));
```

- useUIStore().openModal('edit')
- useUIStore().showToast('저장 완료', 'success')

---

## 4. Storybook 문서화 전략

각 컴포넌트를 독립적으로 문서화하여 아래 항목 포함:
- 다양한 variant
- 상태 예시 (disabled, error 등)
- interaction 테스트 가능

```ts
export default {
  title: 'Components/Input',
  component: TextInput,
};

export const Basic = () => (
  <TextInput label="할 일 제목" placeholder="입력하세요" />
);
```

---

## 5. 앞으로 확장할 공통 컴포넌트들

### ✅ DropdownMenu
- 예: 점 세 개 메뉴 (수정, 삭제)
- headlessui or radix-ui
  
### ✅ Toast 알림
- useToast() 훅 + ToastProvider 구조
- sonner 라이브러리 추천


### ✅ Badge / Tag
- 예: 상태 뱃지 (RETRY, SUCCESS, NEUTRAL)
- variant 기반 색상 매핑
  

### ✅ Toggle / Switch
- 체크박스 대체용 스위치 컴포넌트
- on, disabled, label 포함

---

## 🧩 디자인 시스템 설계 철학
- **재사용성과 일관성** 중심
- **CVA**로 variant 분기 관리
- **Zustand**로 UI 상태 전역 관리
- **Storybook**으로 디자인 협업 및 테스트 용이
