# React 변환 규칙

Figma 디자인을 React 컴포넌트로 변환할 때의 가이드.

## 컴포넌트 구조

```tsx
// 함수형 컴포넌트 (기본)
interface ComponentProps {
  // Figma variant가 있으면 props로 매핑
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children?: React.ReactNode;
}

export function Component({ variant = 'primary', size = 'md', children }: ComponentProps) {
  return (
    <div className="container">
      {children}
    </div>
  );
}
```

## Figma 구조 → JSX 매핑

| Figma 요소 | React/JSX |
|-----------|-----------|
| Frame | `<div>` (또는 시맨틱 태그) |
| Text | `<p>`, `<h1>`~`<h6>`, `<span>` |
| Rectangle | `<div>` (CSS로 스타일) |
| Image | `<img src={...} alt="..." />` |
| Button (instance) | `<button>` 또는 기존 Button 컴포넌트 |
| Input (instance) | `<input>` 또는 기존 Input 컴포넌트 |
| Icon (SVG) | 인라인 SVG 또는 `<img>` |
| Group | `<div>` 또는 `<>...</>` (Fragment) |
| Component instance | 재사용 컴포넌트로 추출 |

## 컴포넌트 분리 기준

Figma에서 **Component**로 정의된 요소는 React 컴포넌트로 분리한다:
- 반복되는 요소 (카드, 리스트 아이템)
- Variant가 있는 요소 (버튼, 배지)
- 독립적인 UI 블록 (헤더, 푸터, 사이드바)

```tsx
// Figma Component "Card" → React Component
interface CardProps {
  title: string;
  description: string;
  image: string;
}

export function Card({ title, description, image }: CardProps) {
  return (
    <article className="card">
      <img src={image} alt={title} className="card-image" />
      <div className="card-content">
        <h3 className="card-title">{title}</h3>
        <p className="card-description">{description}</p>
      </div>
    </article>
  );
}
```

## Variant → Props 매핑

Figma Component의 variant 속성을 React props로 변환:

```tsx
// Figma: Button component with variants
// - Type: Primary, Secondary, Ghost
// - Size: Small, Medium, Large
// - State: Default, Hover, Disabled

interface ButtonProps {
  type?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}
```

## 리스트 렌더링

Figma에서 동일한 컴포넌트가 반복되면 `map`으로 렌더링:

```tsx
<div className="card-grid">
  {items.map((item) => (
    <Card key={item.id} {...item} />
  ))}
</div>
```

## 이벤트 핸들링

인터랙티브 요소에는 적절한 이벤트 핸들러를 추가:

```tsx
<button onClick={handleClick}>
<input onChange={handleChange} />
<a href={url} onClick={handleNavigation}>
```

## 접근성

- `img`에는 반드시 `alt` 속성
- 버튼/링크에는 의미 있는 텍스트
- 아이콘만 있는 버튼에는 `aria-label`
- 폼 요소에는 `<label>` 연결

## 파일 구조

```
src/
├── components/
│   ├── Header.tsx
│   ├── Card.tsx
│   └── Button.tsx
├── pages/
│   └── Home.tsx
└── styles/
    └── (CSS 방식에 따라 다름)
```

프로젝트에 기존 파일 구조가 있으면 그것을 따른다.

## 기존 컴포넌트 재사용

프로젝트에 이미 Button, Input, Card 등의 컴포넌트가 있으면:
1. 기존 컴포넌트를 import하여 사용
2. 필요하면 새 variant/prop을 추가
3. 새로 만들기보다 확장을 우선
