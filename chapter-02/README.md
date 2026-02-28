# Chapter 2: Claude Code로 코드 생성

> CLAUDE.md 문서, Thinking 모드, 서브에이전트 등 Claude Code의 핵심 기능을 활용하여 학습 트래커의 주요 기능을 구현합니다.

## 학습 목표

이 챕터를 완료하면 다음을 할 수 있습니다:

- **문서 기반 코드 생성**의 중요성을 이해하고 CLAUDE.md를 활용할 수 있습니다
- **서버 컴포넌트**에서 데이터를 패칭하는 패턴을 구현할 수 있습니다
- **Thinking 모드**를 사용하여 복잡한 로직을 설계할 수 있습니다
- **서버 액션**으로 데이터 변이(mutation)를 처리할 수 있습니다
- **Skills(커스텀 슬래시 명령)**을 만들어 반복 작업을 자동화할 수 있습니다
- **서브에이전트**를 활용하여 코드 탐색과 리뷰를 수행할 수 있습니다

## 챕터 구성

| 섹션 | 주제 | 핵심 Claude Code 기능 |
|------|------|----------------------|
| [2.1](01-dashboard-first-attempt.md) | 대시보드 첫 시도 (문서 없이) | 컨텍스트의 중요성, `/rewind` |
| [2.2](02-documentation-driven.md) | 문서 기반 코드 생성 | `/init`, CLAUDE.md, 가이드 문서 |
| [2.3](03-data-fetching.md) | 서버 컴포넌트 데이터 패칭 | 문서 참조 패턴, 쿼리 함수 |
| [2.4](04-thinking-mode.md) | Thinking 모드 & URL SearchParams | Extended Thinking, 복잡한 로직 |
| [2.5](05-data-mutations.md) | 서버 액션 & 데이터 변이 | Server Actions, Zod 검증 |
| [2.6](06-branch-management.md) | 브랜치 관리 & Skills | Git 워크플로, Skills(커스텀 슬래시 명령) |
| [2.7](07-edit-page-subagent.md) | 편집 페이지 & 서브에이전트 | Subagents, 코드 탐색/리뷰 |

## 이 챕터에서 구현하는 기능

```
학습 트래커 앱
├── /dashboard                  ← 대시보드 (통계 카드, 최근 세션)
├── /dashboard/sessions         ← 학습 세션 목록 (필터링, 정렬)
├── /dashboard/sessions/new     ← 새 학습 세션 생성
├── /dashboard/sessions/[id]    ← 학습 세션 상세
├── /dashboard/sessions/[id]/edit ← 학습 세션 편집
└── /dashboard/subjects         ← 과목 관리 (CRUD)
```

## 사전 준비

Chapter 1을 완료하여 다음이 준비되어 있어야 합니다:

- [x] Claude Code가 Amazon Bedrock에 연결된 상태
- [x] Next.js 15 프로젝트가 생성된 상태
- [x] Clerk 인증이 설정된 상태
- [x] Neon Postgres 데이터베이스가 연결된 상태
- [x] Drizzle ORM 스키마가 정의된 상태 (`study_sessions`, `subjects`, `session_subjects`, `study_blocks`)

## DB 스키마 참고

```
study_sessions: id, userId, date, startTime, endTime, notes, createdAt, updatedAt
subjects: id, userId, name, color, createdAt
session_subjects: id, sessionId, subjectId
study_blocks: id, sessionSubjectId, durationMinutes, pagesRead, notes, createdAt
```

## 소요 시간

약 90~120분

---

> **다음 단계**: [2.1 대시보드 첫 시도](01-dashboard-first-attempt.md)에서 문서 없이 코드를 생성해보고, 왜 컨텍스트가 중요한지 직접 체험합니다.
