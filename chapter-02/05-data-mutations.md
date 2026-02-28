# 2.5 서버 액션 & 데이터 변이

> 데이터 변이(mutation) 가이드 문서를 작성하고, Server Actions로 학습 세션 생성 및 과목 관리 기능을 구현합니다.

## 학습 목표

- 데이터 변이 규칙 문서를 작성하여 일관된 Server Action 패턴을 정의합니다
- Next.js Server Actions로 데이터 생성/수정/삭제를 구현합니다
- Zod를 사용한 서버 측 유효성 검사를 적용합니다
- `useActionState`를 사용한 폼 상태 관리를 배웁니다

## 배경: Next.js Server Actions

Server Actions는 서버에서 실행되는 비동기 함수로, 폼 제출과 데이터 변이를 처리합니다. `"use server"` 지시어로 선언하며, 클라이언트에서 직접 호출할 수 있습니다.

```typescript
// actions/example.ts
"use server";

export async function createItem(formData: FormData) {
  // 서버에서 실행됨
  const name = formData.get("name");
  await db.insert(items).values({ name });
  revalidatePath("/items");
}
```

## 실습: 데이터 변이 가이드 문서 및 기능 구현

### Step 1: docs/data-mutations.md 작성

데이터 변이의 규칙과 패턴을 문서로 정의합니다:

```
데이터 변이 패턴을 문서화하는 docs/data-mutations.md를 생성해줘:

# Data Mutation Patterns

## Core Principles
- Use Next.js Server Actions for ALL data mutations
- Never use API routes for mutations
- Always authenticate before mutating data
- Always validate input on the server

## Server Action Structure
- Place server actions in the actions/ directory
- One file per domain entity (e.g., actions/sessions.ts, actions/subjects.ts)
- Each action file starts with "use server" directive
- Every action must check userId from Clerk's auth()

## Input Validation
- Use Zod schemas for all input validation
- Define schemas alongside the action functions
- Return validation errors in a structured format

## Return Type
All server actions return a consistent type:
```typescript
type ActionResult = {
  success: boolean;
  error?: string;
  data?: any;
};
```

## After Mutation
- Call revalidatePath() to refresh affected pages
- For creation: redirect to the new item's page or list page
- For deletion: redirect to the list page

## Form Handling on Client
- Use React's useActionState hook for form state management
- Show loading state while action is pending
- Display validation errors from the server
- Use shadcn/ui form components
```

### Step 2: 학습 세션 생성 폼 구현

가이드 문서를 참조하여 학습 세션 생성 기능을 구현합니다:

```
docs/data-mutations.md를 읽어줘. "새 학습 세션" 폼을 구현해줘:

1. 세션 생성을 위한 Zod 유효성 검사 스키마 만들기
2. actions/sessions.ts에 server action 만들기:
   - 입력 유효성 검사, 인증 확인, DB 삽입을 수행하는 createSession action
   - { success: boolean, error?: string } 반환
   - 성공 후 revalidatePath("/dashboard/sessions") 호출

3. 폼 UI 만들기:
   - shadcn/ui Dialog를 사용해서 모달로 폼 표시
   - 폼 필드: 날짜 (Calendar picker), 시작 시간 (Input), 종료 시간 (Input), 메모 (Textarea)
   - useActionState로 폼 상태 관리
   - 유효성 검사 에러를 인라인으로 표시
   - 제출 중 로딩 스피너 표시
   - 성공 시 다이얼로그 닫고 새로고침

4. 세션 목록 페이지에 다이얼로그를 여는 "새 세션" 버튼 추가
```

### Step 3: 생성된 Server Action 확인

Claude가 생성한 `actions/sessions.ts`를 확인합니다:

```typescript
// actions/sessions.ts - 기대하는 패턴
"use server";

import { auth } from "@clerk/nextjs/server";
import { db } from "@/db";
import { studySessions } from "@/db/schema";
import { revalidatePath } from "next/cache";
import { z } from "zod";

// Zod 스키마 정의
const createSessionSchema = z.object({
  date: z.string().min(1, "Date is required"),
  startTime: z.string().min(1, "Start time is required"),
  endTime: z.string().min(1, "End time is required"),
  notes: z.string().optional(),
});

// ActionResult 타입 정의
type ActionResult = {
  success: boolean;
  error?: string;
};

export async function createSession(
  prevState: ActionResult | null,
  formData: FormData
): Promise<ActionResult> {
  // 1. 인증 확인
  const { userId } = await auth();
  if (!userId) {
    return { success: false, error: "Unauthorized" };
  }

  // 2. 입력 유효성 검사
  const rawData = {
    date: formData.get("date") as string,
    startTime: formData.get("startTime") as string,
    endTime: formData.get("endTime") as string,
    notes: formData.get("notes") as string,
  };

  const parsed = createSessionSchema.safeParse(rawData);
  if (!parsed.success) {
    return {
      success: false,
      error: parsed.error.issues[0].message,
    };
  }

  // 3. 데이터베이스 삽입
  try {
    await db.insert(studySessions).values({
      userId,
      date: parsed.data.date,
      startTime: parsed.data.startTime,
      endTime: parsed.data.endTime,
      notes: parsed.data.notes ?? null,
    });
  } catch (error) {
    return { success: false, error: "Failed to create session" };
  }

  // 4. 캐시 무효화
  revalidatePath("/dashboard/sessions");
  revalidatePath("/dashboard");

  return { success: true };
}
```

확인할 포인트:

- [x] `"use server"` 지시어
- [x] `auth()`로 인증 확인
- [x] Zod 스키마로 유효성 검사
- [x] `try/catch`로 에러 처리
- [x] `revalidatePath()`로 캐시 무효화
- [x] 일관된 반환 타입 `{ success, error }`

### Step 4: 폼 컴포넌트의 useActionState 확인

```typescript
// components/create-session-dialog.tsx - 기대하는 패턴
"use client";

import { useActionState } from "react";
import { createSession } from "@/actions/sessions";
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";

export function CreateSessionDialog() {
  const [state, formAction, isPending] = useActionState(createSession, null);

  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button>New Session</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Create Study Session</DialogTitle>
        </DialogHeader>
        <form action={formAction} className="space-y-4">
          <div>
            <Label htmlFor="date">Date</Label>
            <Input type="date" id="date" name="date" required />
          </div>
          <div>
            <Label htmlFor="startTime">Start Time</Label>
            <Input type="time" id="startTime" name="startTime" required />
          </div>
          <div>
            <Label htmlFor="endTime">End Time</Label>
            <Input type="time" id="endTime" name="endTime" required />
          </div>
          <div>
            <Label htmlFor="notes">Notes</Label>
            <Textarea id="notes" name="notes" />
          </div>

          {state?.error && (
            <p className="text-sm text-red-500">{state.error}</p>
          )}

          <Button type="submit" disabled={isPending}>
            {isPending ? "Creating..." : "Create Session"}
          </Button>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

{% hint style="info" %}
**`useActionState`**는 React 19에서 도입된 훅으로, Server Action의 결과 상태와 pending 상태를 관리합니다. 이전의 `useFormState`를 대체합니다.

```typescript
const [state, formAction, isPending] = useActionState(action, initialState);
```
- `state`: Server Action의 마지막 반환값
- `formAction`: `<form action={formAction}>`에 전달할 함수
- `isPending`: 액션이 처리 중인지 여부
{% endhint %}

### Step 5: 과목 관리 기능 구현

과목(Subject) CRUD 기능을 구현합니다:

```
docs/data-mutations.md 패턴을 따라 과목 관리를 위한 CRUD actions과 UI를 만들어줘:

1. actions/subjects.ts 만들기:
   - createSubject(prevState, formData): name과 color 필드, Zod로 유효성 검사
   - updateSubject(prevState, formData): id로 name과 color 업데이트
   - deleteSubject(subjectId): 소유권 확인 후 id로 삭제

2. /dashboard/subjects에 과목 관리 페이지 만들기:
   - 현재 사용자의 모든 과목 목록 표시
   - 각 과목은 컬러 badge와 함께 이름 표시
   - "과목 추가" 버튼으로 다이얼로그 폼 열기
   - 각 과목에 편집/삭제 버튼
   - 삭제 확인용 shadcn/ui AlertDialog 사용

3. lib/queries/subjects.ts에 쿼리 함수 만들기:
   - getSubjectsByUserId(userId)
```

### Step 6: Zod 유효성 검사 확인

생성된 코드에서 Zod 스키마가 올바르게 정의되었는지 확인합니다:

```typescript
// actions/subjects.ts - Zod 스키마 예시
const createSubjectSchema = z.object({
  name: z
    .string()
    .min(1, "Subject name is required")
    .max(50, "Subject name must be 50 characters or less"),
  color: z
    .string()
    .regex(/^#[0-9A-Fa-f]{6}$/, "Invalid color format"),
});
```

### Step 7: 전체 테스트

개발 서버에서 기능을 테스트합니다:

```
pnpm dev를 실행하고 테스트해줘:
1. 새 학습 세션 생성 - 목록에 표시되는지 확인
2. 이름과 색상으로 새 과목 생성
3. 과목 이름 편집
4. 과목 삭제 (확인 다이얼로그 체크)
5. 모든 유효성 검사 에러가 올바르게 표시되는지 확인 (빈 폼 제출 시도)
발견한 이슈가 있으면 수정해줘.
```

## Server Action 패턴 요약

```
┌──────────────────────┐
│   Client Component   │
│   (Form UI)          │
│                      │
│  useActionState()    │
│  ├── state (결과)     │
│  ├── formAction      │
│  └── isPending       │
└──────────┬───────────┘
           │ form action
           ▼
┌──────────────────────┐
│   Server Action      │
│   (actions/*.ts)     │
│                      │
│  1. auth() 확인       │
│  2. Zod 유효성 검사   │
│  3. DB 변이           │
│  4. revalidatePath() │
│  5. return result    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Database           │
│   (insert/update/    │
│    delete)           │
└──────────────────────┘
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Server Actions | `"use server"` 지시어로 서버에서 실행되는 비동기 함수 |
| Zod | 타입 안전한 스키마 유효성 검사 라이브러리 |
| `useActionState` | Server Action의 결과/pending 상태를 관리하는 React 훅 |
| `revalidatePath()` | 특정 경로의 캐시를 무효화하여 데이터 새로고침 |
| `actions/` 디렉토리 | 도메인별로 분리된 Server Action 파일 |

---

> **다음**: [2.6 브랜치 관리 & Custom Slash Commands](06-branch-management.md)에서 Git 워크플로와 Skills(커스텀 슬래시 명령)를 설정합니다.
