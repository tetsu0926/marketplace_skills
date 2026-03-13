---
name: figma-to-code
description: Figma 디자인을 프로덕션 코드로 변환한다. 피그마 URL을 받아 React, Vue, Next.js, Svelte, Angular, HTML 코드를 생성하며 Tailwind, Bootstrap, styled-components, CSS Modules, 순수 CSS를 지원한다. "피그마를 코드로", "Figma to React", "피그마 변환", "디자인 구현", "figma url을 변환", "피그마 디자인을 코드로" 등 Figma URL과 함께 코드 생성을 요청할 때 반드시 이 스킬을 사용한다. 사용자가 Figma URL을 제공하고 특정 프레임워크나 CSS 방식을 언급하면 항상 이 스킬을 트리거한다.
metadata:
  mcp-server: figma
---

# Figma-to-Code

Figma 디자인을 픽셀 퍼펙트 프로덕션 코드로 변환하는 스킬.

## 전제 조건

- **Figma MCP 서버** 연결 필수
- **Playwright MCP 서버** 연결 권장
- 사용자 제공: Figma URL + 대상 프레임워크 + CSS 방식

## Context 관리 (핵심)

**이 스킬의 최대 위험은 context 소진이다. 아래 규칙을 반드시 따른다.**

### 규칙 1: MCP 응답 즉시 요약 후 폐기

`get_design_context` 호출 후, 전체 코드를 context에 유지하지 않는다. **즉시 필요한 값만 추출하여 요약**한다:

```
[섹션명 요약]
- 에셋: img1="URL1", img2="URL2"
- 레이아웃: flex-col, gap:24px, padding:80px 320px
- 타이포: h1(48px/700/#111827), body(24px/400/#374151)
- 색상: bg:#fff, accent:#0f4c5c, border:#e5e7eb
- 텍스트: "제목 텍스트", "본문 텍스트..."
- 그라디언트: radial(#0f4c5c 50%, #268691 75%, #3cc0c7 100%)
- 버튼: 248x64, bg:#0f4c5c, radius:4px, text:22px/500/white
- 인스턴스 변형: [동일 컴포넌트의 인스턴스별 차이 — 아래 "규칙 6" 참조]
- 활성/선택 상태: [활성 탭 border-bottom:4px #a258a2, font-weight:700 등]
- 버튼 변형: [btn1: filled(bg:#0f4c5c, text:white), btn2: outlined(border:#a258a2, bg:#f9fafb)]
```

이 요약만 기억하고 코드를 작성한다.

### 규칙 2: 스크린샷 최소화

- `get_design_context`는 기본으로 스크린샷을 포함한다. **별도로 `get_screenshot`을 호출하지 않는다.**
- 비교 검증 시에만 `get_screenshot`을 추가로 호출한다.
- 전체 페이지 스크린샷은 Phase 1에서 1회만 캡처한다.

### 규칙 3: reference 파일은 필요한 것만 로드

- `references/frameworks/{framework}.md` — 해당 프레임워크만
- `references/css-approaches/{css-approach}.md` — 해당 CSS 방식만
- `references/design-token-resolution.md` — 토큰 처리 필요 시에만
- 나머지 reference는 로드하지 않는다

### 규칙 4: 서브에이전트로 섹션 구현 위임

**큰 페이지(5개 이상 섹션)는 반드시 Agent 도구를 사용하여 섹션 구현을 위임한다.**

각 서브에이전트에게 전달할 것:
- 섹션 nodeId, fileKey
- 프레임워크 + CSS 방식
- 프로젝트의 기존 파일 구조와 컨벤션
- 디자인 토큰/CSS 변수 정의 (이미 생성된 것)
- 구현할 파일 경로

서브에이전트가 하는 것:
1. `get_design_context` 호출 → 값 추출
2. 코드 작성 → 파일 저장
3. 완료 보고 (생성된 파일 목록)

**메인 context에는 서브에이전트의 전체 작업 내용이 포함되지 않는다.**

### 규칙 5: 섹션 그룹핑

작은 섹션은 묶어서 처리한다:
- Header + Tab Navigation → 1개 서브에이전트
- Hero → 1개 (배경 이미지가 복잡하므로 단독)
- Section01 + Section02 → 1개 (또는 크기에 따라 분리)
- Footer → 1개

목표: **서브에이전트 호출을 5개 이내로 유지**

### 규칙 6: 컴포넌트 인스턴스별 상세 분석 (핵심)

**동일 컴포넌트가 반복 사용될 때, 각 인스턴스의 개별 속성을 반드시 추출한다.**

이 규칙이 필요한 이유: Figma에서 동일 컴포넌트를 여러 번 배치하더라도, 인스턴스마다 **높이, 텍스트 줄 수, 활성 상태, 색상 변형**이 다를 수 있다. `get_design_context`나 `get_metadata` 응답에서 각 인스턴스의 개별 치수를 반드시 확인해야 한다.

#### 추출해야 할 인스턴스별 속성:

1. **개별 치수 (width/height)**: 같은 컴포넌트라도 인스턴스마다 height가 다르면 교차 패턴(alternating pattern) 등의 디자인 의도가 있음
2. **텍스트 줄 수/내용**: 각 인스턴스의 텍스트 내용과 줄 바꿈 위치
3. **활성/비활성 상태**: 탭, 버튼 등에서 선택된 항목 vs 기본 항목의 스타일 차이
   - 선택 상태: border 색상/두께, font-weight, text 색상 변화
4. **버튼 변형(variant)**: 같은 버튼 컴포넌트라도 filled/outlined/ghost 등 변형이 있음
   - bg color, border color, border width, text color를 각각 추출

#### 요약 템플릿 (인스턴스별):

```
[컴포넌트 리스트: component/list/history/base × 7개]
- 인스턴스1: nodeId="xxx", h=520px, text="굿서비스 대리운전 설립" (1줄)
- 인스턴스2: nodeId="yyy", h=420px, text="법인사업자 전환" (1줄)
- 인스턴스3: nodeId="zzz", h=720px, text="인천 아시안 게임 VIP..." (3줄)
→ 패턴: 홀수(1,3,5,7)=길고, 짝수(2,4,6)=짧음

[버튼 그룹: btn/sub × 2개]
- btn1: "계약문의" — filled, bg:#0f4c5c, text:white, rounded-full
- btn2: "기사등록" — outlined, border:#a258a2, bg:#f9fafb, text:#111827, rounded-full

[탭 내비게이션: btn/tabs/menu × 5개]
- 활성탭: "굿서비스 소개" — border-bottom:4px solid #a258a2, font-weight:700, text:#0f4c5c
- 비활성탭: font-weight:500, text:#111827, border:none
```

#### 분석 방법:

1. `get_metadata`에서 동일 name의 instance 목록 수집 → 각각의 x, y, width, height 확인
2. `get_design_context`에서 각 인스턴스의 className/style 비교 → 차이점 추출
3. 특히 **height 차이**, **border 유무**, **font-weight 차이**, **color 차이**에 주목
4. 차이가 있으면 반드시 요약에 인스턴스별로 기록

### 규칙 7: 레이어 속성 심층 파악

`get_design_context` 응답에서 다음 정보를 반드시 추출한다:

1. **레이아웃 모드**: flex direction, align, justify, gap, padding — 컨테이너와 자식 모두
2. **색상 체계**: bg, text, border, shadow — 특히 semantic color(var)의 fallback 값
3. **선택/활성 색상**: 활성 상태의 정확한 색상 (border-color, text-color, background)
4. **에셋과 아이콘**: 이미지 URL + 크기 + 위치 (absolute/relative)
5. **컴포넌트 설명(description)**: MCP 응답의 "Component descriptions" 섹션에 사용 지침이 있으면 반드시 따른다
6. **숨겨진 요소**: `hidden="true"` 속성이 있는 노드는 구현에서 제외

## 워크플로우

### Phase 1: 전체 구조 파악 (메인 context)

#### 1-1: URL 파싱

URL: `figma.com/design/:fileKey/:fileName?node-id=1-2`
- fileKey: `/design/` 뒤 세그먼트
- nodeId: `node-id` 파라미터 (하이픈 형식 그대로)
- Branch URL: `branch/:branchKey/` → branchKey를 fileKey로

#### 1-2: 설정 확인

1. 프레임워크 확인 (React/Vue/Next.js/Svelte/Angular/HTML)
2. CSS 방식 확인 (Tailwind/Bootstrap/styled-components/CSS Modules/Pure CSS)
3. 프로젝트 코드 분석 (package.json, 기존 컴포넌트, 파일 구조)

#### 1-3: 전체 구조 수집

```
get_design_context(fileKey, nodeId)  → 메타데이터 (섹션 트리)
get_screenshot(fileKey, nodeId)      → 전체 스크린샷 (1회만)
```

메타데이터에서 **섹션 목록 + nodeId** 추출 → 작업 로드맵으로 사용.

#### 1-4: 컴포넌트 인스턴스 심층 분석 (필수)

**메타데이터 또는 `get_design_context` 응답에서 반복 컴포넌트를 발견하면, 반드시 인스턴스별 차이를 분석한다.**

절차:
1. 동일 `name`의 instance 노드를 모두 찾는다
2. 각 인스턴스의 **width, height, x, y** 값을 비교한다
3. 높이/너비가 다르면 → **각 인스턴스별 치수를 기록** (교차 패턴, 점진 증가 등)
4. 필요 시 개별 인스턴스에 `get_design_context`를 호출하여 세부 스타일 차이를 확인:
   - 활성/비활성 상태 (border, font-weight, color 차이)
   - 버튼 변형 (filled vs outlined — bg color, border color 차이)
   - 텍스트 내용과 줄 수

**분석 대상 패턴:**

| 패턴 | 감지 방법 | 추출할 것 |
|------|----------|----------|
| 교차 높이 (alternating) | 같은 컴포넌트 인스턴스의 height가 번갈아 변함 | 각 인스턴스의 개별 height |
| 탭/네비 활성 상태 | 같은 탭 컴포넌트 중 하나만 border/bold | 활성 탭의 border 색상/두께, font-weight |
| 버튼 변형 | 같은 btn 컴포넌트지만 bg/border가 다름 | 각 버튼의 bg, border, text color |
| 카드 크기 변형 | 같은 카드 컴포넌트지만 w/h가 다름 | 각 카드의 개별 dimensions |

**이 단계를 건너뛰면 결과물이 디자인과 크게 달라진다. 반드시 수행한다.**

### Phase 2: 기반 코드 생성 (메인 context)

1. **웹폰트 로딩** 설정 (Google Fonts link)
2. **CSS 변수/토큰** 정의 — `references/design-token-resolution.md` 참조
3. **전체 스켈레톤** 생성 — 빈 섹션 컨테이너 배치
4. **공통 스타일** 정의 (body, 섹션 기본 padding, 반응형 breakpoint)

### Phase 3: 섹션별 구현 (서브에이전트 위임)

**각 섹션/그룹을 Agent 도구로 서브에이전트에 위임한다.**

서브에이전트 프롬프트 템플릿:

```
Figma 디자인의 [섹션명]을 구현해줘.

정보:
- fileKey: "xxx", nodeId: "yyy"
- 프레임워크: React + Tailwind (또는 해당 스택)
- 파일 경로: src/components/SectionName.tsx
- CSS 변수는 이미 정의됨: [변수 목록 또는 파일 경로]
- 인스턴스별 상세 정보: [메인에서 분석한 인스턴스별 치수/상태 목록을 여기에 전달]

작업 절차:
1. get_design_context(fileKey="xxx", nodeId="yyy") 호출
2. 반환된 코드에서 에셋 URL, 수치, 텍스트만 추출 (코드 복사 금지)
3. **인스턴스별 차이 분석** (핵심):
   a. 동일 컴포넌트의 인스턴스를 모두 찾아 className/style 비교
   b. height, border, font-weight, color 등의 차이를 기록
   c. 각 인스턴스에 개별 스타일을 적용 (하드코딩 OK — 디자인 의도 그대로 반영)
4. MCP 코드의 var(--spacing\/...) 같은 이스케이프 변수는 fallback 값(px)으로 직접 사용
5. 인라인 SVG data URI 그라디언트는 CSS gradient로 변환
6. 스크린샷을 참고하여 직접 코드 작성
7. 에셋 URL을 fetch하여 프로젝트에 저장
8. 파일 저장 후 완료 보고

인스턴스 분석 체크리스트:
- [ ] 반복 컴포넌트의 각 인스턴스 높이가 동일한가? → 다르면 각각 개별 height 적용
- [ ] 탭/네비게이션에 활성 상태가 있는가? → border-bottom 색상/두께, font-weight 차이 적용
- [ ] 버튼이 여러 개인가? → 각 버튼의 bg, border, text color를 개별 적용 (filled vs outlined)
- [ ] 리스트 아이템의 텍스트 줄 수가 다른가? → 각 아이템별 텍스트와 줄 바꿈을 정확히 재현

주의:
- MCP 출력 코드를 그대로 복사하면 안 된다 (비표준 클래스, 이스케이프 변수 포함)
- data-name, data-node-id 속성 제거
- content-stretch 등 비표준 클래스 제거
- 불필요한 div 중첩 제거
- 동일 컴포넌트라도 인스턴스별로 다른 속성이 있으면 반드시 개별 적용
```

**서브에이전트 병렬 실행:**
독립적인 섹션은 병렬로 실행하여 시간을 절약한다.

### Phase 4: 통합 & 검증 (메인 context)

1. 서브에이전트 결과 확인 (생성된 파일 목록)
2. 메인 컴포넌트에서 섹션 import 연결
3. Playwright로 전체 비교 (가능한 경우)
4. 불일치 수정 (해당 섹션 파일만 수정)

## MCP 출력 처리 (핵심 규칙)

### 절대 하지 말 것

- MCP 코드를 그대로 복사하여 사용
- `var(--spacing\/space-24, 24px)` 형태의 이스케이프 변수를 코드에 포함
- `content-stretch`, `data-name`, `data-node-id` 등 MCP 전용 속성 포함
- 인라인 SVG data URI 그라디언트를 그대로 사용
- `get_design_context` + `get_screenshot`을 동시 호출 (context 낭비)

### 반드시 할 것

- MCP 코드에서 **수치, 색상, 에셋 URL, 텍스트만 추출**
- 이스케이프 변수의 **fallback 값을 직접 사용** (예: 24px)
- SVG 그라디언트 → **CSS gradient로 변환**
- 폰트: `'Noto_Sans_KR:Bold'` → `'Noto Sans KR'` + `font-weight: 700`
- 스크린샷을 보면서 **직접 코드 작성**

### 그라디언트 변환 요약

| MCP 출력 | CSS 변환 |
|----------|---------|
| 인라인 SVG radialGradient | `radial-gradient(ellipse at 20% 0%, ...)` |
| 인라인 SVG linearGradient | `linear-gradient(90deg, ...)` |
| 텍스트에 적용 시 | + `-webkit-background-clip: text; -webkit-text-fill-color: transparent` |

## 반응형

데스크톱 퍼스트 (Figma 기본이 보통 1920px):

| 브레이크포인트 | page-padding |
|-------------|-------------|
| 1920px | 320px |
| 1440px | 160px |
| 1024px | 80px |
| 768px | 40px |
| <640px | 20px |

## 트러블슈팅

| 문제 | 해결 |
|------|------|
| context 부족 | 서브에이전트 위임, 스크린샷 최소화, MCP 응답 즉시 요약 |
| 디자인 데이터 잘림 | 전체 노드 → 메타데이터만 반환됨 (정상). 섹션별로 개별 호출 |
| MCP 코드가 깨짐 | 코드 복사 금지. 수치/에셋만 추출하여 직접 작성 |
| 그라디언트 미표시 | SVG → CSS gradient 변환. `-webkit-background-clip: text` 확인 |
| 폰트 다름 | Google Fonts link + preconnect 확인 |
| 버튼 스타일 다름 | MCP 데이터에서 정확한 크기/색상/radius 추출하여 적용 |
| 에셋 미로드 | URL fetch하여 프로젝트에 저장. 7일 유효 |
| 리스트 아이템 높이 불일치 | 규칙 6 적용: 각 인스턴스의 개별 height를 메타데이터에서 추출하여 하드코딩 |
| 탭/네비 활성 상태 누락 | 규칙 6 적용: 인스턴스별 className 비교 → border/font-weight/color 차이 추출 |
| 버튼 변형(filled/outlined) 구분 안됨 | 규칙 6 적용: 각 버튼 인스턴스의 bg, border, text color를 개별 추출 |
| 교차 패턴(alternating) 미반영 | 메타데이터에서 인스턴스별 height 비교 → 홀짝 또는 교차 패턴 감지 → 개별 적용 |
| 컴포넌트 내부 텍스트 줄 수 다름 | 각 인스턴스의 텍스트 내용/줄 바꿈을 정확히 추출, 텍스트 높이에 따라 컨테이너 높이 조정 |
