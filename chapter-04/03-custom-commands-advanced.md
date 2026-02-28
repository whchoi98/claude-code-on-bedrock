# 4.3 사용자 범위 슬래시 명령 (User-scope Skills)

> 프로젝트 범위를 넘어 모든 프로젝트에서 재사용할 수 있는 사용자 범위 스킬을 만듭니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 사용자 범위 Skills (~/.claude/skills/)

> ⭐⭐ **난이도**: 보통

## 스킬(Skills) 개요

Claude Code의 **스킬(Skills)**은 `SKILL.md` 파일로 정의되는 확장 기능입니다. Claude의 동작을 커스터마이즈하고, 반복 작업을 자동화할 수 있습니다. 스킬은 슬래시 명령(`/skill-name`)으로 직접 호출하거나, Claude가 상황에 맞게 자동으로 사용합니다.

> **공식 문서 참고**: [https://code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

{% hint style="info" %}
기존의 `.claude/commands/` 디렉토리에 있던 **커스텀 슬래시 명령**은 스킬에 통합되었습니다. `.claude/commands/` 파일들은 여전히 작동하지만, 스킬이 더 많은 기능(프론트매터, 지원 파일, 호출 제어 등)을 제공합니다.
{% endhint %}

## 프로젝트 범위 vs 사용자 범위

스킬은 저장 위치에 따라 적용 범위가 달라집니다:

| 범위 | 경로 | 적용 대상 |
|------|------|-----------|
| 프로젝트 | `.claude/skills/<skill-name>/SKILL.md` | 이 프로젝트에서만 사용 |
| 사용자 | `~/.claude/skills/<skill-name>/SKILL.md` | 모든 프로젝트에서 사용 |
| 엔터프라이즈 | 관리형 설정 참조 | 조직 전체 |

**사용자 범위 스킬**은 어떤 프로젝트에서든 사용할 수 있으므로, 코드 리뷰, 디버깅, 커밋 관련 등 범용적인 워크플로에 적합합니다.

## Step 1: 코드 리뷰 스킬 만들기

모든 프로젝트에서 사용할 수 있는 코드 리뷰 스킬을 만들어봅시다.

### 디렉토리 생성

```bash
mkdir -p ~/.claude/skills/review
```

### SKILL.md 작성

`~/.claude/skills/review/SKILL.md` 파일을 생성합니다:

```yaml
---
name: review
description: 코드 변경사항의 품질, 보안, 모범 사례를 리뷰
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash
---

현재 코드 변경사항을 리뷰해줘:

1. `git diff`를 실행하여 커밋되지 않은 변경사항 확인
2. 변경된 각 파일을 다음 기준으로 분석:
   - 코드 품질과 가독성
   - 잠재적 버그 또는 엣지 케이스
   - 보안 이슈
   - 성능 문제
   - 프로젝트 패턴과의 일관성
3. 다음과 같이 요약 제공:
   - **심각한 이슈** (반드시 수정)
   - **경고** (수정 권장)
   - **제안** (있으면 좋은 것)
4. 변경사항이 없으면 `git diff --staged`로 스테이징된 변경사항 확인
```

### 프론트매터 설명

| 필드 | 값 | 의미 |
|------|-----|------|
| `name` | `review` | `/review`로 호출 가능 |
| `description` | 설명 텍스트 | Claude가 언제 사용할지 판단하는 기준 |
| `disable-model-invocation` | `true` | Claude가 자동으로 호출하지 않음. 사용자가 직접 `/review`로만 실행 |
| `allowed-tools` | `Read, Grep, Glob, Bash` | 이 스킬 실행 시 허용할 도구 |

### 사용 방법

Claude Code에서 다음과 같이 호출합니다:

→ *이 프롬프트는 생성한 코드 리뷰 스킬을 실행하기 위한 것입니다:*

```
/review
```

Claude는 `git diff`를 실행하고, 변경된 파일들을 분석하여 리뷰 결과를 보고합니다.

## Step 2: 디버그 스킬 만들기

에러 메시지를 전달하면 자동으로 원인을 분석하고 수정안을 제시하는 스킬입니다.

### 디렉토리 생성

```bash
mkdir -p ~/.claude/skills/debug
```

### SKILL.md 작성

`~/.claude/skills/debug/SKILL.md`:

```yaml
---
name: debug
description: 에러 또는 이슈 디버그
disable-model-invocation: true
---

다음 이슈를 디버그해줘: $ARGUMENTS

1. 에러 메시지 또는 설명 분석
2. Grep과 Glob을 사용하여 관련 코드 검색
3. 근본 원인 파악
4. 코드 예시와 함께 수정안 제시
5. 수정이 간단하면 직접 구현
```

### $ARGUMENTS 변수

`$ARGUMENTS`는 슬래시 명령 호출 시 전달된 인자로 대체됩니다. 예를 들어:

→ *이 프롬프트는 디버그 스킬에 에러 메시지를 인자로 전달하기 위한 것입니다:*

```
/debug TypeError: Cannot read property 'map' of undefined in dashboard page
```

이 경우 `$ARGUMENTS`가 `TypeError: Cannot read property 'map' of undefined in dashboard page`로 치환됩니다.

{% hint style="info" %}
`$ARGUMENTS`가 스킬 본문에 포함되어 있지 않으면, 전달된 인자는 스킬 내용 끝에 `ARGUMENTS: <value>` 형태로 자동 추가됩니다.
{% endhint %}

### 위치 기반 인자 접근

여러 인자를 개별적으로 접근할 수도 있습니다:

```yaml
---
name: compare
description: 두 파일 비교
disable-model-invocation: true
---

다음 두 파일을 비교해줘:
- 파일 1: $ARGUMENTS[0]
- 파일 2: $ARGUMENTS[1]

주요 차이점을 보여주고 어떤 접근 방식이 더 나은지 설명해줘.
```

사용 예: `/compare src/old-auth.ts src/new-auth.ts`

축약형 `$0`, `$1`도 사용 가능합니다 (`$0`은 `$ARGUMENTS[0]`과 동일).

## Step 3: 스킬 사용해보기

생성한 스킬들을 학습 트래커 프로젝트에서 사용해봅시다.

### 코드 리뷰 스킬 테스트

→ *이 프롬프트는 코드 변경사항에 대해 리뷰 스킬을 실행하기 위한 것입니다:*

```bash
# 먼저 코드 변경을 만들어봅니다
# (이전 섹션에서 UI 개선 작업을 했다면 변경사항이 있을 것입니다)

# Claude Code에서:
/review
```

### 디버그 스킬 테스트

→ *이 프롬프트는 특정 버그를 디버그 스킬에 전달하여 원인 분석을 요청하기 위한 것입니다:*

```
/debug 학습 블록을 삭제할 때 합계가 업데이트되지 않아
```

## Step 4: 스킬 고급 팁

### 팁 1: disable-model-invocation 사용

`disable-model-invocation: true`를 설정하면 Claude가 자동으로 이 스킬을 호출하지 않습니다. 다음과 같은 경우에 권장합니다:

- 부작용이 있는 워크플로 (배포, 커밋 등)
- 실행 타이밍을 사용자가 제어해야 하는 경우
- 리소스를 많이 사용하는 작업

### 팁 2: allowed-tools로 도구 제한

스킬이 실행될 때 사용 가능한 도구를 제한할 수 있습니다:

```yaml
---
name: safe-reader
description: 변경 없이 코드를 읽고 분석
allowed-tools: Read, Grep, Glob
---
```

이 스킬은 파일을 읽기만 할 수 있고, 수정이나 Bash 명령 실행은 불가능합니다.

### 팁 3: context: fork로 서브에이전트에서 실행

스킬을 메인 대화와 분리된 서브에이전트에서 실행할 수 있습니다:

```yaml
---
name: analyze
description: 프로젝트 구조 분석
context: fork
agent: Explore
---

프로젝트 구조를 분석하고 요약해줘.
```

`context: fork`를 사용하면:
- 메인 대화의 컨텍스트 창을 소비하지 않음
- 독립된 컨텍스트에서 실행되어 탐색 결과가 메인 대화에 영향을 주지 않음
- 결과만 요약되어 메인 대화로 반환됨

{% hint style="warning" %}
`context: fork`는 명확한 작업 지시가 있는 스킬에만 사용하세요. "이 API 규칙을 따르세요"와 같은 참조용 스킬에는 적합하지 않습니다. 서브에이전트는 대화 히스토리 없이 스킬 내용만 받기 때문입니다.
{% endhint %}

### 팁 4: 지원 파일 추가

스킬 디렉토리에 추가 파일을 포함할 수 있습니다:

```
~/.claude/skills/review/
├── SKILL.md           # 메인 지침 (필수)
├── checklist.md       # 리뷰 체크리스트
└── examples/
    └── good-review.md # 좋은 리뷰 예시
```

`SKILL.md`에서 지원 파일을 참조합니다:

```markdown
## 추가 리소스
- 상세 체크리스트: [checklist.md](checklist.md)
- 좋은 리뷰 예시: [examples/good-review.md](examples/good-review.md)
```

## 정리

이 섹션에서 배운 내용:

- **스킬(Skills)**: `SKILL.md` 파일로 정의하는 Claude Code 확장 기능
- **프로젝트 vs 사용자 범위**: `.claude/skills/`는 프로젝트, `~/.claude/skills/`는 전역
- **프론트매터**: `name`, `description`, `disable-model-invocation`, `allowed-tools` 등 설정
- **$ARGUMENTS**: 슬래시 명령 호출 시 인자 전달
- **context: fork**: 서브에이전트에서 격리 실행
- **지원 파일**: 스킬 디렉토리에 체크리스트, 템플릿 등 추가 가능

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] `~/.claude/skills/` 디렉토리에 스킬 파일 생성 확인
- [ ] `/review` 스킬이 정상적으로 동작하는지 확인
- [ ] `/debug` 스킬에 인자 전달이 올바르게 되는지 확인
- [ ] 프론트매터 설정 (name, description, disable-model-invocation 등) 이해

---

다음: [4.4 에이전트 스킬](04-skills.md)
