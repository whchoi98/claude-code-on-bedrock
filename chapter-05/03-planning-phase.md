# 5.3 계획 수립 단계

> research.md를 기반으로 상세한 구현 계획을 plan.md 파일에 작성하고, 기존 구현을 명시적으로 참조합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: Plan 모드 (`Shift+Tab`), 파일 기반 계획 (plan.md), 기존 구현 참조 지시

> ⭐⭐⭐ **난이도**: 어려움

## 학습 목표

- Plan 모드에서 구현 방향을 토론하는 방법을 배웁니다
- Claude에게 plan.md 파일로 상세 계획을 작성시키는 방법을 익힙니다
- 기존 구현을 참조하도록 지시하여 일관성을 확보하는 기법을 배웁니다

## Plan 모드 vs 파일 기반 계획

Claude Code에는 빌트인 Plan 모드(`Shift+Tab`)가 있습니다. 하지만 Boris Tane 워크플로에서는 **빌트인 Plan 모드와 파일 기반 계획을 함께 사용**합니다.

| 방식 | 장점 | 단점 |
|------|------|------|
| **빌트인 Plan 모드** | 코드를 수정하지 않음, 빠른 토론 | 계획이 대화에 남아 지나가면 찾기 어려움 |
| **파일 기반 계획** (plan.md) | 에디터에서 검토 가능, 주석 추가 가능 | 파일 관리 필요 |
| **둘 다 사용** (권장) | Plan 모드로 토론 → plan.md로 정리 | - |

{% hint style="tip" %}
**권장 접근법**: Plan 모드(`Shift+Tab`)에서 먼저 구현 방향을 토론한 뒤, 합의된 방향을 plan.md 파일로 정리합니다. 이렇게 하면 빠른 토론의 장점과 검토 가능한 문서의 장점을 모두 얻을 수 있습니다.
{% endhint %}

## 실습: /stats 대시보드 계획 수립

### Step 1: Plan 모드에서 구현 방향 토론

먼저 `Shift+Tab`으로 Plan 모드를 활성화하고, 대시보드의 구현 방향을 토론합니다.

→ *이 프롬프트는 Plan 모드에서 대시보드 구현의 큰 방향을 먼저 잡기 위한 것입니다:*

```
research.md를 읽고, /stats 대시보드 페이지의 구현 방향에 대해 토론하자.

다음 질문에 대해 의견을 줘:
1. 차트를 외부 라이브러리(recharts 등)로 만들까, Tailwind CSS만으로 만들까?
2. 통계 쿼리를 하나의 함수로 합칠까, 각각 분리할까?
3. 클라이언트 컴포넌트가 필요한 부분은 어디인가?
4. 페이지 로딩 전략은? (하나의 await vs Suspense 각각)
```

{% hint style="info" %}
**Plan 모드 복습**: Chapter 1.4에서 배운 Plan 모드는 `Shift+Tab`으로 활성화합니다. 이 모드에서 Claude는 코드를 수정하지 않고 계획만 수립합니다. 다시 `Shift+Tab`을 누르면 일반 모드로 돌아갑니다.
{% endhint %}

Plan 모드에서의 토론 예시:

```
Claude (Plan 모드):
1. 차트 라이브러리: 이 프로젝트는 아직 차트 라이브러리가 없습니다.
   - 옵션 A: Tailwind CSS만 사용 (바 차트, 진행 바 등 간단한 시각화)
   - 옵션 B: recharts 추가 (파이 차트, 라인 차트 등 복잡한 시각화)
   - 추천: 옵션 A로 시작하고 필요 시 라이브러리 추가

2. 쿼리 함수:
   - 분리를 추천합니다. 각 통계가 독립적이므로:
     - getCompletionStats(): 완료율 통계
     - getCategoryDistribution(): 카테고리별 분포
     - getPriorityDistribution(): 우선순위별 분포
     - getWeeklyTrend(): 7일 완료 추이
     - getProductivityScore(): 생산성 점수

3. 클라이언트 컴포넌트:
   - 바 차트, 파이 차트 등 인터랙티브 UI만 클라이언트
   - 통계 카드는 서버 컴포넌트로 충분

4. 로딩 전략:
   - 각 섹션을 Suspense로 감싸면 점진적 로딩 가능
   - 하지만 이 프로젝트는 단순한 await Promise.all 패턴 사용 중
   - 기존 패턴과 일관되게 Promise.all 추천
```

### Step 2: plan.md 작성 요청

Plan 모드에서 방향이 합의되면, 일반 모드로 돌아와서 plan.md 작성을 요청합니다.

→ *이 프롬프트는 합의된 구현 방향을 상세한 계획 문서로 정리하여, 다음 단계(주석 사이클)에서 검토할 수 있게 하기 위한 것입니다:*

```
상세한 구현 계획을 plan.md에 작성해줘. research.md를 참고해서 다음을 포함해줘:

1. ## 접근 방식
   - 전체 구현 방향 요약
   - 차트를 Tailwind CSS만으로 구현하는 접근

2. ## 생성/수정할 파일 목록
   - 각 파일의 역할 설명
   - 파일 간 의존 관계

3. ## 쿼리 함수 설계
   - 각 쿼리 함수의 시그니처와 반환 타입
   - SQL 쿼리의 개요 (GROUP BY, COUNT 등)
   - lib/queries/stats.ts에 모든 통계 쿼리 배치

4. ## UI 컴포넌트 구조
   - 페이지 레이아웃 (그리드 배치)
   - 각 컴포넌트의 props 인터페이스
   - 서버/클라이언트 분리 결정 근거

5. ## 코드 스니펫
   - 핵심 쿼리 함수의 예시 코드
   - 주요 컴포넌트의 JSX 구조 예시
   - app/stats/page.tsx의 전체 구조

6. ## 트레이드오프 & 대안
   - 선택한 접근의 장단점
   - 고려했지만 선택하지 않은 대안

아직 구현하지 마. plan.md 작성만 해줘.
```

{% hint style="warning" %}
**중요**: "아직 구현하지 마"를 반드시 포함하세요. 이 지시가 없으면 Claude가 계획을 작성한 후 바로 코드 구현까지 진행할 수 있습니다. Boris Tane 워크플로의 핵심은 **계획 승인 전까지 코드를 작성하지 않는 것**입니다.
{% endhint %}

### Step 3: 기존 구현 참조 지시

plan.md 작성 시, Claude에게 기존 코드를 명시적으로 참조하도록 지시합니다. 이렇게 하면 새로 만드는 코드가 기존 패턴과 일관됩니다.

→ *이 프롬프트는 기존 코드 패턴을 명시적으로 참조시켜, 새로운 통계 쿼리가 기존 쿼리와 일관된 구조를 갖도록 하기 위한 것입니다:*

```
plan.md를 작성할 때 다음 기존 구현을 참조해줘:

1. 쿼리 패턴: lib/queries/todos.ts의 함수 구조를 그대로 따라줘
   - 같은 import 패턴
   - 같은 에러 처리 방식
   - 같은 반환 타입 패턴

2. 페이지 구조: app/todos/page.tsx와 동일한 패턴을 따라줘
   - 서버 컴포넌트에서 데이터 패칭
   - props로 클라이언트 컴포넌트에 전달

3. UI 패턴: 기존 카드 컴포넌트 사용 방식을 따라줘
   - 같은 Card, CardHeader, CardTitle, CardContent 구조
   - 같은 grid 레이아웃 패턴

plan.md의 코드 스니펫에서 기존 파일의 패턴과 어떻게 일치하는지 명시해줘.
```

### plan.md에 포함되어야 할 내용

완성된 plan.md는 다음과 같은 구조를 가집니다:

```markdown
# Plan: /stats 대시보드 구현

## 접근 방식
- Tailwind CSS만으로 차트 구현 (외부 라이브러리 없음)
- 기존 쿼리/페이지 패턴을 그대로 따름
- 서버 컴포넌트에서 데이터 패칭, 클라이언트는 차트 UI만

## 생성/수정할 파일 목록

### 새로 생성
| 파일 | 역할 |
|------|------|
| `lib/queries/stats.ts` | 통계 쿼리 함수 모음 |
| `app/stats/page.tsx` | 대시보드 페이지 (서버 컴포넌트) |
| `components/stats/completion-cards.tsx` | 완료율 통계 카드 |
| `components/stats/category-chart.tsx` | 카테고리별 바 차트 |
| `components/stats/priority-chart.tsx` | 우선순위별 분포 |
| `components/stats/weekly-trend.tsx` | 7일 완료 추이 차트 |
| `components/stats/productivity-score.tsx` | 생산성 점수 표시 |

### 수정
| 파일 | 변경 내용 |
|------|----------|
| `components/nav.tsx` (또는 레이아웃) | /stats 링크 추가 |

## 쿼리 함수 설계 (lib/queries/stats.ts)

### getCompletionStats()
```ts
// lib/queries/todos.ts의 패턴을 따름
export async function getCompletionStats() {
  // 전체 완료율 + 이번 주 완료율
  // Returns: { total: number, completed: number, rate: number,
  //            weekTotal: number, weekCompleted: number, weekRate: number }
}
```

### getCategoryDistribution()
```ts
export async function getCategoryDistribution() {
  // GROUP BY categoryId, COUNT(*), JOIN categories
  // Returns: { categoryName: string, color: string, count: number }[]
}
```

### getPriorityDistribution()
```ts
export async function getPriorityDistribution() {
  // GROUP BY priority, COUNT(*)
  // Returns: { priority: string, count: number }[]
}
```

### getWeeklyTrend()
```ts
export async function getWeeklyTrend() {
  // 최근 7일 날짜별 완료 건수
  // Returns: { date: string, count: number }[]
}
```

### getProductivityScore()
```ts
export async function getProductivityScore() {
  // 완료율 × 40 + 이번 주 완료율 × 30 + 기한 준수율 × 30
  // Returns: { score: number, breakdown: {...} }
}
```

## UI 컴포넌트 구조

### 페이지 레이아웃
```tsx
// app/stats/page.tsx (서버 컴포넌트)
// app/todos/page.tsx와 동일한 패턴
<div className="space-y-6">
  <h1>통계 대시보드</h1>

  {/* 통계 카드 그리드 */}
  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
    <CompletionCards stats={completionStats} />
  </div>

  {/* 차트 그리드 */}
  <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
    <CategoryChart data={categoryData} />
    <PriorityChart data={priorityData} />
  </div>

  {/* 주간 추이 */}
  <WeeklyTrend data={weeklyData} />

  {/* 생산성 점수 */}
  <ProductivityScore data={productivityData} />
</div>
```

## 트레이드오프 & 대안

### 선택: Tailwind CSS만으로 차트 구현
- 장점: 추가 의존성 없음, 기존 스타일과 일관
- 단점: 복잡한 차트(파이, 라인) 구현이 어려움
- 대안: recharts 라이브러리 사용 → 더 풍부한 시각화 가능

### 선택: 분리된 쿼리 함수
- 장점: 각 통계를 독립적으로 테스트/수정 가능
- 단점: DB 호출이 여러 번 발생
- 대안: 하나의 복합 쿼리로 합치기 → 성능 좋으나 유지보수 어려움
```

## 계획 수립 단계 팁

### 1. "아직 구현하지 마"를 반드시 포함

이 한 문장이 Boris Tane 워크플로의 전체 흐름을 지탱합니다. 이것이 없으면 Claude는 자연스럽게 구현까지 진행합니다.

### 2. 기존 파일을 구체적으로 지목

```
❌ "기존 패턴을 따라줘"
✅ "lib/queries/todos.ts의 getTodos 함수와 동일한 패턴을 따라줘"
```

파일 경로와 함수 이름을 구체적으로 지목하면 Claude가 정확히 어떤 패턴을 참조해야 하는지 알 수 있습니다.

### 3. 코드 스니펫은 핵심만

plan.md의 코드 스니펫은 전체 구현이 아니라 **시그니처, 구조, 패턴**만 포함하면 됩니다. 상세한 구현은 Implementation 단계에서 합니다.

### 4. 트레이드오프를 명시적으로 요청

```
"선택한 접근의 장단점과, 고려했지만 선택하지 않은 대안도 작성해줘"
```

이렇게 하면 왜 특정 접근을 선택했는지 나중에 되돌아볼 수 있습니다.

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Plan 모드 토론 | `Shift+Tab`으로 구현 방향을 먼저 토론 |
| plan.md 파일 | 상세 구현 계획을 검토 가능한 파일로 작성 |
| 기존 구현 참조 | 구체적인 파일 경로와 함수 이름으로 참조 지시 |
| "아직 구현하지 마" | 코드 작성을 방지하는 핵심 지시 |
| 코드 스니펫 | 시그니처와 구조만 포함, 전체 구현은 나중에 |
| 트레이드오프 | 선택한 접근의 장단점과 대안을 명시 |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Plan 모드에서 구현 방향 토론을 수행함
- [ ] plan.md 파일이 프로젝트 루트에 생성됨
- [ ] plan.md에 파일 목록, 쿼리 시그니처, UI 구조, 트레이드오프가 포함됨
- [ ] 기존 구현(lib/queries/todos.ts, app/todos/page.tsx)을 참조하도록 지시함
- [ ] Claude가 코드를 작성하지 않고 계획만 작성했음을 확인함

---

> **다음**: [5.4 주석 사이클](04-annotation-cycle.md)에서 plan.md에 인라인 주석을 추가하고 Claude에게 반영을 요청하는 반복 과정을 수행합니다.
