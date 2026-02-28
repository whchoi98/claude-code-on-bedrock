# 2.4 Thinking 모드 & URL SearchParams

> Claude Code의 Extended Thinking 기능을 활용하여 복잡한 URL 기반 필터링 로직을 설계하고 구현합니다.

## 학습 목표

- Claude Code의 Extended Thinking이 무엇이고 어떻게 동작하는지 이해합니다
- Thinking 모드를 활용하여 복잡한 로직을 설계하는 방법을 배웁니다
- Next.js의 `searchParams`를 사용한 서버 사이드 필터링을 구현합니다

## Extended Thinking이란?

[Extended Thinking](https://code.claude.com/docs/en/common-workflows#use-extended-thinking-thinking-mode)은 Claude가 응답하기 전에 **단계별로 깊이 사고**하는 기능입니다. 복잡한 문제를 분석할 때 특히 유용합니다.

### 동작 방식

Claude Code에서 Extended Thinking은 **기본적으로 활성화**되어 있습니다. Claude는 문제의 복잡도에 따라 적응적으로(adaptive) 사고 깊이를 조절합니다.

| 설정 | 방법 | 설명 |
|------|------|------|
| 토글 | `Option+T` (macOS) / `Alt+T` (Linux) | 현재 세션에서 Thinking on/off 전환 |
| Effort 레벨 | `/model` 메뉴에서 조정 | low, medium, high (기본: high) |
| 사고 과정 보기 | `Ctrl+O` | Verbose 모드에서 사고 과정을 회색 이탤릭으로 표시 |
| 전역 설정 | `/config`에서 토글 | 모든 프로젝트에 적용 |

{% hint style="info" %}
**참고**: 공식 문서에 따르면, "think", "think hard", "ultrathink" 등의 키워드는 일반적인 프롬프트 지시로 해석됩니다. Extended Thinking 토큰을 직접 할당하는 것은 아니지만, Claude에게 더 신중하게 문제를 분석하도록 요청하는 효과가 있습니다.
{% endhint %}

### 언제 사용하면 좋은가?

- **복잡한 아키텍처 결정**: 여러 파일에 걸친 변경, 트레이드오프 분석
- **디버깅**: 까다로운 버그의 근본 원인 추적
- **여러 단계의 구현 계획**: 순서와 의존성이 중요한 구현
- **비즈니스 로직**: 다양한 엣지 케이스가 있는 로직

## 실습: URL SearchParams 기반 필터링 구현

세션 목록 페이지에 날짜, 과목, 정렬 기능을 URL 파라미터로 구현합니다. 이 기능은 여러 파일의 변경이 필요하고 서버/클라이언트 컴포넌트 간의 협업이 필요한 복잡한 작업입니다.

### Step 1: Thinking을 유도하는 프롬프트 작성

Claude Code에서 다음과 같이 입력합니다:

```
이것에 대해 깊이 생각해줘. URL search params를 사용해서 세션 목록 페이지에 필터링 기능을 추가해줘:

1. 날짜 범위 필터 (startDate, endDate)
2. 과목별 필터
3. 날짜순 정렬 (최신/오래된 순)

요구사항:
- 서버 컴포넌트에서 Next.js searchParams를 사용해줘 (page.tsx가 searchParams를 prop으로 받음)
- lib/queries/sessions.ts의 쿼리 함수가 필터 파라미터를 받을 수 있도록 업데이트해줘
- 필터 UI용 클라이언트 컴포넌트를 만들어줘:
  - 과목 필터용 shadcn/ui Select
  - 날짜 범위용 shadcn/ui Calendar/DatePicker
  - 정렬 순서용 shadcn/ui Select
- 필터가 변경될 때 useRouter와 useSearchParams를 사용해서 URL이 업데이트되도록 해줘
- 필터가 URL에 있으므로 페이지 새로고침 시에도 유지되어야 해
- 필터가 설정되지 않으면 모든 세션을 표시해줘

데이터 흐름을 잘 생각해줘: URL params → 서버 컴포넌트 → 쿼리 함수 → 데이터베이스
```

### Step 2: Thinking 과정 관찰

{% hint style="tip" %}
**사고 과정 보기**: `Ctrl+O`를 눌러 verbose 모드를 활성화하면 Claude의 사고 과정을 직접 관찰할 수 있습니다. 회색 이탤릭 텍스트로 표시됩니다.
{% endhint %}

Claude가 이 문제를 어떻게 분석하는지 관찰합니다. 다음과 같은 사고 단계를 거칠 것입니다:

**1단계: 데이터 흐름 분석**
```
URL: /dashboard/sessions?startDate=2025-01-01&endDate=2025-01-31&subject=math&sort=newest
                    │
                    ▼
Server Component (page.tsx): searchParams에서 필터 값 추출
                    │
                    ▼
Query Function (lib/queries/sessions.ts): 필터 조건을 SQL WHERE 절로 변환
                    │
                    ▼
Database: 필터링된 결과 반환
```

**2단계: 서버/클라이언트 컴포넌트 분리**
```
서버 컴포넌트 (page.tsx)
├── searchParams 읽기
├── 쿼리 함수 호출
└── 결과 렌더링

클라이언트 컴포넌트 (components/session-filters.tsx)
├── "use client"
├── 필터 UI 렌더링
├── useRouter()로 URL 업데이트
└── useSearchParams()로 현재 필터 상태 읽기
```

**3단계: 엣지 케이스 고려**
- 유효하지 않은 날짜 파라미터 처리
- 존재하지 않는 과목 ID 처리
- 빈 결과 상태 UI

### Step 3: 생성된 코드 확인

Claude가 생성할 파일들을 확인합니다:

#### 쿼리 함수 업데이트 (`lib/queries/sessions.ts`)

```typescript
import { db } from "@/db";
import { studySessions, sessionSubjects, subjects } from "@/db/schema";
import { eq, desc, asc, and, gte, lte, sql } from "drizzle-orm";

export interface SessionFilters {
  startDate?: string;
  endDate?: string;
  subjectId?: string;
  sort?: "newest" | "oldest";
}

export async function getFilteredSessions(
  userId: string,
  filters: SessionFilters
) {
  const conditions = [eq(studySessions.userId, userId)];

  if (filters.startDate) {
    conditions.push(gte(studySessions.date, filters.startDate));
  }
  if (filters.endDate) {
    conditions.push(lte(studySessions.date, filters.endDate));
  }

  const orderBy =
    filters.sort === "oldest"
      ? asc(studySessions.date)
      : desc(studySessions.date);

  let query = db
    .select()
    .from(studySessions)
    .where(and(...conditions))
    .orderBy(orderBy);

  // subjectId 필터는 JOIN 필요
  // ... (Claude가 구현)

  return query;
}
```

#### 필터 UI 컴포넌트 (`components/session-filters.tsx`)

```typescript
"use client";

import { useRouter, useSearchParams } from "next/navigation";
import { useCallback } from "react";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";

interface SessionFiltersProps {
  subjects: { id: string; name: string }[];
}

export function SessionFilters({ subjects }: SessionFiltersProps) {
  const router = useRouter();
  const searchParams = useSearchParams();

  const updateFilter = useCallback(
    (key: string, value: string | null) => {
      const params = new URLSearchParams(searchParams.toString());
      if (value) {
        params.set(key, value);
      } else {
        params.delete(key);
      }
      router.push(`/dashboard/sessions?${params.toString()}`);
    },
    [router, searchParams]
  );

  return (
    <div className="flex gap-4 items-center">
      <Select
        value={searchParams.get("subject") ?? "all"}
        onValueChange={(v) =>
          updateFilter("subject", v === "all" ? null : v)
        }
      >
        <SelectTrigger className="w-40">
          <SelectValue placeholder="All Subjects" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="all">All Subjects</SelectItem>
          {subjects.map((s) => (
            <SelectItem key={s.id} value={s.id}>
              {s.name}
            </SelectItem>
          ))}
        </SelectContent>
      </Select>
      {/* 날짜 범위, 정렬 등 추가 필터 */}
    </div>
  );
}
```

#### 서버 컴포넌트 업데이트 (`page.tsx`)

```typescript
import { auth } from "@clerk/nextjs/server";
import { redirect } from "next/navigation";
import { getFilteredSessions } from "@/lib/queries/sessions";
import { getSubjectsByUserId } from "@/lib/queries/subjects";
import { SessionFilters } from "@/components/session-filters";

interface Props {
  searchParams: Promise<{
    startDate?: string;
    endDate?: string;
    subject?: string;
    sort?: string;
  }>;
}

export default async function SessionsPage({ searchParams }: Props) {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");

  const params = await searchParams;

  const [sessions, subjects] = await Promise.all([
    getFilteredSessions(userId, {
      startDate: params.startDate,
      endDate: params.endDate,
      subjectId: params.subject,
      sort: (params.sort as "newest" | "oldest") ?? "newest",
    }),
    getSubjectsByUserId(userId),
  ]);

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">Study Sessions</h1>
      <SessionFilters subjects={subjects} />
      {/* 세션 테이블 렌더링 */}
    </div>
  );
}
```

### Step 4: 결과 테스트

개발 서버에서 필터링이 정상 동작하는지 확인합니다:

```
pnpm dev를 실행하고 다음 시나리오를 테스트해줘:
1. 기본 뷰 (필터 없음)에서 모든 세션이 표시되는지
2. 과목을 선택하면 세션이 올바르게 필터링되는지
3. 날짜 범위를 설정하면 날짜별로 필터링되는지
4. 정렬 순서 변경이 동작하는지
5. URL이 현재 필터 상태를 반영하는지
6. 페이지 새로고침 시 필터가 유지되는지
발견한 이슈가 있으면 수정해줘.
```

### Step 5: Effort 레벨 조정 (선택)

매우 복잡한 비즈니스 로직이나 아키텍처 결정이 필요한 경우, `/model` 메뉴에서 effort 레벨을 조정할 수 있습니다:

| Effort 레벨 | 사용 시기 |
|-------------|----------|
| **low** | 간단한 변경, 코드 포맷팅, 파일 이름 변경 등 |
| **medium** | 일반적인 기능 구현, 버그 수정 |
| **high** (기본값) | 복잡한 아키텍처, 다중 파일 변경, 디버깅 |

```
/model
```

메뉴에서 effort 레벨을 선택할 수 있습니다.

## URL SearchParams 패턴 정리

```
┌─────────────────┐     URL 변경      ┌──────────────────┐
│  Client Component│ ───────────────► │  Server Component │
│  (Filters UI)   │  useRouter.push  │  (Page)           │
│                 │                   │                   │
│  useSearchParams │ ◄─────────────── │  searchParams prop│
│  현재 필터 읽기   │    자동 동기화    │  필터 값 추출      │
└─────────────────┘                   └────────┬──────────┘
                                               │
                                      쿼리 함수 호출
                                               │
                                               ▼
                                      ┌──────────────────┐
                                      │  Query Function   │
                                      │  필터 조건 → SQL   │
                                      └────────┬──────────┘
                                               │
                                               ▼
                                      ┌──────────────────┐
                                      │  Database         │
                                      │  필터링된 결과     │
                                      └──────────────────┘
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Extended Thinking | Claude가 응답 전에 단계별로 깊이 사고하는 기능 (기본 활성화) |
| Effort 레벨 | `/model`에서 low/medium/high로 사고 깊이 조절 |
| `Ctrl+O` | Verbose 모드에서 사고 과정 관찰 |
| searchParams | 서버 컴포넌트에서 URL 쿼리 파라미터를 받는 prop |
| useSearchParams | 클라이언트 컴포넌트에서 현재 URL 파라미터를 읽는 훅 |
| useRouter | 클라이언트 컴포넌트에서 URL을 프로그래밍적으로 변경하는 훅 |

---

> **다음**: [2.5 서버 액션 & 데이터 변이](05-data-mutations.md)에서 Server Actions를 사용하여 학습 세션 생성 및 과목 관리 기능을 구현합니다.
