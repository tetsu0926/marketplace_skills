---
name: figma-to-code
description: Figma 디자인을 프로덕션 코드로 변환한다. 피그마 URL을 받아 React, Vue, Next.js, Svelte, Angular, HTML 코드를 생성하며 Tailwind, Bootstrap, styled-components, CSS Modules, 순수 CSS를 지원한다. "피그마를 코드로", "Figma to React", "피그마 변환", "디자인 구현", "figma url을 변환", "피그마 디자인을 코드로" 등 Figma URL과 함께 코드 생성을 요청할 때 반드시 이 스킬을 사용한다. 사용자가 Figma URL을 제공하고 특정 프레임워크나 CSS 방식을 언급하면 항상 이 스킬을 트리거한다.
metadata:
  mcp-server: figma
---

# Figma-to-Code

Figma 디자인을 픽셀 퍼펙트 프로덕션 코드로 변환하는 스킬. 레이아웃과 스타일을 분리하여 단계적으로 구현하고, Playwright로 결과물을 비교 검증한다.

## 전제 조건

- **Figma MCP 서버** 연결 필수 — `get_design_context`, `get_screenshot` 등 MCP 도구 사용 가능해야 함
- **Playwright MCP 서버** 연결 권장 — 비주얼 비교 루프에 필요
- 사용자 제공 정보: Figma URL + 대상 프레임워크 + CSS 방식

Figma MCP 도구가 사용 불가하면 사용자에게 Figma MCP 서버 활성화를 안내한다.

## 지원 구성

| 프레임워크 | Tailwind | Bootstrap | styled-components | CSS Modules | Pure CSS |
|-----------|----------|-----------|-------------------|-------------|----------|
| React     | O | O | O | O | O |
| Vue       | O | O | - | O | O |
| Next.js   | O | - | O | O | O |
| Svelte    | O | - | - | - | O |
| Angular   | O | O | - | - | O |
| HTML      | O | O | - | - | O |

사용자가 프레임워크나 CSS 방식을 지정하지 않으면 확인한다. 프로젝트에 이미 설정된 프레임워크가 있다면 그것을 기본값으로 제안한다.

## 워크플로우

**아래 단계를 순서대로 실행한다. 단계를 건너뛰지 않는다.**

### Step 1: 입력 파싱 & 설정 확인

#### Figma URL 파싱

URL 형식: `https://figma.com/design/:fileKey/:fileName?node-id=1-2`

추출 항목:
- **fileKey**: `/design/` 뒤의 세그먼트
- **nodeId**: `node-id` 쿼리 파라미터 값 (URL의 하이픈 형식 그대로 사용)

Branch URL인 경우: `figma.com/design/:fileKey/branch/:branchKey/:fileName` → branchKey를 fileKey로 사용

**예시:**
- URL: `https://figma.com/design/kL9xQn2VwM8pYrTb4ZcHjF/MyDesign?node-id=42-15`
- fileKey: `kL9xQn2VwM8pYrTb4ZcHjF`
- nodeId: `42-15`

#### 설정 확인

1. 대상 프레임워크 확인 (React / Vue / Next.js / Svelte / Angular / HTML)
2. CSS 방식 확인 (Tailwind / Bootstrap / styled-components / CSS Modules / Pure CSS)
3. 해당 reference 파일 로드:
   - `references/frameworks/{framework}.md`
   - `references/css-approaches/{css-approach}.md`
4. 부분 구현 요청인지 확인 — 특정 섹션만 구현하는 경우

### Step 2: 디자인 데이터 수집

**병렬로 실행하여 시간을 절약한다:**

```
get_design_context(fileKey=":fileKey", nodeId="1-2")
get_screenshot(fileKey=":fileKey", nodeId="1-2")
```

`get_design_context`가 반환하는 데이터:
- 레이아웃 속성 (Auto Layout, constraints, sizing)
- 타이포그래피 스펙 (font-family, size, weight, line-height, letter-spacing)
- 색상 값과 디자인 토큰
- 컴포넌트 구조와 variants
- 간격(spacing)과 패딩(padding) 값

**응답이 잘리거나 너무 큰 경우:**
1. `get_metadata(fileKey=":fileKey", nodeId="1-2")`로 노드 트리 구조 파악
2. 메타데이터에서 주요 자식 노드 ID 식별
3. 각 자식 노드에 대해 개별적으로 `get_design_context` 호출

**디자인 토큰이 있는 경우:**
```
get_variable_defs(fileKey=":fileKey")
```
이 토큰을 CSS 변수나 프레임워크 토큰으로 매핑한다.

**부분 구현 요청 시:**
- `get_metadata`로 전체 노드 트리 파악
- 사용자가 지정한 영역의 노드만 `get_design_context`로 fetch

### Step 3: 디자인 토큰 해석 & 에셋 다운로드

> 상세 참조: `references/design-token-resolution.md`

#### 3-1: 디자인 토큰 해석

Figma MCP 출력에는 CSS 변수 형식의 디자인 토큰이 포함된다:
```
var(--spacing/space-24, 24px)
var(--text/body/large/size, 24px)
var(--color/semantic/text/primary, #111827)
```

**반드시 다음 절차를 따른다:**

1. **프로젝트에 디자인 토큰 시스템이 있는 경우**: MCP 토큰을 프로젝트 토큰에 매핑한다
2. **프로젝트에 디자인 토큰이 없는 경우**: fallback 값(쉼표 뒤의 값)을 추출하여 CSS 변수로 정의한다
3. **CSS 변수 정의 파일을 생성**하여 모든 토큰을 한곳에서 관리한다:

```css
:root {
  /* Spacing */
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

  /* Colors */
  --color-primary: #0f4c5c;
  --color-text-primary: #111827;
  --color-text-secondary: #374151;
  --color-text-tertiary: #6b7280;
  --color-text-inverse: #ffffff;
  --color-surface-bg: #ffffff;
  --color-surface-secondary: #f9fafb;
  --color-border-default: #e5e7eb;
  --color-border-subtle: #d1d5db;

  /* Typography */
  --font-heading-h1: 48px;
  --font-heading-h2: 36px;
  --font-title-t1: 28px;
  --font-title-t2: 26px;
  --font-title-t3: 24px;
  --font-body-large: 24px;
  --font-body-medium: 20px;
  --font-body-small: 16px;

  /* Radius */
  --radius-small: 4px;
  --radius-large: 12px;
  --radius-2xl: 48px;
  --radius-full: 9999px;

  /* Page Layout */
  --page-padding: 320px;
  --content-width: 1280px;
}
```

4. **MCP 출력의 var() 표현을 프로젝트 변수로 변환한다**:
   - `var(--spacing\/space-24, 24px)` → `var(--spacing-24)` 또는 직접 `24px`
   - `var(--text\/heading\/h1\/size, 48px)` → `var(--font-heading-h1)` 또는 `48px`
   - `var(--color\/semantic\/text\/primary, #111827)` → `var(--color-text-primary)` 또는 `#111827`

#### 3-2: 그라디언트 텍스트 & 복잡 그라디언트 처리

MCP 출력에 인라인 SVG data URI 그라디언트가 포함될 수 있다:
```jsx
style={{ backgroundImage: "url('data:image/svg+xml;utf8,<svg ...>')" }}
```

**이를 안정적인 CSS로 변환한다:**

```css
/* 인라인 SVG radial-gradient → CSS radial-gradient */
.gradient-text {
  background: radial-gradient(ellipse at left, #0f4c5c 50%, #268691 75%, #3cc0c7 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 프로세스 바 같은 복잡 그라디언트 → CSS linear-gradient */
.gradient-bar {
  background: linear-gradient(90deg, #a258a2 0%, #6059a7 26%, #2860a3 53%, #009ede 79%, #46bc96 100%);
  border-radius: 9999px;
}
```

**SVG 그라디언트 → CSS 변환 규칙:**
- `<radialGradient>` → `radial-gradient()` 또는 `linear-gradient()` (방향에 따라)
- `<stop stop-color>` → CSS 색상 스톱
- `gradientTransform` → CSS gradient 방향으로 근사 변환
- 항상 스크린샷과 비교하여 색상이 일치하는지 확인

#### 3-3: 웹폰트 로딩

MCP 출력에 포함된 모든 폰트를 식별하고 로드한다:

```html
<!-- Figma에서 주로 사용하는 폰트 로딩 예시 -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
```

**폰트 규칙:**
- MCP 출력의 `font-family` 값에서 폰트명 추출: `'Noto_Sans_KR:Bold'` → `Noto Sans KR`, weight `700`
- Google Fonts, Adobe Fonts 등에서 해당 폰트를 로드한다
- 폰트가 로드되지 않으면 레이아웃이 깨질 수 있으므로 반드시 `font-display: swap` 사용
- `fontVariationSettings: "'wdth' 100"` 같은 속성도 그대로 적용한다

#### 3-4: 에셋 다운로드

`get_design_context` 결과에서 이미지, SVG, 아이콘 등 에셋을 추출하여 다운로드한다.

**에셋 규칙:**
- Figma MCP가 `localhost` 소스를 반환하면 그 URL을 직접 사용한다
- Figma MCP가 `figma.com/api/mcp/asset/` URL을 반환하면 해당 URL을 fetch하여 프로젝트 에셋 폴더에 저장한다
- 외부 아이콘 패키지(lucide, heroicons 등)를 임의로 추가하지 않는다 — 모든 에셋은 Figma 페이로드에서 가져온다
- 플레이스홀더 이미지를 사용하지 않는다 — 에셋 소스가 있으면 반드시 그것을 사용한다
- 에셋은 Figma MCP 서버의 빌트인 에셋 엔드포인트로 제공된다

### Step 4: 레이아웃 단계 (Layout Phase)

> 상세 참조: `references/layout-patterns.md`

**이 단계에서는 구조적 골격만 생성한다. 비주얼 스타일은 적용하지 않는다.**

레이아웃 단계의 목표: Figma 디자인의 구조를 정확하게 코드로 옮기되, 색상·타이포그래피·그림자 같은 시각적 요소는 다음 단계로 미룬다. 이렇게 분리하면 구조 문제와 스타일 문제를 독립적으로 해결할 수 있어 정확도가 높아진다.

#### 핵심 변환 규칙

| Figma 개념 | CSS 출력 |
|-----------|----------|
| Frame (auto-layout, horizontal) | `display: flex; flex-direction: row` |
| Frame (auto-layout, vertical) | `display: flex; flex-direction: column` |
| Gap between items | `gap: {value}px` |
| Padding | `padding: {top}px {right}px {bottom}px {left}px` |
| Fill container | `flex: 1` 또는 `width: 100%` |
| Fixed size | `width: {value}px; height: {value}px` |
| Hug contents | `width: fit-content` |
| Min/Max constraints | `min-width`, `max-width`, `min-height`, `max-height` |
| Space between (justify) | `justify-content: space-between` |
| Alignment | `align-items: center/flex-start/flex-end` |

#### MCP 출력 코드 정리 규칙

Figma MCP가 생성하는 참조 코드는 직접 사용하지 않는다. 반드시 다음을 정리한다:

1. **data-name, data-node-id 속성 제거** — 디버깅용이므로 프로덕션 코드에 포함하지 않는다
2. **불필요한 래퍼 div 병합** — MCP는 Figma 레이어 구조를 1:1로 변환하여 과도한 중첩이 발생한다. 시맨틱하게 의미 없는 래퍼는 병합한다
3. **CSS 변수 표현 정리** — `var(--spacing\/space-24, 24px)` 같은 이스케이프 슬래시를 제거하고, Step 3에서 정의한 토큰으로 변환한다
4. **인라인 스타일 최소화** — MCP가 생성한 `style={{}}` 인라인 스타일을 CSS 클래스로 이동한다 (그라디언트 등 복잡한 스타일 포함)
5. **content-stretch 등 비표준 클래스 제거** — MCP 전용 유틸리티 클래스를 표준 CSS로 변환한다

#### 복잡 레이아웃 패턴

MCP 출력의 레이아웃을 참고하되, 다음 패턴은 직접 구현한다:

**1. 섹션 레이아웃 (전폭 배경 + 제한 너비 콘텐츠)**
```css
.section {
  width: 100%;
  padding: 120px var(--page-padding);  /* Figma의 px 값 그대로 */
}
/* 반응형에서 page-padding을 줄인다 */
@media (max-width: 1024px) { .section { padding: 80px 40px; } }
@media (max-width: 768px) { .section { padding: 60px 20px; } }
```

**2. 타임라인/히스토리 레이아웃**
- Figma의 수평 타임라인: `display: flex; justify-content: space-between`
- 각 항목의 높이가 다른 경우: `align-items: flex-start` + 개별 높이 지정
- 연결선: `position: absolute`로 도트 사이를 잇는 선을 그린다
- 아이콘-라벨-도트-설명 순서의 수직 구조

**3. 지그재그 프로세스 레이아웃**
- 상단 행: Step 1, 3, 5 (홀수)
- 프로그레스 바: 전폭 그라디언트 바
- 하단 행: Step 2, 4 (짝수)
- 수직 연결선으로 프로그레스 바와 연결

**4. 이미지+텍스트 가로 레이아웃**
```css
.content-row {
  display: flex;
  gap: 80px;
  align-items: center;
}
.content-image {
  flex: 0 0 408px;  /* Figma의 정확한 이미지 크기 */
  border-radius: 48px;
  overflow: hidden;
}
.content-text { flex: 1; }
```

**5. 2x2 카드 그리드**
```css
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 32px;
}
.card-grid > * {
  flex: 1 1 calc(50% - 16px);
  min-width: 496px;
}
```

#### 반응형 전략

기본적으로 모바일 퍼스트 반응형으로 구현한다:

```
기본: 320px ~ (모바일)
sm: 640px ~
md: 768px ~
lg: 1024px ~
xl: 1440px ~
```

Figma의 constraints를 반응형 동작으로 변환:
- `Left & Right` → `width: 100%` (부모에 맞춤)
- `Center` → `margin: 0 auto` (가운데 정렬)
- `Scale` → 비율 기반 사이즈 (`%` 또는 `vw`)
- `px(--responsive/side/page-padding, 320px)` → 반응형 패딩으로 변환 (xl: 320px, lg: 80px, md: 40px, sm: 20px)

사용자가 특정 브레이크포인트 구간을 요청하면 해당 구간에 맞춰 작업한다.

#### 출력

이 단계의 결과물은 **올바른 구조의 컴포넌트 파일**이다:
- 정확한 HTML/JSX/template 구조
- flex/grid 레이아웃 적용
- 반응형 브레이크포인트 설정
- 컨테이너, 간격, 정렬 설정 완료
- 색상, 폰트, 그림자 등 비주얼 스타일은 아직 없음

### Step 5: 스타일 단계 (Style Phase)

> 상세 참조: `references/style-patterns.md`

**레이아웃 위에 모든 비주얼 속성을 적용한다.**

스타일 단계에서는 Figma 디자인의 모든 시각적 속성을 하나도 빠짐없이 코드로 옮긴다. 디자인 토큰이 있으면 토큰을 우선 사용하고, 없으면 Figma에서 직접 추출한 값을 사용한다.

#### 적용 항목

**타이포그래피:**
- `font-family` — Figma에서 사용한 폰트 그대로 (웹폰트 로드 포함)
- `font-size` — px 단위 그대로 또는 rem으로 변환
- `font-weight` — 정확한 weight 값
- `line-height` — px 또는 비율
- `letter-spacing` — px 또는 em
- `text-align`, `text-decoration`, `text-transform`

**색상:**
- 배경색 (`background-color`, 그라디언트 포함)
- 텍스트 색상 (`color`)
- 보더 색상 (`border-color`)
- 디자인 토큰이 있으면 CSS 변수로 매핑: `var(--color-primary)`

**그라디언트 (핵심 — 반드시 정확히 변환):**
- MCP의 인라인 SVG data URI 그라디언트를 CSS gradient로 변환한다 (Step 3-2 참조)
- `background-clip: text` + `-webkit-text-fill-color: transparent` 조합으로 그라디언트 텍스트 구현
- 섹션 배경 그라디언트: `linear-gradient(to bottom, transparent, #f9fafb)` 등
- 프로그레스 바 그라디언트: 여러 색상 스톱이 있는 `linear-gradient(90deg, ...)`
- **반드시 Figma 스크린샷과 비교하여 색상이 일치하는지 확인**

**효과:**
- `box-shadow` — Figma의 Drop Shadow, Inner Shadow 변환
- `filter: blur()` — Figma의 Layer Blur
- `backdrop-filter: blur()` — Figma의 Background Blur
- `opacity`
- `border-radius` — 각 모서리별 값 지원 (48px 같은 큰 값도 정확히 적용)

**보더:**
- `border-width`, `border-style`, `border-color`
- 개별 보더 (top/right/bottom/left)
- 왼쪽 보더로 구분선 표현: `border-left: 1px solid var(--color-border-subtle)`

**인터랙션 상태:**
- `:hover`, `:active`, `:focus`, `:disabled` 스타일
- `transition` 속성 (부드러운 상태 전환)
- `cursor` 속성

#### CSS 방식별 적용

선택된 CSS 방식에 따라 적절한 reference 파일의 매핑 규칙을 따른다:
- Tailwind → `references/css-approaches/tailwind.md`
- Pure CSS → `references/css-approaches/pure-css.md`
- (기타 CSS 방식도 해당 reference 참조)

### Step 6: 비주얼 비교 루프 (Playwright)

> 상세 참조: `references/comparison-guide.md`

코드 생성이 완료되면 Playwright로 렌더링 결과를 Figma 스크린샷과 비교한다. 이 과정이 피그마 디자인과 100% 일치를 달성하는 핵심이다.

#### 비교 프로세스

1. **로컬 서빙**: 생성된 코드를 브라우저에서 볼 수 있도록 준비한다
   - HTML 파일이면 직접 열기
   - React/Vue/Next.js 등이면 dev server 실행 또는 임시 HTML로 래핑

2. **브라우저 스크린샷 캡처**:
   ```
   browser_navigate(url="http://localhost:PORT")
   browser_take_screenshot()
   ```

3. **비교**: Step 2에서 저장한 Figma 스크린샷과 Playwright 스크린샷을 나란히 비교한다

   **필수 체크 항목 (우선순위 순):**

   a. **전체 레이아웃 구조**:
      - 섹션 순서와 배치가 동일한가
      - 전폭 섹션 vs 제한 너비 콘텐츠 구분이 맞는가
      - 가로 배치(이미지+텍스트)의 비율이 맞는가

   b. **간격과 여백**:
      - 섹션 간 padding이 Figma와 일치하는가 (80px, 120px, 200px 등)
      - 요소 간 gap이 정확한가
      - page-padding (좌우 여백)이 맞는가

   c. **타이포그래피**:
      - 폰트가 정상 로드되었는가 (Noto Sans KR, Roboto 등)
      - font-size, font-weight, line-height가 일치하는가
      - 텍스트 색상이 정확한가 (#111827, #374151, #6b7280 등)

   d. **그라디언트 & 색상**:
      - 그라디언트 텍스트(숫자 등)의 색상이 일치하는가
      - 섹션 배경 그라디언트가 적용되었는가
      - 프로그레스 바 등의 그라디언트 방향과 색상이 맞는가

   e. **컴포넌트 스타일**:
      - 카드의 border, border-radius, background가 맞는가
      - 버튼의 색상, 크기, radius가 맞는가
      - 이미지의 border-radius (48px 등 큰 값 포함)

   f. **이미지와 에셋**:
      - 모든 이미지가 로드되었는가
      - 이미지 크기와 위치가 맞는가
      - 아이콘이 올바른 크기로 표시되는가

4. **불일치 수정**: 발견된 차이를 우선순위 순으로 수정한다. 가장 눈에 띄는 차이부터 해결한다.

5. **재비교**: 수정 후 다시 스크린샷 → 비교 → 수정 반복

6. **최대 5회 반복**: 3회 후에도 차이가 크면 추가 2회 반복. 5회 후에도 남은 불일치는 사용자에게 보고한다.

   **자주 발생하는 불일치와 해결법:**
   - 폰트 미로드 → Google Fonts link 확인, preconnect 추가
   - 그라디언트 텍스트 미표시 → `-webkit-background-clip: text` + `-webkit-text-fill-color: transparent` 확인
   - 간격 불일치 → MCP의 var() fallback 값을 직접 px로 적용
   - 이미지 미표시 → 에셋 URL 접근 가능 여부 확인, 404라면 재다운로드
   - 배경색 미적용 → 섹션 배경 gradient 코드 확인

#### Playwright 사용 불가 시

Playwright MCP가 연결되지 않은 경우:
- Step 7의 수동 검증 체크리스트로 대체한다
- 사용자에게 브라우저에서 직접 확인하도록 안내한다

### Step 7: 최종 검증 & 전달

#### 검증 체크리스트

구현 완료 전에 다음 항목을 확인한다:

- [ ] **레이아웃**: 간격, 정렬, 크기가 Figma와 일치
- [ ] **타이포그래피**: 폰트, 크기, 굵기, line-height 일치
- [ ] **색상**: 모든 색상 값이 정확히 일치
- [ ] **인터랙션**: hover, active, disabled 상태 동작
- [ ] **반응형**: 모바일/태블릿/데스크톱에서 올바르게 표시
- [ ] **에셋**: 이미지와 아이콘이 올바르게 렌더링
- [ ] **접근성**: 시맨틱 HTML, ARIA 속성, 키보드 네비게이션

#### 전달

- 최종 코드를 사용자에게 전달한다
- 부분 구현인 경우 나머지 부분과의 통합 방법을 설명한다
- 프로젝트에 기존 디자인 시스템이 있다면 토큰 매핑 내역을 안내한다

## 부분 구현

사용자가 디자인의 특정 부분만 구현을 요청할 수 있다:

1. `get_metadata`로 전체 노드 트리를 파악한다
2. 사용자가 지정한 영역(헤더, 사이드바, 카드 등)의 노드를 식별한다
3. 해당 노드만 `get_design_context`로 상세 데이터를 가져온다
4. 동일한 레이아웃 → 스타일 파이프라인으로 구현한다
5. 나머지 부분과의 통합 가이드를 제공한다

## 프로젝트 통합 규칙

### 기존 컴포넌트 재사용
- 프로젝트에 이미 있는 컴포넌트(버튼, 인풋, 카드 등)가 있으면 재사용한다
- 새 컴포넌트를 만들기 전에 기존 것을 확장할 수 있는지 확인한다

### 디자인 토큰 매핑
- Figma 디자인 토큰이 있으면 프로젝트의 토큰 시스템에 매핑한다
- 프로젝트에 토큰이 없으면 CSS 변수로 정리하여 제공한다

### 코드 컨벤션
- 프로젝트의 기존 파일 구조, 네이밍, import 패턴을 따른다
- Figma MCP 출력(React + Tailwind)은 참조용이지 최종 코드가 아니다 — 반드시 대상 프레임워크와 CSS 방식으로 변환한다

## 예시

### 예시 1: React + Tailwind

사용자: "https://figma.com/design/kL9xQn2VwM8pYrTb4ZcHjF/MyApp?node-id=42-15 를 React로 변환해줘. 테일윈드 사용해줘."

1. URL 파싱 → fileKey=`kL9xQn2VwM8pYrTb4ZcHjF`, nodeId=`42-15`
2. `references/frameworks/react.md`, `references/css-approaches/tailwind.md` 로드
3. `get_design_context` + `get_screenshot` 병렬 실행
4. 에셋 다운로드
5. 레이아웃 단계: JSX 구조 + Tailwind flex/grid 클래스
6. 스타일 단계: Tailwind 유틸리티 클래스로 모든 스타일 적용
7. Playwright 비교 → 불일치 수정 → 최종 전달

### 예시 2: HTML + Pure CSS (부분 구현)

사용자: "이 피그마 디자인에서 헤더 부분만 HTML/CSS로 만들어줘. https://figma.com/design/pR8mN/Site?node-id=10-5"

1. URL 파싱
2. `get_metadata`로 전체 구조 파악 → 헤더 노드 식별
3. 헤더 노드만 `get_design_context` + `get_screenshot`
4. 레이아웃 단계: 시맨틱 HTML 구조 + CSS flex/grid
5. 스타일 단계: 별도 CSS 파일에 모든 스타일 작성
6. Playwright 비교 → 최종 전달

### 예시 3: Vue + Tailwind (반응형)

사용자: "피그마 디자인을 Vue 컴포넌트로 변환해줘. 모바일부터 데스크톱까지 반응형으로. https://figma.com/design/xxx?node-id=1-1"

1. URL 파싱
2. `references/frameworks/vue.md`, `references/css-approaches/tailwind.md` 로드
3. 디자인 데이터 수집
4. 레이아웃 단계: Vue SFC `<template>` + Tailwind 반응형 클래스 (sm:, md:, lg:)
5. 스타일 단계: Tailwind 유틸리티로 모든 비주얼 적용
6. Playwright에서 여러 뷰포트 크기로 비교 (320px, 768px, 1440px)
7. 최종 전달

## 트러블슈팅

### 디자인 데이터가 잘림
`get_metadata`로 노드 트리를 먼저 파악한 후, 주요 섹션별로 `get_design_context`를 개별 호출한다.

### 구현 결과가 디자인과 다름
Step 2의 스크린샷과 코드를 나란히 비교한다. spacing, color, typography 값을 디자인 컨텍스트 데이터에서 정확히 확인한다.

### 에셋 로딩 안 됨
Figma MCP 서버의 에셋 엔드포인트가 접근 가능한지 확인한다. localhost URL을 수정 없이 직접 사용한다.

### 디자인 토큰 값이 프로젝트와 다름
프로젝트 토큰을 우선 사용하되, 시각적 일치를 위해 spacing/sizing을 미세 조정한다.
