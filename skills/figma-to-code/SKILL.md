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

**섹션별 순차 작업 방식으로 진행한다. 전체를 한 번에 생성하지 않는다.**

핵심 원칙:
1. 먼저 전체 구조(스켈레톤)를 파악하고 큰 레이아웃을 생성한다
2. 그 다음 각 섹션을 순서대로 하나씩 구현한다
3. 각 섹션마다 개별적으로 `get_design_context`를 호출하여 정확한 데이터를 얻는다
4. MCP 출력 코드를 복사하지 않는다 — 스크린샷과 데이터를 참고하여 직접 코드를 작성한다

### Phase 1: 전체 구조 파악

#### 1-1: 입력 파싱

URL 형식: `https://figma.com/design/:fileKey/:fileName?node-id=1-2`

추출 항목:
- **fileKey**: `/design/` 뒤의 세그먼트
- **nodeId**: `node-id` 쿼리 파라미터 값 (URL의 하이픈 형식 그대로 사용)

Branch URL인 경우: `figma.com/design/:fileKey/branch/:branchKey/:fileName` → branchKey를 fileKey로 사용

#### 1-2: 설정 확인

1. 대상 프레임워크 확인 (React / Vue / Next.js / Svelte / Angular / HTML)
2. CSS 방식 확인 (Tailwind / Bootstrap / styled-components / CSS Modules / Pure CSS)
3. 해당 reference 파일 로드:
   - `references/frameworks/{framework}.md`
   - `references/css-approaches/{css-approach}.md`
   - `references/design-token-resolution.md`

#### 1-3: 전체 구조 파악

**반드시 `get_design_context`를 먼저 호출한다.** 대부분의 전체 페이지는 "너무 큰" 응답으로 메타데이터만 반환된다. 이것이 정상이다.

```
get_design_context(fileKey, nodeId)   → 메타데이터 (섹션 구조)
get_screenshot(fileKey, nodeId)       → 전체 스크린샷 (비교 기준)
```

메타데이터에서 **섹션 목록과 각 섹션의 nodeId**를 추출한다:

```
예시 결과:
├── Header (nodeId: "1231:26920")
├── Hero (nodeId: "3635:26403")
├── Tab (nodeId: "1231:28073")
├── Container (nodeId: "1231:26777")
│   ├── Section01 (nodeId: "1231:26778")
│   ├── Section02 (nodeId: "1231:26802")
│   ├── Section03 (nodeId: "1231:26868")
│   ├── Section04 (nodeId: "1231:26883")
│   ├── Section05 (nodeId: "1231:26897")
│   └── Section06 (nodeId: "1231:26766")
└── Footer (nodeId: "1231:26765")
```

이 섹션 목록이 이후 작업의 로드맵이 된다.

**메타데이터에서 반복 컴포넌트의 인스턴스별 차이도 확인한다:**

동일 `name`의 instance가 여러 개 있으면 각각의 **width, height**를 비교한다. 차이가 있으면 기록해둔다:
- 높이가 다른 리스트 아이템 → **스크린샷으로 실제 레이아웃을 반드시 확인**:
  - 모든 아이템이 같은 방향(위→아래)으로 흐르되 커넥터 길이만 다른 경우 (타임라인에서 흔함)
  - 교차 패턴(alternating)인 경우 — 반드시 스크린샷에서 위/아래 교차가 시각적으로 확인되어야 함
- 같은 버튼 컴포넌트지만 bg/border 색상이 다름 → filled vs outlined 변형
- 같은 탭 컴포넌트지만 하나만 border/bold → 활성 상태 표시

**⚠️ 높이가 다르다고 자동으로 지그재그(교차) 패턴을 가정하지 않는다.** 반드시 스크린샷을 기준으로 판단한다.

이 정보는 Phase 3 섹션 구현에서 각 인스턴스에 개별 스타일을 적용할 때 사용한다.

#### 1-4: 프로토타입 인터랙션 분석

`get_design_context`와 `get_metadata`의 응답에서 **프로토타입 설정(reactions, interactions)**을 확인한다.

**확인 방법:**
1. `get_design_context` 응답의 코드/메타데이터에서 `reaction`, `interaction`, `trigger`, `action`, `navigation`, `transition` 관련 정보를 찾는다
2. `get_metadata` 응답의 XML에서 각 노드의 프로토타입 연결 정보를 확인한다
3. 스크린샷에서 클릭 가능한 요소(버튼, 링크, 탭, 카드 등)를 시각적으로 파악한다

**추출할 프로토타입 정보:**

| Figma 프로토타입 속성 | 추출 내용 |
|----------------------|----------|
| Trigger | On Click, On Hover, While Hovering, After Delay, Mouse Enter/Leave |
| Action | Navigate To, Open Overlay, Close Overlay, Back, Open Link, Swap With |
| Animation | Instant, Dissolve, Move In/Out, Push, Slide In/Out, Smart Animate |
| Duration | 전환 시간 (ms) |
| Easing | Linear, Ease In, Ease Out, Ease In and Out, Custom Bezier |
| Overlay 설정 | 위치, 배경 딤 처리, 외부 클릭 시 닫기 |
| Scroll 설정 | Scroll To 대상 노드, 오프셋 |

**프로토타입 정보를 섹션 목록에 기록한다:**

```
예시 결과:
├── Header (nodeId: "1231:26920")
│   ├── Logo → Navigate To: Hero 섹션
│   ├── Nav Item "서비스" → On Hover: Open Overlay (드롭다운 메뉴)
│   └── CTA 버튼 → On Click: Open Link (외부 URL)
├── Hero (nodeId: "3635:26403")
│   └── "자세히 보기" 버튼 → On Click: Scroll To Section01
├── Tab (nodeId: "1231:28073")
│   ├── Tab01 → On Click: Navigate To Section01 (Smart Animate, 300ms, Ease Out)
│   ├── Tab02 → On Click: Navigate To Section02
│   └── Tab03 → On Click: Navigate To Section03
├── Section03 (nodeId: "1231:26868")
│   └── 카드 → On Click: Open Overlay (상세 모달, 배경 딤)
└── Footer (nodeId: "1231:26765")
    └── 맵 영역 → On Click: Open Link (Google Maps)
```

이 인터랙션 목록은 Phase 3에서 각 섹션 구현 시 참조한다.

#### 1-5: 프로젝트 기반 작업 & 디자인 토큰

> 상세 참조: `references/design-token-resolution.md`

**프로젝트 코드 분석 (필수):**
- 프로젝트의 기존 파일 구조, 프레임워크, CSS 방식을 파악한다
- package.json, tsconfig, tailwind.config 등을 확인한다
- 기존 컴포넌트가 있으면 재사용 가능한지 파악한다

**웹폰트 로딩 설정:**
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
```

MCP 출력의 `font-family` 패턴 변환:
- `'Noto_Sans_KR:Bold'` → `'Noto Sans KR', sans-serif` + `font-weight: 700`
- `'Roboto:Medium'` → `'Roboto', sans-serif` + `font-weight: 500`

### Phase 2: 전체 레이아웃 스켈레톤 생성

**이 단계에서 전체 페이지의 뼈대를 만든다.** 각 섹션은 빈 컨테이너로 두고, 전체 구조만 잡는다.

```
[Header] ─ 고정 상단, 전폭
[Hero] ─ 전폭, 배경 이미지
[Tab Navigation] ─ 전폭, 고정/스티키 가능
[Container] ─ 전폭
  [Section01] ─ 빈 섹션 (추후 구현)
  [Section02] ─ 빈 섹션
  [Section03] ─ 빈 섹션
  ...
[Footer] ─ 전폭
```

**스켈레톤에서 확정하는 것:**
- 전체 페이지 레이아웃 (세로 스택)
- 각 섹션의 전폭 배경 vs 제한 너비 콘텐츠 구분
- 페이지 기본 스타일 (body margin/padding, font-family, color)
- CSS 변수/토큰 정의 (colors, spacing, typography)

### Phase 3: 섹션별 순차 구현

**각 섹션을 아래 절차로 하나씩 완성한다. 한 섹션이 끝나야 다음 섹션으로 넘어간다.**

#### 섹션 구현 절차 (매 섹션마다 반복)

**Step A: 섹션 데이터 수집**

```
get_design_context(fileKey, nodeId="섹션ID")
get_screenshot(fileKey, nodeId="섹션ID")    ← 섹션별 스크린샷
```

- `get_design_context`가 코드와 에셋 URL을 반환한다
- 반환된 코드는 **참고 자료일 뿐 최종 코드가 아니다**
- 스크린샷이 해당 섹션의 "정답"이다 — 이것과 동일하게 보여야 한다

**Step B: MCP 출력 분석 (코드 복사 금지)**

MCP가 반환하는 코드에서 추출할 것:
1. **에셋 URL** — 이미지, 아이콘의 다운로드 URL
2. **정확한 수치** — spacing, font-size, color 등의 값
3. **레이아웃 구조** — flex 방향, gap, padding 등
4. **텍스트 내용** — 실제 텍스트 콘텐츠
5. **인스턴스별 차이** — 동일 컴포넌트가 반복될 때 각 인스턴스의 개별 속성:
   - 각 인스턴스의 **height가 다르면** 개별 값을 기록 — **스크린샷에서 흐름 방향을 확인** (동일 방향 + 커넥터 길이 차이 vs 지그재그 교차)
   - **border, font-weight, color가 다르면** 활성/비활성 상태 구분
   - 버튼의 **bg, border-color가 다르면** filled/outlined 변형 구분
   - 각 인스턴스의 **텍스트 줄 수**가 다르면 개별 기록
6. **CSS 효과** — `backdrop-filter`, `opacity`, `rgba` 배경 등 시각 효과 추출

MCP 출력에서 무시할 것:
1. `data-name`, `data-node-id` 속성
2. `content-stretch` 등 비표준 클래스
3. `var(--spacing\/space-24, 24px)` 형태의 이스케이프된 CSS 변수 — fallback 값만 사용
4. 과도한 div 중첩 — 시맨틱하게 정리한다
5. 인라인 SVG data URI — CSS gradient로 변환한다

**수치 추출 체크리스트 (반드시 기록할 것):**

MCP 코드의 className에서 다음 값들을 정확히 추출하여 기록한다:
```
[섹션 수치 요약]
- 컨테이너 padding: pt/pb/px 각각 (예: pt-200px pb-40px px-320px)
- 자식 간 gap: (예: gap-80px, gap-32px, gap-64px)
- 고정 크기 요소: (예: w-320px, h-64px, w-248px, w-153px)
- border-radius: (예: rounded-48px, rounded-4px, rounded-full)
- 최소 크기: (예: min-w-496px, min-w-231px)
- 그라디언트 색상 스톱: (SVG stop-color + offset 추출)
- CSS 효과: (예: backdrop-blur, opacity, rgba 배경, box-shadow)
```

**특히 padding의 top/bottom이 비대칭인 경우가 매우 흔하다.** `py-120px`로 단순화하지 말고 `pt-200px pb-40px`처럼 개별 값을 정확히 추출한다.

**헤더와 콘텐츠 영역의 padding이 다를 수 있다.** 헤더는 `px-120px`, 콘텐츠 섹션은 `px-320px`처럼 각 영역의 padding을 개별적으로 확인한다. 전체 페이지의 `page-padding` 값을 모든 곳에 일괄 적용하지 않는다.

**Step C: 레이아웃 구현**

스크린샷을 보면서 해당 섹션의 HTML 구조를 작성한다:

| Figma 개념 | CSS 출력 |
|-----------|----------|
| Frame (horizontal) | `display: flex; flex-direction: row` |
| Frame (vertical) | `display: flex; flex-direction: column` |
| Gap | `gap: {value}px` |
| Padding | `padding: {top}px {right}px {bottom}px {left}px` |
| Fill container | `flex: 1` 또는 `width: 100%` |
| Fixed size | `width: {value}px; height: {value}px` |
| Hug contents | `width: fit-content` |
| Space between | `justify-content: space-between` |

**공통 섹션 패턴:**

```css
/* 전폭 배경 + 제한 너비 콘텐츠 */
.section {
  width: 100%;
  padding: 120px 320px;  /* Figma의 padding 값 그대로 */
}

/* 이미지 + 텍스트 가로 레이아웃 */
.row-layout {
  display: flex;
  gap: 80px;
  align-items: center;
}
.row-image {
  flex: 0 0 408px;
  border-radius: 48px;
  overflow: hidden;
}
.row-text { flex: 1; }

/* 2x2 카드 그리드 */
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 32px;
}
.card-grid > * {
  flex: 1 1 calc(50% - 16px);
  min-width: 496px;  /* MCP 데이터의 min-w 값 그대로 */
}

/* 숫자 통계 행 (divider 포함) */
.stat-row {
  display: flex;
  align-items: center;
  justify-content: center;
}
.stat-card { width: 320px; text-align: center; }  /* 고정 폭 */
.stat-divider { width: 1px; height: 80px; background: #e5e7eb; }

/* 지그재그 프로세스 (위-바-아래) */
.process-top { display: flex; justify-content: space-between; }
.process-bar { height: 12px; border-radius: 9999px; /* gradient */ }
.process-bottom { display: flex; justify-content: space-between; padding: 0 231px; }

/* 수직 컬럼 타임라인 (아이콘→연도→도트→텍스트, 모든 아이템 동일 방향) */
/* ⚠️ 중요: 아이템별 height가 다르다고 지그재그(위/아래 교차) 패턴이 아니다!
   Figma에서 동일 컴포넌트가 서로 다른 height를 가지면,
   모든 아이템이 같은 방향(위→아래)으로 흐르되 커넥터 길이만 다른 것이다.
   지그재그 패턴은 명시적으로 '위-바-아래' 구조가 보일 때만 사용한다 (예: 프로세스 단계). */
.timeline { display: flex; align-items: flex-start; justify-content: space-between; }
.timeline-item {
  display: flex; flex-direction: column; align-items: center;
  width: 153px;
  /* height는 각 아이템마다 개별 적용 — 커넥터 길이로 조절 */
}
/* 타임라인 수평선은 도트의 중심 높이에 absolute로 배치 */
.timeline-line {
  position: absolute;
  height: 4px;
  /* top 계산: icon(108) + gap(20) + year(68) + gap(20) + dot_radius(22) */
}

/* 헤더 — 콘텐츠와 다른 padding 사용, 반투명 배경 가능 */
.header {
  padding: 0 120px;  /* 헤더는 page-padding(320px)과 다를 수 있다 */
  backdrop-filter: blur(5px);
  background: rgba(255,255,255,0.92);  /* 반투명 + 블러 효과 확인 */
}

/* 섹션 배경 그라디언트 (투명 → 색상) */
.section-gradient-bg {
  background: linear-gradient(to bottom, rgba(249,250,251,0), #f9fafb);
}
```

**Step D: 스타일 적용**

> 상세 참조: `references/style-patterns.md`

레이아웃 위에 모든 비주얼 속성을 적용한다. **Figma 스크린샷을 계속 참고하면서 작업한다.**

**타이포그래피:**
- `font-family` — Figma에서 사용한 폰트 그대로
- `font-size` — MCP 데이터에서 추출한 정확한 px 값
- `font-weight` — 400 (Regular), 500 (Medium), 700 (Bold)
- `line-height` — 정확한 px 값
- `color` — 정확한 hex 값

**색상 & 배경:**
- 배경색, 텍스트 색상, 보더 색상 모두 MCP 데이터의 정확한 값 사용
- 그라디언트는 반드시 CSS gradient로 변환 (인라인 SVG 금지):
  ```css
  /* 그라디언트 텍스트 */
  .gradient-text {
    background: radial-gradient(ellipse at 20% 0%, #0f4c5c 50%, #268691 75%, #3cc0c7 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }
  /* 프로그레스 바 */
  .gradient-bar {
    background: linear-gradient(90deg, #a258a2 0%, #6059a7 26%, #2860a3 53%, #009ede 79%, #46bc96 100%);
  }
  ```

**효과 (CSS Effects):**
- `box-shadow`, `border-radius`, `opacity`
- `border` — 정확한 width, style, color
- `backdrop-filter: blur()` — MCP 출력에서 `backdrop-blur` 클래스 확인
- `background: rgba()` — 반투명 배경이면 정확한 alpha 값 추출 (예: `rgba(255,255,255,0.92)`)
- 헤더, 모달, 오버레이에 반투명 배경 + 블러 조합이 흔함 — `bg-white`로 대체하지 않는다

**버튼 (특히 중요 — 자주 틀리는 부분):**
- MCP 데이터에서 정확한 크기(width, height), padding, border-radius 추출
- 배경색, 텍스트 색상, font-size, font-weight 정확히 적용
- hover/active 상태도 구현
- **같은 버튼 컴포넌트라도 인스턴스별로 스타일이 다를 수 있다** — 각각 확인한다

버튼 변형 예시:
```css
/* CTA 메인 버튼 */
.btn-main { width: 248px; height: 64px; padding: 12px 24px; border-radius: 4px; background: #0f4c5c; color: white; font-size: 22px; font-weight: 500; }
/* 서브 버튼 - filled */
.btn-sub-filled { padding: 16px; border-radius: 9999px; background: #0f4c5c; color: white; font-size: 18px; font-weight: 500; }
/* 서브 버튼 - outlined (보라색) */
.btn-sub-outlined { padding: 16px; border-radius: 9999px; background: #f9fafb; border: 1px solid #a258a2; color: #111827; font-size: 18px; font-weight: 500; }
/* 서브 버튼 - outlined (회색) + 아이콘 */
.btn-sub-icon { padding: 12px 16px 12px 24px; border-radius: 9999px; background: #f9fafb; border: 1px solid #e5e7eb; font-size: 16px; }
```

**탭/네비게이션 활성 상태 (자주 누락됨):**
```css
/* 탭 활성 - 상단 border (GNB) */
.tab-active-top { border-top: 4px solid #0f4c5c; font-weight: 700; color: #0f4c5c; }
/* 탭 활성 - 하단 border (서브 탭) */
.tab-active-bottom { border-bottom: 4px solid #a258a2; font-weight: 700; color: #0f4c5c; }
/* 탭 비활성 */
.tab-inactive { border: none; font-weight: 500; color: #111827; }
```

**Step E: 에셋 적용**

- MCP가 반환한 이미지 URL을 fetch하여 프로젝트에 저장한다
- 외부 아이콘을 임의로 추가하지 않는다 — 모든 에셋은 Figma에서 가져온다
- 이미지에 정확한 크기와 border-radius를 적용한다
- 플레이스홀더 이미지를 사용하지 않는다

**Step F: 프로토타입 인터랙션 구현**

Phase 1-4에서 기록한 인터랙션 목록을 참조하여, 해당 섹션에 포함된 인터랙션을 구현한다.

**Figma Action → 코드 매핑:**

| Figma Action | 구현 방식 |
|-------------|----------|
| Navigate To (같은 페이지 내 섹션) | `scrollIntoView({ behavior: 'smooth' })` 또는 앵커 링크 (`#section-id`) |
| Navigate To (다른 페이지) | 라우터 네비게이션 (`react-router`, `vue-router` 등) 또는 `<a href>` |
| Open Link | `<a href="URL" target="_blank" rel="noopener noreferrer">` |
| Open Overlay | 모달/다이얼로그 컴포넌트 — state로 열기/닫기 제어 |
| Close Overlay | 모달 닫기 핸들러 — 닫기 버튼 + 배경 클릭 + ESC 키 |
| Swap With | 컴포넌트 상태 전환 — hover/active 시 다른 변형 표시 |
| Back | `history.back()` 또는 라우터의 뒤로 가기 |
| Scroll To | `element.scrollIntoView()` 또는 `window.scrollTo()` |

**Figma Trigger → 이벤트 매핑:**

| Figma Trigger | HTML/JS 이벤트 |
|--------------|---------------|
| On Click | `onClick` / `@click` / `click` 이벤트 |
| On Hover / While Hovering | CSS `:hover` 또는 `onMouseEnter`/`onMouseLeave` |
| Mouse Enter | `onMouseEnter` / `@mouseenter` |
| Mouse Leave | `onMouseLeave` / `@mouseleave` |
| After Delay | `setTimeout` + 초기 렌더링 시 실행 |
| On Drag | `onDrag` / 드래그 이벤트 핸들러 |

**Figma Animation → CSS/JS 전환 매핑:**

```css
/* Dissolve (페이드) */
.transition-dissolve {
  transition: opacity {duration}ms {easing};
}

/* Move In (슬라이드 진입) */
.transition-move-in {
  transition: transform {duration}ms {easing}, opacity {duration}ms {easing};
}
.transition-move-in-enter { transform: translateY(20px); opacity: 0; }
.transition-move-in-active { transform: translateY(0); opacity: 1; }

/* Smart Animate (속성 자동 전환) */
.transition-smart {
  transition: all {duration}ms {easing};
}
```

**Easing 변환:**
| Figma Easing | CSS timing-function |
|-------------|-------------------|
| Linear | `linear` |
| Ease In | `ease-in` / `cubic-bezier(0.42, 0, 1, 1)` |
| Ease Out | `ease-out` / `cubic-bezier(0, 0, 0.58, 1)` |
| Ease In and Out | `ease-in-out` / `cubic-bezier(0.42, 0, 0.58, 1)` |
| Custom Bezier | `cubic-bezier(x1, y1, x2, y2)` — Figma 값 그대로 |

**주요 인터랙션 구현 패턴:**

```jsx
/* 1. Scroll To — 탭/네비게이션에서 섹션으로 스크롤 */
const scrollToSection = (sectionId) => {
  document.getElementById(sectionId)?.scrollIntoView({
    behavior: 'smooth',
    block: 'start'
  });
};

/* 2. Open Overlay — 모달/팝업 */
const [isModalOpen, setIsModalOpen] = useState(false);
// 배경 딤 + 외부 클릭 닫기 + ESC 키 닫기
<div className="modal-overlay" onClick={() => setIsModalOpen(false)}>
  <div className="modal-content" onClick={e => e.stopPropagation()}>
    {/* 모달 내용 */}
    <button onClick={() => setIsModalOpen(false)}>닫기</button>
  </div>
</div>

/* 3. Hover 상태 변경 (Swap With) */
const [isHovered, setIsHovered] = useState(false);
<div
  onMouseEnter={() => setIsHovered(true)}
  onMouseLeave={() => setIsHovered(false)}
>
  {isHovered ? <HoverVariant /> : <DefaultVariant />}
</div>

/* 4. 드롭다운 메뉴 (On Hover: Open Overlay) */
const [isDropdownOpen, setIsDropdownOpen] = useState(false);
<div
  onMouseEnter={() => setIsDropdownOpen(true)}
  onMouseLeave={() => setIsDropdownOpen(false)}
>
  <button>메뉴</button>
  {isDropdownOpen && <DropdownContent />}
</div>
```

**모달/오버레이 CSS 패턴:**
```css
/* Figma Open Overlay 설정에 따라 적용 */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);  /* Figma의 딤 배경 색상/투명도 */
  display: flex;
  align-items: center;              /* Figma overlay 위치: center */
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 200ms ease-out;  /* Figma의 animation duration + easing */
}
.modal-content {
  background: white;
  border-radius: 16px;              /* Figma의 border-radius */
  max-width: 600px;
  width: 90%;
  animation: slideUp 300ms ease-out; /* Figma의 Move In 애니메이션 */
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
```

**프로토타입 정보가 없는 경우:**
- MCP 응답에 프로토타입 데이터가 포함되지 않을 수 있다
- 이 경우 스크린샷과 요소의 시각적 특성(버튼 스타일, 커서 포인터 등)을 기반으로 기본 인터랙션을 추론한다:
  - 버튼 → `cursor: pointer` + hover 효과
  - 네비게이션 링크 → 해당 섹션으로 스크롤 또는 페이지 이동
  - 탭 → 탭 전환 (활성 상태 변경)
  - 카드 → 클릭 시 상세 페이지 이동 또는 확대
  - 외부 링크 아이콘이 있는 요소 → 새 탭에서 열기

**Step G: 섹션 비교 검증**

Playwright MCP가 있는 경우:
1. 브라우저에서 해당 섹션을 스크린샷 캡처
2. Figma 섹션 스크린샷과 비교
3. 차이가 있으면 수정 후 재비교
4. 섹션당 최대 3회 반복

Playwright MCP가 없는 경우:
- 사용자에게 해당 섹션의 브라우저 스크린샷을 요청한다
- 또는 구현 완료 후 전체 비교를 진행한다

**비교 체크리스트:**
- [ ] 레이아웃 구조가 동일한가 (가로/세로 배치, 비율)
- [ ] 간격이 정확한가 (padding, gap, margin) — **헤더 px와 콘텐츠 px가 다를 수 있음**
- [ ] 폰트가 맞는가 (family, size, weight, color)
- [ ] 배경색/그라디언트가 일치하는가 — **반투명(rgba) + backdrop-blur 효과 포함**
- [ ] 버튼 크기, 색상, radius가 맞는가 (filled/outlined 변형 구분 포함)
- [ ] 이미지/로고가 올바르게 표시되는가 — **Figma 에셋 URL에서 다운로드한 원본 사용**
- [ ] 보더, 그림자, radius가 맞는가
- [ ] 반복 컴포넌트의 **인스턴스별 치수**가 맞는가 (높이가 다르면 커넥터 길이 차이이지 지그재그가 아님)
- [ ] 탭/네비게이션의 **활성 상태**(border, font-weight, color)가 맞는가
- [ ] 타임라인/프로세스의 **아이템 흐름 방향**이 맞는가 (모두 같은 방향 vs 지그재그)
- [ ] **프로토타입 인터랙션**이 구현되었는가 (클릭, 호버, 스크롤, 오버레이 등)
- [ ] 인터랙션의 **애니메이션**이 Figma 설정과 일치하는가 (duration, easing)

#### 섹션 구현 순서

일반적으로 페이지 상단부터 하단 순서로 진행한다:

```
1. Header (네비게이션)
2. Hero (메인 비주얼)
3. Tab Navigation (서브 메뉴)
4. Section01 (통계/연혁)
5. Section02 (콘텐츠 + 프로세스)
6. Section03 (콘텐츠)
7. Section04 (APP 소개)
8. Section05 (고객사)
9. Section06 (오시는 길)
10. Footer
```

각 섹션 구현 사이에 코드를 저장하고, 다음 섹션으로 넘어간다.

### Phase 4: 전체 비교 & 최종 검증

모든 섹션 구현이 끝나면 전체 페이지를 Figma 스크린샷과 비교한다.

#### 비교 프로세스

1. **전체 페이지 스크린샷** 캡처 (Playwright 또는 사용자 요청)
2. **Figma 전체 스크린샷**과 나란히 비교
3. **섹션별 불일치 확인** — 개별 섹션은 맞는데 섹션 간 간격이 다를 수 있다
4. **수정 → 재비교** 최대 3회 반복

**자주 발생하는 전체 레벨 문제:**
- 섹션 간 간격(margin/padding)이 Figma와 다름
- 전폭 배경색이 누락됨
- 스크롤 시 헤더 겹침 (z-index 문제)
- 폰트 로딩 지연으로 레이아웃 시프트

#### 검증 체크리스트

- [ ] **구조**: 모든 섹션이 올바른 순서로 표시됨
- [ ] **레이아웃**: 간격, 정렬, 크기가 Figma와 일치
- [ ] **타이포그래피**: 폰트, 크기, 굵기, line-height 일치
- [ ] **색상**: 모든 색상 값이 정확히 일치
- [ ] **버튼**: 크기, 색상, radius, 텍스트가 Figma와 동일
- [ ] **이미지/에셋**: 모든 이미지가 올바르게 로드됨
- [ ] **그라디언트**: 텍스트 그라디언트, 배경 그라디언트, 프로그레스 바 정확
- [ ] **기본 인터랙션**: hover, active, focus 상태 동작
- [ ] **프로토타입 인터랙션**: Figma에서 설정된 모든 인터랙션이 구현됨
  - [ ] Navigate To → 스크롤/라우팅이 올바른 대상으로 이동
  - [ ] Open Overlay → 모달/팝업이 올바르게 열리고 닫힘 (딤 배경, ESC 키, 외부 클릭)
  - [ ] Open Link → 외부 링크가 새 탭에서 올바르게 열림
  - [ ] Swap With → hover/click 시 컴포넌트 변형이 전환됨
  - [ ] 애니메이션 → duration, easing이 Figma 설정과 일치
- [ ] **접근성**: 시맨틱 HTML, alt 텍스트, 키보드 내비게이션, 모달 포커스 트랩

#### 전달

- 최종 코드를 사용자에게 전달한다
- 남은 불일치가 있으면 목록으로 안내한다
- 프로젝트에 기존 디자인 시스템이 있다면 토큰 매핑 내역을 안내한다

---

## MCP 출력 처리 규칙 (중요)

### MCP 출력은 참고 자료이다 — 복사하여 사용하지 않는다

Figma MCP의 `get_design_context`가 반환하는 코드는 Figma 레이어 구조를 그대로 변환한 것이다. 이 코드에는 다음 문제가 있다:

1. **과도한 div 중첩** — Figma의 모든 Frame이 div가 됨
2. **비표준 CSS 클래스** — `content-stretch` 등 MCP 전용 유틸리티
3. **이스케이프된 CSS 변수** — `var(--spacing\/space-24, 24px)` (브라우저에서 동작 안함)
4. **인라인 SVG data URI** — 그라디언트가 인라인 SVG로 표현됨 (불안정)
5. **data 속성** — `data-name`, `data-node-id` (프로덕션 불필요)
6. **Tailwind + React 고정 출력** — 대상 프레임워크와 무관하게 항상 React+Tailwind

### MCP 출력에서 추출할 것

| 추출 대상 | 사용 방법 |
|-----------|----------|
| 에셋 URL (const 선언) | fetch하여 프로젝트에 저장 |
| spacing, padding 수치 | CSS에 px 값으로 직접 사용 |
| font-size, weight, line-height | CSS typography에 적용 |
| color hex/rgba 값 | CSS color에 적용 |
| flex 방향, gap, alignment | CSS layout에 적용 |
| 텍스트 콘텐츠 | HTML에 그대로 사용 |
| SVG gradient 색상 스톱 | CSS gradient로 변환 |

### SVG 그라디언트 → CSS 변환

MCP 출력:
```jsx
style={{ backgroundImage: "url('data:image/svg+xml;utf8,<svg>...<radialGradient>...<stop stop-color=\"rgba(15,76,92,1)\" offset=\"0.5\"/>...')" }}
```

CSS 변환:
```css
background: radial-gradient(ellipse at 20% 0%, #0f4c5c 50%, #1a6977 62.5%, #268691 75%, #3cc0c7 100%);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
```

**변환 규칙:** SVG `<stop>` 요소에서 `stop-color`와 `offset`을 추출하여 CSS gradient 스톱으로 변환한다.

**자주 사용되는 그라디언트 패턴:**
```css
/* 숫자/제목 텍스트 그라디언트 (teal 계열) */
.gradient-text-teal {
  background: radial-gradient(ellipse at 20% 0%, #0f4c5c 50%, #1a6977 62.5%, #268691 75%, #3cc0c7 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
/* 프로그레스 바 그라디언트 (보라→파랑→청록) */
.gradient-bar {
  background: linear-gradient(90deg, #a258a2 0%, #6059a7 26%, #2860a3 53%, #009ede 79%, #46bc96 100%);
  height: 12px;
  border-radius: 9999px;
}
/* 섹션 배경 그라디언트 (투명 → 색상) */
.section-fade-bg {
  background: linear-gradient(to bottom, rgba(249,250,251,0), #f9fafb);
}
```

---

## 반응형 전략

기본적으로 데스크톱 퍼스트로 구현한다 (Figma가 보통 데스크톱 레이아웃):

```
데스크톱: 1920px (Figma 기본)
xl: 1440px ~
lg: 1024px ~
md: 768px ~
sm: 640px ~
모바일: ~ 639px
```

**page-padding 반응형:**
- 1920px: 320px (Figma 기본값)
- 1440px: 160px
- 1024px: 80px
- 768px: 40px
- 640px이하: 20px

사용자가 특정 브레이크포인트를 요청하면 해당 구간에 맞춰 작업한다.

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

### 예시 1: React + Tailwind (전체 페이지)

사용자: "https://figma.com/design/xxx/MyApp?node-id=42-15 를 React로 변환해줘."

```
Phase 1: 전체 구조 파악
  ├── get_design_context → 메타데이터 (섹션 목록)
  ├── get_screenshot → 전체 스크린샷 저장
  └── 섹션 목록: Header, Hero, Section01~06, Footer

Phase 2: 스켈레톤
  ├── App 컴포넌트에 섹션 컨테이너 배치
  ├── CSS 변수 정의 (colors, spacing, typography)
  └── 웹폰트 로딩 설정

Phase 3: 섹션별 구현 (반복)
  ├── Header:
  │   ├── get_design_context(nodeId="header-id")
  │   ├── get_screenshot(nodeId="header-id")
  │   ├── 레이아웃 + 스타일 구현
  │   └── 스크린샷 비교 검증
  ├── Hero:
  │   ├── get_design_context(nodeId="hero-id")
  │   └── ... (동일 절차)
  └── ... (나머지 섹션)

Phase 4: 전체 비교 & 최종 검증
```

### 예시 2: Vue + CSS (부분 구현)

사용자: "피그마 디자인에서 헤더와 히어로만 Vue로 만들어줘."

```
Phase 1: 전체 구조 파악 → 헤더, 히어로 nodeId 식별
Phase 2: 스켈레톤 (헤더 + 히어로만)
Phase 3:
  ├── Header: get_design_context → 구현 → 비교
  └── Hero: get_design_context → 구현 → 비교
Phase 4: 전체 비교 & 통합 가이드 제공
```

## 트러블슈팅

### 디자인 데이터가 잘림
`get_design_context`를 전체 노드에 호출하면 메타데이터만 반환된다. 이것이 정상이다.
→ 메타데이터에서 섹션 nodeId를 추출하여 각 섹션별로 개별 호출한다.

### MCP 출력 코드가 브라우저에서 깨짐
MCP 코드를 그대로 사용하면 깨진다.
→ MCP 코드를 복사하지 말고, 수치와 에셋만 추출하여 직접 코드를 작성한다.
→ `var(--spacing\/space-24, 24px)` 같은 이스케이프 변수는 `24px`로 직접 사용한다.

### 구현 결과가 디자인과 다름
→ 해당 섹션의 `get_screenshot`과 브라우저 스크린샷을 나란히 비교한다.
→ spacing, color, typography 값을 `get_design_context` 데이터에서 정확히 추출한다.
→ 특히 **버튼**(크기, 색상, radius)과 **그라디언트**(텍스트, 배경)를 재확인한다.

### 에셋 로딩 안 됨
→ Figma MCP의 에셋 URL은 7일간 유효하다. URL을 fetch하여 프로젝트에 저장한다.
→ 외부 아이콘을 임의로 추가하지 않는다 — Figma 에셋만 사용한다.

### 그라디언트가 표시 안 됨
→ 인라인 SVG data URI를 사용하면 불안정하다.
→ CSS gradient로 변환한다: SVG의 `<stop>` 색상을 CSS gradient 스톱으로 매핑.
→ 텍스트 그라디언트: `-webkit-background-clip: text` + `-webkit-text-fill-color: transparent` 확인.

### 폰트가 다르게 보임
→ Google Fonts 링크가 HTML에 포함되었는지 확인한다.
→ `preconnect` 태그가 있는지 확인한다.
→ MCP의 `'Noto_Sans_KR:Bold'` → `'Noto Sans KR'` + `font-weight: 700`으로 변환했는지 확인한다.

### 반복 컴포넌트의 높이/스타일이 균일하게 나옴
Figma에서 동일 컴포넌트를 반복 사용하더라도 인스턴스마다 height, border, color가 다를 수 있다.
→ `get_metadata` 또는 `get_design_context` 응답에서 동일 name의 instance를 모두 찾아 **각각의 height를 비교**한다.
→ 높이가 다르면 **커넥터/연결선 길이 차이**를 의미한다 — 각 인스턴스에 **개별 height를 적용**한다.
→ 탭/네비 컴포넌트 중 하나만 border-bottom이나 bold가 있으면 **활성 상태** — 해당 인스턴스에만 활성 스타일을 적용한다.
→ 같은 버튼 컴포넌트지만 bg/border 색상이 다르면 **filled vs outlined 변형** — 각 버튼에 개별 스타일을 적용한다.

### 타임라인/연혁이 지그재그로 잘못 구현됨
타임라인 컴포넌트에서 아이템별 height가 다르다고 해서 지그재그(위/아래 교차) 배치가 아니다.
→ **Figma 스크린샷을 반드시 확인**하여 아이템의 실제 흐름 방향을 파악한다.
→ 대부분의 타임라인은 모든 아이템이 **동일한 방향**(위→아래: 아이콘→연도→도트→텍스트)으로 흐른다.
→ height 차이는 **커넥터(세로 연결선) 길이**와 **텍스트 줄 수**에서 발생한다.
→ 지그재그 패턴은 프로세스 단계(Step 1 위, Step 2 아래, Step 3 위...)처럼 **명시적으로 위-아래 교차가 보일 때만** 사용한다.

### 헤더 배경이 디자인과 다름
헤더에 반투명 배경 + 블러 효과가 적용된 경우가 흔하다.
→ MCP 출력에서 `backdrop-blur`, `bg-[rgba(...)]` 클래스를 확인한다.
→ `bg-white`(불투명)로 단순화하지 않는다 — `background: rgba(255,255,255,0.92)` + `backdrop-filter: blur(5px)` 등을 정확히 적용한다.
→ 헤더의 **padding**도 콘텐츠 섹션과 다를 수 있다 (예: 헤더 `px-120px`, 콘텐츠 `px-320px`).

### 로고 이미지가 디자인과 다름
Figma의 로고가 여러 SVG 파트(Union, Vector 등)로 구성된 경우가 있다.
→ `get_design_context`에서 반환된 **에셋 URL을 반드시 다운로드**하여 사용한다.
→ 로고를 텍스트나 대체 이미지로 임의 구현하지 않는다.
→ 다운로드한 이미지가 Figma 스크린샷과 동일한지 비교 확인한다.
