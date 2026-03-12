# HTML 변환 규칙

Figma 디자인을 순수 HTML + CSS/JS로 변환할 때의 가이드.

## 기본 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>페이지 제목</title>
  <!-- 웹폰트 -->
  <link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
  <!-- CSS (방식에 따라 다름) -->
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>

  <!-- JS (필요 시) -->
  <script src="script.js"></script>
</body>
</html>
```

## Figma 구조 → HTML 매핑

| Figma 요소 | HTML 태그 | 선택 기준 |
|-----------|----------|----------|
| Frame (페이지 헤더) | `<header>` | 페이지 상단 영역 |
| Frame (네비게이션) | `<nav>` | 네비게이션 링크 그룹 |
| Frame (메인 콘텐츠) | `<main>` | 주요 콘텐츠 영역 |
| Frame (사이드바) | `<aside>` | 보조 콘텐츠 |
| Frame (섹션) | `<section>` | 주제별 콘텐츠 블록 |
| Frame (독립 콘텐츠) | `<article>` | 카드, 게시물 등 |
| Frame (하단) | `<footer>` | 페이지 하단 |
| Frame (일반 그룹) | `<div>` | 위에 해당하지 않을 때 |
| Text (제목) | `<h1>`~`<h6>` | 제목 계층에 따라 |
| Text (본문) | `<p>` | 문단 텍스트 |
| Text (인라인) | `<span>` | 인라인 텍스트 |
| Text (링크) | `<a href="...">` | 클릭 가능한 텍스트 |
| Rectangle | `<div>` | CSS로 스타일 |
| Image | `<img src="..." alt="...">` | 이미지 |
| Button | `<button>` | 액션 버튼 |
| Input | `<input type="...">` | 입력 필드 |
| Icon (SVG) | `<svg>` 또는 `<img>` | 아이콘 |
| 리스트 | `<ul>/<ol>` + `<li>` | 반복 아이템 |

## 시맨틱 구조 예시

### 네비게이션 바
```html
<header class="header">
  <nav class="nav">
    <a href="/" class="nav-logo">
      <img src="logo.svg" alt="로고">
    </a>
    <ul class="nav-links">
      <li><a href="/about">소개</a></li>
      <li><a href="/services">서비스</a></li>
      <li><a href="/contact">문의</a></li>
    </ul>
    <div class="nav-actions">
      <button class="btn btn-primary">시작하기</button>
    </div>
  </nav>
</header>
```

### 카드 그리드
```html
<section class="card-section">
  <h2 class="section-title">서비스</h2>
  <div class="card-grid">
    <article class="card">
      <img src="card1.jpg" alt="서비스 1" class="card-image">
      <div class="card-content">
        <h3 class="card-title">서비스 제목</h3>
        <p class="card-description">서비스 설명...</p>
      </div>
    </article>
    <!-- 반복 -->
  </div>
</section>
```

### 폼
```html
<form class="form">
  <div class="form-group">
    <label for="email" class="form-label">이메일</label>
    <input type="email" id="email" class="form-input" placeholder="이메일 입력">
  </div>
  <div class="form-group">
    <label for="message" class="form-label">메시지</label>
    <textarea id="message" class="form-textarea" rows="4"></textarea>
  </div>
  <button type="submit" class="btn btn-primary">전송</button>
</form>
```

## CSS 클래스 네이밍

BEM(Block Element Modifier) 패턴 권장:

```css
/* Block */
.card { }

/* Element */
.card__title { }
.card__image { }
.card__content { }

/* Modifier */
.card--featured { }
.card--compact { }
```

또는 간단한 하이픈 네이밍:

```css
.card { }
.card-title { }
.card-image { }
.card-content { }
.card-featured { }
```

프로젝트에 기존 네이밍 규칙이 있으면 그것을 따른다.

## 반복 요소 처리

HTML에서는 동적 렌더링이 없으므로, Figma에서 반복되는 요소를 직접 마크업:

```html
<!-- 3개의 카드가 반복되는 경우 -->
<div class="card-grid">
  <article class="card">...</article>
  <article class="card">...</article>
  <article class="card">...</article>
</div>
```

실제 데이터가 필요하면 JS로 동적 생성하거나 사용자에게 안내한다.

## JavaScript (필요 시)

인터랙션이 필요한 경우 최소한의 JS를 추가:

```html
<script>
// 모바일 메뉴 토글
const menuToggle = document.querySelector('.menu-toggle');
const navLinks = document.querySelector('.nav-links');
menuToggle?.addEventListener('click', () => {
  navLinks?.classList.toggle('active');
});

// 탭 전환
document.querySelectorAll('.tab-button').forEach(button => {
  button.addEventListener('click', () => {
    // 탭 전환 로직
  });
});
</script>
```

## 파일 구조

```
project/
├── index.html          # 메인 HTML
├── styles.css          # 메인 CSS (또는 css/ 디렉토리)
├── script.js           # JS (필요 시)
└── assets/
    ├── images/         # 이미지 파일
    └── icons/          # SVG 아이콘
```

## 접근성

- 모든 `img`에 `alt` 속성
- 제목 태그 순서: `h1` → `h2` → `h3` (건너뛰지 않기)
- 버튼은 `<button>`, 링크는 `<a>` (역할 혼용 금지)
- 폼 요소에 `<label>` 연결
- `tabindex` 순서가 자연스러운지 확인
- 충분한 색상 대비 (WCAG AA 기준: 4.5:1)
