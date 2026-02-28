# 2.3 서버 컴포넌트 데이터 패칭

> 데이터 패칭 가이드 문서를 작성하고, 서버 컴포넌트에서 Drizzle ORM으로 데이터를 패칭하는 패턴을 구현합니다.

## 학습 목표

- 데이터 패칭 규칙 문서를 작성하여 일관된 패턴을 정의합니다
- 서버 컴포넌트에서 직접 데이터베이스를 쿼리하는 방법을 이해합니다
- `lib/queries/` 디렉토리에 재사용 가능한 쿼리 함수를 만듭니다
- React Suspense와 로딩/에러 상태를 처리합니다

## 배경: Next.js App Router의 데이터 패칭

Next.js App Router에서는 서버 컴포넌트가 기본값입니다. 서버 컴포넌트에서는 `async/await`를 사용하여 데이터베이스에 직접 접근할 수 있으므로, API 라우트나 `useEffect`가 필요 없습니다.

```typescript
// 서버 컴포넌트 - 직접 데이터 패칭 가능
export default async function Page() {
  const data = await db.select().from(table);
  return <div>{/* data 렌더링 */}</div>;
}
```

## 실습: 데이터 패칭 가이드 문서 및 세션 목록 구현

### Step 1: docs/data-fetching.md 작성

데이터 패칭의 규칙과 패턴을 문서로 정의합니다:

```
데이터 패칭 패턴을 문서화하는 docs/data-fetching.md를 생성해줘:

# Data Fetching Patterns

## Core Principles
- Use server components for data fetching (no useEffect/useState for initial data)
- Never create API routes for data that server components can fetch directly
- Always authenticate with Clerk's auth() before querying

## Query Functions
- Create reusable query functions in lib/queries/ directory
- Each domain entity gets its own file (e.g., lib/queries/sessions.ts, lib/queries/subjects.ts)
- All query functions must accept userId as a parameter
- Use Drizzle ORM's query builder with proper relations
- Return typed results

## Example Query Function Pattern
```typescript
// lib/queries/sessions.ts
import { db } from "@/db";
import { studySessions, sessionSubjects, subjects } from "@/db/schema";
import { eq, desc } from "drizzle-orm";

export async function getSessionsByUserId(userId: string) {
  return db
    .select()
    .from(studySessions)
    .where(eq(studySessions.userId, userId))
    .orderBy(desc(studySessions.date));
}
```

## Loading States
- Use React Suspense boundaries for loading states
- Create loading.tsx in each route for automatic loading UI
- Use shadcn/ui Skeleton component for loading placeholders

## Error Handling
- Create error.tsx in each route for error boundaries
- Log errors server-side, show user-friendly messages client-side
- Handle "not found" cases with notFound() from next/navigation
```

### Step 2: 학습 세션 목록 페이지 구현

가이드 문서를 참조하여 세션 목록 페이지를 생성합니다:

```
docs/data-fetching.md를 읽어줘. 다음 요구사항으로 /dashboard/sessions에 학습 세션 목록 페이지를 만들어줘:
- 서버 컴포넌트를 사용해서 현재 사용자의 모든 세션을 패칭
- 각 세션의 날짜, 총 학습 시간, 학습한 과목 표시
- shadcn/ui의 Table 컴포넌트 사용
- Skeleton 컴포넌트로 loading.tsx 포함
- 사용자 친화적인 에러 메시지로 error.tsx 포함
문서화된 패턴에 따라 lib/queries/sessions.ts에 쿼리 함수를 생성해줘.
```

### Step 3: 생성된 파일 구조 확인

Claude Code가 생성한 파일들을 확인합니다. 다음과 같은 구조가 되어야 합니다:

```
app/
└── dashboard/
    └── sessions/
        ├── page.tsx        # 세션 목록 페이지 (서버 컴포넌트)
        ├── loading.tsx     # 로딩 상태 UI
        └── error.tsx       # 에러 상태 UI

lib/
└── queries/
    └── sessions.ts         # 세션 쿼리 함수
```

### Step 4: 쿼리 함수 패턴 확인

`lib/queries/sessions.ts` 파일을 살펴보고 다음 패턴이 적용되었는지 확인합니다:

```typescript
// lib/queries/sessions.ts - 기대하는 패턴
import { db } from "@/db";
import { studySessions, sessionSubjects, subjects } from "@/db/schema";
import { eq, desc } from "drizzle-orm";

export async function getSessionsByUserId(userId: string) {
  return db
    .select({
      id: studySessions.id,
      date: studySessions.date,
      startTime: studySessions.startTime,
      endTime: studySessions.endTime,
      notes: studySessions.notes,
      createdAt: studySessions.createdAt,
    })
    .from(studySessions)
    .where(eq(studySessions.userId, userId))
    .orderBy(desc(studySessions.date));
}

export async function getSessionWithDetails(sessionId: string, userId: string) {
  // 세션 상세 정보 + 관련 과목 + 학습 블록
  const session = await db
    .select()
    .from(studySessions)
    .where(eq(studySessions.id, sessionId))
    .limit(1);

  if (!session[0] || session[0].userId !== userId) {
    return null;
  }

  return session[0];
}
```

확인할 포인트:

- [x] `userId`를 파라미터로 받아서 데이터 필터링
- [x] Drizzle ORM의 query builder 사용
- [x] 타입 안전한 반환값
- [x] 정렬(orderBy) 적용

### Step 5: 서버 컴포넌트 페이지 확인

`app/dashboard/sessions/page.tsx` 파일을 살펴봅니다:

```typescript
// app/dashboard/sessions/page.tsx - 기대하는 패턴
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";
import { getSessionsByUserId } from "@/lib/queries/sessions";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";

export default async function SessionsPage() {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");

  const sessions = await getSessionsByUserId(userId);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Study Sessions</h1>
      </div>
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Date</TableHead>
            <TableHead>Duration</TableHead>
            <TableHead>Subjects</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {sessions.map((session) => (
            <TableRow key={session.id}>
              <TableCell>{session.date}</TableCell>
              <TableCell>{/* duration 계산 */}</TableCell>
              <TableCell>{/* subjects 표시 */}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </div>
  );
}
```

확인할 포인트:

- [x] `"use client"` 없음 (서버 컴포넌트)
- [x] `async` 함수로 선언
- [x] `auth()`로 인증 확인
- [x] 쿼리 함수를 직접 호출 (API 라우트 불필요)
- [x] shadcn/ui Table 컴포넌트 사용

### Step 6: Suspense 경계와 로딩 상태 확인

`loading.tsx` 파일을 확인합니다:

```typescript
// app/dashboard/sessions/loading.tsx
import { Skeleton } from "@/components/ui/skeleton";

export default function SessionsLoading() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-8 w-48" />
      <div className="space-y-2">
        {Array.from({ length: 5 }).map((_, i) => (
          <Skeleton key={i} className="h-12 w-full" />
        ))}
      </div>
    </div>
  );
}
```

{% hint style="info" %}
Next.js App Router는 `loading.tsx` 파일을 자동으로 React Suspense 경계로 감쌉니다. 별도의 `<Suspense>` 래퍼를 작성할 필요가 없습니다.
{% endhint %}

### Step 7: 개발 서버에서 확인

개발 서버를 실행하여 결과를 확인합니다:

```
개발 서버를 실행하고 세션 페이지가 올바르게 동작하는지 확인해줘. TypeScript 에러나 누락된 import가 있으면 수정해줘.
```

## 데이터 패칭 패턴 요약

```
┌─────────────────────────┐
│   Server Component      │
│   (page.tsx)            │
│                         │
│  1. auth() 호출          │
│  2. 쿼리 함수 호출       │
│  3. JSX 렌더링           │
└─────────┬───────────────┘
          │ import
          ▼
┌─────────────────────────┐
│   Query Function        │
│   (lib/queries/*.ts)    │
│                         │
│  - userId 파라미터       │
│  - Drizzle query builder│
│  - 타입 안전한 반환값     │
└─────────┬───────────────┘
          │ Drizzle ORM
          ▼
┌─────────────────────────┐
│   Database              │
│   (Neon Postgres)       │
└─────────────────────────┘
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| 서버 컴포넌트 | `async` 함수로 선언하여 직접 DB 쿼리 가능 |
| 쿼리 함수 | `lib/queries/`에 도메인별로 분리된 재사용 가능한 함수 |
| `loading.tsx` | 자동 Suspense 경계로 로딩 UI 표시 |
| `error.tsx` | 에러 경계로 에러 UI 표시 |
| `auth()` | 모든 쿼리에서 userId 확인 필수 |

---

> **다음**: [2.4 Thinking 모드 & URL SearchParams](04-thinking-mode.md)에서 Thinking 모드를 활용하여 URL 기반 필터링 기능을 구현합니다.
