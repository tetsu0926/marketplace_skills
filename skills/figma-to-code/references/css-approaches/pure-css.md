# Pure CSS 매핑 규칙

Figma 디자인 속성을 순수 CSS로 변환할 때의 가이드.

## CSS 파일 구조

```css
/* ===== CSS Variables (Design Tokens) ===== */
:root {
  /* Colors */
  --color-primary: #3B82F6;
  --color-primary-hover: #2563EB;
  --color-secondary: #10B981;
  --color-text: #111827;
  --color-text-secondary: #6B7280;
  --color-bg: #FFFFFF;
  --color-bg-secondary: #F3F4F6;
  --color-border: #E5E7EB;

  /* Typography */
  --font-family: 'Inter', sans-serif;
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 1.875rem;  /* 30px */
  --font-size-4xl: 2.25rem;   /* 36px */

  /* Spacing */
  --spacing-1: 0.25rem;  /* 4px */
  --spacing-2: 0.5rem;   /* 8px */
  --spacing-3: 0.75rem;  /* 12px */
  --spacing-4: 1rem;     /* 16px */
  --spacing-5: 1.25rem;  /* 20px */
  --spacing-6: 1.5rem;   /* 24px */
  --spacing-8: 2rem;     /* 32px */
  --spacing-10: 2.5rem;  /* 40px */
  --spacing-12: 3rem;    /* 48px */
  --spacing-16: 4rem;    /* 64px */

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);

  /* Transitions */
  --transition-fast: 150ms ease;
  --transition-normal: 200ms ease;
  --transition-slow: 300ms ease;
}

/* ===== Reset ===== */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  color: var(--color-text);
  background-color: var(--color-bg);
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

img {
  max-width: 100%;
  display: block;
}

/* ===== Layout ===== */
/* 여기에 레이아웃 CSS */

/* ===== Components ===== */
/* 여기에 컴포넌트 CSS */

/* ===== Responsive ===== */
/* 여기에 반응형 미디어 쿼리 */
```

## Figma 속성 → CSS 직접 매핑

### 레이아웃

```css
/* Figma Auto Layout (Horizontal, gap: 16, padding: 24) */
.container {
  display: flex;
  flex-direction: row;
  gap: 16px;
  padding: 24px;
}

/* Figma Auto Layout (Vertical, align: center) */
.stack {
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Fill container */
.fill {
  flex: 1;
}

/* Hug contents */
.hug {
  width: fit-content;
}
```

### 타이포그래피

```css
/* Figma Text Style 변환 */
.heading-1 {
  font-family: var(--font-family);
  font-size: var(--font-size-4xl);
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: -0.02em;
  color: var(--color-text);
}

.body-text {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  font-weight: 400;
  line-height: 1.5;
  color: var(--color-text-secondary);
}
```

### 색상 & 배경

```css
/* Solid color */
.element {
  background-color: var(--color-primary);
  color: #FFFFFF;
}

/* Gradient */
.gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

/* Opacity */
.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}
```

### 효과

```css
/* Shadow */
.card {
  box-shadow: var(--shadow-md);
}

/* Multiple shadows */
.elevated {
  box-shadow:
    0 4px 6px -1px rgba(0, 0, 0, 0.1),
    0 2px 4px -2px rgba(0, 0, 0, 0.1);
}

/* Blur */
.blurred {
  filter: blur(8px);
}

.glass {
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  background: rgba(255, 255, 255, 0.1);
}

/* Border radius */
.rounded {
  border-radius: var(--radius-md);
}
```

### 보더

```css
/* Figma Stroke */
.bordered {
  border: 1px solid var(--color-border);
}

/* 특정 면만 */
.bottom-border {
  border-bottom: 1px solid var(--color-border);
}

/* Dashed */
.dashed {
  border: 2px dashed var(--color-border);
}
```

## 인터랙션 상태

```css
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-2) var(--spacing-4);
  background-color: var(--color-primary);
  color: #FFFFFF;
  border: none;
  border-radius: var(--radius-md);
  font-size: var(--font-size-sm);
  font-weight: 500;
  cursor: pointer;
  transition: all var(--transition-normal);
}

.button:hover {
  background-color: var(--color-primary-hover);
}

.button:active {
  transform: scale(0.98);
}

.button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

## 반응형 미디어 쿼리

```css
/* 모바일 퍼스트 */

/* 모바일 (기본) */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: var(--spacing-4);
}

.sidebar {
  display: none;
}

/* 태블릿 (768px+) */
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
    gap: var(--spacing-6);
  }
}

/* 데스크톱 (1024px+) */
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .sidebar {
    display: block;
    width: 240px;
  }

  .layout {
    display: grid;
    grid-template-columns: 240px 1fr;
  }
}

/* 와이드 (1440px+) */
@media (min-width: 1440px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

## 디자인 토큰 활용

`get_variable_defs`에서 가져온 Figma 토큰을 CSS 변수로 매핑:

```css
/* Figma variable: color/primary → CSS variable */
:root {
  --color-primary: #3B82F6;       /* Figma: color/primary */
  --color-primary-hover: #2563EB; /* Figma: color/primary-hover */
  --spacing-sm: 8px;              /* Figma: spacing/sm */
  --spacing-md: 16px;             /* Figma: spacing/md */
  --radius-default: 8px;          /* Figma: radius/default */
}
```

토큰이 있으면 하드코딩 대신 변수를 사용한다. 일관성과 유지보수성이 향상된다.

## 주의사항

- CSS 변수(`:root`)에 디자인 토큰을 정리하면 나중에 테마 변경이 쉬워진다
- Reset CSS를 포함하여 브라우저 기본 스타일 차이를 제거한다
- `-webkit-` 프리픽스가 필요한 속성: `backdrop-filter`, `-webkit-font-smoothing`
- `box-sizing: border-box`를 전역 적용하여 Figma의 크기 계산과 일치시킨다
- CSS 파일이 길어지면 섹션 주석으로 구분한다
