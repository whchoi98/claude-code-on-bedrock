# 2.2 문서 기반 코드 생성 (CLAUDE.md & 가이드 문서)

> `/init`으로 CLAUDE.md를 생성하고, 프로젝트 가이드 문서를 작성하여 Claude Code가 일관된 코드를 생성하도록 만듭니다.

## 학습 목표

- `/init` 명령어로 CLAUDE.md를 생성하는 방법을 배웁니다
- 프로젝트 규칙을 문서화하여 Claude Code의 코드 품질을 높이는 방법을 이해합니다
- 문서 유무에 따른 코드 생성 품질 차이를 직접 비교합니다

## CLAUDE.md란?

[CLAUDE.md](https://code.claude.com/docs/en/memory)는 Claude Code가 **매 세션 시작 시 자동으로 읽는** 마크다운 파일입니다. 프로젝트 루트에 위치하며, 다음과 같은 역할을 합니다:

- 빌드/개발 명령어 정의
- 코딩 컨벤션과 규칙 설정
- 프로젝트 구조 설명
- 사용할 라이브러리와 패턴 지정

Claude Code의 메모리 시스템은 계층적 구조를 가집니다:

| 메모리 유형 | 위치 | 범위 |
|------------|------|------|
| 프로젝트 메모리 | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀 전체 (소스 컨트롤 공유) |
| 프로젝트 규칙 | `./.claude/rules/*.md` | 팀 전체 (모듈별 규칙) |
| 사용자 메모리 | `~/.claude/CLAUDE.md` | 개인 전체 프로젝트 |
| 로컬 메모리 | `./CLAUDE.local.md` | 개인 현재 프로젝트 |
| 자동 메모리 | `~/.claude/projects/<project>/memory/` | 자동 학습 내용 |

## 실습: CLAUDE.md 생성 및 가이드 문서 작성

### Step 1: `/init`으로 CLAUDE.md 초기 생성

Claude Code 터미널에서 `/init` 명령어를 실행합니다:

```
/init
```

{% hint style="info" %}
**`/init`** 명령어는 현재 프로젝트를 분석하여 CLAUDE.md 파일을 자동으로 생성합니다. 프로젝트 구조, package.json, 설정 파일 등을 읽고 초기 규칙을 만들어줍니다.
{% endhint %}

Claude Code가 프로젝트를 분석하고 CLAUDE.md 초안을 생성합니다. 생성된 파일을 확인해봅니다.

### Step 2: CLAUDE.md에 프로젝트 규칙 추가

`/init`으로 생성된 CLAUDE.md의 내용을 검토한 후, 우리 프로젝트에 맞게 다음 내용이 포함되도록 수정을 요청합니다:

```
CLAUDE.md에 다음 프로젝트별 규칙을 추가해줘:

# Study Tracker - CLAUDE.md

## Build & Dev Commands
- Dev: pnpm dev
- Build: pnpm build
- Lint: pnpm lint

## Code Style
- Use server components by default, "use client" only when needed
- Use Drizzle ORM for all database queries
- Use Clerk's auth() for server-side authentication
- Use shadcn/ui components for UI
- File naming: kebab-case for files, PascalCase for components

## Project Structure
- app/: Next.js App Router pages
- components/: Reusable UI components
- db/: Database schema and connection
- lib/: Utility functions and query functions
- actions/: Server actions

## Data Fetching Rules
- Use server components for data fetching (no useEffect for initial data)
- Create query functions in lib/queries/ directory
- Always filter by userId from Clerk auth()

## Mutation Rules
- Use Next.js Server Actions for all mutations
- Place server actions in actions/ directory
- Always validate input with zod
- Use revalidatePath() after mutations
```

{% hint style="tip" %}
CLAUDE.md는 구체적일수록 좋습니다. "코드를 잘 작성하세요"보다 "서버 컴포넌트를 기본으로 사용하고, 클라이언트 상태가 필요한 경우에만 'use client'를 추가하세요"가 훨씬 효과적입니다.
{% endhint %}

### Step 3: docs/ui.md 가이드 문서 작성

프로젝트에 `docs/` 디렉토리를 만들고 UI 가이드 문서를 작성합니다:

```
UI 컨벤션을 문서화하는 docs/ui.md 파일을 생성해줘:

# UI Conventions

## Component Library
- Use shadcn/ui as the primary component library
- Import components from @/components/ui/

## Layout
- Use a sidebar navigation layout for the dashboard
- Main content area with max-width container
- Responsive design with mobile-first approach

## Data Display
- Use shadcn/ui Card component for statistics and summary data
- Use shadcn/ui Table component for list views
- Use shadcn/ui Badge for status indicators and subject tags

## Color Scheme
- Primary color: blue (default shadcn theme)
- Use subject's own color property for subject badges
- Use muted variants for secondary information

## Forms
- Use shadcn/ui form components (Input, Select, Textarea)
- Use shadcn/ui Calendar for date picking
- Use shadcn/ui Dialog for modal forms
- Show validation errors inline below each field

## Icons
- Use lucide-react for all icons
- Keep icon usage consistent across similar actions
```

### Step 4: 문서를 참조한 대시보드 재생성

이제 CLAUDE.md와 docs/ui.md가 있는 상태에서 대시보드를 다시 생성합니다:

```
먼저 CLAUDE.md와 docs/ui.md를 읽어줘. 그다음 우리 컨벤션에 따라 /dashboard에 대시보드 페이지를 만들어줘. 포함할 내용:
- 네비게이션 링크가 있는 사이드바 레이아웃
- 총 학습 시간, 세션 수, 과목 수를 보여주는 통계 카드
- Table 컴포넌트를 사용한 최근 세션 목록
- "새 세션" 버튼
서버 컴포넌트에서 Drizzle ORM으로 데이터를 패칭하고, Clerk의 auth()로 인증해줘.
```

### Step 5: 결과 비교

이번에 생성된 코드가 이전과 어떻게 다른지 비교합니다:

#### 문서 없이 생성한 코드 (2.1)

```typescript
// 클라이언트 컴포넌트, useEffect, 비일관적 UI
"use client";
export default function Dashboard() {
  const [sessions, setSessions] = useState([]);
  useEffect(() => {
    fetch("/api/sessions")...
  }, []);
  return <div className="bg-white rounded-lg shadow p-4">...</div>;
}
```

#### 문서와 함께 생성한 코드 (2.2)

```typescript
// 서버 컴포넌트, Drizzle ORM, shadcn/ui, Clerk auth
import { auth } from "@clerk/nextjs/server";
import { db } from "@/db";
import { studySessions } from "@/db/schema";
import { eq } from "drizzle-orm";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default async function DashboardPage() {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");

  const sessions = await db
    .select()
    .from(studySessions)
    .where(eq(studySessions.userId, userId))
    .orderBy(desc(studySessions.date));

  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1 p-6">
        <div className="grid gap-4 md:grid-cols-3">
          <Card>
            <CardHeader>
              <CardTitle>Total Study Time</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-3xl font-bold">{totalTime}</p>
            </CardContent>
          </Card>
          {/* ... */}
        </div>
      </main>
    </div>
  );
}
```

{% hint style="success" %}
**핵심 차이점:**
- 서버 컴포넌트로 직접 데이터 패칭 (API 라우트 불필요)
- Clerk `auth()`로 인증 처리
- Drizzle ORM의 query builder 사용
- shadcn/ui 컴포넌트 일관 사용
- 사이드바 레이아웃 적용
{% endhint %}

## CLAUDE.md 활용 팁

### 1. `@` 구문으로 파일 임포트

CLAUDE.md에서 다른 파일을 참조할 수 있습니다:

```markdown
# Project Rules
- UI conventions: @docs/ui.md
- Data fetching patterns: @docs/data-fetching.md
- API conventions: @docs/api.md
```

### 2. `.claude/rules/` 디렉토리로 모듈화

규칙이 많아지면 파일을 분리할 수 있습니다:

```
.claude/
├── CLAUDE.md           # 메인 규칙
└── rules/
    ├── code-style.md   # 코드 스타일 규칙
    ├── testing.md      # 테스트 컨벤션
    └── api-design.md   # API 설계 규칙
```

### 3. `/memory` 명령어로 편집

세션 중에 `/memory` 명령어를 사용하면 시스템 에디터에서 직접 CLAUDE.md를 편집할 수 있습니다:

```
/memory
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| `/init` | 프로젝트 분석 후 CLAUDE.md 자동 생성 |
| CLAUDE.md | 매 세션 시작 시 자동 로드되는 프로젝트 규칙 파일 |
| 가이드 문서 (docs/) | 특정 영역의 상세 컨벤션을 정의하는 참조 문서 |
| `@` 임포트 | CLAUDE.md에서 다른 파일을 참조하는 구문 |
| `/memory` | 세션 중 메모리 파일을 편집하는 명령어 |

---

> **다음**: [2.3 서버 컴포넌트 데이터 패칭](03-data-fetching.md)에서 데이터 패칭 가이드 문서를 작성하고 학습 세션 목록 페이지를 구현합니다.
