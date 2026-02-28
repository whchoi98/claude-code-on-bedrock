# 2.6 브랜치 관리 & Custom Slash Commands (Skills)

> Git 저장소를 초기화하고, Skills(커스텀 슬래시 명령)를 만들어 반복적인 Git 작업을 자동화합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: Git 명령 실행, Skills (커스텀 슬래시 명령)

> ⭐⭐ **난이도**: 보통

## 학습 목표

- Claude Code로 Git 저장소를 초기화하고 관리하는 방법을 배웁니다
- Skills(커스텀 슬래시 명령)의 개념과 구조를 이해합니다
- 프로젝트 전용 커밋 스킬을 만들어 활용합니다

## Skills(커스텀 슬래시 명령)란?

> 💡 **비유**: Skills는 "나만의 단축키 만들기"와 같습니다. 매번 같은 내용을 반복 입력하는 대신, 한 번 정의해두면 `/commit` 같은 짧은 명령으로 실행할 수 있습니다.

매번 같은 커밋 규칙을 반복하지 않으려면? → Skills를 사용합니다.

[Skills](https://code.claude.com/docs/en/skills)는 Claude Code의 기능을 확장하는 재사용 가능한 지시사항입니다. `SKILL.md` 파일에 지시를 작성하면, Claude가 자동으로 사용하거나 `/skill-name`으로 직접 호출할 수 있습니다.

### Skills의 핵심 특징

| 특징 | 설명 |
|------|------|
| 위치 | `.claude/skills/<skill-name>/SKILL.md` (프로젝트) 또는 `~/.claude/skills/<skill-name>/SKILL.md` (개인) |
| 호출 | `/skill-name`으로 직접 호출하거나, Claude가 자동으로 사용 |
| 구성 | YAML frontmatter (설정) + Markdown (지시 내용) |
| 공유 | 프로젝트 스킬은 소스 컨트롤로 팀과 공유 가능 |

### Skills 저장 위치와 범위

| 위치 | 경로 | 범위 |
|------|------|------|
| 개인 스킬 | `~/.claude/skills/<name>/SKILL.md` | 모든 프로젝트 |
| 프로젝트 스킬 | `.claude/skills/<name>/SKILL.md` | 현재 프로젝트만 |
| 플러그인 스킬 | `<plugin>/skills/<name>/SKILL.md` | 플러그인 활성화된 곳 |

### Frontmatter 주요 필드

```yaml
---
name: skill-name              # 슬래시 명령 이름
description: What it does      # Claude가 언제 사용할지 판단하는 설명
disable-model-invocation: true # true면 사용자만 호출 가능 (Claude 자동 사용 차단)
user-invocable: true           # false면 /메뉴에서 숨김
allowed-tools: Read, Grep      # 사용 가능한 도구 제한
context: fork                  # fork면 서브에이전트에서 실행
---
```

## 실습: Git 초기화 및 Skills 생성

### Step 1: Git 저장소 초기화 및 첫 커밋

Claude Code에서 Git 저장소를 초기화합니다:

→ *이 프롬프트는 Git 저장소를 처음부터 설정하고 첫 커밋까지 한 번에 완료하기 위한 것입니다:*

```
이 프로젝트의 Git 리포지토리를 초기화해줘:
1. Next.js에 적합한 .gitignore 파일 생성 (node_modules, .next, .env*.local 등 포함)
2. Git 초기화
3. 현재 모든 파일 추가
4. "feat: initial project setup with auth, database, and dashboard" 메시지로 초기 커밋 생성
```

Claude Code는 Git 명령어를 직접 실행할 수 있으므로, 저장소 초기화부터 커밋까지 한 번에 처리합니다.

### Step 2: 기능 브랜치 생성

다음 작업을 위한 브랜치를 만듭니다:

→ *이 프롬프트는 기능별 브랜치를 생성하여 체계적으로 코드를 관리하기 위한 것입니다:*

```
세션 상세 페이지 구현을 위해 "feature/session-detail" 브랜치를 생성해줘.
```

{% hint style="info" %}
Claude Code는 Git 명령어를 이해하고 실행할 수 있습니다. 브랜치 생성, 커밋, 체크아웃 등의 작업을 자연어로 요청할 수 있습니다.
{% endhint %}

### Step 3: 커밋 스킬 만들기

프로젝트에서 자주 사용할 커밋 스킬을 만듭니다. 이 스킬은 스테이징된 변경 사항을 분석하고 [Conventional Commits](https://www.conventionalcommits.org/) 형식의 커밋 메시지를 자동 생성합니다.

먼저 디렉토리를 생성합니다:

→ *이 프롬프트는 커밋 스킬을 저장할 디렉토리를 만들기 위한 것입니다:*

```
.claude/skills/commit/ 디렉토리를 생성해줘
```

그런 다음 SKILL.md 파일을 작성합니다:

→ *이 프롬프트는 Conventional Commits 형식의 커밋 메시지를 자동 생성하는 스킬을 정의하기 위한 것입니다:*

```
다음 내용으로 .claude/skills/commit/SKILL.md를 생성해줘:

---
name: commit
description: 스테이징된 변경사항을 검토하고 설명이 포함된 conventional commit을 생성
disable-model-invocation: true
---

스테이징된 변경사항을 검토하고 커밋을 생성해줘:

1. `git diff --staged`를 실행해서 스테이징된 내용 확인
2. 스테이징된 것이 없으면 `git status`를 실행하고 스테이징할 내용 제안
3. 변경사항을 분석하고 타입 결정:
   - feat: 새 기능
   - fix: 버그 수정
   - docs: 문서
   - style: 포맷팅, 로직 변경 없음
   - refactor: 코드 구조 변경
   - test: 테스트 추가
   - chore: 유지보수 작업
4. 간결한 conventional commit 메시지 작성:
   - 형식: type(scope): description
   - scope는 선택사항, 영향받는 주요 영역 사용
   - description은 명령형, 소문자, 마침표 없음
   - 예시: feat(sessions): add filtering by date range
5. 생성된 메시지로 커밋 생성
6. 커밋 로그 항목 표시
```

{% hint style="warning" %}
**`disable-model-invocation: true`**를 설정하면 Claude가 자동으로 이 스킬을 호출하지 않습니다. 커밋은 사용자의 의도적인 행동이므로, `/commit`으로 직접 호출할 때만 실행됩니다.
{% endhint %}

### Step 4: 커밋 스킬 사용

코드를 변경한 후 스킬을 사용해봅니다:

→ *이 프롬프트는 방금 만든 커밋 스킬을 실행하여 자동으로 커밋 메시지를 생성하고 커밋하기 위한 것입니다:*

```
/commit
```

Claude Code가 다음을 수행합니다:

1. `git diff --staged`로 변경 사항 확인
2. 변경 내용 분석
3. Conventional Commit 형식의 메시지 생성
4. 커밋 실행
5. 결과 표시

예상 출력:

```
Analyzing staged changes...

Changes detected:
- Modified: app/dashboard/sessions/page.tsx (added filtering)
- New file: components/session-filters.tsx
- Modified: lib/queries/sessions.ts (added filter params)

Creating commit...
feat(sessions): add URL-based filtering for session list

[feature/session-detail abc1234] feat(sessions): add URL-based filtering for session list
 3 files changed, 145 insertions(+), 12 deletions(-)
 create mode 100644 components/session-filters.tsx
```

### Step 5: PR 생성 스킬 만들기 (선택)

GitHub에 PR을 생성하는 스킬도 만들 수 있습니다:

→ *이 프롬프트는 PR 생성을 자동화하는 스킬을 정의하기 위한 것입니다:*

```
다음 내용으로 .claude/skills/pr/SKILL.md를 생성해줘:

---
name: pr
description: 현재 브랜치를 push하고 pull request를 생성
disable-model-invocation: true
---

현재 브랜치의 pull request를 생성해줘:

1. `git log --oneline main..HEAD`를 실행해서 이 브랜치의 커밋 확인
2. 현재 브랜치를 origin에 push: `git push -u origin HEAD`
3. 모든 커밋을 분석해서 PR 제목과 설명 생성
4. PR 생성: `gh pr create --title "..." --body "..."`
   - 제목: 변경사항의 간결한 요약
   - 본문: 주요 변경사항을 bullet point로 정리한 요약 섹션 포함
5. PR URL 표시
```

{% hint style="info" %}
PR 생성 스킬은 GitHub CLI(`gh`)가 설치되어 있어야 합니다. `gh auth login`으로 사전 인증이 필요합니다.
{% endhint %}

### Step 6: 스킬 목록 확인

현재 사용 가능한 스킬을 확인합니다:

→ *이 프롬프트는 프로젝트와 개인 스킬 목록을 확인하기 위한 것입니다:*

```
사용 가능한 스킬이 뭐가 있어?
```

Claude Code가 프로젝트와 개인 스킬 목록을 보여줍니다.

## Skills 구조 심화

### 지원 파일 포함

스킬에 템플릿이나 스크립트를 함께 포함할 수 있습니다:

```
commit/
├── SKILL.md           # 메인 지시 (필수)
├── template.md        # 커밋 메시지 템플릿 (선택)
└── examples/
    └── good-commits.md  # 좋은 커밋 메시지 예시 (선택)
```

### `$ARGUMENTS` 변수

스킬에 인자를 전달할 수 있습니다:

```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.
```

사용:

```
/fix-issue 42
```

### 동적 컨텍스트 주입

`!`command`` 구문으로 셸 명령의 출력을 스킬에 주입할 수 있습니다:

```yaml
---
name: pr-review
description: Review the current PR
context: fork
agent: Explore
---

## Current PR Context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`

Review this pull request for issues and improvements.
```

## 핵심 정리

| 개념 | 설명 |
|------|------|
| Skills | `.claude/skills/`에 `SKILL.md`로 정의하는 커스텀 명령 |
| frontmatter | YAML 형식의 스킬 설정 (name, description, disable-model-invocation 등) |
| `disable-model-invocation` | `true`면 사용자가 `/name`으로만 호출 가능 |
| `$ARGUMENTS` | 스킬 호출 시 전달된 인자를 참조하는 변수 |
| `/commit` | 커밋 스킬 호출 예시 |
| 프로젝트 스킬 | `.claude/skills/`에 저장하여 팀과 공유 |
| 개인 스킬 | `~/.claude/skills/`에 저장하여 모든 프로젝트에서 사용 |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Git 저장소 초기화 및 첫 커밋 완료
- [ ] `/commit` 스킬이 동작함
- [ ] 기능 브랜치가 생성됨

---

> **다음**: [2.7 편집 페이지 & 서브에이전트](07-edit-page-subagent.md)에서 서브에이전트를 활용하여 기존 코드를 분석하고 편집 페이지를 구현합니다.
