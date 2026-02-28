# 2.1 대시보드 첫 시도 (문서 없이)

> 의도적으로 문서 없이 코드를 생성하여 Claude Code에서 컨텍스트가 왜 중요한지 직접 체험합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: `/rewind`

> ⭐ **난이도**: 쉬움

## 학습 목표

- Claude Code에 충분한 컨텍스트를 제공하지 않았을 때 발생하는 문제를 이해합니다
- CLAUDE.md 문서의 필요성을 체감합니다
- `/rewind` 명령어로 변경 사항을 되돌리는 방법을 배웁니다

## 실습: 두 가지 방법 비교하기

### Step 1: 방법 A — 프롬프트만으로 대시보드 생성

CLAUDE.md나 가이드 문서를 만들지 않은 상태에서, 바로 대시보드 생성을 요청해봅니다.

Claude Code 터미널에서 다음과 같이 입력합니다:

→ *이 프롬프트는 CLAUDE.md 없이 최소한의 지시만으로 대시보드를 생성하기 위한 것입니다:*

```
/dashboard에 대시보드 페이지를 만들어줘:
- 최근 학습 세션 목록
- 총 학습 시간
- 새 세션 생성 버튼
db/schema.ts의 데이터베이스 스키마를 사용해줘.
```

Claude Code가 코드를 생성하는 과정을 지켜봅니다. 파일 생성과 수정에 대한 권한 요청이 오면 허용합니다.

### Step 2: 결과 관찰 - 어떤 차이가 있나?

Claude가 생성한 코드를 살펴봅니다. 다음과 같은 차이점들을 확인해보세요:

#### 차이점 1: 프로젝트 컨벤션을 모름

```typescript
// Claude가 만들 수 있는 문제 코드 예시

// 클라이언트 컴포넌트에서 useEffect로 데이터 패칭 (우리는 서버 컴포넌트를 원함)
"use client";
import { useEffect, useState } from "react";

export default function Dashboard() {
  const [sessions, setSessions] = useState([]);

  useEffect(() => {
    fetch("/api/sessions").then(res => res.json()).then(setSessions);
  }, []);
  // ...
}
```

- 서버 컴포넌트 대신 클라이언트 컴포넌트를 사용할 수 있음
- `useEffect`로 데이터를 패칭할 수 있음 (Next.js App Router에서는 비효율적)
- API 라우트를 불필요하게 만들 수 있음

#### 차이점 2: UI 일관성 없음

```typescript
// shadcn/ui 대신 직접 스타일링하거나 다른 패턴 사용
<div className="bg-white rounded-lg shadow p-4">
  <h2 className="text-xl font-bold">Total Study Time</h2>
  <p className="text-3xl">{totalTime}</p>
</div>
```

- shadcn/ui 컴포넌트(`Card`, `Table` 등)를 사용하지 않을 수 있음
- 프로젝트의 디자인 시스템과 맞지 않는 스타일링
- 하드코딩된 색상값

#### 차이점 3: 인증 패턴을 모름

```typescript
// Clerk auth() 대신 다른 방식을 사용할 수 있음
const userId = cookies().get("userId"); // Clerk를 모름
```

- Clerk의 `auth()` 함수를 사용하지 않을 수 있음
- 사용자 인증 체크가 누락될 수 있음

#### 차이점 4: 데이터베이스 쿼리 패턴 불일치

```typescript
// Drizzle ORM 대신 raw SQL이나 다른 패턴 사용
import { sql } from "drizzle-orm";
const sessions = await db.execute(sql`SELECT * FROM study_sessions`);
```

- Drizzle의 query builder를 제대로 활용하지 않을 수 있음
- userId 필터링이 빠질 수 있음

### Step 3: 핵심 포인트

{% hint style="info" %}
**Claude Code는 컨텍스트가 중요합니다.**

Claude Code는 매우 유능한 AI 코딩 어시스턴트이지만, 프로젝트의 컨벤션과 규칙을 알려주지 않으면 **일반적인 패턴을 추측**하여 코드를 생성합니다. 이는 자연스러운 동작이며, 다음 섹션에서 CLAUDE.md 문서를 작성하면 쉽게 개선할 수 있습니다:

- 프로젝트 내에서 **일관성 없는 코드 패턴** → CLAUDE.md로 해결
- 의도하지 않은 **아키텍처 결정** → 가이드 문서로 해결
- 나중에 **수정해야 할 기술 부채** → 사전 문서화로 예방
{% endhint %}

공식 문서([code.claude.com/docs](https://code.claude.com/docs))에서도 강조하듯이, **CLAUDE.md는 Claude Code가 매 세션 시작 시 읽는 마크다운 파일**로, 코딩 표준, 아키텍처 결정, 선호하는 라이브러리 등을 설정하는 데 사용합니다.

| 문서 없이 | 문서와 함께 |
|-----------|------------|
| 일반적인 React 패턴 추측 | 프로젝트 규칙에 맞는 코드 생성 |
| 일관성 없는 UI | shadcn/ui 컴포넌트 일관 사용 |
| 다양한 데이터 패칭 방식 | 서버 컴포넌트 + Drizzle 패턴 통일 |
| 인증 방식 불명확 | Clerk auth() 자동 사용 |

### Step 4: `/rewind`로 되돌리기

이 실습에서 만든 코드는 문서 기반으로 다시 생성할 것이므로, 되돌립니다.

Claude Code 터미널에서 다음과 같이 입력합니다:

→ *이 프롬프트는 방금 생성한 코드를 되돌려서 깨끗한 상태에서 다시 시작하기 위한 것입니다:*

```
/rewind
```

{% hint style="info" %}
**`/rewind` 명령어**는 대화 내에서 이전 시점으로 되돌아갈 수 있게 해주는 Claude Code의 빌트인 명령어입니다. 코드 변경 사항을 취소하고 해당 시점의 대화로 복원합니다. 실험적인 시도를 안전하게 할 수 있어 매우 유용합니다.
{% endhint %}

되돌아갈 시점을 선택하라는 프롬프트가 나타나면, 대시보드 생성 요청 **이전** 시점을 선택합니다.

## 다음 단계

문서 없이 코드를 생성했을 때의 차이점을 확인했습니다. 다음 섹션에서는 CLAUDE.md와 가이드 문서를 작성하여 **일관되고 품질 높은 코드**를 생성하는 방법을 배웁니다.

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] 문서 없이 생성된 코드의 차이점을 관찰함
- [ ] `/rewind`로 변경사항을 되돌리기 완료

---

> **다음**: [2.2 문서 기반 코드 생성](02-documentation-driven.md)
