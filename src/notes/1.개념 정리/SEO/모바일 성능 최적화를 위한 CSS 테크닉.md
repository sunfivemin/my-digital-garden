# 모바일 CSS 성능 최적화 실무 가이드

> 모바일에서 버벅거리는 웹사이트를 60fps로 부드럽게 만드는 방법

## 왜 모바일이 느릴까?

모바일 기기는 컴퓨터보다 성능이 떨어집니다. 특히 다음과 같은 상황에서 버벅거림이 발생해요:

- 스크롤할 때 애니메이션이 끊어짐
- 버튼을 누를 때 반응이 늦음
- 화면 전환이 뚝뚝 끊어짐

## 핵심 해결책 3가지

### 1. GPU 가속 사용하기

**문제**: 애니메이션이 CPU에서 처리되어 느림 **해결**: GPU에서 처리하도록 강제로 변경

```css
/* 이렇게 하면 GPU가 처리해서 빨라집니다 */
.smooth-element {
  transform: translateZ(0);        /* GPU 레이어 생성 */
  will-change: transform;          /* 브라우저에게 미리 알림 */
}
```

**언제 사용?**

- 애니메이션이 있는 모든 요소
- 스크롤할 때 움직이는 요소
- 호버 효과가 있는 버튼

### 2. 레이아웃 변경 방지하기

**문제**: 하나의 요소가 변경되면 전체 페이지가 다시 계산됨 **해결**: 영향 범위를 제한

```css
/* 이 요소 안의 변경사항이 밖으로 퍼지지 않음 */
.isolated-component {
  contain: layout;
}
```

**언제 사용?**

- 리스트 아이템들
- 카드 컴포넌트들
- 사이드바, 모달 등

### 3. 불필요한 최적화 제거하기

**문제**: 모든 요소에 최적화를 적용하면 오히려 느려짐 **해결**: 필요한 곳에만 적용하고, 끝나면 해제

```css
/* 모든 요소의 기본값을 리셋 */
* {
  will-change: auto;
}

/* 필요할 때만 활성화 */
.button:hover {
  will-change: transform;
}
```

## 실무 적용 가이드

### 버튼 최적화

```css
/* Before: 버벅거리는 버튼 */
.button {
  transition: all 0.3s ease;
}

/* After: 부드러운 버튼 */
.button {
  transform: translateZ(0);           /* GPU 가속 */
  transition: transform 0.2s ease;   /* transform만 애니메이션 */
}

.button:hover {
  will-change: transform;             /* 호버 시에만 최적화 */
  transform: translateZ(0) scale(1.05);
}

.button:not(:hover) {
  will-change: auto;                  /* 호버 끝나면 해제 */
}
```

### 모달/팝업 최적화

```css
/* 모달 배경 */
.modal-overlay {
  will-change: opacity;               /* 페이드 효과 최적화 */
  transition: opacity 0.3s ease;
}

/* 모달 내용 */
.modal-content {
  transform: translateZ(0);           /* GPU 가속 */
  contain: strict;                    /* 완전 격리 */
  will-change: transform, opacity;
}
```

### 리스트/카드 최적화

```css
/* 리스트 컨테이너 */
.card-list {
  contain: layout;                    /* 레이아웃 격리 */
}

/* 개별 카드 */
.card-item {
  will-change: auto;                  /* 기본은 최적화 없음 */
  transition: transform 0.2s ease;
}

.card-item:hover {
  will-change: transform;             /* 호버 시에만 최적화 */
  transform: translateY(-5px);
}
```

### 네비게이션 메뉴 최적화

```css
/* 햄버거 메뉴 */
.mobile-menu {
  transform: translateZ(0);           /* GPU 가속 */
  contain: strict;                    /* 완전 격리 */
}

/* 메뉴 애니메이션 */
.menu-item {
  will-change: auto;
}

.menu-item.animating {
  will-change: transform, opacity;    /* 애니메이션 중에만 */
}
```

## React/Vue에서 사용하기

### React 예시

```jsx
const AnimatedButton = ({ children, onClick }) => {
  const [isHovered, setIsHovered] = useState(false);
  
  return (
    <button 
      className={`btn ${isHovered ? 'optimized' : ''}`}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// CSS
.btn {
  transform: translateZ(0);
  transition: transform 0.2s ease;
}

.btn.optimized {
  will-change: transform;
}
```

### Vue 예시

```vue
<template>
  <div 
    class="card"
    :class="{ optimized: isAnimating }"
    @mouseenter="isAnimating = true"
    @mouseleave="isAnimating = false"
  >
    <slot />
  </div>
</template>

<script>
export default {
  data() {
    return {
      isAnimating: false
    }
  }
}
</script>

<style>
.card {
  transform: translateZ(0);
  contain: layout;
  transition: transform 0.2s ease;
}

.card.optimized {
  will-change: transform;
}
</style>
```

## 성능 개선 체크리스트

### ✅ 즉시 적용할 것들

1. **애니메이션 있는 모든 요소**
    
    ```css
    transform: translateZ(0);
    ```
    
2. **전역 will-change 리셋**
    
    ```css
    * { will-change: auto; }
    ```
    
3. **리스트/카드 컴포넌트**
    
    ```css
    contain: layout;
    ```
    

### ✅ 상황별 적용

- **호버 효과**: `will-change: transform` (호버 시에만)
- **페이드 효과**: `will-change: opacity` (애니메이션 시에만)
- **독립적인 컴포넌트**: `contain: strict`

### ✅ 성능 확인 방법

1. **Chrome 개발자 도구**
    
    - F12 → Performance 탭
    - 빨간 점 눌러서 기록 시작
    - 애니메이션 실행 후 기록 정지
    - FPS 그래프가 60에 가까우면 성공!
2. **실제 모바일에서 테스트**
    
    - 저사양 안드로이드 폰에서 확인
    - 부드럽게 움직이는지 직접 확인

## 주의사항

### ❌ 하지 말아야 할 것

```css
/* 모든 요소에 적용하면 오히려 느려짐 */
* {
  will-change: transform;
  transform: translateZ(0);
}

/* 너무 많은 속성을 한번에 애니메이션 */
transition: all 0.3s ease;
```

### ✅ 올바른 방법

```css
/* 필요한 요소에만 적용 */
.animated-element {
  will-change: transform;
  transform: translateZ(0);
}

/* 구체적인 속성만 애니메이션 */
transition: transform 0.2s ease, opacity 0.2s ease;
```

## 마무리

이 방법들을 적용하면:

- **모바일에서 60fps 부드러운 애니메이션**
- **배터리 수명 향상**
- **사용자 만족도 증가**

복잡해 보이지만 핵심은 3가지입니다:

1. GPU 가속 활용 (`transform: translateZ(0)`)
2. 영향 범위 제한 (`contain: layout`)
3. 필요할 때만 최적화 (`will-change: auto`)

작은 변경으로 큰 차이를 만들 수 있어요! 🚀