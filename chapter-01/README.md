# Chapter 1: 프로젝트 설정 & Claude Code 소개

> AWS Bedrock 환경 설정부터 Claude Code 설치, Next.js 프로젝트 생성, 데이터베이스 구성까지 학습 트래커의 기반을 만듭니다.

## 이 챕터에서 배우는 것

이 챕터에서는 Claude Code를 Amazon Bedrock 환경에서 사용하기 위한 모든 준비 과정을 진행합니다. 챕터를 마치면 인증, 데이터베이스, UI 프레임워크가 갖춰진 Next.js 프로젝트가 완성됩니다.

## 학습 목표

| 섹션 | 내용 | 소요 시간 |
|------|------|-----------|
| [1.1 AWS 환경 설정](01-environment-setup.md) | Bedrock 모델 액세스 활성화, IAM 정책 구성, AWS 자격증명 설정 | 15분 |
| [1.2 Claude Code 설치 및 Bedrock 연결](02-claude-code-install.md) | Claude Code 설치, 환경변수 설정, Bedrock 연결 확인 | 10분 |
| [1.3 Next.js 프로젝트 생성](03-project-creation.md) | Claude Code를 사용하여 Next.js 15 프로젝트 생성 | 10분 |
| [1.4 Claude Code 기본 명령어 & 모드](04-claude-code-basics.md) | 슬래시 명령어, Plan 모드, 대화 관리, 키보드 단축키 | 15분 |
| [1.5 Clerk 인증 설정](05-auth-setup.md) | Clerk 계정 생성, Next.js 인증 통합, 로그인 페이지 구성 | 15분 |
| [1.6 컨텍스트 창 개념](06-context-window.md) | 컨텍스트 윈도우 관리, 압축, 서브에이전트 활용 | 10분 |
| [1.7 데이터베이스 설정](07-database-setup.md) | Neon Postgres 생성, Drizzle ORM 설정, shadcn/ui 초기화 | 15분 |
| [1.8 스키마 설계 & MCP](08-schema-design-mcp.md) | Plan 모드로 스키마 설계, 마이그레이션, Neon MCP 시드 데이터 | 20분 |

## 사전 준비

이 챕터를 시작하기 전에 다음이 필요합니다:

- **AWS 계정**: Amazon Bedrock 서비스 접근 권한
- **Node.js 18+**: `node --version`으로 확인
- **pnpm**: `npm install -g pnpm`으로 설치
- **Git**: 버전 관리용
- **터미널**: macOS Terminal, iTerm2, Windows WSL 등

## 최종 결과물

이 챕터를 완료하면 다음과 같은 프로젝트 구조가 만들어집니다:

```
study-tracker/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── sign-in/[[...sign-in]]/page.tsx
│   └── sign-up/[[...sign-up]]/page.tsx
├── db/
│   ├── index.ts
│   └── schema.ts
├── components/ui/        # shadcn/ui 컴포넌트
├── drizzle/              # 마이그레이션 파일
├── drizzle.config.ts
├── middleware.ts          # Clerk 인증 미들웨어
├── .env.local
├── CLAUDE.md
├── package.json
└── tailwind.config.ts
```

다음 섹션인 [1.1 AWS 환경 설정](01-environment-setup.md)으로 진행하세요.
