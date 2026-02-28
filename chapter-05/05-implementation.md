# 5.5 구현 단계

> 승인된 계획을 기반으로 일괄 구현을 수행하고, 결과를 테스트하며, 반복적 피드백으로 완성도를 높입니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 일괄 구현 프롬프트, plan.md 연동, `/rewind`, 간결한 피드백, `/commit`

> ⭐⭐⭐ **난이도**: 어려움

## 학습 목표

- 승인된 계획을 기반으로 "모두 구현해줘" 프롬프트를 사용하는 방법을 배웁니다
- Claude가 plan.md에 완료 표시를 하며 진행하는 과정을 관찰합니다
- 간결한 피드백으로 수정을 요청하는 기법을 익힙니다
- 잘못된 방향일 때 `/rewind`로 되돌리고 재시작하는 판단 기준을 배웁니다

## 드디어 코드 작성!

리서치 → 계획 → 주석 사이클을 거쳐 충분히 검토된 계획이 준비되었습니다. 이제 이 계획을 기반으로 코드를 작성합니다.

{% hint style="info" %}
여기까지 코드를 한 줄도 작성하지 않았지만, 이 시점에서 구현의 80%는 이미 결정된 것이나 다름없습니다. 어떤 파일을 만들고, 어떤 패턴을 따르고, 어떤 쿼리를 사용할지 모두 plan.md에 정의되어 있기 때문입니다.
{% endhint %}

## 실습: /stats 대시보드 구현

### Step 1: 일괄 구현 프롬프트

→ *이 프롬프트는 승인된 계획의 모든 작업을 한번에 구현하고, 진행 상황을 plan.md에 기록하며 진행하도록 하기 위한 것입니다:*

```
plan.md의 구현 작업 목록을 모두 구현해줘. 작업이나 단계를 완료하면 plan.md에 완료 표시(✅)해줘. 모든 작업과 단계가 완료될 때까지 멈추지 마.
```

이 프롬프트의 핵심 요소:

| 요소 | 역할 |
|------|------|
| "plan.md의 구현 작업 목록" | 어떤 계획을 따를지 명확히 지정 |
| "완료 표시(✅)해줘" | 진행 상황을 추적할 수 있게 함 |
| "멈추지 마" | 중간에 확인을 구하지 않고 일괄 진행 |

### Step 2: 실행 중 관찰

Claude가 코드를 생성하는 과정을 관찰합니다. 다음과 같은 순서로 진행될 것입니다:

```
Phase 1: 쿼리 함수
├── lib/queries/stats.ts 생성
├── getCompletionStats() 구현 → plan.md에 ✅ 표시
├── getCategoryDistribution() 구현 → plan.md에 ✅ 표시
├── getPriorityDistribution() 구현 → plan.md에 ✅ 표시
├── getWeeklyTrend() 구현 → plan.md에 ✅ 표시
└── getProductivityScore() 구현 → plan.md에 ✅ 표시

Phase 2: UI 컴포넌트
├── components/stats/completion-cards.tsx 생성 → ✅
├── components/stats/category-chart.tsx 생성 → ✅
├── components/stats/priority-chart.tsx 생성 → ✅
├── components/stats/weekly-trend.tsx 생성 → ✅
└── components/stats/productivity-score.tsx 생성 → ✅

Phase 3: 페이지 조립
├── app/stats/page.tsx 생성 → ✅
└── 네비게이션에 /stats 링크 추가 → ✅
```

{% hint style="tip" %}
Claude가 진행하는 동안 plan.md를 에디터에서 열어두면 체크박스가 실시간으로 채워지는 것을 관찰할 수 있습니다.
{% endhint %}

### Step 3: 결과 테스트

구현이 완료되면 개발 서버를 실행하여 결과를 확인합니다.

→ *이 프롬프트는 구현된 대시보드가 정상적으로 동작하는지 빌드와 브라우저에서 검증하기 위한 것입니다:*

```
pnpm build를 실행해줘. TypeScript 에러가 있으면 수정해줘.
```

빌드가 성공하면:

```bash
pnpm dev
```

브라우저에서 `http://localhost:3000/stats`로 접속하여 대시보드를 확인합니다.

### Step 4: 반복적 수정 — 간결한 피드백

대시보드를 확인하면서 수정이 필요한 부분을 발견하면, **간결한 피드백**으로 수정을 요청합니다.

Boris Tane 워크플로에서 수정 피드백은 짧고 명확해야 합니다:

#### 레이아웃 수정

→ *이 프롬프트는 통계 카드의 너비가 부족한 UI 문제를 간결하게 수정하기 위한 것입니다:*

```
통계 카드가 너무 좁아. 더 넓게 해줘.
```

#### 스타일 수정

→ *이 프롬프트는 차트 색상이 너무 연한 시각적 문제를 간결하게 수정하기 위한 것입니다:*

```
바 차트 색상이 너무 연해. 더 진하게 해줘.
```

#### 패턴 일관성

→ *이 프롬프트는 기존 UI 패턴과 동일한 스타일을 적용하도록 기존 구현을 참조하기 위한 것입니다:*

```
통계 카드는 할 일 목록 페이지의 카드와 정확히 같은 스타일이어야 해.
```

#### 데이터 표시 수정

→ *이 프롬프트는 차트에서 텍스트가 잘리는 문제를 간결하게 수정하기 위한 것입니다:*

```
카테고리 이름이 잘려. 말줄임이 아니라 전체 이름이 보여야 해.
```

{% hint style="tip" %}
**간결한 피드백의 원칙**: Claude는 전체 컨텍스트를 가지고 있으므로, "통계 카드가 너무 좁아. 더 넓게 해줘."만으로 충분합니다. 파일 경로, 클래스 이름 등을 일일이 지정할 필요가 없습니다.
{% endhint %}

### Step 5: 잘못된 방향일 때 — revert 후 재시작

때로는 수정으로 해결할 수 없는 근본적인 문제가 발견됩니다. 이런 경우:

#### `/rewind` 사용

```
/rewind
```

`/rewind`는 마지막 변경사항을 되돌립니다. 여러 파일의 변경을 한 번에 되돌릴 수 있습니다.

#### 패치보다 처음부터 다시 하는 것이 나은 경우

| 상황 | 접근 |
|------|------|
| 사소한 UI 수정 | 간결한 피드백으로 수정 |
| 쿼리 로직 오류 | 해당 함수만 수정 요청 |
| 전체 구조가 잘못됨 | `/rewind`로 되돌리고 plan.md 수정 후 재구현 |
| 2번 수정해도 해결 안 됨 | `/rewind`로 되돌리고 재시작 |

{% hint style="warning" %}
**Boris Tane의 규칙**: 같은 문제로 **2번 수정을 요청해도 해결되지 않으면**, 패치를 계속하지 말고 `/rewind`로 되돌린 뒤 처음부터 다시 접근하세요. 작은 패치를 쌓아가면 코드가 점점 더 복잡해지고, Claude의 컨텍스트도 혼란스러워집니다.
{% endhint %}

### Step 6: 자체 검증

모든 수정이 완료되면 6가지 기준으로 자체 검증을 수행합니다.

→ *이 프롬프트는 구현된 대시보드가 6가지 품질 기준을 모두 충족하는지 체계적으로 검증하기 위한 것입니다:*

```
/stats 대시보드를 다음 6가지 기준으로 검증해줘:

1. 빌드 통과: pnpm build가 에러 없이 완료되는가?
2. 데이터 정확성: 통계 숫자가 실제 할 일 데이터와 일치하는가?
3. 빈 상태 처리: 할 일이 없을 때 적절한 메시지를 보여주는가?
4. 패턴 일관성: 기존 코드(쿼리, 컴포넌트, 스타일)와 일관되는가?
5. 반응형 레이아웃: 모바일/데스크톱에서 적절하게 표시되는가?
6. 타입 안전성: TypeScript 에러가 없는가?

각 기준에 대해 통과/실패 여부와 수정 필요 사항을 보고해줘.
```

### Step 7: 커밋

모든 검증이 통과하면 `/commit` 스킬로 변경 사항을 커밋합니다.

→ *이 프롬프트는 Chapter 2에서 만든 커밋 스킬로 모든 변경사항을 커밋하기 위한 것입니다:*

```
/commit
```

{% hint style="info" %}
**Skills 복습**: Chapter 2.6에서 만든 `/commit` 스킬은 변경된 파일을 분석하고, 적절한 커밋 메시지를 생성하여 커밋합니다.
{% endhint %}

## 구현 단계에서 생성되는 파일 구조

최종적으로 다음과 같은 파일들이 생성/수정됩니다:

```
lib/queries/
└── stats.ts                    # 통계 쿼리 함수 모음

components/stats/
├── completion-cards.tsx        # 완료율 통계 카드
├── category-chart.tsx          # 카테고리별 바 차트
├── priority-chart.tsx          # 우선순위별 분포 (도넛 차트)
├── weekly-trend.tsx            # 7일 완료 추이 바 차트
└── productivity-score.tsx      # 생산성 점수 표시

app/stats/
└── page.tsx                    # 대시보드 페이지 (서버 컴포넌트)

components/
└── nav.tsx (수정)              # /stats 링크 추가
```

### 핵심 코드 예시: 서버 컴포넌트

```typescript
// app/stats/page.tsx
import {
  getCompletionStats,
  getCategoryDistribution,
  getPriorityDistribution,
  getWeeklyTrend,
  getProductivityScore,
} from "@/lib/queries/stats";
import { CompletionCards } from "@/components/stats/completion-cards";
import { CategoryChart } from "@/components/stats/category-chart";
import { PriorityChart } from "@/components/stats/priority-chart";
import { WeeklyTrend } from "@/components/stats/weekly-trend";
import { ProductivityScore } from "@/components/stats/productivity-score";

export default async function StatsPage() {
  const [completion, categories, priorities, weekly, productivity] =
    await Promise.all([
      getCompletionStats(),
      getCategoryDistribution(),
      getPriorityDistribution(),
      getWeeklyTrend(),
      getProductivityScore(),
    ]);

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">통계 대시보드</h1>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <CompletionCards stats={completion} />
        <ProductivityScore data={productivity} />
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <CategoryChart data={categories} />
        <PriorityChart data={priorities} />
      </div>

      <WeeklyTrend data={weekly} />
    </div>
  );
}
```

## 구현 단계 팁

### 1. "멈추지 마"의 중요성

"멈추지 마" 없이 구현을 요청하면, Claude가 Phase 1을 완료한 후 "Phase 2를 진행할까요?"라고 확인을 구합니다. 일괄 구현에서는 확인 없이 끝까지 진행하는 것이 효율적입니다.

### 2. 검증 기준을 미리 정의

구현 후 "잘 동작해?"가 아니라, **구체적인 검증 기준**을 제시하세요. Step 6의 6가지 기준처럼 명확한 기준이 있으면 Claude가 체계적으로 검증합니다.

### 3. 긴 세션에서는 `/compact` 활용

구현 단계에서 대화가 길어지면 컨텍스트가 커집니다. 중간에 `/compact`를 사용하면 이전 대화를 요약하여 컨텍스트를 절약할 수 있습니다.

```
/compact
```

{% hint style="tip" %}
**`/clear` vs `/compact`**: `/clear`는 컨텍스트를 완전히 초기화합니다. 긴 구현 세션 중에는 `/compact`를 사용하여 이전 맥락을 유지하면서 컨텍스트를 줄이세요. plan.md 파일은 `/clear` 후에도 디스크에 남아 있으므로 참조할 수 있습니다.
{% endhint %}

## 핵심 정리

| 개념 | 설명 |
|------|------|
| 일괄 구현 | "모두 구현해줘. 멈추지 마." 프롬프트 |
| plan.md 완료 표시 | Claude가 작업 완료 시 체크박스에 ✅ 표시 |
| 간결한 피드백 | "더 넓게", "색상 더 진하게" 등 짧고 명확한 수정 요청 |
| 기존 패턴 참조 | "할 일 목록 테이블과 같은 스타일로" |
| `/rewind` | 잘못된 방향일 때 되돌리기 |
| 2번 규칙 | 같은 문제로 2번 실패하면 되돌리고 재시작 |
| 자체 검증 | 6가지 기준으로 체계적 검증 |
| `/compact` | 긴 세션에서 컨텍스트 관리 |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] plan.md 기반으로 일괄 구현을 수행함
- [ ] plan.md의 작업 목록에 완료 표시(✅)가 모두 채워짐
- [ ] `pnpm build`가 에러 없이 통과함
- [ ] 브라우저에서 `/stats` 페이지가 정상 표시됨
- [ ] 간결한 피드백으로 UI 수정을 수행함
- [ ] 6가지 검증 기준을 모두 통과함
- [ ] `/commit`으로 변경 사항을 커밋함

---

> **다음**: [5.6 리뷰 & 프로 팁 종합](06-review-and-tips.md)에서 전체 워크플로를 복습하고, Boris Tane의 프로 팁 10선을 정리합니다.
