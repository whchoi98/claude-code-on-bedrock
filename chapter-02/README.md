# Chapter 2: Claude Code로 코드 생성

> CLAUDE.md 문서, Thinking 모드, 서브에이전트 등 Claude Code의 핵심 기능을 활용하여 Todo 앱의 주요 기능을 구현합니다.

## 학습 목표

이 챕터를 완료하면 다음을 할 수 있습니다:

- **문서 기반 코드 생성**의 중요성을 이해하고 CLAUDE.md를 활용할 수 있습니다
- **서버 컴포넌트**에서 데이터를 패칭하는 패턴을 구현할 수 있습니다
- **Thinking 모드**를 사용하여 복잡한 로직을 설계할 수 있습니다
- **서버 액션**으로 데이터 변이(mutation)를 처리할 수 있습니다
- **Skills(커스텀 슬래시 명령)**을 만들어 반복 작업을 자동화할 수 있습니다
- **서브에이전트**를 활용하여 코드 탐색과 리뷰를 수행할 수 있습니다

### 이 챕터에서 배우는 Claude Code 기능

| 기능 | 배우는 섹션 | 설명 |
|------|-----------|------|
| `/rewind` | 2.1 | 변경사항 되돌리기 |
| `/init`, CLAUDE.md | 2.2 | 프로젝트 규칙 문서화 |
| 문서 기반 코드 생성 | 2.2, 2.3 | CLAUDE.md + docs/로 일관된 코드 생성 |
| Extended Thinking | 2.4 | 복잡한 로직을 깊이 사고하여 설계 |
| Skills (커스텀 슬래시 명령) | 2.6 | 반복 작업 자동화 |
| 서브에이전트 | 2.7 | 별도 컨텍스트에서 탐색/리뷰 위임 |

## 챕터 구성

| 섹션 | 주제 | 핵심 Claude Code 기능 |
|------|------|----------------------|
| [2.1](01-dashboard-first-attempt.md) | 할 일 목록 첫 시도 (문서 없이) | 컨텍스트의 중요성, `/rewind` |
| [2.2](02-documentation-driven.md) | 문서 기반 코드 생성 | `/init`, CLAUDE.md, 가이드 문서 |
| [2.3](03-data-fetching.md) | 서버 컴포넌트 데이터 패칭 | 문서 참조 패턴, 쿼리 함수 |
| [2.4](04-thinking-mode.md) | Thinking 모드 & URL SearchParams | Extended Thinking, 복잡한 로직 |
| [2.5](05-data-mutations.md) | 서버 액션 & 데이터 변이 | Server Actions, Zod 검증 |
| [2.6](06-branch-management.md) | 브랜치 관리 & Skills | Git 워크플로, Skills(커스텀 슬래시 명령) |
| [2.7](07-edit-page-subagent.md) | 편집 페이지 & 서브에이전트 | Subagents, 코드 탐색/리뷰 |

## 이 챕터에서 구현하는 기능

```
Todo 앱
├── /todos                    ← 할 일 목록 (필터링, 정렬)
├── /todos/new                ← 새 할 일 생성
├── /todos/[id]/edit          ← 할 일 편집
└── /categories               ← 카테고리 관리 (CRUD)
```

## 사전 준비

Chapter 1을 완료하여 다음이 준비되어 있어야 합니다:

- [x] Claude Code가 Amazon Bedrock에 연결된 상태
- [x] Next.js 15 프로젝트가 생성된 상태
- [x] 로컬 PostgreSQL 데이터베이스가 연결된 상태
- [x] Drizzle ORM 스키마가 정의된 상태 (`todos`, `categories`)

## Next.js 개념 Quick Reference

이 챕터에서 자주 등장하는 Next.js 개념을 간단히 정리합니다. 몰라도 괜찮습니다 — Claude Code가 코드를 생성해주지만, 알아두면 이해하기 쉽습니다.

| 개념 | 한 줄 설명 | 예시 |
|------|-----------|------|
| **Server Component** | 서버에서 실행되는 컴포넌트. 데이터베이스에 직접 접근 가능 | `export default async function Page() { ... }` |
| **Client Component** | 브라우저에서 실행되는 컴포넌트. 사용자 상호작용 처리 | `"use client"` 선언 필요 |
| **Server Action** | 서버에서 실행되는 함수. 폼 제출 등 데이터 변경에 사용 | `"use server"` 선언 필요 |
| **Drizzle ORM** | TypeScript로 SQL 쿼리를 작성하는 라이브러리 | `db.select().from(todos).where(...)` |
| **`searchParams`** | URL의 쿼리 파라미터를 서버 컴포넌트에서 읽는 방법 | `?sort=newest` → `searchParams.sort` |
| **`revalidatePath()`** | 데이터 변경 후 페이지를 새로고침하는 함수 | 할 일 생성 후 목록 페이지 갱신 |

## DB 스키마 참고

```
todos: id (UUID PK), title (TEXT NOT NULL), description (TEXT), completed (BOOLEAN DEFAULT false), priority (TEXT 'low'|'medium'|'high' DEFAULT 'medium'), dueDate (DATE), categoryId (UUID FK → categories.id), createdAt (TIMESTAMP), updatedAt (TIMESTAMP)
categories: id (UUID PK), name (TEXT NOT NULL), color (TEXT NOT NULL), createdAt (TIMESTAMP)
```

### 다음 챕터 미리보기

Chapter 2를 완료하면, [Chapter 3: GitHub 저장소 설정 및 GitHub Actions 자동화](../chapter-03/README.md)에서 GitHub 저장소를 설정하고 GitHub Actions와 Claude Code를 연동하여 Issue 기반 자동 개발 워크플로를 구축합니다.

## 소요 시간

약 90~120분

---

> **다음 단계**: [2.1 할 일 목록 첫 시도](01-dashboard-first-attempt.md)에서 문서 없이 코드를 생성해보고, 왜 컨텍스트가 중요한지 직접 체험합니다.
