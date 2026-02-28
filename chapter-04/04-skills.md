# 4.4 에이전트 스킬 (Agent Skills)

> Claude가 자동으로 활용하는 스킬과 커스텀 서브에이전트를 만들어 학습 트래커 개발을 효율화합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 자동 스킬, 서브에이전트, 커스텀 에이전트 (.claude/agents/)

> ⭐⭐⭐ **난이도**: 어려움

## 개요

이전 섹션에서는 사용자가 직접 호출하는 스킬(`disable-model-invocation: true`)을 만들었습니다. 이 섹션에서는 Claude가 **자동으로 사용하는 스킬**과 **커스텀 서브에이전트**를 다룹니다.

> **공식 문서 참고**:
> - 스킬: [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)
> - 서브에이전트: [https://code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)

## 스킬 호출 방식 비교

| 프론트매터 설정 | 사용자가 호출 | Claude가 자동 호출 | 용도 |
|----------------|-------------|------------------|------|
| (기본값) | O | O | 범용 가이드/작업 |
| `disable-model-invocation: true` | O | X | 배포, 커밋 등 수동 작업 |
| `user-invocable: false` | X | O | 백그라운드 지식 |

## Step 1: 자동 스킬 만들기 (Model-invocable)

Claude가 서버 액션이나 API를 수정할 때 자동으로 참조하는 컨벤션 스킬을 만듭니다.

### 디렉토리 생성

```bash
mkdir -p .claude/skills/api-conventions
```

### SKILL.md 작성

`.claude/skills/api-conventions/SKILL.md`:

```yaml
---
name: api-conventions
description: Server action and API conventions for the study tracker. Use when creating or modifying server actions and data access patterns.
---

# Study Tracker API Conventions

## Server Actions
- Place in `actions/` directory
- Always validate with zod
- Always check `auth()` for userId
- Return `{ success: boolean, error?: string, data?: T }`
- Use `revalidatePath()` after mutations

## Data Access
- Place query functions in `lib/queries/`
- Always filter by userId
- Use Drizzle query builder with relations
- Handle pagination with limit/offset

## Example Server Action

```typescript
"use server";

import { auth } from "@clerk/nextjs/server";
import { z } from "zod";
import { db } from "@/db";
import { studyBlocks } from "@/db/schema";
import { revalidatePath } from "next/cache";

const createBlockSchema = z.object({
  sessionSubjectId: z.number(),
  durationMinutes: z.number().min(1).max(480),
  pagesRead: z.number().optional(),
  notes: z.string().optional(),
});

export async function createStudyBlock(input: z.infer<typeof createBlockSchema>) {
  const { userId } = await auth();
  if (!userId) return { success: false, error: "Unauthorized" };

  const validated = createBlockSchema.safeParse(input);
  if (!validated.success) return { success: false, error: "Invalid input" };

  const block = await db.insert(studyBlocks).values({
    ...validated.data,
  }).returning();

  revalidatePath("/dashboard");
  return { success: true, data: block[0] };
}
```
```

### 이 스킬이 동작하는 방식

`disable-model-invocation`이 설정되지 않았으므로(기본값 `false`), Claude는 서버 액션을 생성하거나 수정할 때 이 스킬의 `description`을 보고 자동으로 로드합니다. 별도의 슬래시 명령 없이도 컨벤션이 적용됩니다.

{% hint style="info" %}
스킬의 `description`은 항상 컨텍스트에 로드되어 Claude가 언제 사용할지 판단합니다. 하지만 전체 스킬 내용은 실제로 호출될 때만 로드됩니다.
{% endhint %}

## Step 2: 서브에이전트에서 실행하는 스킬

`context: fork`를 사용하면 스킬이 독립된 서브에이전트에서 실행됩니다. 메인 대화의 컨텍스트 창을 소비하지 않으므로 탐색이나 분석 작업에 적합합니다.

### 코드베이스 리서치 스킬

```bash
mkdir -p .claude/skills/deep-research
```

`.claude/skills/deep-research/SKILL.md`:

```yaml
---
name: deep-research
description: 코드베이스에서 주제를 철저히 조사
context: fork
agent: Explore
---

$ARGUMENTS를 철저히 조사해줘:

1. Glob과 Grep을 사용하여 관련 파일 모두 찾기
2. 코드를 읽고 분석
3. 의존성과 관계 매핑
4. 구체적인 파일 참조와 라인 번호와 함께 결과 요약
```

사용 예:

→ *이 프롬프트는 코드베이스 리서치 스킬을 실행하여 인증 구현을 조사하기 위한 것입니다:*

```
/deep-research 앱 전체에서 인증이 어떻게 구현되어 있는지 조사해줘
```

`agent: Explore`를 지정하면 읽기 전용 도구만 사용하는 빠른 Explore 에이전트에서 실행됩니다. 사용 가능한 빌트인 에이전트:

| 에이전트 | 모델 | 도구 | 용도 |
|---------|------|------|------|
| `Explore` | Haiku (빠름) | 읽기 전용 | 탐색, 분석 |
| `Plan` | 상속 | 읽기 전용 | 계획 수립 |
| `general-purpose` | 상속 | 전체 | 복합 작업 |

## Step 3: 동적 컨텍스트 주입

스킬 내에서 `` !`command` `` 구문을 사용하면 셸 명령의 출력을 동적으로 주입할 수 있습니다. 명령은 스킬 내용이 Claude에 전달되기 전에 실행되며, 출력으로 대체됩니다.

### PR 리뷰 스킬

```bash
mkdir -p .claude/skills/pr-review
```

`.claude/skills/pr-review/SKILL.md`:

```yaml
---
name: pr-review
description: 현재 PR 변경사항을 리뷰
context: fork
agent: Explore
---

## PR Context
- Diff: !`git diff main...HEAD`
- Changed files: !`git diff main...HEAD --name-only`

## 리뷰 지침
다음 기준으로 변경사항을 리뷰해줘:
1. 코드 품질과 일관성
2. 잠재적 버그 또는 엣지 케이스
3. 보안 문제
4. 성능 영향
5. 테스트 커버리지

파일별로 정리하여 피드백을 제공하고, 심각도(critical/warning/suggestion)를 표시해줘.
```

실행 흐름:

1. `` !`git diff main...HEAD` ``와 `` !`git diff main...HEAD --name-only` ``가 먼저 실행됨
2. 명령 출력이 스킬 내용에 삽입됨
3. 완성된 내용이 Explore 서브에이전트에 전달됨
4. 서브에이전트가 분석 후 요약을 메인 대화에 반환

## Step 4: 커스텀 서브에이전트

스킬의 `agent` 필드에 빌트인 에이전트 대신 **커스텀 서브에이전트**를 지정할 수 있습니다. 서브에이전트는 `.claude/agents/` 디렉토리에 마크다운 파일로 정의합니다.

### 에이전트 디렉토리 생성

```bash
mkdir -p .claude/agents
```

### 학습 트래커 전문 에이전트 정의

`.claude/agents/study-tracker-expert.md`:

```yaml
---
name: study-tracker-expert
description: Expert on the study tracker codebase. Use proactively when working on features.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are an expert on the Study Tracker application.
The app uses Next.js 15 App Router, Drizzle ORM, Clerk auth, and shadcn/ui.

Key directories:
- app/: Pages and layouts (App Router)
- db/: Schema and database connection
- actions/: Server actions for mutations
- lib/queries/: Data fetching functions
- components/: Shared UI components

Key patterns:
- Server components for data fetching, client components for interactivity
- All mutations go through server actions with zod validation
- Authentication via Clerk's auth() function
- Data always filtered by userId for multi-tenancy

Always check existing patterns before suggesting changes.
```

### 프론트매터 필드 설명

| 필드 | 값 | 의미 |
|------|-----|------|
| `name` | `study-tracker-expert` | 에이전트 식별자 |
| `description` | 설명 | Claude가 언제 위임할지 판단. "Use proactively"는 적극적 위임 유도 |
| `tools` | `Read, Grep, Glob, Bash` | 이 에이전트가 사용할 수 있는 도구 |
| `model` | `sonnet` | 사용할 모델. `sonnet`, `opus`, `haiku`, `inherit` 중 선택 |

### 에이전트 범위

서브에이전트도 스킬과 마찬가지로 저장 위치에 따라 범위가 달라집니다:

| 위치 | 범위 |
|------|------|
| `.claude/agents/` | 현재 프로젝트 |
| `~/.claude/agents/` | 모든 프로젝트 |

## Step 5: 에이전트 관리

### /agents 명령

Claude Code에서 `/agents` 명령으로 에이전트를 관리할 수 있습니다:

→ *이 프롬프트는 사용 가능한 에이전트 목록을 확인하고 관리하기 위한 것입니다:*

```
/agents
```

이 명령으로:
- 사용 가능한 모든 에이전트 확인 (빌트인 + 커스텀)
- 새 에이전트 생성 (가이드 설정 또는 Claude 생성)
- 기존 에이전트 편집/삭제
- 에이전트 도구 접근 설정

### CLI에서 에이전트 목록 확인

```bash
claude agents
```

### 스킬에서 커스텀 에이전트 사용

스킬의 `agent` 필드에 커스텀 에이전트 이름을 지정할 수 있습니다:

```yaml
---
name: feature-plan
description: 학습 트래커의 새로운 기능 계획
context: fork
agent: study-tracker-expert
---

다음의 구현을 계획해줘: $ARGUMENTS

1. 변경이 필요한 파일 식별
2. 필요한 데이터베이스 스키마 변경 목록 (있는 경우)
3. 필요한 server actions 개요
4. UI 컴포넌트 설명
5. 엣지 케이스 또는 잠재적 이슈 기록
```

## 스킬과 서브에이전트의 관계

스킬과 서브에이전트는 두 가지 방향으로 결합됩니다:

| 접근 방식 | 시스템 프롬프트 | 작업 내용 |
|-----------|--------------|-----------|
| 스킬 + `context: fork` | 에이전트 타입에서 가져옴 | SKILL.md 내용이 작업 지시 |
| 서브에이전트 + `skills` 필드 | 에이전트 마크다운 본문 | Claude의 위임 메시지가 작업 지시 |

서브에이전트에 스킬을 사전 로드할 수도 있습니다:

```yaml
---
name: api-developer
description: 팀 컨벤션에 따라 API 엔드포인트 구현
skills:
  - api-conventions
---

API 엔드포인트를 구현해줘. 사전 로드된 스킬의 컨벤션을 따라줘.
```

이 경우 `api-conventions` 스킬의 전체 내용이 에이전트 컨텍스트에 주입됩니다.

## 정리

이 섹션에서 배운 내용:

- **자동 스킬**: `description`만 잘 작성하면 Claude가 필요할 때 자동 호출
- **context: fork**: 서브에이전트에서 격리 실행하여 메인 컨텍스트 절약
- **동적 컨텍스트 주입**: `` !`command` ``로 셸 명령 출력을 스킬에 삽입
- **커스텀 서브에이전트**: `.claude/agents/`에 전문 에이전트 정의
- **/agents 명령**: 에이전트 관리 인터페이스
- **스킬 + 에이전트 결합**: `agent` 필드로 커스텀 에이전트에서 스킬 실행

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] 커스텀 에이전트 정의 파일 생성 확인 (`.claude/agents/study-tracker-expert.md`)
- [ ] 자동 스킬 (api-conventions)이 서버 액션 작성 시 자동 호출되는지 확인
- [ ] `context: fork` 스킬이 서브에이전트에서 격리 실행되는지 확인
- [ ] `/agents` 명령으로 에이전트 목록 확인

---

다음: [4.5 Bash 명령 & 팁](05-bash-and-tips.md)
