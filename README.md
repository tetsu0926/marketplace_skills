# Marketplace Skills

Claude Code 스킬을 마켓플레이스를 통해 설치하고 사용할 수 있는 프로젝트입니다.

## 스킬 설치 방법

### 1. Claude Code CLI로 설치 (권장)

다른 프로젝트에서 이 마켓플레이스의 스킬을 설치하려면 아래 단계를 따릅니다.

#### Step 1: 마켓플레이스 등록

```bash
# GitHub 저장소로 등록
claude plugin marketplace add <your-github-username>/marketplace_skills

# 또는 로컬 경로로 등록
claude plugin marketplace add /path/to/marketplace_skills
```

#### Step 2: 스킬(플러그인) 설치

```bash
# figma-to-code 스킬 설치 (사용자 범위)
claude plugin install figma-to-code

# 특정 프로젝트에만 설치
claude plugin install figma-to-code --scope project
```

#### 설치 확인

```bash
# 설치된 플러그인 목록 확인
claude plugin list
```

#### 마켓플레이스 관리

```bash
# 등록된 마켓플레이스 확인
claude plugin marketplace list

# 마켓플레이스 업데이트 (최신 스킬 동기화)
claude plugin marketplace update

# 마켓플레이스 제거
claude plugin marketplace remove marketplace-skills

# 스킬(플러그인) 제거
claude plugin uninstall figma-to-code
```

## 사용 예시

스킬 설치 후 Claude Code에서 바로 사용할 수 있습니다.

```
# Figma 디자인을 React + Tailwind 코드로 변환
> 이 피그마 디자인을 React + Tailwind로 구현해줘
  https://figma.com/design/abc123/MyDesign?node-id=1-2

# Vue + Pure CSS로 변환
> 피그마를 Vue 코드로 변환해줘 (순수 CSS 사용)
  https://figma.com/design/abc123/MyDesign?node-id=3-4
```

## 사용 가능한 스킬 목록

| 스킬 | 설명 | 필수 MCP 서버 |
|------|------|---------------|
| `figma-to-code` | Figma 디자인을 프로덕션 코드로 변환 (React, Vue, Next.js, Svelte, Angular, HTML 지원) | Figma, Playwright(권장) |

## 라이선스

MIT
