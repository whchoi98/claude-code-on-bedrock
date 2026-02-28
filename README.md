# Claude Code on Amazon Bedrock 핸즈온 워크샵

> Claude Code를 Amazon Bedrock과 함께 사용하여 풀스택 웹 애플리케이션을 만드는 실습형 워크샵

## 워크샵 소개

이 워크샵에서는 **Claude Code**를 **Amazon Bedrock** 환경에서 사용하여 "**학습 트래커(Study Tracker)**" 웹 애플리케이션을 처음부터 끝까지 만들어봅니다. Claude Code의 에이전틱 코딩 기능을 활용해 프로젝트 생성, 코드 생성, 디버깅, 배포, 그리고 GitHub Actions 자동화까지 전 과정을 경험합니다.

## 만들게 될 앱: 학습 트래커 (Study Tracker)

학습 세션을 기록하고 관리하는 풀스택 웹 애플리케이션입니다.

- 학습 세션 생성/조회/수정/삭제
- 과목별 학습 블록 기록 (시간, 페이지수)
- 대시보드에서 학습 현황 확인
- 사용자 인증 (Clerk)

## 기술 스택

| 구분 | 기술 |
|------|------|
| AI 코딩 도구 | Claude Code (Amazon Bedrock) |
| 프레임워크 | Next.js 15 (App Router) |
| 언어 | TypeScript |
| 인증 | Clerk |
| 데이터베이스 | Neon Postgres |
| ORM | Drizzle ORM |
| UI | shadcn/ui + Tailwind CSS |
| 배포 | Vercel |
| CI/CD | GitHub Actions + Claude Code Action |

## 워크샵 구성

### [Chapter 1: 프로젝트 설정 & Claude Code 소개](chapter-01/README.md)

AWS Bedrock 환경 설정부터 Claude Code 설치, Next.js 프로젝트 생성, 인증 설정, 데이터베이스 구성까지 프로젝트의 기반을 만듭니다.

### [Chapter 2: Claude Code로 코드 생성](chapter-02/README.md)

Claude Code의 다양한 기능(문서 기반 생성, Thinking 모드, 서브에이전트 등)을 활용하여 학습 트래커의 핵심 기능을 구현합니다.

### [Chapter 3: GitHub Actions 자동화](chapter-03/README.md)

Vercel 배포 후 GitHub Actions와 Claude Code를 연동하여 Issue 기반 자동 개발 워크플로를 구축합니다.

### [Chapter 4: 고급 기능](chapter-04/README.md)

UI 개선, 커스텀 슬래시 명령, 에이전트 스킬, Bash 활용 등 Claude Code의 고급 기능을 학습합니다.

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
```

각 기능은 처음 등장하는 섹션에서 상세히 설명되며, 이후 챕터에서는 자연스럽게 활용됩니다.

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
