# 1.7 데이터베이스 설정 (Neon Postgres + Drizzle ORM + shadcn/ui)

> Neon Postgres 데이터베이스를 생성하고, Drizzle ORM과 shadcn/ui를 설치 및 구성합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 자연어로 패키지 설치 및 설정, `/clear`

> ⭐⭐ **난이도**: 보통

{% hint style="info" %}
**용어 설명**
- **Neon**: 클라우드에서 동작하는 PostgreSQL 데이터베이스 서비스입니다. 서버를 직접 관리할 필요 없이 데이터베이스를 사용할 수 있습니다.
- **Drizzle ORM**: TypeScript 코드로 데이터베이스를 다룰 수 있게 해주는 라이브러리입니다. SQL을 직접 작성하지 않아도 됩니다.
- **shadcn/ui**: 미리 만들어진 UI 컴포넌트 모음입니다. 버튼, 카드, 테이블 등을 쉽게 사용할 수 있습니다.
{% endhint %}

### 왜 Neon + Drizzle + shadcn/ui 조합을 사용하나요?

| 도구 | 역할 | 선택 이유 |
|------|------|-----------|
| **Neon** | 데이터베이스 | 무료 티어 제공, 서버리스로 관리 부담 없음, Vercel과 궁합이 좋음 |
| **Drizzle** | ORM | TypeScript 친화적, 타입 안전성, 가볍고 빠름 |
| **shadcn/ui** | UI 컴포넌트 | 커스터마이징 가능, Tailwind CSS 기반, Next.js에 최적화 |

## 개요

이 섹션에서는 학습 트래커의 데이터 저장소로 사용할 Neon Postgres 서버리스 데이터베이스를 생성하고, TypeScript 친화적인 Drizzle ORM을 설정합니다. 또한 UI 프레임워크인 shadcn/ui를 초기화하여 다음 챕터에서 사용할 컴포넌트를 준비합니다.

{% hint style="info" %}
이전 섹션에서 배운 대로, 새 작업을 시작하기 전에 `/clear`로 컨텍스트를 초기화하세요.
{% endhint %}

## 1단계: Neon 계정 및 프로젝트 생성

### Neon 계정 생성

1. [neon.tech](https://neon.tech)에 접속합니다.
2. **Sign up**을 클릭하여 계정을 생성합니다 (GitHub 계정으로 로그인 가능).

### 프로젝트 생성

1. 대시보드에서 **New Project**를 클릭합니다.
2. 프로젝트 설정:
   - **Project name**: `study-tracker`
   - **Region**: 가장 가까운 리전 선택 (예: `US East (Ohio)`)
   - **Postgres version**: 기본값 사용
3. **Create Project**를 클릭합니다.

## 2단계: 연결 문자열 복사

프로젝트가 생성되면 **Connection Details** 섹션에서 연결 문자열을 확인합니다.

1. **Connection string** 탭에서 전체 연결 문자열을 복사합니다.
2. 형식: `postgresql://username:password@host/database?sslmode=require`

### .env.local에 추가

`.env.local` 파일에 데이터베이스 연결 문자열을 추가합니다:

→ *이 프롬프트는 데이터베이스 연결 문자열을 환경변수 파일에 추가하기 위한 것입니다:*

```
.env.local에 Neon 연결 문자열로 DATABASE_URL을 추가해줘:
DATABASE_URL=postgresql://username:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
```

{% hint style="danger" %}
**중요**: 위의 연결 문자열은 예시입니다. Neon 대시보드에서 복사한 **실제 연결 문자열**을 사용하세요.
{% endhint %}

현재 `.env.local` 파일의 전체 내용:

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_your-key-here
CLERK_SECRET_KEY=sk_test_your-key-here
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# Database
DATABASE_URL=postgresql://username:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## 3단계: Drizzle ORM 및 shadcn/ui 설치

Claude Code에 다음 프롬프트를 입력합니다:

→ *이 프롬프트는 Drizzle ORM, Neon 드라이버, shadcn/ui를 한 번에 설치하고 설정하기 위한 것입니다:*

```
Neon Postgres serverless 드라이버와 함께 Drizzle ORM을 설치하고 설정해줘.
shadcn/ui도 기본 테마로 초기화하고, 다음 shadcn 컴포넌트를 추가해줘: button, card, input, label, select, table, dialog, popover, calendar.

세부 요구사항:
1. drizzle-orm, @neondatabase/serverless 설치, drizzle-kit은 dev dependency로 설치
2. 프로젝트 루트에 drizzle.config.ts 생성
3. db/index.ts에 데이터베이스 연결 유틸리티 생성
4. shadcn/ui 초기화 및 나열된 컴포넌트 추가
5. package.json에 drizzle-kit 스크립트 추가
```

### Claude Code가 수행하는 작업

Claude Code는 다음을 자동으로 처리합니다:

1. **패키지 설치**:
   ```bash
   pnpm add drizzle-orm @neondatabase/serverless
   pnpm add -D drizzle-kit
   ```

2. **shadcn/ui 초기화**:
   ```bash
   pnpm dlx shadcn@latest init
   ```

3. **shadcn/ui 컴포넌트 추가**:
   ```bash
   pnpm dlx shadcn@latest add button card input label select table dialog popover calendar
   ```

4. **설정 파일 생성**: `drizzle.config.ts`, `db/index.ts`

5. **package.json 스크립트 추가**: `db:generate`, `db:migrate` 등

## 4단계: 설정 파일 확인

### drizzle.config.ts

→ *이 프롬프트는 Drizzle 설정 파일이 올바르게 생성되었는지 확인하기 위한 것입니다:*

```
drizzle.config.ts의 내용을 보여줘
```

다음과 유사한 구조여야 합니다:

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  out: './drizzle',
  schema: './db/schema.ts',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

### db/index.ts

→ *이 프롬프트는 데이터베이스 연결 유틸리티가 올바르게 생성되었는지 확인하기 위한 것입니다:*

```
db/index.ts의 내용을 보여줘
```

Neon 서버리스 드라이버를 사용한 연결 유틸리티:

```typescript
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle({ client: sql });
```

{% hint style="info" %}
Neon의 서버리스 드라이버(`@neondatabase/serverless`)는 HTTP 기반으로 동작하여 서버리스 환경(Vercel 등)에서 Cold Start 시간을 최소화합니다.
{% endhint %}

### package.json 스크립트

→ *이 프롬프트는 데이터베이스 관련 스크립트가 올바르게 추가되었는지 확인하기 위한 것입니다:*

```
package.json의 scripts 섹션을 보여줘
```

다음 스크립트가 추가되어 있어야 합니다:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio"
  }
}
```

## 5단계: shadcn/ui 설정 확인

### components.json

→ *이 프롬프트는 shadcn/ui 설정 파일이 올바르게 생성되었는지 확인하기 위한 것입니다:*

```
components.json을 보여줘
```

shadcn/ui 설정 파일이 생성되어 있는지 확인합니다:

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

### UI 컴포넌트 확인

→ *이 프롬프트는 shadcn/ui 컴포넌트 파일들이 모두 생성되었는지 확인하기 위한 것입니다:*

```
components/ui 디렉토리의 모든 파일을 보여줘
```

다음 컴포넌트 파일들이 생성되어 있어야 합니다:

```
components/ui/
├── button.tsx
├── calendar.tsx
├── card.tsx
├── dialog.tsx
├── input.tsx
├── label.tsx
├── popover.tsx
├── select.tsx
└── table.tsx
```

## 6단계: CLAUDE.md 업데이트

데이터베이스 및 UI 관련 정보를 CLAUDE.md에 추가합니다:

→ *이 프롬프트는 CLAUDE.md에 데이터베이스 및 UI 설정 정보를 기록하기 위한 것입니다:*

```
CLAUDE.md에 다음 섹션을 추가해줘:

## 데이터베이스
- Neon Postgres + Drizzle ORM
- 연결: db/index.ts (neon-http 드라이버)
- 스키마: db/schema.ts
- 설정: drizzle.config.ts
- 명령어: pnpm db:generate, pnpm db:migrate, pnpm db:studio

## UI
- shadcn/ui + Tailwind CSS
- 컴포넌트 위치: components/ui/
- 사용 가능: button, card, input, label, select, table, dialog, popover, calendar
```

## 7단계: 연결 테스트

데이터베이스 연결이 정상적으로 작동하는지 확인합니다:

→ *이 프롬프트는 데이터베이스 연결이 정상인지 테스트하기 위한 임시 API를 만들기 위한 것입니다:*

```
app/api/test-db/route.ts에 데이터베이스에 연결하고 성공 메시지를 반환하는 임시 API 라우트를 만들어줘.
개발 서버를 실행하고 테스트해줘.
```

테스트가 성공하면 임시 API 라우트를 삭제합니다:

→ *이 프롬프트는 테스트 완료 후 불필요한 파일을 정리하기 위한 것입니다:*

```
test-db API 라우트를 삭제해줘. 데이터베이스 연결 테스트용이었어.
```

## 요약

| 항목 | 상태 |
|------|------|
| Neon Postgres 프로젝트 생성 | &#x2705; |
| DATABASE_URL 환경변수 설정 | &#x2705; |
| Drizzle ORM 설치 및 설정 | &#x2705; |
| shadcn/ui 초기화 및 컴포넌트 추가 | &#x2705; |
| DB 연결 테스트 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Neon Postgres 프로젝트가 생성되고 연결 문자열이 `.env.local`에 설정됨
- [ ] Drizzle ORM이 설치되고 `drizzle.config.ts`, `db/index.ts`가 생성됨
- [ ] shadcn/ui가 초기화되고 필요한 컴포넌트들이 `components/ui/`에 생성됨
- [ ] 데이터베이스 연결 테스트가 성공함
- [ ] CLAUDE.md에 데이터베이스 및 UI 정보가 추가됨

다음 섹션에서는 [Plan 모드로 스키마를 설계하고 MCP를 통해 시드 데이터를 삽입](08-schema-design-mcp.md)합니다.
