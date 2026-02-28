# Chapter 4: 고급 기능

> Claude Code의 고급 기능을 활용하여 Todo 앱을 한 단계 더 발전시킵니다.

## 이 챕터에서 다루는 내용

이전 챕터들에서 Todo 앱의 기본 기능을 구현하고 GitHub 저장소 설정까지 완료했습니다. 이제 Claude Code의 고급 기능들을 활용하여 앱을 더욱 완성도 높게 만들어보겠습니다.

### 학습 목표

| 섹션 | 내용 | 핵심 기능 |
|------|------|-----------|
| [4.1 UI 개선](01-ui-refinement.md) | 팝오버 캘린더, 통계 카드, 차트 | 이미지 기반 피드백, 반복적 개선 |
| [4.2 할 일 일괄 관리 기능](02-exercise-logging.md) | 일괄 완료, 일괄 삭제, 일괄 카테고리 변경 | 복합 기능 구현, 자체 검증 |
| [4.3 사용자 범위 슬래시 명령](03-custom-commands-advanced.md) | 사용자 범위 스킬 생성 | Skills, `~/.claude/skills/` |
| [4.4 에이전트 스킬](04-skills.md) | 자동 스킬, 서브에이전트, 커스텀 에이전트 | context: fork, .claude/agents/ |
| [4.5 Bash 명령 & 팁](05-bash-and-tips.md) | CLI 파이프, 비대화형 모드, Hooks | 생산성 향상 팁 |

### 사전 조건

- Chapter 1~3 완료 (Todo 앱 기본 기능 + GitHub 저장소 설정)
- 데이터베이스에 `todos`, `categories` 테이블 생성 완료
- 개발 서버가 정상 작동하는 상태

### DB 스키마 참고

이 챕터에서는 특히 `todos`와 `categories` 테이블을 적극 활용합니다.

```
todos: id, title, description, completed, priority, dueDate, categoryId, createdAt, updatedAt
categories: id, name, color, createdAt
```

### 여기까지 배운 Claude Code 기능 (Chapter 1~3 복습)

| 기능 | 배운 챕터 |
|------|----------|
| 기본 명령어 (`/clear`, `/compact`, `/rewind`) | Chapter 1 |
| Plan 모드 (`Shift+Tab`) | Chapter 1 |
| `/init`, CLAUDE.md | Chapter 2 |
| Extended Thinking | Chapter 2 |
| Skills (커스텀 슬래시 명령) | Chapter 2 |
| 서브에이전트 | Chapter 2 |
| GitHub Actions 연동 | Chapter 3 |

### 이 챕터에서 새로 배울 기능

| 기능 | 배우는 섹션 | 설명 |
|------|-----------|------|
| 이미지 기반 피드백 | 4.1 | 스크린샷을 붙여넣어 UI 수정 요청 |
| 사용자 범위 Skills | 4.3 | `~/.claude/skills/`에 전역 스킬 생성 |
| 에이전트 정의 | 4.4 | `.claude/agents/`에 커스텀 에이전트 생성 |
| Hooks | 4.5 | 도구 실행 전후에 자동 스크립트 실행 |
| CLI 파이프 (`claude -p`) | 4.5 | 다른 도구의 출력을 Claude에 전달 |

### 핵심 개념 미리보기

이 챕터에서 배우게 될 Claude Code 고급 기능들입니다.

- **스킬(Skills)**: Claude의 동작을 확장하는 `SKILL.md` 파일. 프로젝트 범위(`.claude/skills/`)와 사용자 범위(`~/.claude/skills/`)로 나뉨
- **서브에이전트(Sub-agents)**: 특정 작업을 위임할 수 있는 독립된 AI 에이전트. `.claude/agents/`에 정의
- **Hooks**: Claude의 도구 실행 전후에 자동으로 스크립트를 실행하는 자동화 기능
- **CLI 파이프**: `claude -p` 명령으로 다른 도구의 출력을 Claude에 전달
- **이미지 피드백**: 스크린샷을 붙여넣어 UI 수정 요청

> **참고**: Claude Code의 스킬, 서브에이전트, Hooks에 대한 공식 문서는 [https://code.claude.com/docs](https://code.claude.com/docs)에서 확인할 수 있습니다.

### 다음 챕터 미리보기

Chapter 4를 완료하면, [Chapter 5: 슈퍼랩 — 프로처럼 Claude Code 사용하기](../chapter-05/README.md)에서 이 모든 기능을 **Boris Tane의 프로 워크플로**(Research → Plan → Annotate → Implement)로 통합하여 통계 대시보드를 처음부터 끝까지 만드는 종합 실습을 수행합니다.

---

준비가 되었다면 [4.1 UI 개선](01-ui-refinement.md)부터 시작하겠습니다.
