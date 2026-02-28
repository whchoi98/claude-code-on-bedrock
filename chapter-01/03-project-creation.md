# 1.3 Next.js 프로젝트 생성

> Claude Code를 사용하여 학습 트래커 프로젝트의 기반이 될 Next.js 15 프로젝트를 생성합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: `claude` 명령으로 대화 시작, 자연어로 프로젝트 생성

> ⭐ **난이도**: 쉬움

## 개요

이 섹션에서는 Claude Code에게 프롬프트를 주어 Next.js 15 프로젝트를 처음부터 생성합니다. Claude Code의 에이전틱 코딩 능력을 처음으로 체험하는 단계입니다. 프로젝트 생성 명령어 실행부터 초기 설정까지 Claude Code가 자동으로 처리합니다.

## 1단계: 프로젝트 디렉토리 생성 및 Claude Code 시작

터미널에서 프로젝트 디렉토리를 만들고 Claude Code를 시작합니다:

```bash
mkdir study-tracker && cd study-tracker
claude
```

Claude Code 인터페이스가 나타나면 준비가 완료된 것입니다.

## 2단계: 프로젝트 생성 프롬프트

Claude Code에 다음 프롬프트를 입력합니다:

→ *이 프롬프트는 Next.js 15 프로젝트를 원하는 설정으로 생성하기 위한 것입니다:*

```
Next.js 15 프로젝트를 App Router, TypeScript, Tailwind CSS, ESLint와 함께 생성해줘.
패키지 매니저는 pnpm을 사용하고, 프로젝트 이름은 "study-tracker"로 해줘.
현재 디렉토리에 초기화해줘 (하위 디렉토리가 아닌).
```

{% hint style="info" %}
Claude Code는 이 프롬프트를 받으면 `pnpm create next-app` 명령을 실행합니다. 명령 실행 전에 권한 승인을 요청할 수 있습니다. **Allow** 또는 **Yes**를 선택하여 진행하세요.
{% endhint %}

### Claude Code가 수행하는 작업

1. `pnpm create next-app@latest . --typescript --tailwind --eslint --app --use-pnpm` 실행
2. 프로젝트 초기화 옵션을 자동 선택
3. 의존성 패키지 설치

## 3단계: 프로젝트 구조 확인

프로젝트가 생성되면 Claude Code에 구조를 확인해 봅니다:

→ *이 프롬프트는 생성된 프로젝트의 파일 구조를 확인하기 위한 것입니다:*

```
프로젝트 구조를 보여줘. 어떤 파일들이 생성되었어?
```

다음과 유사한 구조가 생성되어야 합니다:

```
study-tracker/
├── app/
│   ├── favicon.ico
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── public/
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── .eslintrc.json
├── .gitignore
├── next-env.d.ts
├── next.config.ts
├── package.json
├── pnpm-lock.yaml
├── postcss.config.mjs
├── tailwind.config.ts
└── tsconfig.json
```

## 4단계: CLAUDE.md 초기화

프로젝트의 컨텍스트를 Claude Code에 알려주는 `CLAUDE.md` 파일을 생성합니다. Claude Code의 `/init` 명령어를 사용합니다:

→ *이 프롬프트는 프로젝트를 분석하여 CLAUDE.md 파일을 자동 생성하기 위한 것입니다:*

```
/init
```

`/init` 명령은 현재 프로젝트를 분석하여 빌드 시스템, 테스트 프레임워크, 코드 패턴 등을 자동으로 감지하고 `CLAUDE.md` 파일을 생성합니다.

생성된 `CLAUDE.md`를 확인한 후 다음 내용이 포함되어 있는지 확인하고, 없으면 추가합니다:

```markdown
# Study Tracker

## 프로젝트
- Next.js 15 App Router, TypeScript
- 패키지 매니저: pnpm
- 스타일링: Tailwind CSS

## 명령어
- 개발 서버: pnpm dev
- 빌드: pnpm build
- 린트: pnpm lint
```

{% hint style="info" %}
`CLAUDE.md`는 매 세션 시작 시 Claude Code가 자동으로 읽는 특수 파일입니다. 프로젝트의 규칙, 명령어, 코딩 스타일 등을 여기에 기록하면 Claude Code가 일관된 작업을 수행합니다.
{% endhint %}

## 5단계: 개발 서버 실행

프로젝트가 정상적으로 생성되었는지 개발 서버를 실행하여 확인합니다.

### Claude Code에서 직접 실행

→ *이 프롬프트는 개발 서버를 실행하여 프로젝트가 정상 동작하는지 확인하기 위한 것입니다:*

```
pnpm dev로 개발 서버를 실행해줘
```

### 또는 터미널에서 직접 실행

Claude Code에서 `Esc`를 눌러 나간 후:

```bash
pnpm dev
```

## 6단계: 브라우저에서 확인

웹 브라우저에서 [http://localhost:3000](http://localhost:3000)에 접속합니다.

Next.js 기본 페이지가 표시되면 프로젝트 생성이 성공한 것입니다.

## 7단계: Git 초기화

아직 Git이 초기화되지 않았다면 초기화합니다:

→ *이 프롬프트는 Git 리포지토리를 초기화하고 첫 커밋을 생성하기 위한 것입니다:*

```
Git 리포지토리를 초기화하고 "Initial Next.js 15 project setup" 메시지로 첫 커밋을 만들어줘.
```

Claude Code가 다음을 수행합니다:
1. `git init`
2. `.gitignore` 확인 (Next.js가 이미 생성)
3. `git add .`
4. `git commit -m "Initial Next.js 15 project setup"`

## 프롬프트 팁: 구체적으로 요청하기

Claude Code에 프롬프트를 작성할 때는 **구체적일수록 좋습니다**.

| 이렇게 해도 되지만 | 이렇게 하면 더 좋아요 |
|---------------------|-------------------|
| "프로젝트 만들어줘" | "Next.js 15 프로젝트를 App Router, TypeScript, Tailwind CSS, ESLint와 함께 생성해줘. 패키지 매니저는 pnpm을 사용해줘." |
| "서버 실행해" | "pnpm dev로 개발 서버를 실행해줘" |
| "코드 보여줘" | "app/layout.tsx와 app/page.tsx의 내용을 보여줘" |

## 요약

| 항목 | 상태 |
|------|------|
| Next.js 15 프로젝트 생성 | &#x2705; |
| CLAUDE.md 초기화 | &#x2705; |
| 개발 서버 동작 확인 | &#x2705; |
| Git 초기 커밋 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Next.js 개발 서버가 정상 실행됨 (`pnpm dev` 후 localhost:3000 접속 가능)
- [ ] CLAUDE.md 파일이 생성됨 (`/init` 실행 완료)
- [ ] Git 리포지토리가 초기화되고 첫 커밋이 생성됨

다음 섹션에서는 [Claude Code의 기본 명령어와 모드](04-claude-code-basics.md)를 학습합니다.
