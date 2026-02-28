# 4.2 학습 블록 기록 기능

> 세션 상세 페이지에서 과목별 학습 블록을 기록하고, 실시간 합산 및 자체 검증까지 수행합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 복합 기능 구현, 자체 검증

> ⭐⭐⭐ **난이도**: 어려움

## 개요

이 섹션에서는 학습 세션의 상세 페이지(`/dashboard/sessions/[id]`)를 구현합니다. 사용자는 세션에 연결된 과목별로 학습 블록을 추가/수정/삭제할 수 있으며, 전체 학습 시간이 실시간으로 합산됩니다.

### DB 스키마 복습

```
session_subjects: id, sessionId, subjectId
study_blocks: id, sessionSubjectId, durationMinutes, pagesRead, notes, createdAt
```

- 하나의 세션에 여러 과목이 연결됨 (`session_subjects`)
- 각 세션-과목 조합에 여러 학습 블록이 연결됨 (`study_blocks`)
- 학습 블록에는 시간(분), 읽은 페이지 수(선택), 메모(선택)가 포함

## Step 1: 학습 블록 상세 기록 기능 구현

복합적인 기능을 요청할 때는 Claude에게 먼저 생각할 시간을 주는 것이 좋습니다. `Think about this carefully`를 프롬프트에 포함하면 Claude가 extended thinking 모드를 활성화하여 더 체계적으로 접근합니다.

### Claude에게 요청

→ *이 프롬프트는 학습 블록 CRUD 기능을 종합적으로 구현하기 위한 것입니다 (Extended Thinking 활성화):*

```
이것에 대해 깊이 생각해줘. 상세 학습 블록 기록 기능을 구현해줘:

학습 세션을 볼 때, 사용자가 다음을 할 수 있어야 해:
1. 세션에 연결된 모든 과목 확인
2. 각 과목에 대해 학습 블록 추가/수정/삭제
3. 각 학습 블록: durationMinutes, pagesRead (선택), notes (선택)
4. 과목별, 전체 세션의 누적 합계 표시
5. startTime + 총 시간 기반으로 세션 endTime 자동 계산

/dashboard/sessions/[id]에 인라인 편집이 가능한 UI를 만들어줘.
변이에는 server actions를 사용해줘. docs/data-mutations.md 패턴을 따라줘.
```

### Claude가 생성할 것들

이 요청으로 Claude는 대략 다음과 같은 파일들을 생성합니다:

| 파일 | 역할 |
|------|------|
| `app/dashboard/sessions/[id]/page.tsx` | 세션 상세 서버 컴포넌트 |
| `components/study-blocks/block-list.tsx` | 학습 블록 목록 (클라이언트 컴포넌트) |
| `components/study-blocks/block-form.tsx` | 블록 추가/수정 인라인 폼 |
| `actions/study-blocks.ts` | 학습 블록 CRUD 서버 액션 |
| `lib/queries/study-blocks.ts` | 블록 데이터 조회 함수 |

## Step 2: 서버 컴포넌트 + 클라이언트 인터랙션

학습 블록 기능은 서버 컴포넌트와 클라이언트 컴포넌트를 적절히 조합해야 합니다.

### 서버 컴포넌트가 담당하는 부분

- 세션 데이터 조회 (Drizzle ORM 쿼리)
- 과목 목록 + 학습 블록 데이터 페칭
- 인증 확인 (`auth()`)

### 클라이언트 컴포넌트가 담당하는 부분

- 인라인 편집 UI (`"use client"`)
- 실시간 합산 로직 (블록 추가/수정 시 즉시 반영)
- 낙관적 업데이트 (서버 응답 전에 UI 반영)

### 실시간 합산 로직

과목별 합산과 전체 합산이 실시간으로 업데이트되어야 합니다:

```typescript
// 예시: 합산 로직
const subjectTotal = blocks
  .filter(b => b.sessionSubjectId === subjectId)
  .reduce((sum, b) => sum + b.durationMinutes, 0);

const sessionTotal = blocks
  .reduce((sum, b) => sum + b.durationMinutes, 0);

// endTime 자동 계산
const endTime = addMinutes(session.startTime, sessionTotal);
```

Claude가 서버 컴포넌트와 클라이언트 컴포넌트 분리를 잘못할 경우, 다음과 같이 수정 요청합니다:

→ *이 프롬프트는 서버/클라이언트 컴포넌트 분리가 잘못되었을 때 수정을 요청하기 위한 것입니다:*

```
block-list.tsx는 인라인 편집 상태를 관리하기 때문에 "use client"가 필요해.
하지만 세션 데이터 페칭은 서버 컴포넌트(page.tsx)에 남겨야 해.
초기 데이터를 클라이언트 컴포넌트에 props로 전달해줘.
```

## Step 3: 테스트 및 검증

기능 구현 후 Claude에게 직접 테스트를 요청합니다.

### Claude에게 검증 요청

→ *이 프롬프트는 구현된 기능을 Claude에게 직접 테스트하도록 요청하기 위한 것입니다:*

```
학습 블록 기록 기능을 테스트해줘: 세션을 생성하고, 3개 과목을 추가하고, 각각에 학습 블록을
추가해줘. 합계가 정확한지, 세션 종료 시간이 업데이트되는지 확인해줘.
```

Claude는 다음과 같은 과정을 수행합니다:

1. 개발 서버 실행 (또는 이미 실행 중인지 확인)
2. 데이터베이스에 테스트 데이터 삽입
3. API 또는 서버 액션을 통한 기능 테스트
4. 결과 검증

## Step 4: Claude에게 자체 검증 시키기

좋은 프롬프트 습관 중 하나는 **검증 기준을 함께 제공하는 것**입니다. Claude가 스스로 결과를 확인할 수 있도록 기준을 명시합니다.

### 검증 기준과 함께 요청

→ *이 프롬프트는 검증 기준을 명시하여 Claude가 체계적으로 자체 테스트하도록 하기 위한 것입니다:*

```
학습 블록 기능이 올바르게 작동하는지 검증해줘:

테스트 기준:
1. 학습 블록을 추가하면 해당 과목의 총 시간이 즉시 업데이트되어야 함
2. 여러 과목에 블록을 추가하면 세션 전체 합계가 업데이트되어야 함
3. 세션 endTime은 startTime + 모든 블록의 총 분과 같아야 함
4. 블록을 삭제하면 합계가 감소해야 함
5. pagesRead와 notes가 비어 있어도 허용되어야 함 (선택 항목)
6. 세션 소유자(일치하는 userId)만 블록을 보고 편집할 수 있어야 함

개발 서버를 실행하고 각 기준을 검증해줘. 각 항목의 통과/실패를 보고해줘.
```

### 자체 검증의 장점

- Claude가 빌드 에러를 직접 발견하고 수정
- 런타임 에러를 서버 로그에서 확인
- 데이터 무결성 문제를 미리 감지
- 인증 관련 버그 사전 발견

{% hint style="info" %}
**Best Practice**: 복잡한 기능을 구현할 때는 항상 검증 기준을 함께 제공하세요. Claude가 "완료"라고 말해도 실제로는 에지 케이스가 남아있을 수 있습니다. 검증 기준이 있으면 Claude가 스스로 확인합니다.
{% endhint %}

## Step 5: 추가 개선

기본 기능이 완성되면, 추가적인 개선을 요청할 수 있습니다:

→ *이 프롬프트는 기본 기능 완성 후 추가적인 UX 개선을 요청하기 위한 것입니다:*

```
학습 블록 기능에 다음 개선사항을 추가해줘:
1. 과목 내에서 블록 순서를 드래그 앤 드롭으로 변경
2. 한 과목에서 다른 과목으로 블록 복사
3. 빠른 추가 템플릿 (예: "30분 독서", "1시간 문제 풀기")
```

{% hint style="warning" %}
한 번에 너무 많은 기능을 요청하면 품질이 떨어질 수 있습니다. 핵심 기능을 먼저 완성하고, 부가 기능은 별도의 요청으로 나누세요.
{% endhint %}

## 정리

이 섹션에서 배운 내용:

- **`Think about this carefully`** 프롬프트로 Claude의 extended thinking 활성화
- **서버/클라이언트 컴포넌트 분리**: 데이터 페칭은 서버, 인터랙션은 클라이언트
- **검증 기준 제공**: Claude가 스스로 테스트하고 결과를 보고하도록 유도
- **단계적 구현**: 핵심 기능 먼저, 부가 기능은 별도 요청
- **자체 검증 패턴**: "Run the dev server and verify" 요청으로 Claude에게 검증 위임

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] 학습 블록 CRUD 동작 확인 (추가/수정/삭제)
- [ ] 과목별, 전체 세션 합산이 실시간으로 업데이트됨
- [ ] 세션 endTime이 자동 계산됨
- [ ] 서버 컴포넌트와 클라이언트 컴포넌트가 올바르게 분리됨

---

다음: [4.3 사용자 범위 슬래시 명령](03-custom-commands-advanced.md)
