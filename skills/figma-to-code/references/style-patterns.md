# 스타일 패턴 참조

스타일 단계에서 Figma 디자인의 모든 비주얼 속성을 CSS로 변환할 때 참조하는 상세 가이드.

## 타이포그래피

### Figma → CSS 매핑

| Figma 속성 | CSS 속성 | 변환 규칙 |
|-----------|---------|----------|
| Font family | `font-family` | 웹폰트 로드 필요 시 Google Fonts 또는 프로젝트 폰트 사용 |
| Font size | `font-size` | `{value}px` 또는 `{value/16}rem` |
| Font weight | `font-weight` | Regular=400, Medium=500, SemiBold=600, Bold=700 |
| Line height | `line-height` | `{value}px` 또는 비율 `{value/fontSize}` |
| Letter spacing | `letter-spacing` | `{value}px` 또는 `{value/fontSize}em` |
| Text align | `text-align` | left, center, right, justify |
| Text decoration | `text-decoration` | underline, line-through, none |
| Text case | `text-transform` | uppercase, lowercase, capitalize |
| Paragraph spacing | `margin-bottom` | 문단 간 간격 |

### Font Weight 매핑

| Figma 이름 | CSS font-weight |
|-----------|----------------|
| Thin | 100 |
| Extra Light / UltraLight | 200 |
| Light | 300 |
| Regular / Normal | 400 |
| Medium | 500 |
| Semi Bold / Demi Bold | 600 |
| Bold | 700 |
| Extra Bold / Ultra Bold | 800 |
| Black / Heavy | 900 |

### 웹폰트 로드

Figma에서 사용한 폰트가 웹폰트로 사용 가능하면 로드한다:

```html
<!-- Google Fonts 예시 -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

프로젝트에 이미 폰트가 설정되어 있으면 그것을 우선 사용한다.

## 색상

### Figma Fill → CSS 배경

| Figma Fill 타입 | CSS 변환 |
|----------------|---------|
| Solid color | `background-color: #{hex}` 또는 `rgba(r, g, b, a)` |
| Linear gradient | `background: linear-gradient({angle}deg, #{color1} {stop1}%, #{color2} {stop2}%)` |
| Radial gradient | `background: radial-gradient(circle at {x}% {y}%, #{color1}, #{color2})` |
| Image fill | `background-image: url(...)` + `background-size: cover/contain` |

### Figma 색상 투명도

Figma에서 fill의 opacity가 100%가 아니면:
```css
/* opacity가 fill에 적용된 경우 → rgba 사용 */
background-color: rgba(0, 0, 0, 0.5);

/* opacity가 레이어에 적용된 경우 → opacity 속성 사용 */
opacity: 0.5;
```

이 구분이 중요하다 — fill opacity와 layer opacity는 다르게 동작한다.

### 그라디언트 변환

Figma의 그라디언트 각도를 CSS로 변환:

```
Figma 그라디언트 핸들 위치 → CSS angle 계산
- 위→아래: 180deg
- 좌→우: 90deg
- 대각선: atan2 계산
```

```css
/* Figma: Linear gradient, 2 stops */
background: linear-gradient(180deg, #FF5733 0%, #33FF57 100%);

/* Figma: Multi-stop gradient */
background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
```

### 디자인 토큰 매핑

`get_variable_defs`로 가져온 토큰을 CSS 변수로 변환:

```css
:root {
  --color-primary: #3B82F6;
  --color-primary-hover: #2563EB;
  --color-secondary: #10B981;
  --color-text-primary: #111827;
  --color-text-secondary: #6B7280;
  --color-bg-primary: #FFFFFF;
  --color-bg-secondary: #F3F4F6;
  --color-border: #E5E7EB;
}
```

토큰이 있으면 하드코딩 대신 변수를 사용한다: `color: var(--color-text-primary)`

## 효과 (Effects)

### 그림자 (Shadow)

| Figma Effect | CSS 변환 |
|-------------|---------|
| Drop Shadow | `box-shadow: {x}px {y}px {blur}px {spread}px rgba(r,g,b,a)` |
| Inner Shadow | `box-shadow: inset {x}px {y}px {blur}px {spread}px rgba(r,g,b,a)` |
| 다중 그림자 | 쉼표로 구분하여 나열 |

```css
/* 단일 그림자 */
box-shadow: 0px 4px 6px -1px rgba(0, 0, 0, 0.1);

/* 다중 그림자 (Figma에서 여러 shadow 적용 시) */
box-shadow:
  0px 4px 6px -1px rgba(0, 0, 0, 0.1),
  0px 2px 4px -2px rgba(0, 0, 0, 0.1);

/* Inner Shadow */
box-shadow: inset 0px 2px 4px rgba(0, 0, 0, 0.06);
```

### 블러 (Blur)

| Figma Effect | CSS 변환 |
|-------------|---------|
| Layer blur | `filter: blur({value}px)` |
| Background blur | `backdrop-filter: blur({value}px)` |

```css
/* Glassmorphism 패턴 */
.glass {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px); /* Safari 지원 */
}
```

### Border Radius

```css
/* 균일한 radius */
border-radius: 8px;

/* 개별 모서리 */
border-radius: {topLeft}px {topRight}px {bottomRight}px {bottomLeft}px;

/* 원형 (Figma에서 정원의 radius가 너비/2일 때) */
border-radius: 50%;

/* 캡슐형 (Figma에서 radius가 높이/2일 때) */
border-radius: 9999px;
```

## 보더 (Stroke)

### Figma Stroke → CSS Border

| Figma 속성 | CSS 변환 |
|-----------|---------|
| Stroke color | `border-color: #{hex}` |
| Stroke weight | `border-width: {value}px` |
| Stroke align: Inside | `border: {weight}px solid {color}` (+ `box-sizing: border-box`) |
| Stroke align: Outside | `outline: {weight}px solid {color}` 또는 `box-shadow: 0 0 0 {weight}px {color}` |
| Stroke align: Center | `border: {weight}px solid {color}` |
| Stroke per side | `border-top/right/bottom/left: ...` |
| Dashed stroke | `border-style: dashed` |

## 인터랙션 상태

Figma에 variant나 interactive component가 있으면 해당 상태를 CSS로 변환한다:

```css
.button {
  background-color: var(--color-primary);
  transition: all 0.2s ease;
  cursor: pointer;
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

Figma에 상태별 variant가 없어도, 인터랙티브 요소(버튼, 링크, 입력 필드)에는 기본적인 hover/focus 상태를 추가한다.

## Opacity & Blend Mode

```css
/* Layer opacity */
opacity: 0.8;

/* Figma Blend Mode → CSS mix-blend-mode */
mix-blend-mode: multiply;    /* Multiply */
mix-blend-mode: screen;      /* Screen */
mix-blend-mode: overlay;     /* Overlay */
mix-blend-mode: darken;      /* Darken */
mix-blend-mode: lighten;     /* Lighten */
```

## 이미지 스타일

```css
/* Figma의 Image Fill 모드 */
.image-fill {
  background-size: cover;     /* Fill */
  background-position: center;
}

.image-fit {
  background-size: contain;   /* Fit */
  background-repeat: no-repeat;
  background-position: center;
}

/* img 태그 사용 시 */
img {
  width: 100%;
  height: 100%;
  object-fit: cover;          /* Fill */
  /* 또는 */
  object-fit: contain;        /* Fit */
}
```

## 스크롤바 스타일

Figma에서 스크롤 가능한 영역이 커스텀 스크롤바를 사용하면:

```css
.scrollable::-webkit-scrollbar {
  width: 6px;
}
.scrollable::-webkit-scrollbar-track {
  background: transparent;
}
.scrollable::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}
```

## 주의사항

- Figma의 색상은 sRGB 색공간이다. CSS에서도 동일하게 사용한다.
- Figma에서 `opacity`가 fill/stroke에 적용된 것과 레이어에 적용된 것을 구분한다.
- 다중 fill이 있으면 CSS에서도 다중 배경으로 표현한다.
- Figma의 `Auto` line-height는 보통 `1.2` (font-size × 1.2)이다 — 정확한 값은 디자인 컨텍스트에서 확인한다.
- 텍스트의 `Resizing: Auto height`는 CSS에서 높이를 지정하지 않으면 된다 (기본 동작).
