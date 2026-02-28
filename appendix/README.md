# 참고 자료 & 문제 해결

## 참고 링크 모음

### Claude Code

| 자료 | URL |
|------|-----|
| Claude Code 공식 문서 | [https://code.claude.com/docs](https://code.claude.com/docs) |
| Amazon Bedrock 연동 가이드 | [https://code.claude.com/docs/en/amazon-bedrock](https://code.claude.com/docs/en/amazon-bedrock) |
| Best Practices | [https://code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices) |
| Skills (Custom Slash Commands) | [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills) |
| Subagents | [https://code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents) |
| GitHub Actions | [https://code.claude.com/docs/en/github-actions](https://code.claude.com/docs/en/github-actions) |
| MCP 통합 | [https://code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp) |
| CLI 레퍼런스 | [https://code.claude.com/docs/en/cli-reference](https://code.claude.com/docs/en/cli-reference) |

### AWS

| 자료 | URL |
|------|-----|
| Amazon Bedrock 문서 | [https://docs.aws.amazon.com/bedrock/](https://docs.aws.amazon.com/bedrock/) |
| Bedrock 가격 | [https://aws.amazon.com/bedrock/pricing/](https://aws.amazon.com/bedrock/pricing/) |
| Bedrock Inference Profiles | [https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html) |
| IAM 문서 | [https://docs.aws.amazon.com/IAM/latest/UserGuide/](https://docs.aws.amazon.com/IAM/latest/UserGuide/) |

### 프레임워크 & 라이브러리

| 자료 | URL |
|------|-----|
| Next.js 문서 | [https://nextjs.org/docs](https://nextjs.org/docs) |
| Drizzle ORM 문서 | [https://orm.drizzle.team/docs/overview](https://orm.drizzle.team/docs/overview) |
| Clerk 문서 | [https://clerk.com/docs](https://clerk.com/docs) |
| Neon 문서 | [https://neon.tech/docs](https://neon.tech/docs) |
| shadcn/ui 문서 | [https://ui.shadcn.com](https://ui.shadcn.com) |
| Tailwind CSS 문서 | [https://tailwindcss.com/docs](https://tailwindcss.com/docs) |
| Vercel 문서 | [https://vercel.com/docs](https://vercel.com/docs) |
| Zod 문서 | [https://zod.dev](https://zod.dev) |

---

## 문제 해결 (Troubleshooting)

### AWS 자격증명 오류

#### `Unable to locate credentials`

AWS 자격증명이 설정되지 않았습니다.

```bash
# 자격증명 확인
aws sts get-caller-identity

# 자격증명 설정
aws configure

# 또는 환경변수로 설정
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_REGION=us-east-1
```

#### `ExpiredTokenException`

AWS 자격증명이 만료되었습니다.

```bash
# SSO 사용 시 재로그인
aws sso login --profile your-profile

# 임시 자격증명 사용 시 세션 토큰 갱신
aws sts get-session-token
```

#### `AccessDeniedException`

IAM 권한이 부족합니다. 다음 권한이 있는지 확인하세요:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:ListInferenceProfiles"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/*",
        "arn:aws:bedrock:*:*:application-inference-profile/*",
        "arn:aws:bedrock:*:*:foundation-model/*"
      ]
    }
  ]
}
```

### Bedrock 모델 액세스 오류

#### `You don't have access to the model with the specified model ID`

해당 리전에서 Claude 모델 액세스가 활성화되지 않았습니다.

1. AWS 콘솔에서 **Amazon Bedrock > Model access**로 이동
2. Anthropic 모델에 대해 **Request model access** 클릭
3. 처음 사용 시 Use case details 제출 필요
4. 활성화까지 몇 분 소요될 수 있음

#### `on-demand throughput isn't supported`

Inference Profile을 사용해야 합니다.

```bash
# 사용 가능한 inference profile 확인
aws bedrock list-inference-profiles --region us-east-1

# 모델 ID를 inference profile ID로 변경
export ANTHROPIC_MODEL='us.anthropic.claude-sonnet-4-6'
```

#### 리전별 모델 가용성

모든 Claude 모델이 모든 리전에서 사용 가능하지는 않습니다.

```bash
# 다른 리전으로 전환
export AWS_REGION=us-east-1

# Cross-region inference profile 사용 (권장)
export ANTHROPIC_MODEL='us.anthropic.claude-sonnet-4-6'
```

### Claude Code 연결 문제

#### Claude Code가 Bedrock에 연결되지 않음

```bash
# 필수 환경변수 확인
echo $CLAUDE_CODE_USE_BEDROCK  # 1이어야 함
echo $AWS_REGION               # 리전이 설정되어 있어야 함

# 환경변수 설정
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1

# AWS 자격증명 확인
aws sts get-caller-identity
```

{% hint style="warning" %}
`AWS_REGION`은 반드시 **환경변수**로 설정해야 합니다. Claude Code는 `.aws/config` 파일에서 리전을 읽지 않습니다.
{% endhint %}

#### `/login`, `/logout`이 동작하지 않음

Bedrock 모드에서는 인증이 AWS 자격증명으로 처리되므로 `/login`과 `/logout` 명령이 비활성화됩니다. 이는 정상 동작입니다.

### Neon DB 연결 오류

#### `connection refused` 또는 `ECONNREFUSED`

```bash
# .env.local에서 DATABASE_URL 확인
cat .env.local | grep DATABASE_URL

# 연결 문자열 형식 확인
# postgresql://username:password@host/database?sslmode=require
```

#### SSL 관련 오류

Neon은 SSL 연결이 필수입니다. 연결 문자열에 `?sslmode=require`가 포함되어 있는지 확인하세요.

### Clerk 인증 오류

#### `Clerk: Missing publishable key`

```bash
# .env.local 파일에 키가 있는지 확인
cat .env.local | grep CLERK

# 필요한 환경변수
# NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
# CLERK_SECRET_KEY=sk_test_...
```

#### 미들웨어 관련 오류

```bash
# middleware.ts 파일이 프로젝트 루트에 있는지 확인
ls middleware.ts

# 미들웨어 내용 확인 - clerkMiddleware()가 export되어 있어야 함
```

### 빌드 오류

#### TypeScript 오류

```bash
# Claude Code에 수정 요청
claude -p "Run pnpm build and fix all TypeScript errors"
```

#### 패키지 의존성 오류

```bash
# node_modules 재설치
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

---

## 비용 관리 팁

### Bedrock 토큰 비용

Claude Code는 Amazon Bedrock의 토큰 기반 과금 모델을 사용합니다.

| 모델 | 입력 토큰 (1M) | 출력 토큰 (1M) |
|------|----------------|----------------|
| Claude Sonnet 4 | $3.00 | $15.00 |
| Claude Haiku 3.5 | $0.80 | $4.00 |

> 최신 가격은 [Amazon Bedrock 가격 페이지](https://aws.amazon.com/bedrock/pricing/)에서 확인하세요.

### 비용 절감 방법

1. **`max-turns` 제한**: GitHub Actions에서 `--max-turns 10`으로 불필요한 반복 방지
2. **모델 선택**: 간단한 작업에는 Haiku, 복잡한 작업에는 Sonnet 사용
3. **컨텍스트 관리**: `/clear`로 불필요한 컨텍스트 정리
4. **CLAUDE.md 간결 유지**: 불필요한 내용은 제거하여 토큰 소비 최소화
5. **서브에이전트 활용**: Explore 에이전트는 Haiku를 사용하여 비용 효율적
6. **프롬프트 캐싱**: 가능한 리전에서는 프롬프트 캐싱 활용

### AWS 비용 모니터링

```bash
# AWS Cost Explorer에서 Bedrock 비용 확인
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics "BlendedCost" \
  --filter '{"Dimensions":{"Key":"SERVICE","Values":["Amazon Bedrock"]}}'
```

---

## DB 스키마 참고

학습 트래커 애플리케이션의 데이터베이스 스키마입니다.

### ERD (Entity Relationship Diagram)

```
study_sessions ──┬── session_subjects ──┬── study_blocks
                 │                      │
                 └── (1:N)              └── (1:N)

subjects ────────── session_subjects
                    (N:M 관계)
```

### 테이블 구조

#### study_sessions (학습 세션)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID | Primary Key |
| userId | TEXT | Clerk 사용자 ID |
| date | DATE | 학습 날짜 |
| startTime | TIMESTAMP | 시작 시간 |
| endTime | TIMESTAMP | 종료 시간 |
| notes | TEXT | 메모 (선택) |
| createdAt | TIMESTAMP | 생성 시각 |
| updatedAt | TIMESTAMP | 수정 시각 |

#### subjects (과목)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID | Primary Key |
| userId | TEXT | Clerk 사용자 ID |
| name | TEXT | 과목명 (예: 수학, 영어) |
| color | TEXT | 색상 코드 (예: #3B82F6) |
| createdAt | TIMESTAMP | 생성 시각 |

#### session_subjects (세션-과목 연결)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID | Primary Key |
| sessionId | UUID | FK → study_sessions.id |
| subjectId | UUID | FK → subjects.id |

#### study_blocks (학습 블록)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | UUID | Primary Key |
| sessionSubjectId | UUID | FK → session_subjects.id |
| durationMinutes | INTEGER | 학습 시간 (분) |
| pagesRead | INTEGER | 읽은 페이지 수 (선택) |
| notes | TEXT | 메모 (선택) |
| createdAt | TIMESTAMP | 생성 시각 |

### Drizzle ORM 스키마 예시

```typescript
import { pgTable, uuid, text, date, timestamp, integer } from 'drizzle-orm/pg-core';

export const studySessions = pgTable('study_sessions', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: text('user_id').notNull(),
  date: date('date').notNull(),
  startTime: timestamp('start_time'),
  endTime: timestamp('end_time'),
  notes: text('notes'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const subjects = pgTable('subjects', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: text('user_id').notNull(),
  name: text('name').notNull(),
  color: text('color').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

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
```
