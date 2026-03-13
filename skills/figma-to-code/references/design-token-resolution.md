# 디자인 토큰 해석 가이드

Figma MCP 출력에는 CSS 변수 형식의 디자인 토큰이 포함된다. 이 토큰을 프로덕션 코드에서 사용할 수 있도록 해석하고 변환하는 방법을 정리한다.

## MCP 출력의 토큰 형식

Figma MCP가 생성하는 CSS 변수 패턴:

```
var(--spacing\/space-24, 24px)          → spacing 토큰
var(--text\/body\/large\/size, 24px)    → typography 토큰
var(--color\/semantic\/text\/primary, #111827)  → color 토큰
var(--radius\/large\(card\), 12px)     → radius 토큰
var(--responsive\/side\/page-padding, 320px)   → responsive 토큰
```

**핵심: 슬래시(`\/`)와 이스케이프 문자(`\(`, `\)`)는 MCP 직렬화 아티팩트이다. 실제 CSS에서 사용할 수 없다.**

## 토큰 변환 전략

### 전략 1: CSS 변수로 정의 (권장)

모든 토큰을 하이픈 기반 CSS 변수로 변환하여 `:root`에 정의한다.

```css
:root {
  /* ========================
     Spacing Scale
     ======================== */
  --spacing-0: 0px;
  --spacing-4: 4px;
  --spacing-8: 8px;
  --spacing-12: 12px;
  --spacing-16: 16px;
  --spacing-20: 20px;
  --spacing-24: 24px;
  --spacing-32: 32px;
  --spacing-40: 40px;
  --spacing-48: 48px;
  --spacing-64: 64px;
  --spacing-80: 80px;
  --spacing-120: 120px;
  --spacing-200: 200px;

  /* ========================
     Colors - Semantic
     ======================== */
  /* Text */
  --color-text-primary: #111827;
  --color-text-secondary: #374151;
  --color-text-tertiary: #6b7280;
  --color-text-inverse: #ffffff;
  --color-text-inverse-alpha80: rgba(255, 255, 255, 0.8);
  --color-text-muted: #828282;

  /* Surface / Background */
  --color-surface-bg: #ffffff;
  --color-surface-secondary: #f9fafb;

  /* Border */
  --color-border-default: #e5e7eb;
  --color-border-subtle: #d1d5db;
  --color-border-focus: #0f4c5c;

  /* Brand / Action */
  --color-primary: #0f4c5c;
  --color-primary-hover: #374151;
  --color-info: #0f4c5c;

  /* ========================
     Typography Scale
     ======================== */
  /* Display */
  --text-display-d1-size: 52px;
  --text-display-d1-weight: 700;
  --text-display-d1-line: 78px;
  --text-display-d1-font: 'Roboto', 'Noto Sans KR', sans-serif;

  /* Heading */
  --text-heading-h1-size: 48px;
  --text-heading-h1-weight: 700;
  --text-heading-h1-line: 72px;
  --text-heading-h1-font: 'Noto Sans KR', sans-serif;

  --text-heading-h2-size: 36px;
  --text-heading-h2-weight: 700;
  --text-heading-h2-line: 48px;
  --text-heading-h2-font: 'Noto Sans KR', sans-serif;

  /* Title */
  --text-title-t1-size: 28px;
  --text-title-t1-weight: 700;
  --text-title-t1-line: 38px;

  --text-title-t2-size: 26px;
  --text-title-t2-weight: 700;
  --text-title-t2-line: 36px;

  --text-title-t3-size: 24px;
  --text-title-t3-weight: 700;
  --text-title-t3-line: 34px;

  /* Body */
  --text-body-large-size: 24px;
  --text-body-large-weight: 400;
  --text-body-large-line: 34px;

  --text-body-medium-size: 20px;
  --text-body-medium-weight: 400;
  --text-body-medium-line: 28px;

  --text-body-small-size: 16px;
  --text-body-small-weight: 400;
  --text-body-small-line: 24px;

  /* Label */
  --text-label-large-size: 15px;
  --text-label-large-weight: 500;
  --text-label-large-line: 21px;

  /* Button */
  --text-button-large-size: 22px;
  --text-button-large-weight: 500;
  --text-button-large-line: 20px;

  /* ========================
     Border Radius
     ======================== */
  --radius-small: 4px;       /* btn */
  --radius-large: 12px;      /* card */
  --radius-2xl: 48px;        /* large image */
  --radius-full: 9999px;     /* pill, circle */

  /* ========================
     Layout
     ======================== */
  --page-padding: 320px;
  --content-max-width: 1280px;
}

/* 반응형 토큰 오버라이드 */
@media (max-width: 1440px) {
  :root { --page-padding: 80px; }
}
@media (max-width: 1024px) {
  :root { --page-padding: 40px; }
}
@media (max-width: 768px) {
  :root { --page-padding: 20px; }
}
```

### 전략 2: 직접 값 사용

프로젝트가 간단하거나 CSS 변수 오버헤드가 불필요한 경우, fallback 값을 직접 사용한다.

```css
/* MCP 출력 */
padding: var(--spacing\/space-80, 80px);
/* 변환 결과 */
padding: 80px;
```

### 전략 3: Tailwind 매핑

Tailwind 프로젝트의 경우 `tailwind.config.js`에 토큰을 추가한다:

```js
module.exports = {
  theme: {
    extend: {
      spacing: {
        '80': '80px',
        '120': '120px',
        '200': '200px',
        'page': '320px',
      },
      colors: {
        'primary': '#0f4c5c',
        'text-primary': '#111827',
        'text-secondary': '#374151',
        'surface-secondary': '#f9fafb',
      },
      borderRadius: {
        'card': '12px',
        '2xl-custom': '48px',
      },
    },
  },
}
```

## 그라디언트 토큰 변환

### 텍스트 그라디언트 (gradient-text)

MCP 출력 (인라인 SVG radial gradient):
```jsx
style={{
  backgroundImage: "url('data:image/svg+xml;utf8,<svg>...<radialGradient>...')"
}}
className="bg-clip-text text-transparent"
```

CSS 변환:
```css
.gradient-text-teal {
  background: radial-gradient(
    ellipse at 20% 0%,
    #0f4c5c 50%,
    #1a6977 62.5%,
    #268691 75%,
    #3cc0c7 100%
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

### 프로그레스 바 그라디언트

MCP 출력 (인라인 SVG with 10+ 색상 스톱):
```jsx
style={{
  backgroundImage: "url('data:image/svg+xml;utf8,<svg>...<radialGradient>...')"
}}
```

CSS 변환:
```css
.gradient-bar {
  background: linear-gradient(
    90deg,
    #a258a2 0%,
    #8159a5 13%,
    #6059a7 26%,
    #445da5 39.5%,
    #2860a3 53%,
    #147fc1 66%,
    #0a8fcf 72.5%,
    #009ede 79%,
    #12a6cc 84.25%,
    #23adba 89.5%,
    #46bc96 100%
  );
  border-radius: 9999px;
  height: 12px;
}
```

### 섹션 배경 그라디언트

```css
/* 투명 → 색상 (위에서 아래) */
.section-fade-bg {
  background: linear-gradient(to bottom, transparent, #f9fafb);
}

/* 어두운 오버레이 (히어로 섹션) */
.hero-overlay {
  background: linear-gradient(90deg, rgba(15, 76, 92, 0.9) 0%, rgba(60, 192, 199, 0.6) 100%);
}
```

## MCP 토큰 → CSS 변수 매핑 테이블

| MCP 토큰 패턴 | CSS 변수 | 값 |
|---|---|---|
| `--spacing\/space-{n}` | `--spacing-{n}` | `{n}px` |
| `--text\/heading\/h1\/size` | `--text-heading-h1-size` | `48px` |
| `--text\/heading\/h1\/weight` | `--text-heading-h1-weight` | `700` |
| `--text\/heading\/h1\/line` | `--text-heading-h1-line` | `72px` |
| `--color\/semantic\/text\/primary` | `--color-text-primary` | `#111827` |
| `--color\/semantic\/surface\/bg` | `--color-surface-bg` | `#ffffff` |
| `--color\/semantic\/border\/default` | `--color-border-default` | `#e5e7eb` |
| `--color\/base\/brand\/primary` | `--color-primary` | `#0f4c5c` |
| `--radius\/large\(card\)` | `--radius-large` | `12px` |
| `--radius\/full` | `--radius-full` | `9999px` |
| `--responsive\/side\/page-padding` | `--page-padding` | `320px` |

## 폰트 매핑

MCP 출력의 font-family 패턴과 실제 CSS 매핑:

| MCP font-family | CSS font-family | Google Fonts weight |
|---|---|---|
| `'Noto_Sans_KR:Regular'` | `'Noto Sans KR', sans-serif` | `400` |
| `'Noto_Sans_KR:Medium'` | `'Noto Sans KR', sans-serif` | `500` |
| `'Noto_Sans_KR:Bold'` | `'Noto Sans KR', sans-serif` | `700` |
| `'Roboto:Regular'` | `'Roboto', sans-serif` | `400` |
| `'Roboto:Medium'` | `'Roboto', sans-serif` | `500` |
| `'Roboto:Bold'` | `'Roboto', sans-serif` | `700` |

Google Fonts 로드 태그:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
```
