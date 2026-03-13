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
- 높이가 다른 리스트 아이템 → 교차 패턴(alternating) 가능성 (예: 타임라인에서 홀수=길고, 짝수=짧음)
- 같은 버튼 컴포넌트지만 bg/border 색상이 다름 → filled vs outlined 변형
- 같은 탭 컴포넌트지만 하나만 border/bold → 활성 상태 표시

이 정보는 Phase 3 섹션 구현에서 각 인스턴스에 개별 스타일을 적용할 때 사용한다.

#### 1-4: 프로젝트 기반 작업 & 디자인 토큰

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
   - 각 인스턴스의 **height가 다르면** 개별 값을 기록 (교차 패턴 등)
   - **border, font-weight, color가 다르면** 활성/비활성 상태 구분
   - 버튼의 **bg, border-color가 다르면** filled/outlined 변형 구분
   - 각 인스턴스의 **텍스트 줄 수**가 다르면 개별 기록

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
```

**특히 padding의 top/bottom이 비대칭인 경우가 매우 흔하다.** `py-120px`로 단순화하지 말고 `pt-200px pb-40px`처럼 개별 값을 정확히 추출한다.

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

/* 교차 높이 타임라인 (각 인스턴스별 height 개별 적용) */
.timeline { display: flex; align-items: flex-start; justify-content: space-between; }
.timeline-item { width: 153px; /* height는 각 아이템마다 다름 */ }

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

**효과:**
- `box-shadow`, `border-radius`, `opacity`
- `border` — 정확한 width, style, color

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

**Step F: 섹션 비교 검증**

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
- [ ] 간격이 정확한가 (padding, gap, margin)
- [ ] 폰트가 맞는가 (family, size, weight, color)
- [ ] 배경색/그라디언트가 일치하는가
- [ ] 버튼 크기, 색상, radius가 맞는가 (filled/outlined 변형 구분 포함)
- [ ] 이미지가 올바르게 표시되는가
- [ ] 보더, 그림자, radius가 맞는가
- [ ] 반복 컴포넌트의 **인스턴스별 치수**가 맞는가 (높이 교차 패턴 등)
- [ ] 탭/네비게이션의 **활성 상태**(border, font-weight, color)가 맞는가

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
- [ ] **인터랙션**: hover, active 상태 동작
- [ ] **접근성**: 시맨틱 HTML, alt 텍스트, 키보드 내비게이션

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
→ 높이가 다르면 교차 패턴(alternating)이나 텍스트 줄 수 차이를 의미한다 — 각 인스턴스에 **개별 height를 적용**한다.
→ 탭/네비 컴포넌트 중 하나만 border-bottom이나 bold가 있으면 **활성 상태** — 해당 인스턴스에만 활성 스타일을 적용한다.
→ 같은 버튼 컴포넌트지만 bg/border 색상이 다르면 **filled vs outlined 변형** — 각 버튼에 개별 스타일을 적용한다.
