# 4.5 Bash 명령 & 팁

> Claude Code에서 Bash를 활용하는 방법과 CLI 파이프, 비대화형 모드, Hooks, 그리고 최종 생산성 팁을 다룹니다.

## Claude Code에서 Bash 활용

Claude Code는 Bash 명령을 직접 실행할 수 있습니다. 파일 탐색, git 작업, 패키지 매니저 실행, 테스트 실행 등 터미널에서 할 수 있는 거의 모든 작업이 가능합니다.

### 유용한 패턴들

#### 빌드 에러 자동 수정

```
pnpm build를 실행하고 TypeScript 에러가 있으면 수정해줘
```

Claude는 빌드를 실행하고, 에러가 있으면 자동으로 파일을 수정한 뒤 다시 빌드를 시도합니다.

#### 린트 에러 자동 수정

```
pnpm lint를 실행하고 모든 린팅 이슈를 수정해줘
```

#### Git 히스토리 요약

```
최근 5개 커밋의 git log를 확인하고 변경사항을 요약해줘
```

#### 의존성 감사

```
pnpm audit를 실행하고 발견된 취약점을 설명해줘
```

#### 테스트 실행 후 실패 수정

```
pnpm test를 실행해줘. 실패하는 테스트가 있으면 분석하고 수정해줘.
```

## CLI 파이프 활용

Claude Code의 비대화형 모드(`claude -p`)를 활용하면 다른 도구의 출력을 Claude에 파이프로 전달할 수 있습니다. 이는 대화형 세션 바깥에서 Claude를 활용하는 강력한 방법입니다.

### 빌드 에러를 Claude에 전달

```bash
pnpm build 2>&1 | claude -p "이 빌드 에러를 수정해줘"
```

`2>&1`은 stderr를 stdout으로 리다이렉트하여 에러 메시지도 함께 전달합니다.

### 로그 분석

```bash
cat error.log | claude -p "이 에러들을 분석하고 수정 방법을 제안해줘"
```

### 변경된 파일만 리뷰

```bash
git diff main --name-only | claude -p "변경된 파일들에 이슈가 없는지 리뷰해줘"
```

### 테스트 실패 분석

```bash
pnpm test 2>&1 | claude -p "테스트 실패를 분석하고 수정 방법을 제안해줘"
```

### DB 스키마 분석

```bash
cat db/schema.ts | claude -p "이 Drizzle 스키마를 분석하고 개선사항을 제안해줘"
```

## 비대화형 모드 활용

`claude -p` 플래그로 대화형 세션 없이 단일 질문/작업을 수행할 수 있습니다.

### 단순 질문

```bash
claude -p "이 프로젝트가 무엇을 하는지 설명해줘"
```

### JSON 출력

```bash
claude -p "이 프로젝트의 모든 API 엔드포인트를 나열해줘" --output-format json
```

구조화된 JSON 형식으로 결과를 받을 수 있어 다른 스크립트와 연동하기 좋습니다.

### 스트리밍 JSON

```bash
claude -p "코드베이스 아키텍처를 분석해줘" --output-format stream-json
```

실시간으로 JSON 이벤트가 스트리밍됩니다. CI/CD 파이프라인이나 자동화 스크립트에서 유용합니다.

### 실용 예시: 커밋 메시지 생성

```bash
git diff --staged | claude -p "이 변경사항에 대한 간결한 커밋 메시지를 작성해줘"
```

### 실용 예시: PR 설명 생성

```bash
git log main..HEAD --oneline | claude -p "이 커밋들을 요약하는 PR 설명을 작성해줘"
```

## Hooks 소개

**Hooks**는 Claude의 도구 실행 전후에 자동으로 스크립트를 실행하는 기능입니다. 예를 들어, 파일 수정 후 자동으로 lint를 실행하거나, 특정 명령어를 차단할 수 있습니다.

> **공식 문서 참고**: [https://code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)

### 주요 Hook 이벤트

| 이벤트 | 실행 시점 | 용도 |
|--------|----------|------|
| `PreToolUse` | 도구 실행 전 | 명령 차단, 입력 수정 |
| `PostToolUse` | 도구 실행 후 | 자동 린트, 포맷팅 |
| `Stop` | Claude 응답 완료 시 | 작업 완료 검증 |
| `SessionStart` | 세션 시작 시 | 환경 설정, 컨텍스트 주입 |
| `Notification` | 알림 발생 시 | 커스텀 알림 처리 |

### Hook 설정 위치

| 위치 | 범위 |
|------|------|
| `~/.claude/settings.json` | 모든 프로젝트 |
| `.claude/settings.json` | 현재 프로젝트 (Git 커밋 가능) |
| `.claude/settings.local.json` | 현재 프로젝트 (로컬 전용) |

### 예시: 파일 수정 후 자동 lint

Claude에게 직접 Hook 설정을 요청할 수 있습니다:

```
파일을 수정할 때마다 pnpm lint --fix를 실행하는 hook을 만들어줘
```

Claude는 `.claude/settings.json`에 다음과 같은 설정을 추가합니다:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint --fix"
          }
        ]
      }
    ]
  }
}
```

이 설정은 Claude가 `Edit` 또는 `Write` 도구를 사용할 때마다 자동으로 `pnpm lint --fix`를 실행합니다.

### 예시: 위험한 명령 차단

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-dangerous.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/block-dangerous.sh`:

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -qE 'rm -rf|drop table|DROP TABLE'; then
  echo "Blocked: Dangerous command detected" >&2
  exit 2  # exit 2는 도구 실행을 차단
fi

exit 0  # exit 0은 도구 실행을 허용
```

### /hooks 명령

대화형 세션에서 `/hooks`를 입력하면 Hooks 관리 인터페이스가 열립니다:

```
/hooks
```

여기서 Hook 조회, 추가, 삭제가 가능합니다.

## 최종 팁 정리

Chapter 1~4를 통해 배운 Claude Code 생산성 팁을 정리합니다.

### 1. /clear를 자주 사용하세요

컨텍스트 창이 커지면 응답 품질이 떨어집니다. 새로운 작업을 시작할 때, 또는 같은 수정을 2번 이상 요청해도 해결되지 않을 때 `/clear`로 초기화하세요.

### 2. CLAUDE.md를 간결하게 유지하세요

CLAUDE.md에 너무 많은 내용을 넣으면 매 요청마다 컨텍스트를 소비합니다. 핵심 규칙과 프로젝트 구조만 포함하고, 상세한 내용은 스킬로 분리하세요.

### 3. 서브에이전트로 탐색을 분리하세요

코드베이스 탐색은 컨텍스트를 많이 소비합니다. `context: fork` 스킬이나 커스텀 서브에이전트를 사용하면 메인 대화의 컨텍스트를 절약할 수 있습니다.

### 4. 구체적인 프롬프트를 작성하세요

| 비효과적 | 효과적 |
|---------|--------|
| "이거 고쳐줘" | "app/dashboard/page.tsx의 date picker가 모바일에서 오버플로됩니다. max-width: 100%를 추가해주세요" |
| "테스트 작성해줘" | "actions/study-blocks.ts의 createStudyBlock 함수에 대한 단위 테스트를 작성해줘. 인증 실패, 유효성 검증 실패, 정상 생성 케이스를 포함해줘" |

### 5. 검증 기준을 함께 제공하세요

```
X를 구현해줘. 그 다음 검증해줘:
1. 기준 A
2. 기준 B
3. 기준 C
```

### 6. Plan 모드로 먼저 계획하세요

복잡한 기능을 구현하기 전에:

```
/plan
```

Plan 모드에서는 Claude가 코드를 수정하지 않고 계획만 수립합니다. 계획을 검토한 후 실행하면 더 좋은 결과를 얻을 수 있습니다.

### 7. 2번 실패하면 /clear 후 재시작하세요

같은 문제로 2번 이상 수정을 요청해도 해결되지 않으면, 컨텍스트가 혼란스러워진 것일 수 있습니다. `/clear` 후 문제를 처음부터 명확하게 설명하세요.

### 8. Bedrock 사용 시 모델 버전 고정을 권장합니다

Amazon Bedrock에서 Claude Code를 사용할 때, 모델 버전을 고정하면 일관된 결과를 얻을 수 있습니다. 환경 변수로 설정합니다:

```bash
# .env 또는 셸 프로파일에 추가
export CLAUDE_CODE_USE_BEDROCK=1
export ANTHROPIC_MODEL="us.anthropic.claude-sonnet-4-20250514-v1:0"
```

{% hint style="info" %}
Bedrock에서 사용 가능한 모델 ID는 AWS 콘솔의 Bedrock 모델 액세스 페이지에서 확인하세요. Cross-region inference를 사용하는 경우 `us.` 접두사가 붙습니다.
{% endhint %}

## 워크샵 마무리

축하합니다! Chapter 1부터 Chapter 4까지 Claude Code on Amazon Bedrock 핸즈온 워크샵을 완료했습니다.

### 이 워크샵에서 배운 것들

| Chapter | 핵심 내용 |
|---------|-----------|
| Chapter 1 | 프로젝트 설정, Bedrock 연결, CLAUDE.md, 컨텍스트 창 개념 |
| Chapter 2 | 문서 기반 코드 생성, Thinking 모드, 서버 액션, 슬래시 명령, 서브에이전트 |
| Chapter 3 | Vercel 배포, GitHub Actions + Claude Code, Issue 기반 자동 개발 |
| Chapter 4 | UI 개선, 스킬, 커스텀 에이전트, Hooks, CLI 파이프, 생산성 팁 |

### 다음 단계

- [Claude Code 공식 문서](https://code.claude.com/docs)에서 더 많은 기능 탐색
- [부록: 참고 자료 & 문제 해결](../appendix/README.md)에서 추가 리소스 확인
- 학습 트래커에 새로운 기능을 추가하며 실습 계속

---

[부록: 참고 자료 & 문제 해결](../appendix/README.md)로 이동
