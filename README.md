# Claude Code on Amazon Bedrock 핸즈온 워크샵

> Claude Code를 Amazon Bedrock과 함께 사용하여 풀스택 웹 애플리케이션을 만드는 실습형 워크샵

## 워크샵 소개

이 워크샵에서는 **Claude Code**를 **Amazon Bedrock** 환경에서 사용하여 "**할 일 관리 앱(Todo App)**" 웹 애플리케이션을 처음부터 끝까지 만들어봅니다. Claude Code의 에이전틱 코딩 기능을 활용해 프로젝트 생성, 코드 생성, 디버깅, GitHub 연동, 그리고 GitHub Actions 자동화까지 전 과정을 경험합니다.

## 만들게 될 앱: 할 일 관리 앱 (Todo App)

할 일을 생성하고 관리하는 풀스택 웹 애플리케이션입니다.

- 할 일 생성/조회/수정/삭제 (CRUD)
- 카테고리별 분류
- 우선순위 설정 (높음/중간/낮음)
- 마감일 관리

## 기술 스택

| 구분 | 기술 |
|------|------|
| AI 코딩 도구 | Claude Code (Amazon Bedrock) |
| 프레임워크 | Next.js 15 (App Router) |
| 언어 | TypeScript |
| 데이터베이스 | 로컬 PostgreSQL (EC2) |
| ORM | Drizzle ORM |
| UI | shadcn/ui + Tailwind CSS |
| CI/CD | GitHub Actions + Claude Code Action |

## 워크샵 구성

### [Chapter 1: 프로젝트 설정 & Claude Code 소개](chapter-01/README.md)

AWS Bedrock 환경 설정부터 Claude Code 설치, Next.js 프로젝트 생성, PostgreSQL 데이터베이스 구성까지 프로젝트의 기반을 만듭니다.

### [Chapter 2: Claude Code로 코드 생성](chapter-02/README.md)

Claude Code의 다양한 기능(문서 기반 생성, Thinking 모드, 서브에이전트 등)을 활용하여 Todo 앱의 핵심 기능을 구현합니다.

### [Chapter 3: GitHub Actions 자동화](chapter-03/README.md)

GitHub 저장소 설정 후 GitHub Actions와 Claude Code를 연동하여 Issue 기반 자동 개발 워크플로를 구축합니다.

### [Chapter 4: 고급 기능](chapter-04/README.md)

UI 개선, 커스텀 슬래시 명령, 에이전트 스킬, Bash 활용 등 Claude Code의 고급 기능을 학습합니다.

### [Chapter 5: 슈퍼랩 — 프로처럼 Claude Code 사용하기](chapter-05/README.md)

Boris Tane의 프로페셔널 워크플로(Research → Plan → Annotate → Implement)를 적용하여 Todo 앱에 통계 대시보드를 처음부터 끝까지 만드는 종합 실습입니다.

## Claude Code 기능 학습 로드맵

이 워크샵에서는 Claude Code의 기능을 단계적으로 학습합니다. 각 챕터에서 배우는 주요 기능은 다음과 같습니다.

```
Chapter 1: 기초
├── claude 명령으로 대화 시작
├── 기본 명령어 (/help, /clear, /compact)
├── Plan 모드 (Shift+Tab)
├── 되돌리기 (/rewind, Esc)
└── MCP 서버 연결

Chapter 2: 핵심 기능
├── /init으로 CLAUDE.md 생성
├── 문서 기반 코드 생성 (CLAUDE.md + docs/)
├── Thinking 모드 (복잡한 로직)
├── Skills - 커스텀 슬래시 명령 (/commit)
└── 서브에이전트 (탐색 위임)

Chapter 3: 자동화
├── Git 연동 (커밋, 푸시, PR)
└── GitHub Actions + @claude

Chapter 4: 고급 기능
├── 이미지 기반 UI 피드백
├── 사용자 범위 Skills (~/.claude/skills/)
├── 에이전트 정의 (.claude/agents/)
├── Hooks (자동 실행)
└── CLI 파이프 (claude -p)

Chapter 5: 슈퍼랩 (종합 실습)
├── Boris Tane 프로 워크플로
├── 서브에이전트 리서치 → research.md
├── 파일 기반 계획 → plan.md
├── 인라인 주석 사이클 (NOTE/CHANGE/REJECT)
└── 계획 기반 일괄 구현
```

각 기능은 처음 등장하는 섹션에서 상세히 설명되며, 이후 챕터에서는 자연스럽게 활용됩니다.

## Claude Code 기능 커리큘럼

이 워크샵은 [Claude Code 공식 문서](https://code.claude.com/docs)의 주요 기능을 단계적으로 학습할 수 있도록 설계되었습니다.

| Claude Code 기능 | 공식 문서 | 워크샵 섹션 | 설명 |
|------------------|----------|------------|------|
| 설치 및 설정 | [Setup](https://code.claude.com/docs/en/setup) | 1.1~1.2 | CLI 설치, Bedrock 연결 |
| CLAUDE.md (메모리) | [Memory](https://code.claude.com/docs/en/claude-md) | 1.3, 2.2 | 프로젝트 규칙, 코딩 컨벤션 정의 |
| 기본 명령어 | [Quickstart](https://code.claude.com/docs/en/quickstart) | 1.4 | /help, /clear, /compact, /rewind |
| Plan 모드 | [Common Workflows](https://code.claude.com/docs/en/common-workflows) | 1.4, 1.8 | Shift+Tab으로 읽기 전용 탐색 |
| 컨텍스트 관리 | [Best Practices](https://code.claude.com/docs/en/best-practices) | 1.6 | 컨텍스트 윈도우 관리, 자동 압축 |
| MCP 서버 | [MCP](https://code.claude.com/docs/en/mcp) | 1.8 | 외부 도구 연결 (PostgreSQL 등) |
| Extended Thinking | [Common Workflows](https://code.claude.com/docs/en/common-workflows) | 2.4 | 복잡한 로직의 깊은 사고 |
| Skills (슬래시 명령) | [Skills](https://code.claude.com/docs/en/skills) | 2.6, 4.3, 4.4 | 커스텀 명령 정의 및 자동화 |
| 서브에이전트 | [Sub-agents](https://code.claude.com/docs/en/sub-agents) | 2.7, 4.4 | 독립 컨텍스트에서 탐색/리뷰 |
| Git 연동 | [Common Workflows](https://code.claude.com/docs/en/common-workflows) | 2.6, 3.1 | 커밋, 푸시, PR 생성 |
| GitHub Actions | [GitHub Actions](https://code.claude.com/docs/en/github-actions) | 3.2, 3.3 | @claude 멘션으로 자동 개발 |
| 이미지 피드백 | [Common Workflows](https://code.claude.com/docs/en/common-workflows) | 4.1 | 스크린샷 기반 UI 수정 |
| 커스텀 에이전트 | [Sub-agents](https://code.claude.com/docs/en/sub-agents) | 4.4 | .claude/agents/ 에이전트 정의 |
| Hooks | [Hooks](https://code.claude.com/docs/en/hooks) | 4.5 | 도구 실행 전후 자동 스크립트 |
| CLI 파이프 | [CLI Reference](https://code.claude.com/docs/en/cli-reference) | 4.5 | claude -p 비대화형 모드 |
| 프로 워크플로 | [Best Practices](https://code.claude.com/docs/en/best-practices) | 5.1~5.6 | Research → Plan → Annotate → Implement |
| 파일 기반 계획 | [Common Workflows](https://code.claude.com/docs/en/common-workflows) | 5.2~5.4 | research.md, plan.md 작성 및 주석 사이클 |
| VS Code 확장 | [VS Code](https://code.claude.com/docs/en/vs-code) | 사전 준비 | IDE 내 Claude Code 사용 |
| 설정 관리 | [Settings](https://code.claude.com/docs/en/settings) | 전체 | /config, /permissions 등 |

## 사전 요구사항

자세한 내용은 [사전 준비사항](prerequisites/README.md)을 참고하세요.

- AWS 계정 (Bedrock 액세스 가능)
- GitHub 계정
- Node.js 18+ 설치
- 기본적인 TypeScript/React 지식

## 참고 자료

- [Claude Code 공식 문서](https://code.claude.com/docs)
- [Claude Code on Amazon Bedrock 가이드](https://code.claude.com/docs/en/amazon-bedrock)
- [Amazon Bedrock 문서](https://docs.aws.amazon.com/bedrock/)
