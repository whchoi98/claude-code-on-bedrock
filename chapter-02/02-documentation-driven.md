# 2.2 문서 기반 코드 생성 (CLAUDE.md & 가이드 문서)

> `/init`으로 CLAUDE.md를 생성하고, 프로젝트 가이드 문서를 작성하여 Claude Code가 일관된 코드를 생성하도록 만듭니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: `/init`, CLAUDE.md, 문서 기반 코드 생성

> ⭐⭐ **난이도**: 보통

## 학습 목표

- `/init` 명령어로 CLAUDE.md를 생성하는 방법을 배웁니다
- 프로젝트 규칙을 문서화하여 Claude Code의 코드 품질을 높이는 방법을 이해합니다
- 문서 유무에 따른 코드 생성 품질 차이를 직접 비교합니다

## CLAUDE.md란?

[CLAUDE.md](https://code.claude.com/docs/en/memory)는 Claude Code가 **매 세션 시작 시 자동으로 읽는** 마크다운 파일입니다. 프로젝트 루트에 위치하며, 다음과 같은 역할을 합니다:

> 💡 **비유**: CLAUDE.md는 "새로운 팀원에게 프로젝트 규칙을 알려주는 온보딩 문서"와 같습니다. Claude Code는 매번 새로운 세션이 시작될 때 이 문서를 읽고, 여러분의 프로젝트 규칙을 따르게 됩니다.

CLAUDE.md의 주요 역할:

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

→ *이 프롬프트는 프로젝트를 분석하여 CLAUDE.md 초안을 자동 생성하기 위한 것입니다:*

```
/init
```

{% hint style="info" %}
**`/init`** 명령어는 현재 프로젝트를 분석하여 CLAUDE.md 파일을 자동으로 생성합니다. 프로젝트 구조, package.json, 설정 파일 등을 읽고 초기 규칙을 만들어줍니다.
{% endhint %}

Claude Code가 프로젝트를 분석하고 CLAUDE.md 초안을 생성합니다. 생성된 파일을 확인해봅니다.

### Step 2: CLAUDE.md에 프로젝트 규칙 추가

`/init`으로 생성된 CLAUDE.md의 내용을 검토한 후, 우리 프로젝트에 맞게 다음 내용이 포함되도록 수정을 요청합니다:

→ *이 프롬프트는 자동 생성된 CLAUDE.md에 우리 프로젝트의 구체적인 규칙을 추가하기 위한 것입니다:*

```
CLAUDE.md에 다음 프로젝트별 규칙을 추가해줘:

# Todo App - CLAUDE.md

## 빌드 & 개발 명령어
- 개발 서버: pnpm dev
- 빌드: pnpm build
- 린트: pnpm lint

## 코드 스타일
- 서버 컴포넌트를 기본으로 사용, "use client"는 필요한 경우에만
- 모든 데이터베이스 쿼리에 Drizzle ORM 사용
- 
- UI에 shadcn/ui 컴포넌트 사용
- 파일 이름: kebab-case, 컴포넌트: PascalCase

## 프로젝트 구조
- app/: Next.js App Router 페이지
- components/: 재사용 가능한 UI 컴포넌트
- db/: 데이터베이스 스키마 및 연결
- lib/: 유틸리티 함수 및 쿼리 함수
- actions/: Server Actions

## 데이터 패칭 규칙
- 데이터 패칭에 서버 컴포넌트 사용 (초기 데이터에 useEffect 사용 금지)
- lib/queries/ 디렉토리에 쿼리 함수 생성
- 

## 데이터 변이 규칙
- 모든 데이터 변이에 Next.js Server Actions 사용
- Server Actions는 actions/ 디렉토리에 배치
- 입력값은 항상 Zod로 유효성 검사
- 변이 후 revalidatePath() 호출
```

{% hint style="tip" %}
CLAUDE.md는 구체적일수록 좋습니다. "코드를 잘 작성하세요"보다 "서버 컴포넌트를 기본으로 사용하고, 클라이언트 상태가 필요한 경우에만 'use client'를 추가하세요"가 훨씬 효과적입니다.
{% endhint %}

### Step 3: docs/ui.md 가이드 문서 작성

프로젝트에 `docs/` 디렉토리를 만들고 UI 가이드 문서를 작성합니다:

→ *이 프롬프트는 UI 관련 규칙을 별도 문서로 분리하여 Claude Code가 일관된 UI를 생성하도록 하기 위한 것입니다:*

```
UI 컨벤션을 문서화하는 docs/ui.md 파일을 생성해줘:

# UI 컨벤션

## 컴포넌트 라이브러리
- shadcn/ui를 기본 컴포넌트 라이브러리로 사용
- @/components/ui/에서 컴포넌트 임포트

## 레이아웃
- 대시보드에 사이드바 네비게이션 레이아웃 사용
- 메인 콘텐츠 영역에 max-width 컨테이너 적용
- 모바일 우선 반응형 디자인

## 데이터 표시
- 통계 및 요약 데이터에 shadcn/ui Card 컴포넌트 사용
- 목록 뷰에 shadcn/ui Table 컴포넌트 사용
- 상태 표시와 과목 태그에 shadcn/ui Badge 사용

## 색상 체계
- 기본 색상: blue (shadcn 기본 테마)
- 과목 badge에는 과목 자체의 color 속성 사용
- 보조 정보에는 muted 변형 사용

## 폼
- shadcn/ui 폼 컴포넌트 사용 (Input, Select, Textarea)
- 날짜 선택에 shadcn/ui Calendar 사용
- 모달 폼에 shadcn/ui Dialog 사용
- 유효성 검사 에러는 각 필드 아래 인라인으로 표시

## 아이콘
- 모든 아이콘에 lucide-react 사용
- 유사한 동작에 일관된 아이콘 사용
```

### Step 4: 문서를 참조한 대시보드 재생성

이제 CLAUDE.md와 docs/ui.md가 있는 상태에서 대시보드를 다시 생성합니다:

→ *이 프롬프트는 문서를 먼저 읽게 한 뒤, 문서 규칙에 따라 대시보드를 생성하기 위한 것입니다:*

```
먼저 CLAUDE.md와 docs/ui.md를 읽어줘. 그다음 우리 컨벤션에 따라 /todos에 대시보드 페이지를 만들어줘. 포함할 내용:
- 네비게이션 링크가 있는 사이드바 레이아웃
- 총 학습 시간, 세션 수, 과목 수를 보여주는 통계 카드
- Table 컴포넌트를 사용한 최근 세션 목록
- "새 세션" 버튼
서버 컴포넌트에서 Drizzle ORM으로 데이터를 패칭해줘.
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
// 서버 컴포넌트, Drizzle ORM, shadcn/ui

import { db } from "@/db";
import { todos } from "@/db/schema";
import { eq } from "drizzle-orm";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default async function DashboardPage() {
  
  

  const sessions = await db
    .select()
    .from(todos)
    .orderBy(desc(todos.date));

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

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] CLAUDE.md 생성됨
- [ ] docs/ui.md 생성됨
- [ ] 문서 기반 대시보드 생성됨

---

> **다음**: [2.3 서버 컴포넌트 데이터 패칭](03-data-fetching.md)에서 데이터 패칭 가이드 문서를 작성하고 학습 세션 목록 페이지를 구현합니다.
