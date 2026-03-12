# Marketplace Skills

## Project Overview

Claude Code 스킬(Skill)을 생성, 관리하고 마켓플레이스를 통해 배포/설치할 수 있도록 지원하는 프로젝트.

### 핵심 목표
- 스킬 생성 및 편집 도구 제공
- 마켓플레이스 구조를 통한 스킬 검색, 배포, 설치 지원
- 다른 사용자/환경에서 스킬을 쉽게 설치하고 사용할 수 있는 인프라 구축

## Project Structure

```
marketplace_skills/
├── CLAUDE.md                              # 프로젝트 가이드 (이 파일)
├── .claude-plugin/
│   └── plugin.json                        # 플러그인 메타데이터
└── skills/
    └── figma-to-code/
        ├── SKILL.md                       # 메인 스킬 (Figma → Code 변환)
        └── references/
            ├── layout-patterns.md         # 레이아웃 단계 참조
            ├── style-patterns.md          # 스타일 단계 참조
            ├── comparison-guide.md        # Playwright 비주얼 비교 가이드
            ├── frameworks/
            │   ├── react.md               # React 변환 규칙
            │   ├── vue.md                 # Vue 변환 규칙
            │   └── html.md                # HTML 변환 규칙
            └── css-approaches/
                ├── tailwind.md            # Tailwind CSS 매핑 규칙
                └── pure-css.md            # Pure CSS 매핑 규칙
```

## Tech Stack

- Claude Code 스킬 포맷 (마크다운 프론트매터 + 프롬프트)
- Figma MCP 서버 (디자인 데이터 추출)
- Playwright MCP 서버 (비주얼 비교 검증)
- 지원 프레임워크: React, Vue, Next.js, Svelte, Angular, HTML
- 지원 CSS: Tailwind, Bootstrap, styled-components, CSS Modules, Pure CSS

## Development Guidelines

### 코드 컨벤션
- 명확하고 간결한 코드 작성
- 불필요한 추상화 지양
- 기존 코드/패턴을 우선 재사용

### 스킬 관련 규칙
- 스킬 파일은 Claude Code 스킬 포맷(마크다운 프론트매터 + 프롬프트)을 따름
- 스킬 메타데이터(이름, 설명, 버전, 작성자 등)는 반드시 포함
- 설치/배포 가능한 형태로 패키징

### Git
- 커밋 메시지는 한글 또는 영문으로 간결하게 작성
- 기능 단위로 커밋 분리

## Key Concepts

| 용어 | 설명 |
|------|------|
| Skill | Claude Code에서 사용 가능한 재사용 가능한 프롬프트/명령 단위 |
| Marketplace | 스킬을 등록, 검색, 설치할 수 있는 중앙 저장소/플랫폼 |
| Publisher | 스킬을 마켓플레이스에 배포하는 사용자/조직 |
| Consumer | 마켓플레이스에서 스킬을 검색하고 설치하는 사용자 |
