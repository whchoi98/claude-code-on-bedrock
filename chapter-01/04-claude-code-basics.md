# 1.4 Claude Code 기본 명령어 & 모드

> Claude Code의 슬래시 명령어, 작업 모드, 대화 관리, 키보드 단축키를 익힙니다.

## 개요

Claude Code는 단순한 코드 생성 도구가 아닌 **에이전틱 코딩 환경**입니다. 파일을 읽고, 명령을 실행하고, 코드를 수정하며, 문제를 자율적으로 해결합니다. 이 섹션에서는 Claude Code를 효과적으로 사용하기 위한 기본 명령어와 작업 모드를 학습합니다.

## 슬래시 명령어

Claude Code 프롬프트에서 `/`로 시작하는 명령어를 사용할 수 있습니다.

### 핵심 명령어

| 명령어 | 설명 | 사용 시점 |
|--------|------|-----------|
| `/help` | 도움말 및 사용 가능한 명령어 목록 표시 | 명령어가 기억나지 않을 때 |
| `/init` | 프로젝트를 분석하여 CLAUDE.md 자동 생성 | 새 프로젝트 시작 시 |
| `/clear` | 현재 대화의 컨텍스트를 완전히 초기화 | 작업 전환 시, 성능 저하 시 |
| `/compact` | 대화 내용을 요약하여 압축 | 컨텍스트가 차오를 때 |
| `/rewind` | 이전 체크포인트로 되돌리기 | 변경사항을 되돌리고 싶을 때 |
| `/permissions` | 권한 설정 (허용 목록 관리) | 반복 승인이 번거로울 때 |
| `/mcp` | MCP 서버 상태 및 도구 확인 | MCP 연결 상태 확인 시 |
| `/agents` | 서브에이전트 관리 | 사용 가능한 에이전트 확인 시 |

### 실습: 명령어 체험

Claude Code를 실행한 상태에서 다음을 순서대로 시도합니다:

```
/help
```

사용 가능한 모든 명령어가 표시됩니다. 이 중 가장 자주 사용하게 될 명령어는 `/clear`, `/compact`, `/rewind`입니다.

## 작업 모드

Claude Code에는 두 가지 작업 모드가 있습니다.

### Normal 모드 (기본)

기본 모드로 Claude Code의 모든 기능을 사용할 수 있습니다:

- 파일 읽기/쓰기
- 코드 수정
- 명령어 실행 (npm, git 등)
- 파일 생성/삭제

```
# Normal 모드에서의 프롬프트 예시
Fix the TypeScript error in app/page.tsx and run the build to verify.
```

### Plan 모드 (Shift+Tab)

**읽기 전용** 모드로, 코드베이스를 탐색하고 계획을 세울 때 사용합니다. 파일 수정이나 명령 실행은 하지 않습니다.

```
# Plan 모드에서의 프롬프트 예시
How should I structure the database schema for a study tracker app?
What files need to change to add authentication?
```

{% hint style="info" %}
**Plan 모드 활용 패턴**: 공식 문서에서 권장하는 4단계 워크플로입니다.

1. **탐색** (Plan 모드): 코드베이스를 읽고 현재 구조를 파악
2. **계획** (Plan 모드): 구현 계획을 수립
3. **구현** (Normal 모드): 계획에 따라 코딩
4. **커밋** (Normal 모드): 변경사항 커밋 및 PR 생성
{% endhint %}

### 모드 전환

- **Plan 모드 진입**: `Shift+Tab`을 누릅니다.
- **Normal 모드로 복귀**: 다시 `Shift+Tab`을 누릅니다.

프롬프트 입력란 좌측에 현재 모드가 표시됩니다.

## 대화 관리

### 대화 이어가기

Claude Code를 종료한 후에도 이전 대화를 이어갈 수 있습니다.

```bash
# 가장 최근 대화 이어가기
claude --continue

# 이전 대화 목록에서 선택하여 이어가기
claude --resume
```

{% hint style="info" %}
`/rename` 명령으로 세션에 설명적인 이름을 지정할 수 있습니다. 예를 들어 `"auth-setup"`, `"schema-design"` 등으로 이름을 지정하면 나중에 `--resume`으로 세션을 찾기 쉽습니다.
{% endhint %}

### 비대화형 모드

스크립트나 파이프라인에서 Claude Code를 사용할 때는 `-p` 플래그를 사용합니다:

```bash
# 단일 질문
claude -p "Explain what this project does"

# 구조화된 출력
claude -p "List all API endpoints" --output-format json

# 스트리밍 JSON 출력
claude -p "Analyze this log file" --output-format stream-json
```

비대화형 모드는 세션을 생성하지 않으므로 `--continue`나 `--resume`으로 이어갈 수 없습니다.

## 키보드 단축키

| 단축키 | 동작 |
|--------|------|
| `Esc` | 현재 Claude의 작업을 중지. 컨텍스트는 유지됨 |
| `Esc` + `Esc` | Rewind 메뉴 열기 (대화/코드 복원 선택) |
| `Shift+Tab` | Normal 모드 <-> Plan 모드 전환 |
| `Ctrl+B` | 백그라운드 모드 전환 (Claude가 작업하는 동안 다른 일 가능) |

### Rewind (되돌리기)

Claude Code는 모든 변경 전에 자동으로 체크포인트를 생성합니다. `Esc`를 두 번 누르거나 `/rewind` 명령을 사용하면 rewind 메뉴가 열립니다.

Rewind 메뉴에서 선택할 수 있는 옵션:
- **대화만 복원**: 대화 상태만 이전으로 되돌림
- **코드만 복원**: 파일 변경사항만 되돌림
- **모두 복원**: 대화와 코드를 모두 이전 상태로 되돌림
- **여기서부터 요약(Summarize from here)**: 선택한 시점부터 현재까지의 대화를 요약

{% hint style="warning" %}
체크포인트는 **Claude Code가 수행한 변경**만 추적합니다. 외부에서 직접 수행한 변경은 체크포인트에 포함되지 않으므로 Git을 대체할 수 없습니다.
{% endhint %}

## 실습: 기본 명령어 체험

Claude Code에서 다음을 순서대로 실행해 봅니다:

### 1. Plan 모드로 프로젝트 탐색

`Shift+Tab`으로 Plan 모드를 활성화한 후:

```
package.json을 읽고 어떤 의존성과 스크립트가 설정되어 있는지 알려줘.
```

### 2. Normal 모드로 전환 후 파일 수정

`Shift+Tab`으로 Normal 모드로 전환 후:

```
app/layout.tsx의 타이틀을 "학습 트래커 | Study Tracker"로 변경해줘.
```

### 3. Rewind로 되돌리기

`Esc`를 두 번 누르거나 `/rewind`를 입력하여 방금 수행한 변경을 되돌려 봅니다.

### 4. 컨텍스트 초기화

```
/clear
```

새로운 작업을 시작할 준비가 완료됩니다.

## 요약

| 개념 | 핵심 포인트 |
|------|-------------|
| 슬래시 명령어 | `/clear`, `/compact`, `/rewind`가 가장 중요 |
| 모드 | Normal (전체 기능), Plan (읽기 전용 탐색/계획) |
| 대화 관리 | `--continue` (최근), `--resume` (선택) |
| 비대화형 | `claude -p "prompt"` (스크립트용) |
| 단축키 | `Esc` (중지), `Esc+Esc` (되돌리기), `Shift+Tab` (모드 전환) |

다음 섹션에서는 [Clerk 인증을 설정](05-auth-setup.md)합니다.
