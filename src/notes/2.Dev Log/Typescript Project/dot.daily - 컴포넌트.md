
# 🗓 dot.daily — 모달 시스템 설계와 사용법

## 1. 왜 모달 시스템을 따로 설계해야 할까?

모달(Modal)은 사용자 경험에서 매우 중요한 UI 컴포넌트입니다.
단순한 팝업부터, 바텀시트, 풀스크린, 카드형 등 다양한 형태가 필요하고,
전역적으로 관리되어야 하며,
- 딤드(overlay) 처리
- 스크롤 락
- 포탈(Portal) 렌더링
- 여러 종류의 모달을 한 번에 관리
등의 요구사항이 있습니다.

## 2. Provider란? (React Context의 관점에서)
Provider는 React의 Context API에서 "하위 트리 전체에 어떤 값(상태, 함수 등)을 공급하는 컴포넌트"입니다.
```tsx
  <MyContext.Provider value={...}>
    <App />
  </MyContext.Provider>
```

- 하위의 모든 컴포넌트는 useContext로 이 값을 꺼내 쓸 수 있습니다.
- 모달 시스템에서 Provider를 쓰는 이유: 어디서든(심지어 깊은 자식 컴포넌트에서도)모달을 열고 닫을 수 있게 하려면 전역 상태/함수를 Context로 공급해야 합니다.

---

## 3. DOT.DAILY의 모달 시스템 구조
### 3-1. 폴더 구조
```bash
src/components/ui/Modal/
  components/
    BottomSheetModal.tsx
    FullScreenModal.tsx
    ModalItem.tsx
  providers/
    FullScreenModalProvider.tsx
    ModalProvider.tsx
```

- FullScreenModalProvider: 전역 모달 상태/함수 관리, 실제 모달 렌더링
- BottomSheetModal, FullScreenModal, ModalItem: 각각 바텀시트, 풀스크린, 카드형 등 다양한 모달 UI

### 3-2. FullScreenModalProvider의 역할
- Context 생성: 현재 열려있는 모달 이름, props, 열기/닫기 함수 제공
- Provider: 앱 루트에 감싸서 하위 어디서든 모달을 열 수 있게 함
- Renderer: 실제로 어떤 모달이 열릴지 조건부로 렌더링

```tsx
// _app.tsx 또는 layout.tsx
<FullScreenModalProvider>
  <App />
</FullScreenModalProvider>
```

### 3-3. 모달 열고 닫기 (useFullScreenModal)
```tsx
import { useFullScreenModal } from '@/components/ui/Modal/providers/FullScreenModalProvider';

const { openModal, closeModal } = useFullScreenModal();

// 모달 열기
openModal('taskForm', { defaultDate: '2025-06-20' });

// 모달 닫기
closeModal();
```
- 'taskForm', 'retrospectForm', 'dateNavigationForm' 등 등록된 모달 이름을 사용

### 3-4. 실제 모달 컴포넌트 구조

#### 1) FullScreenModal
- props: open, onClose, children, variant
- variant: 'full', 'card', 'bottomSheet' 등
- 딤드/카드/바텀시트 등 다양한 형태 지원
```tsx
<FullScreenModal open={true} onClose={closeModal} variant="card">
  <TaskFormModal onClose={closeModal} />
</FullScreenModal>
```

#### 2) BottomSheetModal
- props: open, onClose, children
- 바텀시트 UX, 딤드, 드래그바 등 구현
```tsx
<BottomSheetModal open={true} onClose={closeModal}>
  <DateNavigationModal onClose={closeModal} />
</BottomSheetModal>
```

#### 3) ModalItem (간단한 카드형 모달)
```tsx
<ModalItem open={open} onClose={closeModal}>
  <div>간단한 내용</div>
</ModalItem>
```

### 3-5. 모달 렌더링 구조 (Renderer)
FullScreenModalProvider 내부에서 현재 열려있는 모달 이름에 따라 적절한 모달 컴포넌트를 렌더링합니다.
```tsx
const FullScreenModalRenderer = () => {
  const { modalName, modalProps, closeModal } = useFullScreenModal();

  return (
    <AnimatePresence>
      {modalName === 'taskForm' && (
        <FullScreenModal open={true} onClose={closeModal} variant="card">
          <TaskFormModal onClose={closeModal} {...modalProps} />
        </FullScreenModal>
      )}
      {modalName === 'retrospectForm' && (
        <FullScreenModal open={true} onClose={closeModal} variant="card">
          <RetrospectModal onClose={closeModal} />
        </FullScreenModal>
      )}
      {modalName === 'dateNavigationForm' && (
        <BottomSheetModal open={true} onClose={closeModal}>
          <DateNavigationModal onClose={closeModal} />
        </BottomSheetModal>
      )}
    </AnimatePresence>
  );
};
```

## 4. 모달 시스템 사용법 요약

1. Provider로 앱을 감싼다
2. useFullScreenModal()로 어디서든 모달을 열고 닫는다
3. 모달 컴포넌트는 props로 onClose를 받아 닫기 처리
4. 새로운 모달을 추가하려면 Provider의 Renderer에 등록

## 5. 실전 예시: 할 일 등록 모달 열기
```tsx
import { useFullScreenModal } from '@/components/ui/Modal/providers/FullScreenModalProvider';

function MyComponent() {
  const { openModal } = useFullScreenModal();

  return (
    <button onClick={() => openModal('taskForm', { defaultDate: '2025-06-20' })}>
      할 일 등록
    </button>
  );
}
```

---

React에서 "포탈(Portal)"은 모달, 드롭다운, 토스트 등 "화면의 특정 DOM 계층 밖에" UI를 렌더링해야 할 때 사용하는 기술입니다.

## 6. 포탈(Portal)이란?

- React Portal은 "컴포넌트 트리의 논리적 위치와는 별개로, 실제 DOM의 원하는 위치(보통 body 바로 아래)에 React 컴포넌트를 렌더링하는 방법"입니다.
```tsx
import { createPortal } from 'react-dom';

function MyModal({ children }) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.body // body 바로 아래에 렌더링
  );
}
```
### 6-1 왜 포탈이 필요한가?

### 1) z-index, overflow, position 문제 해결
- 모달/드롭다운/토스트 등은 부모 컴포넌트의 overflow, z-index, position에 영향을 받지 않고 항상 화면 맨 위에 떠야 합니다.
- 포탈을 쓰면, 부모의 레이아웃/스크롤/클리핑에 상관없이 body(혹은 원하는 DOM) 바로 아래에 렌더링되어 레이어가 깨지지 않습니다.

### 2) 접근성(Accessibility)
- 포커스 트랩, aria-modal 등 접근성 구현이 쉬워집니다.

---

### 6-2. 언제 포탈을 써야 할까?
- 모달(Modal)
- 드롭다운/팝오버
- 토스트(Toast)
- 툴팁(Tooltip)
- 즉, "항상 화면 위에 떠야 하는 UI"는 대부분 포탈로 렌더링합니다.

---

### 6-3. 실제 사용 예시

```tsx
import { createPortal } from 'react-dom';

function Modal({ open, children }) {
  if (!open) return null;
  return createPortal(
    <div className="fixed inset-0 z-50 bg-black/40">{children}</div>,
    document.body
  );
}
```
- 이렇게 하면, Modal 컴포넌트가 어디서 호출되든 실제로는 body 바로 아래에 렌더링됩니다.
- 현재는 대부분의 모달이 body 기준(fixed 등)으로 렌더링되어 포탈을 직접 사용하지 않아도 정상 동작합니다.
- 하지만, 복잡한 레이아웃/중첩/스크롤/레이어링 이슈가 생기면 포탈로 분리하는 것이 더 안전합니다.


## 6. 마치며

- Provider 패턴을 활용하면, 어디서든 모달을 열고 닫을 수 있고, 다양한 모달 형태(풀스크린, 바텀시트, 카드 등)를 일관성 있게 관리할 수 있습니다.
- 새로운 모달을 추가할 때는 Renderer에만 등록하면 되므로 확장성도 뛰어납니다.
- 딤드, 스크롤락, 포탈 등 모달의 필수 UX도 Provider 내부에서 일괄 관리할 수 있습니다.