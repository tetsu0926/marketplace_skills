# Tailwind CSS 매핑 규칙

Figma 디자인 속성을 Tailwind CSS 유틸리티 클래스로 변환할 때의 가이드.

## 레이아웃

### Flexbox

| Figma Auto Layout | Tailwind 클래스 |
|-------------------|----------------|
| Horizontal | `flex flex-row` |
| Vertical | `flex flex-col` |
| Wrap | `flex-wrap` |
| Gap: 4px | `gap-1` |
| Gap: 8px | `gap-2` |
| Gap: 12px | `gap-3` |
| Gap: 16px | `gap-4` |
| Gap: 20px | `gap-5` |
| Gap: 24px | `gap-6` |
| Gap: 32px | `gap-8` |

### 정렬

| Figma 정렬 | Tailwind 클래스 |
|-----------|----------------|
| 주축 Start | `justify-start` |
| 주축 Center | `justify-center` |
| 주축 End | `justify-end` |
| Space between | `justify-between` |
| 교차축 Start | `items-start` |
| 교차축 Center | `items-center` |
| 교차축 End | `items-end` |
| 교차축 Stretch | `items-stretch` |

### 크기

| Figma 설정 | Tailwind 클래스 |
|-----------|----------------|
| Fill container | `flex-1` 또는 `w-full` |
| Fixed width | `w-[{value}px]` |
| Fixed height | `h-[{value}px]` |
| Hug contents | `w-fit` |
| Max width | `max-w-[{value}px]` |
| Min width | `min-w-[{value}px]` |

## 간격 (Spacing)

### Tailwind 기본 스케일

| px | Tailwind |
|----|---------|
| 0 | `0` |
| 1 | `px` |
| 2 | `0.5` |
| 4 | `1` |
| 6 | `1.5` |
| 8 | `2` |
| 10 | `2.5` |
| 12 | `3` |
| 14 | `3.5` |
| 16 | `4` |
| 20 | `5` |
| 24 | `6` |
| 28 | `7` |
| 32 | `8` |
| 36 | `9` |
| 40 | `10` |
| 48 | `12` |
| 56 | `14` |
| 64 | `16` |
| 80 | `20` |
| 96 | `24` |

기본 스케일에 없는 값은 arbitrary value 사용: `p-[13px]`, `gap-[22px]`

### Padding

```
padding: 16px           → p-4
padding: 16px 24px      → py-4 px-6
padding: 12px 16px 20px 16px → pt-3 px-4 pb-5
```

### Margin

```
margin: 0 auto → mx-auto
margin-top: 16px → mt-4
margin-bottom: 24px → mb-6
```

## 타이포그래피

### Font Size

| Figma px | Tailwind 클래스 |
|----------|----------------|
| 12px | `text-xs` |
| 14px | `text-sm` |
| 16px | `text-base` |
| 18px | `text-lg` |
| 20px | `text-xl` |
| 24px | `text-2xl` |
| 30px | `text-3xl` |
| 36px | `text-4xl` |
| 48px | `text-5xl` |
| 60px | `text-6xl` |

맞는 클래스가 없으면: `text-[22px]`

### Font Weight

| Figma Weight | Tailwind 클래스 |
|-------------|----------------|
| 100 (Thin) | `font-thin` |
| 200 (Extra Light) | `font-extralight` |
| 300 (Light) | `font-light` |
| 400 (Regular) | `font-normal` |
| 500 (Medium) | `font-medium` |
| 600 (SemiBold) | `font-semibold` |
| 700 (Bold) | `font-bold` |
| 800 (Extra Bold) | `font-extrabold` |
| 900 (Black) | `font-black` |

### Line Height

| Figma | Tailwind 클래스 |
|-------|----------------|
| 1.0 | `leading-none` |
| 1.25 | `leading-tight` |
| 1.375 | `leading-snug` |
| 1.5 | `leading-normal` |
| 1.625 | `leading-relaxed` |
| 2.0 | `leading-loose` |

정확한 값: `leading-[24px]`

### 기타 텍스트

```
text-align: center → text-center
text-align: right → text-right
letter-spacing: 0.05em → tracking-wide
text-transform: uppercase → uppercase
text-decoration: underline → underline
```

## 색상

### 기본 Tailwind 색상

Figma의 색상이 Tailwind 기본 팔레트와 정확히 일치하면 사용:
```
#3B82F6 → bg-blue-500, text-blue-500
#EF4444 → bg-red-500, text-red-500
#10B981 → bg-emerald-500
#F59E0B → bg-amber-500
```

### Arbitrary Color

정확한 색상 일치를 위해 hex 값 직접 사용:
```
bg-[#1E40AF]
text-[#6B7280]
border-[#E5E7EB]
```

### 투명도

```
bg-black/50     → rgba(0,0,0,0.5)
text-white/80   → rgba(255,255,255,0.8)
bg-[#3B82F6]/20 → rgba(59,130,246,0.2)
```

## 효과

### Border Radius

| Figma px | Tailwind 클래스 |
|----------|----------------|
| 0 | `rounded-none` |
| 2px | `rounded-sm` |
| 4px | `rounded` |
| 6px | `rounded-md` |
| 8px | `rounded-lg` |
| 12px | `rounded-xl` |
| 16px | `rounded-2xl` |
| 24px | `rounded-3xl` |
| 9999px | `rounded-full` |

### Shadow

| Figma Shadow | Tailwind 클래스 |
|-------------|----------------|
| 작은 그림자 | `shadow-sm` |
| 중간 그림자 | `shadow` / `shadow-md` |
| 큰 그림자 | `shadow-lg` |
| 매우 큰 그림자 | `shadow-xl` / `shadow-2xl` |

정확한 값: `shadow-[0_4px_6px_-1px_rgba(0,0,0,0.1)]`

### Opacity

```
opacity: 0.5 → opacity-50
opacity: 0.8 → opacity-80
```

### Blur

```
filter: blur(4px) → blur-sm
filter: blur(8px) → blur
backdrop-filter: blur(12px) → backdrop-blur-md
```

## 반응형

모바일 퍼스트 — 기본이 모바일, 프리픽스로 확장:

```html
<div class="flex flex-col md:flex-row gap-4 md:gap-8">
  <aside class="hidden lg:block w-60">사이드바</aside>
  <main class="flex-1">콘텐츠</main>
</div>

<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- 카드들 -->
</div>

<h1 class="text-2xl md:text-4xl lg:text-5xl">제목</h1>
```

| 프리픽스 | 브레이크포인트 |
|---------|------------|
| (없음) | 0px+ (모바일) |
| `sm:` | 640px+ |
| `md:` | 768px+ |
| `lg:` | 1024px+ |
| `xl:` | 1280px+ |
| `2xl:` | 1536px+ |

## 인터랙션 상태

```html
<button class="
  bg-blue-500 text-white
  hover:bg-blue-600
  active:scale-95
  focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
  disabled:opacity-50 disabled:cursor-not-allowed
  transition-all duration-200
">
  버튼
</button>
```

## 그라디언트

```html
<div class="bg-gradient-to-r from-blue-500 to-purple-600">
  그라디언트 배경
</div>

<!-- 정확한 값 -->
<div class="bg-gradient-to-br from-[#667eea] via-[#764ba2] to-[#f093fb]">
  커스텀 그라디언트
</div>
```

## 주의사항

- Figma 값이 Tailwind 기본 스케일과 정확히 일치하지 않으면 arbitrary value `[]` 사용
- 프로젝트의 `tailwind.config`에 커스텀 테마가 있으면 그것을 우선 사용
- 너무 많은 arbitrary value가 필요하면 `tailwind.config`에 커스텀 값 추가를 제안
- 다크 모드가 필요하면 `dark:` 프리픽스 사용
