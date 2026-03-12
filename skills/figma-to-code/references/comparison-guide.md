# 비주얼 비교 가이드

Playwright를 사용하여 생성된 코드의 렌더링 결과를 Figma 스크린샷과 비교하는 방법론.

## 비교 프로세스 개요

```
[Figma 스크린샷] ← 비교 → [Playwright 브라우저 스크린샷]
         ↓                           ↓
    소스 오브 트루스            생성된 코드 렌더링
         ↓                           ↓
         └──── 불일치 발견 → 코드 수정 → 재비교 ────┘
                        (최대 3회 반복)
```

## Step 1: 코드 서빙 준비

### HTML 파일인 경우
생성된 HTML 파일을 직접 Playwright로 열 수 있다:
```
browser_navigate(url="file:///path/to/generated.html")
```

### 프레임워크 프로젝트인 경우
dev 서버가 실행 중이면 해당 URL을 사용:
```
browser_navigate(url="http://localhost:3000")
```

dev 서버가 없으면, 생성된 컴포넌트를 임시 HTML로 래핑하여 브라우저에서 직접 확인:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>/* 생성된 CSS */</style>
</head>
<body>
  <!-- 생성된 HTML -->
</body>
</html>
```

## Step 2: 스크린샷 캡처

### 기본 캡처
```
browser_navigate(url="...")
browser_take_screenshot()
```

### 반응형 비교 — 여러 뷰포트에서 캡처
```
browser_resize(width=375, height=812)     # 모바일
browser_take_screenshot()

browser_resize(width=768, height=1024)    # 태블릿
browser_take_screenshot()

browser_resize(width=1440, height=900)    # 데스크톱
browser_take_screenshot()
```

### 특정 영역만 캡처
부분 구현의 경우, 해당 영역만 캡처하여 비교한다.

## Step 3: 비교 체크포인트

Playwright 스크린샷과 Figma 스크린샷을 나란히 놓고 다음 항목을 순서대로 확인한다:

### 1순위: 레이아웃 구조
- [ ] 요소들의 배치 순서가 동일한가
- [ ] flex/grid 방향이 올바른가
- [ ] 요소 간 간격(gap)이 일치하는가
- [ ] 패딩이 일치하는가
- [ ] 전체 너비/높이 비율이 유사한가

### 2순위: 타이포그래피
- [ ] 폰트 크기가 일치하는가
- [ ] 폰트 굵기(weight)가 일치하는가
- [ ] 텍스트 색상이 일치하는가
- [ ] line-height가 유사한가 (텍스트 줄 간격)
- [ ] 텍스트 정렬(좌/중/우)이 일치하는가

### 3순위: 색상 & 배경
- [ ] 배경색이 정확히 일치하는가
- [ ] 그라디언트 방향과 색상이 일치하는가
- [ ] 텍스트 색상이 올바른가
- [ ] 보더 색상이 일치하는가

### 4순위: 효과 & 디테일
- [ ] border-radius가 일치하는가
- [ ] 그림자(shadow)가 유사한가
- [ ] opacity가 올바른가
- [ ] 이미지/아이콘 크기와 위치가 일치하는가

## Step 4: 불일치 수정 전략

### 간격(Spacing) 불일치
가장 흔한 문제. Figma 디자인 컨텍스트의 정확한 px 값을 다시 확인하고 적용한다.

```css
/* 잘못된 값 */
gap: 16px;
/* Figma 값 확인 후 수정 */
gap: 12px;
```

### 색상 불일치
Figma의 정확한 hex 값을 다시 확인한다. opacity가 적용된 색상인지도 확인.

```css
/* Figma 값: #3B82F6, opacity 80% */
color: rgba(59, 130, 246, 0.8);
```

### 폰트 불일치
웹폰트가 로드되지 않았을 수 있다. 폰트 로드 상태를 확인한다.

```
browser_evaluate(expression="document.fonts.ready.then(() => document.fonts.check('16px Inter'))")
```

### 레이아웃 구조 불일치
flex 방향, wrap 여부, align-items를 다시 확인한다. Figma의 auto-layout 설정을 정확히 반영한다.

## Step 5: 반복 규칙

### 반복 횟수
- **1차 비교**: 대부분의 불일치를 수정
- **2차 비교**: 세부 조정 (1-2px 차이, 미세한 색상 차이)
- **3차 비교**: 최종 확인. 이후 남은 불일치는 사용자에게 보고

### 3회 후에도 불일치가 있을 때
남은 차이를 목록으로 정리하여 사용자에게 보고한다:

```
## 남은 불일치 항목
1. [헤더 영역] 그림자 값이 미세하게 다름 — Figma: 0 4px 6px rgba(0,0,0,0.1), 현재: 유사하지만 blur 차이
2. [카드 섹션] 특정 폰트 weight가 브라우저에서 미세하게 다르게 렌더링됨
```

## Playwright 사용 불가 시 대체 방안

Playwright MCP가 연결되지 않은 환경에서는:

1. 생성된 코드를 파일로 저장한다
2. 사용자에게 브라우저에서 직접 열어 확인하도록 안내한다
3. Step 7의 검증 체크리스트를 꼼꼼히 수행한다
4. 코드 레벨에서 Figma 디자인 컨텍스트의 정확한 값을 사용했는지 한 번 더 확인한다

## 비교 시 허용 범위

완전히 동일한 렌더링은 브라우저 특성상 어려울 수 있다. 다음은 허용 가능한 차이:

- **안티앨리어싱**: 텍스트/곡선의 렌더링 차이 (브라우저마다 다름)
- **서브픽셀 렌더링**: 1px 미만의 위치 차이
- **폰트 렌더링**: 동일 폰트라도 Figma와 브라우저의 렌더링이 미세하게 다를 수 있음

다음은 반드시 수정해야 하는 차이:
- **2px 이상의 간격 차이**
- **색상 차이** (hex 값이 다른 경우)
- **폰트 크기/weight 차이**
- **레이아웃 구조 차이** (요소 배치 순서, flex 방향)
- **누락된 요소**
