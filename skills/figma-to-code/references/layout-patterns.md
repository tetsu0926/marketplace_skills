# 레이아웃 패턴 참조

레이아웃 단계에서 Figma 디자인의 구조를 CSS로 변환할 때 참조하는 상세 가이드.

## Figma Auto Layout → CSS Flexbox

Figma의 Auto Layout은 CSS Flexbox와 직접 매핑된다.

### 방향 (Direction)

| Figma | CSS |
|-------|-----|
| Horizontal | `display: flex; flex-direction: row` |
| Vertical | `display: flex; flex-direction: column` |
| Wrap | `flex-wrap: wrap` |

### 정렬 (Alignment)

Figma의 정렬은 두 축으로 나뉜다:

**주축(Main Axis) 정렬 — `justify-content`:**

| Figma | CSS |
|-------|-----|
| Packed (start) | `justify-content: flex-start` |
| Packed (center) | `justify-content: center` |
| Packed (end) | `justify-content: flex-end` |
| Space between | `justify-content: space-between` |

**교차축(Cross Axis) 정렬 — `align-items`:**

| Figma | CSS |
|-------|-----|
| Top / Left | `align-items: flex-start` |
| Center | `align-items: center` |
| Bottom / Right | `align-items: flex-end` |
| Stretch | `align-items: stretch` |

### 간격 (Gap & Padding)

```css
/* Figma의 Item spacing → CSS gap */
gap: {itemSpacing}px;

/* Figma의 Padding → CSS padding */
padding: {paddingTop}px {paddingRight}px {paddingBottom}px {paddingLeft}px;
```

균일한 패딩이면 축약형을 사용:
- 4면 동일: `padding: 16px`
- 상하/좌우 동일: `padding: 16px 24px`

### 자식 요소 크기 (Child Sizing)

| Figma 설정 | CSS |
|-----------|-----|
| Fill container (주축) | `flex: 1` |
| Fill container (교차축) | `align-self: stretch` |
| Fixed width/height | `width: {value}px; height: {value}px` |
| Hug contents | `width: fit-content; height: fit-content` |
| Min width/height | `min-width: {value}px; min-height: {value}px` |
| Max width/height | `max-width: {value}px; max-height: {value}px` |

### 개별 자식 정렬

Figma에서 자식 요소의 `Align self`가 부모와 다르면:
```css
.child {
  align-self: center; /* 부모의 align-items를 오버라이드 */
}
```

## Figma Constraints → 반응형 동작

Auto Layout이 아닌 프레임에서 constraints를 사용하는 경우:

| Figma Constraint | CSS 변환 |
|-----------------|---------|
| Left | `position: absolute; left: {value}px` |
| Right | `position: absolute; right: {value}px` |
| Top | `position: absolute; top: {value}px` |
| Bottom | `position: absolute; bottom: {value}px` |
| Left & Right | `position: absolute; left: {L}px; right: {R}px` (너비 자동) |
| Top & Bottom | `position: absolute; top: {T}px; bottom: {B}px` (높이 자동) |
| Center (horizontal) | `position: absolute; left: 50%; transform: translateX(-50%)` |
| Center (vertical) | `position: absolute; top: 50%; transform: translateY(-50%)` |
| Scale | `width: {percentage}%; height: {percentage}%` |

> constraints 기반 레이아웃보다 Auto Layout(flex) 기반이 반응형에 더 적합하다. 가능하면 flex/grid로 변환한다.

## CSS Grid 활용

Figma의 Grid가 명시적이거나 카드 그리드 패턴이 보이면 CSS Grid를 사용한다:

```css
/* 카드 그리드 */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* 고정 컬럼 그리드 */
.fixed-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

/* 사이드바 + 콘텐츠 레이아웃 */
.layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 0;
  min-height: 100vh;
}
```

## 반응형 브레이크포인트 전략

### 모바일 퍼스트 (기본)

```css
/* 기본: 모바일 (320px~) */
.container { ... }

/* sm: 640px~ */
@media (min-width: 640px) { ... }

/* md: 768px~ */
@media (min-width: 768px) { ... }

/* lg: 1024px~ */
@media (min-width: 1024px) { ... }

/* xl: 1440px~ */
@media (min-width: 1440px) { ... }
```

### 반응형 변환 패턴

**수평 → 수직 전환:**
```css
.container {
  display: flex;
  flex-direction: column; /* 모바일: 세로 */
}
@media (min-width: 768px) {
  .container {
    flex-direction: row; /* 태블릿+: 가로 */
  }
}
```

**그리드 컬럼 조정:**
```css
.grid {
  display: grid;
  grid-template-columns: 1fr; /* 모바일: 1열 */
}
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr); /* 태블릿: 2열 */
  }
}
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr); /* 데스크톱: 3열 */
  }
}
```

**사이드바 숨김/표시:**
```css
.sidebar {
  display: none; /* 모바일: 숨김 */
}
@media (min-width: 1024px) {
  .sidebar {
    display: block;
    width: 240px;
  }
}
```

## 일반적인 레이아웃 패턴

### 네비게이션 바
```
[Logo] [------- Nav Links -------] [Actions]
→ display: flex; justify-content: space-between; align-items: center
```

### 히어로 섹션
```
[              Hero Content              ]
[    Heading + Description + CTA         ]
→ display: flex; flex-direction: column; align-items: center; text-align: center
```

### 카드 그리드
```
[Card] [Card] [Card]
[Card] [Card] [Card]
→ display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))
```

### 사이드바 레이아웃
```
[Sidebar] [    Main Content    ]
→ display: grid; grid-template-columns: 260px 1fr
```

### 풋터
```
[Col1] [Col2] [Col3] [Col4]
[--------- Copyright ---------]
→ 상단: grid 4열, 하단: flex center
```

## 시맨틱 HTML 구조

레이아웃에 적절한 시맨틱 태그를 사용한다:

| 영역 | HTML 태그 |
|------|----------|
| 전체 페이지 헤더 | `<header>` |
| 네비게이션 | `<nav>` |
| 메인 콘텐츠 | `<main>` |
| 사이드바 | `<aside>` |
| 섹션 구분 | `<section>` |
| 독립 콘텐츠 | `<article>` |
| 페이지 푸터 | `<footer>` |
| 그룹핑 | `<div>` (시맨틱 태그가 없을 때만) |

## 주의사항

- Figma의 절대 위치(`position: absolute`)가 많이 보이면, 이는 auto-layout을 사용하지 않은 디자인이다. 가능한 한 flex/grid로 재해석한다.
- Figma에서 텍스트의 `width: fixed`는 `max-width`로 변환하는 것이 반응형에 유리하다.
- 중첩된 auto-layout 프레임은 중첩된 flex 컨테이너로 변환한다.
- 스크롤 가능한 영역은 `overflow: auto` 또는 `overflow-y: auto`를 적용한다.
