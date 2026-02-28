# Chapter 5: 슈퍼랩 — 프로처럼 Claude Code 사용하기

> Boris Tane의 프로페셔널 워크플로를 적용하여, Todo 앱에 대시보드 + 통계 분석 페이지를 처음부터 끝까지 만드는 종합 실습입니다.

## 이 챕터에서 다루는 내용

Chapter 1~4에서 Claude Code의 개별 기능을 학습했습니다. 이제 **종합 실습(슈퍼랩)**으로, [Boris Tane의 블로그](https://boristane.com/blog/how-i-use-claude-code/)에서 소개한 프로페셔널 워크플로를 적용하여 **대시보드 + 통계 분석 페이지**를 만들어봅니다.

### 핵심 원칙

> **"계획을 검토하고 승인하기 전까지 Claude가 코드를 작성하지 않게 한다"**

이것이 Boris Tane 워크플로의 가장 중요한 원칙입니다. 코드 작성을 마지막 단계로 미루고, 그 전에 충분한 리서치와 계획 수립, 그리고 반복적인 계획 검토를 거칩니다.

### Boris Tane 워크플로 4단계

```
Research → Planning → Annotation Cycle → Implementation
(리서치)    (계획)     (주석 사이클)       (구현)
```

| 단계 | 목적 | Claude의 역할 |
|------|------|--------------|
| Research | 코드베이스 깊이 이해 | 코드 분석 + 마크다운 파일로 정리 |
| Planning | 구현 계획 수립 | plan.md 파일 작성 |
| Annotation | 계획 검토 & 수정 | 인라인 메모 반영 + 계획 업데이트 |
| Implementation | 코드 작성 | 계획 기반 일괄 구현 |

### 학습 목표

| 섹션 | 내용 | 핵심 기능 |
|------|------|-----------|
| [5.1 프로 워크플로 소개](01-workflow-overview.md) | Boris Tane 워크플로 이해 | 4단계 흐름 이해 |
| [5.2 리서치 단계](02-research-phase.md) | 서브에이전트 리서치 + 마크다운 결과 파일 | 서브에이전트, 파일 기반 결과 |
| [5.3 계획 수립 단계](03-planning-phase.md) | plan.md 파일 작성 + 기존 구현 참조 | Plan 모드, 파일 기반 계획 |
| [5.4 주석 사이클](04-annotation-cycle.md) | 인라인 주석으로 계획 수정 반복 | 주석 기반 협업 |
| [5.5 구현 단계](05-implementation.md) | 계획 기반 일괄 구현 + 자체 검증 | 일괄 구현, 반복 피드백 |
| [5.6 리뷰 & 프로 팁](06-review-and-tips.md) | 전체 복습 + 프로 팁 10선 | 워크플로 종합 |

### 구현할 기능: `/stats` 대시보드

Todo 앱에 통계 분석 대시보드 페이지를 추가합니다:

- 완료율 통계 카드 (전체, 이번 주)
- 카테고리별 할 일 분포 (색상 바 차트)
- 우선순위별 분포 (파이 차트 스타일)
- 최근 7일 완료 추이 (일별 바 차트)
- 생산성 점수 계산 로직

이 기능은 다중 파일 변경, 복합 쿼리, 서버/클라이언트 분리가 필요해 슈퍼랩에 적합합니다.

### 사전 조건

- Chapter 1~4 완료 (Todo 앱 기능 + GitHub 저장소 설정)
- 데이터베이스에 `todos`, `categories` 테이블 생성 완료
- 개발 서버가 정상 작동하는 상태
- 테스트용 할 일 데이터가 충분히 등록된 상태

### DB 스키마 참고

이 챕터에서는 통계 쿼리를 위해 `todos`와 `categories` 테이블을 적극 활용합니다.

```
todos: id, title, description, completed, priority, dueDate, categoryId, createdAt, updatedAt
categories: id, name, color, createdAt
```

### Chapter 1~4 복습 — 이 챕터에서 활용하는 기능

| 기능 | 배운 챕터 | 슈퍼랩에서의 활용 |
|------|----------|-----------------|
| 기본 명령어 (`/clear`, `/compact`, `/rewind`) | Chapter 1 | 긴 세션 관리, 되돌리기 |
| Plan 모드 (`Shift+Tab`) | Chapter 1 | 계획 수립 단계 |
| `/init`, CLAUDE.md | Chapter 2 | 프로젝트 규칙 참조 |
| Extended Thinking | Chapter 2 | 복잡한 쿼리 설계 |
| Skills (커스텀 슬래시 명령) | Chapter 2 | `/commit`으로 커밋 |
| 서브에이전트 | Chapter 2 | 리서치 단계에서 코드 분석 |
| GitHub Actions 연동 | Chapter 3 | 최종 커밋 & 푸시 |
| 이미지 기반 피드백 | Chapter 4 | UI 미세 조정 |
| Hooks | Chapter 4 | 자동 린트 실행 |
| CLI 파이프 (`claude -p`) | Chapter 4 | 빌드 에러 전달 |

{% hint style="info" %}
이 챕터는 **종합 실습**이므로, 새로운 Claude Code 기능을 소개하는 것이 아니라 이전에 배운 기능들을 **프로 워크플로**에 통합하는 데 초점을 맞춥니다.
{% endhint %}

---

준비가 되었다면 [5.1 프로 워크플로 소개](01-workflow-overview.md)부터 시작하겠습니다.
