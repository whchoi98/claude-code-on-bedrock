# 1.8 Plan 모드로 스키마 설계 + Neon MCP로 시드 데이터

> Plan 모드를 활용하여 데이터베이스 스키마를 설계하고, Neon MCP 서버를 연결하여 시드 데이터를 삽입합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: Plan 모드 (`Shift+Tab`), MCP 서버 연결

> ⭐⭐ **난이도**: 보통

## 개요

이 섹션에서는 Claude Code의 **Plan 모드**를 사용하여 코드를 변경하지 않고 스키마를 먼저 설계한 후, Normal 모드에서 구현합니다. 또한 **MCP(Model Context Protocol)** 서버를 통해 Neon 데이터베이스에 직접 시드 데이터를 삽입하는 방법을 학습합니다.

{% hint style="info" %}
새 작업을 시작하기 전에 `/clear`로 컨텍스트를 초기화하세요.
{% endhint %}

## Plan 모드란?

Plan 모드는 Claude Code가 **읽기 전용**으로 동작하는 모드입니다. 파일을 읽고, 분석하고, 계획을 세울 수 있지만 코드를 변경하거나 명령을 실행하지 않습니다.

> 💡 **비유**: Plan 모드는 "먼저 설계도를 그리고, 나중에 건물을 짓는 것"과 같습니다. 코드를 바로 수정하지 않고, 먼저 구조를 파악하고 계획을 세운 뒤에 실제 구현으로 넘어갑니다.

| 모드 | 파일 읽기 | 파일 쓰기 | 명령 실행 |
|------|----------|----------|----------|
| Normal 모드 | O | O | O |
| Plan 모드 | O | X | X |

### 활성화 방법

- **Shift+Tab**: Plan 모드 토글
- 프롬프트 입력 영역 옆에 모드 표시가 나타남

## 1단계: Plan 모드에서 스키마 설계

### Plan 모드 활성화

`Shift+Tab`을 눌러 Plan 모드로 전환합니다. 프롬프트 영역에 **(Plan Mode)** 표시가 나타나는지 확인하세요.

### 스키마 설계 프롬프트

→ *이 프롬프트는 Plan 모드에서 코드 변경 없이 데이터베이스 스키마 구조를 설계하기 위한 것입니다:*

```
학습 트래커 애플리케이션을 위한 Drizzle ORM 스키마를 계획해줘. 다음 테이블들이 필요해:

- study_sessions: id (uuid, primary key), userId (text, Clerk에서 제공), date (date), startTime (timestamp), endTime (timestamp), notes (text, 선택), createdAt (timestamp), updatedAt (timestamp)

- subjects: id (uuid, primary key), userId (text), name (text), color (text), createdAt (timestamp)

- session_subjects: id (uuid, primary key), sessionId (uuid, FK → study_sessions), subjectId (uuid, FK → subjects)

- study_blocks: id (uuid, primary key), sessionSubjectId (uuid, FK → session_subjects), durationMinutes (integer), pagesRead (integer, 선택), notes (text, 선택), createdAt (timestamp)

고려사항:
1. userId와 외래 키에 적절한 인덱스
2. Cascade 삭제 동작
3. Drizzle ORM relations 설정
4. 스키마 파일은 db/schema.ts에 생성
```

### 계획 검토

Claude Code가 다음 내용을 포함한 계획을 제시합니다:

- 테이블 구조 및 컬럼 정의
- 외래 키 관계 및 Cascade 설정
- 인덱스 전략
- Drizzle ORM의 `relations()` 설정

계획을 검토하고, 필요하면 수정을 요청합니다:

→ *이 프롬프트는 설계 계획에 인덱스를 추가하고 Cascade 동작을 조정하기 위한 것입니다:*

```
study_sessions.date에 날짜 범위 조회를 위한 인덱스도 추가해줘.
그리고 session_subjects가 삭제될 때 study_blocks도 cascade 삭제되도록 해줘.
```

{% hint style="info" %}
**Ctrl+G**를 누르면 계획을 텍스트 에디터에서 직접 편집할 수 있습니다. 복잡한 계획을 수정할 때 유용합니다.
{% endhint %}

## 2단계: Normal 모드에서 스키마 구현

### Normal 모드 전환

`Shift+Tab`을 다시 눌러 Normal 모드로 전환합니다.

### 구현 프롬프트

→ *이 프롬프트는 Plan 모드에서 세운 계획을 Normal 모드에서 실제 코드로 구현하기 위한 것입니다:*

```
방금 계획한 스키마를 구현해줘. 4개 테이블, relations, 인덱스를 모두 포함하여 db/schema.ts를 생성해줘. 계획을 정확히 따라줘.
```

### 생성된 스키마 확인

Claude Code가 생성한 `db/schema.ts`는 다음과 같은 구조를 가집니다:

```typescript
import { pgTable, uuid, text, date, timestamp, integer, index } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// ===== Tables =====

export const studySessions = pgTable('study_sessions', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: text('user_id').notNull(),
  date: date('date').notNull(),
  startTime: timestamp('start_time'),
  endTime: timestamp('end_time'),
  notes: text('notes'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => [
  index('study_sessions_user_id_idx').on(table.userId),
  index('study_sessions_date_idx').on(table.date),
]);

export const subjects = pgTable('subjects', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: text('user_id').notNull(),
  name: text('name').notNull(),
  color: text('color').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => [
  index('subjects_user_id_idx').on(table.userId),
]);

export const sessionSubjects = pgTable('session_subjects', {
  id: uuid('id').defaultRandom().primaryKey(),
  sessionId: uuid('session_id').references(() => studySessions.id, { onDelete: 'cascade' }).notNull(),
  subjectId: uuid('subject_id').references(() => subjects.id, { onDelete: 'cascade' }).notNull(),
});

export const studyBlocks = pgTable('study_blocks', {
  id: uuid('id').defaultRandom().primaryKey(),
  sessionSubjectId: uuid('session_subject_id').references(() => sessionSubjects.id, { onDelete: 'cascade' }).notNull(),
  durationMinutes: integer('duration_minutes').notNull(),
  pagesRead: integer('pages_read'),
  notes: text('notes'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ===== Relations =====

export const studySessionsRelations = relations(studySessions, ({ many }) => ({
  sessionSubjects: many(sessionSubjects),
}));

export const subjectsRelations = relations(subjects, ({ many }) => ({
  sessionSubjects: many(sessionSubjects),
}));

export const sessionSubjectsRelations = relations(sessionSubjects, ({ one, many }) => ({
  session: one(studySessions, {
    fields: [sessionSubjects.sessionId],
    references: [studySessions.id],
  }),
  subject: one(subjects, {
    fields: [sessionSubjects.subjectId],
    references: [subjects.id],
  }),
  studyBlocks: many(studyBlocks),
}));

export const studyBlocksRelations = relations(studyBlocks, ({ one }) => ({
  sessionSubject: one(sessionSubjects, {
    fields: [studyBlocks.sessionSubjectId],
    references: [sessionSubjects.id],
  }),
}));
```

## 3단계: 마이그레이션 실행

스키마를 데이터베이스에 적용합니다:

→ *이 프롬프트는 스키마 변경사항을 SQL 마이그레이션으로 변환하고 데이터베이스에 적용하기 위한 것입니다:*

```
Drizzle 마이그레이션을 생성하고 실행해줘:
1. pnpm db:generate를 실행하여 마이그레이션 파일 생성
2. pnpm db:migrate를 실행하여 데이터베이스에 마이그레이션 적용
```

### 마이그레이션 확인

```bash
# Claude Code가 다음 명령을 실행합니다:
pnpm db:generate
pnpm db:migrate
```

`drizzle/` 디렉토리에 마이그레이션 SQL 파일이 생성됩니다. 데이터베이스에 4개 테이블이 생성되었는지 확인합니다.

## 4단계: Neon MCP 서버 연결 (선택사항)

MCP(Model Context Protocol)를 사용하면 Claude Code가 외부 도구와 직접 통신할 수 있습니다. Neon MCP 서버를 연결하면 Claude Code가 데이터베이스를 직접 조회하고 조작할 수 있습니다.

### MCP란?

MCP는 AI 도구를 외부 데이터 소스와 연결하는 오픈 표준입니다.

> 💡 **비유**: MCP는 "Claude Code에 새로운 능력을 추가하는 것"입니다. 기본적으로 Claude Code는 파일을 읽고 쓸 수 있지만, MCP를 통해 데이터베이스를 직접 조회하거나, 외부 서비스와 통신하는 등의 새로운 능력을 얻습니다.

Claude Code는 MCP를 통해 다음과 같은 작업이 가능합니다:

- 데이터베이스 쿼리 실행
- 이슈 트래커 연동
- 모니터링 데이터 분석
- 디자인 도구 통합

### Neon MCP 서버 추가

터미널에서 다음 명령을 실행합니다:

```bash
claude mcp add neon \
  --transport stdio \
  -e NEON_API_KEY=your-neon-api-key \
  -- npx -y @neondatabase/mcp-server-neon
```

{% hint style="info" %}
Neon API 키는 Neon 대시보드의 **Account Settings > API Keys**에서 생성할 수 있습니다.
{% endhint %}

### MCP 서버 확인

Claude Code 내에서 MCP 서버 상태를 확인합니다:

```
/mcp
```

Neon MCP 서버가 연결되어 있으면 사용 가능한 도구 목록이 표시됩니다.

## 5단계: 시드 데이터 삽입

### MCP를 통한 시드 데이터 (MCP 연결 시)

→ *이 프롬프트는 MCP를 통해 데이터베이스에 직접 테스트용 시드 데이터를 삽입하기 위한 것입니다:*

```
Neon MCP 서버를 사용해서 데이터베이스에 시드 데이터를 삽입해줘:

1. 과목 3개 생성:
   - 수학 (color: #3B82F6, 파랑)
   - 영어 (color: #10B981, 초록)
   - 프로그래밍 (color: #8B5CF6, 보라)

2. 오늘과 어제 날짜로 샘플 학습 세션 2개 생성

3. session_subjects를 통해 과목을 세션에 연결

4. 학습 블록 추가:
   - 세션 1: 수학 60분 (30페이지), 영어 45분 (15페이지)
   - 세션 2: 프로그래밍 90분, 수학 30분 (20페이지)

userId는 임시로 'test-user-001'을 사용해줘.
```

### Server Action을 통한 시드 데이터 (MCP 미연결 시)

MCP를 사용하지 않는 경우 시드 스크립트를 생성합니다:

→ *이 프롬프트는 MCP 없이 시드 스크립트를 생성하여 테스트 데이터를 삽입하기 위한 것입니다:*

```
db/seed.ts에 시드 스크립트를 생성해줘:
1. 과목 3개 삽입 (수학, 영어, 프로그래밍)
2. 샘플 학습 세션 2개 삽입
3. 과목을 세션에 연결
4. 현실적인 데이터로 학습 블록 추가
5. userId는 'test-user-001' 사용

package.json에 tsx로 이 파일을 실행하는 "db:seed" 스크립트도 추가해줘.
```

실행:

```bash
pnpm db:seed
```

## 6단계: 데이터 확인

### Drizzle Studio로 확인

```bash
pnpm db:studio
```

브라우저에서 [https://local.drizzle.studio](https://local.drizzle.studio)에 접속하면 Drizzle Studio에서 데이터를 확인할 수 있습니다.

### Claude Code에서 확인

→ *이 프롬프트는 시드 데이터가 올바르게 삽입되었는지 확인하기 위한 것입니다:*

```
데이터베이스를 조회해서 모든 학습 세션과 해당 과목, 학습 블록을 보여줘.
```

## 요약

| 항목 | 상태 |
|------|------|
| Plan 모드로 스키마 설계 | &#x2705; |
| db/schema.ts 구현 | &#x2705; |
| 마이그레이션 생성 및 실행 | &#x2705; |
| Neon MCP 서버 연결 (선택) | &#x2705; |
| 시드 데이터 삽입 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Plan 모드에서 스키마 설계를 검토함 (`Shift+Tab`으로 전환)
- [ ] `db/schema.ts`에 4개 테이블이 생성됨
- [ ] 마이그레이션이 성공적으로 실행됨 (`pnpm db:generate` + `pnpm db:migrate`)
- [ ] 시드 데이터가 데이터베이스에 삽입됨
- [ ] (선택) Neon MCP 서버가 연결되어 동작함

이것으로 Chapter 1이 완료되었습니다. 프로젝트의 기반이 모두 갖춰졌습니다:

- AWS Bedrock 환경 설정 완료
- Claude Code 설치 및 연결 완료
- Next.js 프로젝트 생성 완료
- Clerk 인증 설정 완료
- 데이터베이스 스키마 설계 및 마이그레이션 완료
- 시드 데이터 삽입 완료

[Chapter 2: Claude Code로 코드 생성](../chapter-02/README.md)에서는 이 기반 위에 실제 기능을 구현합니다.
