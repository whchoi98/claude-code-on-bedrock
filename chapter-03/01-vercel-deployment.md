# 3-1. GitHub 저장소 설정

이 섹션에서는 Todo 앱의 코드를 GitHub에 푸시하고, 저장소를 설정합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: Git 명령 실행 (커밋, 푸시, 원격 저장소 연결)

> ⭐ **난이도**: 쉬움

## 1단계: GitHub 리포지토리 생성 및 푸시

### Claude Code로 리포지토리 생성

터미널에서 Claude Code를 실행하여 GitHub 리포지토리를 생성합니다.

```
claude
```

Claude에게 다음과 같이 요청합니다:

→ *이 프롬프트는 GitHub 리포지토리를 생성하고 코드를 푸시하기 위한 것입니다:*

```
이 프로젝트의 GitHub 리포지토리를 생성하고 모든 코드를 push해줘. .env.local은 포함하지 마.
```

Claude가 자동으로 다음 작업을 수행합니다:

1. `.gitignore`에 `.env.local`이 포함되어 있는지 확인
2. `gh repo create todo-app --public --source=. --remote=origin` 명령으로 GitHub 리포지토리 생성
3. Git 초기화 및 첫 커밋
4. 원격 리포지토리에 푸시

### 수동으로 진행하는 경우

Claude Code 없이 직접 진행하려면 다음 명령을 실행합니다:

```bash
# .gitignore 확인 (.env.local이 포함되어 있는지)
cat .gitignore

# Git 초기화 (이미 되어있다면 건너뛰기)
git init

# GitHub 리포지토리 생성
gh repo create todo-app --public --source=. --remote=origin

# 첫 커밋 및 푸시
git add .
git commit -m "Initial commit: Todo App"
git push -u origin main
```

{% hint style="warning" %}
**주의**: `.env.local` 파일에는 데이터베이스 URL 등 민감한 정보가 포함되어 있습니다. 절대 GitHub에 푸시하지 마세요. `.gitignore`에 포함되어 있는지 반드시 확인하세요.
{% endhint %}

## 2단계: 저장소 확인

GitHub에서 코드가 올바르게 푸시되었는지 확인합니다:

- [ ] GitHub 리포지토리에 코드가 업로드되었는지 확인
- [ ] `.env.local`이 포함되지 않았는지 확인
- [ ] README.md가 프로젝트를 설명하는지 확인

## 3단계: 개발 서버 실행 확인

EC2에서 개발 서버가 정상 동작하는지 확인합니다:

```bash
pnpm dev
```

브라우저에서 VS Code Server 포트포워딩을 통해 접속하여 앱이 정상적으로 로드되는지 확인합니다.

### 트러블슈팅

개발 서버 실행 시 문제가 발생하면 다음을 확인합니다:

| 오류 | 원인 | 해결 |
|------|------|------|
| `Module not found` | 의존성 미설치 | `pnpm install` 실행 |
| `DATABASE_URL is not set` | 환경변수 누락 | `.env.local` 파일에 DATABASE_URL 설정 확인 |
| TypeScript 타입 에러 | 빌드 시 타입 검증 실패 | `pnpm build`로 확인 후 수정 |

{% hint style="success" %}
**완료!** Todo 앱의 코드가 GitHub에 푸시되었습니다. 다음 섹션에서는 GitHub Actions와 Claude Code를 설정하여 Issue 기반 자동 개발 환경을 구축합니다.
{% endhint %}

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] GitHub 리포지토리에 코드 푸시 완료
- [ ] `.env.local`이 포함되지 않음
- [ ] 개발 서버 정상 동작

---

**다음**: [3-2. GitHub Actions + Claude Code 설정 (Bedrock)](02-github-actions-setup.md)
