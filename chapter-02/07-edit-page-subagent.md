# 2.7 편집 페이지 & 서브에이전트

> 서브에이전트를 활용하여 기존 코드를 분석하고, 학습 세션 편집 페이지를 구현한 뒤 코드 리뷰를 수행합니다.

## 학습 목표

- 서브에이전트의 개념과 종류를 이해합니다
- 서브에이전트로 코드베이스를 탐색하는 방법을 배웁니다
- 서브에이전트의 분석 결과를 기반으로 편집 페이지를 구현합니다
- 서브에이전트로 코드 리뷰를 수행합니다

## 서브에이전트란?

[서브에이전트](https://code.claude.com/docs/en/sub-agents)는 **별도의 컨텍스트 창에서 실행**되는 특화된 AI 어시스턴트입니다. 메인 대화의 컨텍스트를 오염시키지 않으면서 특정 작업을 수행할 수 있습니다.

### 왜 서브에이전트를 사용하는가?

- **컨텍스트 보존**: 탐색/조사 결과가 메인 대화를 차지하지 않음
- **도구 제한**: 읽기 전용 등 특정 도구만 사용하도록 제한 가능
- **병렬 작업**: 여러 서브에이전트가 동시에 작업 가능
- **특화된 행동**: 각 서브에이전트에 맞춤 시스템 프롬프트 설정

### 빌트인 서브에이전트

| 서브에이전트 | 모델 | 도구 | 용도 |
|------------|------|------|------|
| **Explore** | Haiku (빠름) | 읽기 전용 | 파일 검색, 코드 탐색 |
| **Plan** | 메인 모델 상속 | 읽기 전용 | Plan 모드에서 리서치 |
| **general-purpose** | 메인 모델 상속 | 모든 도구 | 복잡한 다단계 작업 |

### 서브에이전트 동작 방식

```
메인 대화 (Main Conversation)
│
├── "Use a subagent to investigate..."
│   │
│   ▼
│   ┌─────────────────────────────┐
│   │  Subagent (별도 컨텍스트)     │
│   │  - 자체 시스템 프롬프트       │
│   │  - 제한된 도구 접근           │
│   │  - 독립적인 컨텍스트 창       │
│   │  - 결과 요약 후 반환          │
│   └─────────────┬───────────────┘
│                 │ 요약된 결과
│                 ▼
├── 결과를 기반으로 구현 진행
│
```

## 실습: 서브에이전트를 활용한 편집 페이지 구현

### Step 1: 서브에이전트로 기존 코드 분석

편집 페이지를 만들기 전에, 기존 코드 패턴을 분석합니다:

```
서브에이전트를 사용해서 세션 목록 페이지와 생성 폼이 어떻게 동작하는지 조사해줘. 다음을 파악해야 해:
1. sessions page.tsx가 어떻게 데이터를 패칭하는지
2. 세션 생성 폼과 server action이 어떻게 동작하는지
3. 어떤 UI 컴포넌트와 패턴이 사용되는지
4. 유효성 검사와 에러 처리가 어떻게 구현되어 있는지
편집 페이지를 일관되게 만들 수 있도록 패턴을 요약해줘.
```

Claude Code가 Explore 서브에이전트를 생성하여 코드베이스를 분석합니다. 서브에이전트는 읽기 전용 도구만 사용하므로 코드를 변경하지 않습니다.

서브에이전트가 반환하는 요약 예시:

```
Subagent findings:

1. Data Fetching Pattern:
   - Server components use auth() then call query functions from lib/queries/
   - Query functions accept userId and return typed results

2. Form Pattern:
   - Client components with "use client"
   - useActionState hook for form state
   - Server actions in actions/ directory with Zod validation
   - Consistent return type: { success: boolean, error?: string }

3. UI Components:
   - shadcn/ui Dialog for modal forms
   - shadcn/ui Input, Textarea, Select for form fields
   - Inline error messages below fields

4. After Mutation:
   - revalidatePath() for cache invalidation
   - redirect() for navigation
```

{% hint style="tip" %}
서브에이전트를 사용하면 탐색 결과가 메인 대화 컨텍스트를 차지하지 않습니다. 대신 **요약된 결과만** 메인 대화로 돌아옵니다. 이는 긴 대화에서 컨텍스트를 효율적으로 관리하는 데 유용합니다.
{% endhint %}

### Step 2: 세션 상세 페이지 구현

편집 페이지 전에 먼저 상세 페이지를 만듭니다:

```
/dashboard/sessions/[id]에 세션 상세 페이지를 만들어줘:
- 세션과 관련된 과목, 학습 블록을 패칭
- 세션 날짜, 시간 범위, 학습 시간, 메모 표시
- 컬러 badge로 과목 목록 표시
- 각 과목별 학습 블록 목록 표시 (학습 시간, 읽은 페이지, 메모)
- "편집"과 "삭제" 버튼 추가
- lib/queries/sessions.ts에 getSessionWithDetails 쿼리 함수 생성
- notFound()로 존재하지 않는 경우 처리
```

### Step 3: 학습 세션 편집 페이지 구현

서브에이전트의 분석 결과를 기반으로 편집 페이지를 구현합니다:

```
서브에이전트가 파악한 패턴을 기반으로 /dashboard/sessions/[id]/edit에 세션 편집 페이지를 만들어줘:

1. 서버 컴포넌트로 페이지 생성:
   - getSessionWithDetails로 기존 세션 데이터 패칭
   - 클라이언트 폼 컴포넌트에 데이터 전달
   - 존재하지 않는 경우 처리

2. EditSessionForm 클라이언트 컴포넌트 생성:
   - 기존 세션 데이터로 모든 필드 미리 채우기
   - 날짜, 시작 시간, 종료 시간, 메모 편집 기능
   - 세션에 과목 추가/제거 기능
   - 각 과목별 학습 블록 추가/제거 기능
   - 각 학습 블록: 학습 시간 (분), 읽은 페이지, 메모

3. actions/sessions.ts에 updateSession server action 생성:
   - Zod로 유효성 검사
   - 소유권 확인 (userId)
   - 세션 필드 업데이트
   - 과목과 학습 블록 변경 처리 (추가/제거)
   - 업데이트 후 revalidatePath
   - 성공 시 세션 상세 페이지로 redirect

4. 서브에이전트가 파악한 것과 동일한 패턴 따르기:
   - useActionState로 폼 상태 관리
   - Zod 유효성 검사
   - 일관된 에러 처리
   - shadcn/ui 컴포넌트
```

### Step 4: 생성된 편집 페이지 확인

생성된 파일 구조를 확인합니다:

```
app/dashboard/sessions/[id]/
├── page.tsx           # 세션 상세 페이지
├── edit/
│   └── page.tsx       # 편집 페이지 (서버 컴포넌트)
components/
└── edit-session-form.tsx  # 편집 폼 (클라이언트 컴포넌트)
actions/
└── sessions.ts        # updateSession 액션 추가됨
```

편집 페이지 서버 컴포넌트 예시:

```typescript
// app/dashboard/sessions/[id]/edit/page.tsx
import { auth } from "@clerk/nextjs/server";
import { redirect, notFound } from "next/navigation";
import { getSessionWithDetails } from "@/lib/queries/sessions";
import { getSubjectsByUserId } from "@/lib/queries/subjects";
import { EditSessionForm } from "@/components/edit-session-form";

interface Props {
  params: Promise<{ id: string }>;
}

export default async function EditSessionPage({ params }: Props) {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");

  const { id } = await params;
  const [session, subjects] = await Promise.all([
    getSessionWithDetails(id, userId),
    getSubjectsByUserId(userId),
  ]);

  if (!session) notFound();

  return (
    <div className="max-w-2xl mx-auto space-y-6">
      <h1 className="text-2xl font-bold">Edit Study Session</h1>
      <EditSessionForm session={session} subjects={subjects} />
    </div>
  );
}
```

### Step 5: 서브에이전트로 코드 리뷰

구현이 완료되면 서브에이전트를 사용하여 코드 리뷰를 수행합니다:

```
서브에이전트를 사용해서 편집 페이지 구현을 리뷰해줘. 다음을 확인해줘:
1. 잠재적 보안 이슈 (권한 확인, 입력 유효성 검사)
2. 엣지 케이스 (빈 과목, 학습 블록 없음, 동시 편집)
3. 기존 패턴과의 일관성
4. 누락된 에러 처리
5. 폼의 접근성 이슈
구체적인 개선 제안을 해줘.
```

서브에이전트가 반환하는 리뷰 결과 예시:

```
Code Review Findings:

[Critical]
- The updateSession action should verify session ownership before updating

[Warning]
- No optimistic UI update - user sees no feedback until server responds
- Missing loading state on the edit form submit button

[Suggestion]
- Consider adding a "Cancel" button that navigates back
- Add aria-labels to form fields for accessibility
- Consider debouncing the subject add/remove actions
```

리뷰 결과를 기반으로 수정합니다:

```
코드 리뷰에서 발견된 이슈를 수정해줘:
1. updateSession에 소유권 확인 추가
2. useActionState의 isPending을 사용해서 제출 버튼에 로딩 상태 추가
3. 취소 버튼 추가
4. 모든 폼 필드에 적절한 aria-labels 추가
```

### Step 6: 커밋 스킬로 변경 사항 커밋

2.6에서 만든 커밋 스킬을 사용합니다:

```
/commit
```

## 서브에이전트 활용 팁

### 탐색/조사는 서브에이전트로

```
Use a subagent to find all places where we handle authentication in this project.
```

### 큰 출력이 예상되는 작업에 적합

```
Use a subagent to run the test suite and summarize only the failures.
```

### 병렬 탐색

```
Use subagents in parallel to research the authentication module, the database layer, and the API routes.
```

### 커스텀 서브에이전트 만들기

`/agents` 명령어로 커스텀 서브에이전트를 만들 수 있습니다:

```
/agents
```

또는 `.claude/agents/` 디렉토리에 직접 마크다운 파일을 만들 수 있습니다:

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Grep, Glob
model: sonnet
---

You are a senior code reviewer. Analyze code for:
- Security vulnerabilities
- Performance issues
- Code quality and readability
- Consistency with project patterns
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| 서브에이전트 | 별도 컨텍스트에서 실행되는 특화된 AI 어시스턴트 |
| Explore | 읽기 전용 빌트인 서브에이전트 (Haiku 모델) |
| Plan | Plan 모드에서 리서치용 빌트인 서브에이전트 |
| general-purpose | 모든 도구 사용 가능한 빌트인 서브에이전트 |
| `/agents` | 서브에이전트 관리 명령어 |
| 컨텍스트 보존 | 탐색 결과가 메인 대화를 차지하지 않음 |

## Chapter 2 완료

축하합니다! Chapter 2를 완료했습니다. 이 챕터에서 다음을 배웠습니다:

- **CLAUDE.md 문서**로 일관된 코드 생성
- **서버 컴포넌트**에서 데이터 패칭
- **Thinking 모드**로 복잡한 로직 설계
- **Server Actions**로 데이터 변이 처리
- **Skills**로 반복 작업 자동화
- **서브에이전트**로 코드 탐색 및 리뷰

---

> **다음 챕터**: [Chapter 3: GitHub Actions 자동화](../chapter-03/README.md)에서 Vercel 배포와 GitHub Actions를 연동하여 자동화 워크플로를 구축합니다.
