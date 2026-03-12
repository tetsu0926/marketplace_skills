# Vue 변환 규칙

Figma 디자인을 Vue 3 SFC(Single File Component)로 변환할 때의 가이드.

## 컴포넌트 구조

```vue
<script setup lang="ts">
// Props (Figma variant → Vue props)
interface Props {
  variant?: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
})
</script>

<template>
  <div class="container">
    <slot />
  </div>
</template>

<style scoped>
/* CSS 방식에 따라 다름 */
</style>
```

## Figma 구조 → Template 매핑

| Figma 요소 | Vue Template |
|-----------|-------------|
| Frame | `<div>` (또는 시맨틱 태그) |
| Text | `<p>`, `<h1>`~`<h6>`, `<span>` |
| Rectangle | `<div>` (CSS로 스타일) |
| Image | `<img :src="imageUrl" alt="..." />` |
| Button (instance) | `<button>` 또는 기존 컴포넌트 |
| Input (instance) | `<input>` 또는 기존 컴포넌트 |
| Icon (SVG) | 인라인 SVG 또는 `<img>` |
| Group | `<div>` 또는 `<template>` |
| Component instance | 재사용 컴포넌트로 추출 |

## 컴포넌트 분리

Figma Component를 Vue SFC로 분리:

```vue
<!-- Card.vue -->
<script setup lang="ts">
interface Props {
  title: string
  description: string
  image: string
}

defineProps<Props>()
</script>

<template>
  <article class="card">
    <img :src="image" :alt="title" class="card-image" />
    <div class="card-content">
      <h3 class="card-title">{{ title }}</h3>
      <p class="card-description">{{ description }}</p>
    </div>
  </article>
</template>
```

## Variant → Props 매핑

```vue
<script setup lang="ts">
interface Props {
  type?: 'primary' | 'secondary' | 'ghost'
  size?: 'sm' | 'md' | 'lg'
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  type: 'primary',
  size: 'md',
  disabled: false,
})

// 동적 클래스 바인딩
const buttonClass = computed(() => [
  'button',
  `button--${props.type}`,
  `button--${props.size}`,
])
</script>

<template>
  <button :class="buttonClass" :disabled="disabled">
    <slot />
  </button>
</template>
```

## 리스트 렌더링

```vue
<template>
  <div class="card-grid">
    <Card
      v-for="item in items"
      :key="item.id"
      :title="item.title"
      :description="item.description"
      :image="item.image"
    />
  </div>
</template>
```

## 이벤트 핸들링

```vue
<template>
  <button @click="handleClick">Click</button>
  <input @input="handleInput" />
  <div @mouseenter="handleHover" @mouseleave="handleLeave" />
</template>
```

## CSS 스코핑

Vue의 `<style scoped>`는 컴포넌트별 CSS 격리를 제공:

```vue
<style scoped>
/* 이 컴포넌트에만 적용 */
.card {
  display: flex;
  flex-direction: column;
}
</style>
```

Tailwind 사용 시 `scoped`는 생략하고 클래스를 직접 적용.

## 파일 구조

```
src/
├── components/
│   ├── Header.vue
│   ├── Card.vue
│   └── Button.vue
├── views/ (또는 pages/)
│   └── Home.vue
└── assets/
    └── styles/
```

## 접근성

- `img`에 `alt` 속성
- 버튼/링크에 의미 있는 텍스트
- 아이콘 버튼에 `aria-label`
- 폼에 `<label>` + `for` 속성
