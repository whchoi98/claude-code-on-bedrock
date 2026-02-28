# 3.3 Issue 기반 자동 개발

> GitHub Issue를 생성하면 Claude가 자동으로 코드를 구현하고 PR을 생성하는 워크플로를 실습합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: @claude 멘션으로 Issue 기반 자동 개발

> ⭐⭐ **난이도**: 보통

## 개요

앞서 설정한 GitHub Actions 워크플로를 활용하여 **Issue 기반 자동 개발**을 실습합니다. Issue에 `@claude`를 멘션하면 Claude가 코드베이스를 분석하고, CLAUDE.md를 읽어 프로젝트 규칙을 따르며, 코드를 구현하여 PR을 생성합니다.

## 실습 1: 삭제 기능 자동 구현

### Issue 생성

GitHub에서 새 Issue를 생성합니다:

**제목**: 학습 세션 삭제 기능 추가

→ *이 프롬프트는 학습 세션 삭제 기능을 Claude에게 자동 구현 요청하기 위한 것입니다:*

**본문**:
```markdown
@claude

학습 세션 삭제 기능을 구현해주세요.

## 요구사항
- 세션 목록 페이지에서 각 세션에 삭제 버튼 추가
- 삭제 전 확인 다이얼로그 표시 (shadcn/ui Dialog 사용)
- Server action으로 삭제 구현
  - session_subjects와 study_blocks도 cascade로 함께 삭제
- 삭제 후 세션 목록 페이지로 리다이렉트
- revalidatePath로 캐시 갱신

## 기술 규칙
- CLAUDE.md의 코딩 컨벤션을 따라주세요
- docs/data-mutations.md 패턴을 참고해주세요
- Zod로 입력 검증을 해주세요
```

### Claude의 자동 작업 과정

Issue가 생성되면 GitHub Actions 워크플로가 트리거됩니다. Claude는 다음 순서로 작업합니다:

1. **리포지토리 체크아웃**: 코드베이스를 가져옵니다
2. **CLAUDE.md 읽기**: 프로젝트 규칙과 컨벤션을 파악합니다
3. **코드베이스 분석**: 기존 패턴 (server actions, UI 컴포넌트 등)을 이해합니다
4. **코드 구현**:
   - `actions/sessions.ts`에 `deleteSession` server action 추가
   - 세션 목록 UI에 삭제 버튼 추가
   - 확인 다이얼로그 컴포넌트 생성
5. **PR 생성**: 구현한 코드로 Pull Request를 생성합니다

### 결과 확인

GitHub Actions 탭에서 워크플로 실행을 확인합니다. 완료되면:

1. Issue에 Claude의 응답 코멘트가 추가됩니다
2. 새로운 PR이 생성됩니다
3. PR에 변경 사항 요약이 포함됩니다

## 실습 2: PR에서 수정 요청

Claude가 생성한 PR을 리뷰하고 추가 수정을 요청합니다.

### PR 리뷰 코멘트 작성

생성된 PR의 코멘트에 다음과 같이 작성합니다:

→ *이 프롬프트는 Claude가 생성한 PR에 추가 수정 사항을 요청하기 위한 것입니다:*

```
@claude 다음 사항을 수정해주세요:

1. 삭제 확인 다이얼로그에 세션 날짜와 과목 정보도 표시해주세요
2. 삭제 버튼의 색상을 빨간색(destructive variant)으로 변경해주세요
3. 삭제 중 로딩 상태를 표시해주세요
```

Claude는 기존 PR의 코드를 수정하고 새로운 커밋을 추가합니다.

### 코드 라인별 리뷰 코멘트

특정 코드 라인에 대해 리뷰 코멘트를 남길 수도 있습니다:

→ *이 프롬프트는 특정 코드 라인에 대해 에러 처리를 추가 요청하기 위한 것입니다:*

```
@claude 이 함수에서 에러 처리를 추가해주세요.
try-catch로 감싸고 사용자에게 toast 알림을 보여주세요.
```

## 실습 3: 새로운 기능 요청

더 복잡한 기능을 요청해봅니다.

### Issue 생성: 학습 통계 페이지

**제목**: 학습 통계 대시보드 추가

→ *이 프롬프트는 통계 대시보드 페이지를 새로 생성 요청하기 위한 것입니다:*

**본문**:
```markdown
@claude

학습 통계 대시보드 페이지를 만들어주세요.

## 요구사항
- /dashboard/stats 경로에 통계 페이지 생성
- 표시할 통계:
  - 이번 주 총 학습 시간
  - 과목별 학습 시간 비율 (색상 바 차트)
  - 최근 7일간 일별 학습 시간
  - 가장 많이 공부한 과목 TOP 3
- 서버 컴포넌트로 데이터 패칭
- lib/queries/stats.ts에 쿼리 함수 생성
- shadcn/ui Card 컴포넌트로 각 통계 표시
- 차트는 div 기반 간단한 바 차트 (외부 라이브러리 미사용)

## 참고
- CLAUDE.md와 docs/data-fetching.md 패턴을 따라주세요
- 사이드바 네비게이션에 통계 메뉴 항목도 추가해주세요
```

## CLAUDE.md의 중요성

GitHub Actions에서 Claude Code가 실행될 때도 **CLAUDE.md를 자동으로 읽습니다**. 이는 다음을 의미합니다:

- 프로젝트 규칙이 자동으로 적용됩니다
- 코딩 컨벤션이 일관되게 유지됩니다
- 빌드/테스트 명령이 올바르게 실행됩니다

### CLAUDE.md 최적화 팁

```markdown
# CLAUDE.md 예시 (GitHub Actions에 최적화)

## Build & Dev Commands
- Build: pnpm build
- Lint: pnpm lint
- Type check: pnpm tsc --noEmit

## Code Conventions
- Server components by default
- Server actions in actions/ directory
- Validate with zod
- Use shadcn/ui components

## IMPORTANT
- Always run pnpm build before creating a PR
- Always check auth() for userId in server actions
```

{% hint style="info" %}
CLAUDE.md를 **간결하게** 유지하세요. 너무 길면 Claude가 중요한 규칙을 놓칠 수 있습니다.
{% endhint %}

## 비용 관리

### GitHub Actions 비용

- Claude Code는 GitHub 호스팅 러너에서 실행되어 **GitHub Actions 분(minutes)**을 소비합니다
- 자세한 요금은 [GitHub 과금 문서](https://docs.github.com/en/billing/managing-billing-for-your-products/managing-billing-for-github-actions/about-billing-for-github-actions)를 참고하세요

### Bedrock API 비용

각 Claude 상호작용은 프롬프트와 응답 길이에 따라 토큰을 소비합니다.

### 비용 절감 팁

| 방법 | 설명 |
|------|------|
| `--max-turns` 제한 | `claude_args`에서 `--max-turns 10`으로 불필요한 반복 방지 |
| 구체적인 @claude 명령 | 모호한 요청 대신 구체적인 지시 사항 제공 |
| CLAUDE.md 간결 유지 | 불필요한 내용 제거로 토큰 소비 최소화 |
| 복잡한 작업 분할 | 큰 기능은 여러 작은 Issue로 분할 |
| 워크플로 타임아웃 설정 | `timeout-minutes`로 무한 실행 방지 |

```yaml
# 워크플로에 타임아웃 추가
jobs:
  claude:
    runs-on: ubuntu-latest
    timeout-minutes: 15  # 15분 타임아웃
```

## Skills 활용 (선택)

GitHub Actions에서도 Skills(커스텀 슬래시 명령)를 사용할 수 있습니다:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    github_token: ${{ steps.app-token.outputs.token }}
    use_bedrock: "true"
    prompt: "/review"  # .claude/skills/review 스킬 실행
    claude_args: '--max-turns 5'
```

PR이 열릴 때 자동으로 코드 리뷰를 실행하는 워크플로 예시:

```yaml
name: Auto Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4

      - name: Generate GitHub App token
        id: app-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ steps.app-token.outputs.token }}
          use_bedrock: "true"
          prompt: "이 PR의 코드 품질, 보안, CLAUDE.md 컨벤션과의 일관성을 리뷰해줘."
          claude_args: '--model us.anthropic.claude-sonnet-4-6 --max-turns 5'
```

## 요약

| 항목 | 상태 |
|------|------|
| Issue에서 @claude로 기능 요청 | &#x2705; |
| Claude 자동 코드 구현 및 PR 생성 | &#x2705; |
| PR 코멘트에서 수정 요청 | &#x2705; |
| CLAUDE.md 자동 적용 확인 | &#x2705; |
| 비용 관리 설정 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Issue 생성 후 Claude가 자동으로 PR 생성 확인
- [ ] PR 코멘트에서 @claude로 수정 요청 후 코드가 업데이트되는지 확인
- [ ] CLAUDE.md 규칙이 자동으로 적용되는지 확인
- [ ] 비용 관리를 위한 --max-turns 설정 완료

이것으로 Chapter 3이 완료되었습니다. [Chapter 4: 고급 기능](../chapter-04/README.md)에서는 Claude Code의 고급 기능을 학습합니다.
