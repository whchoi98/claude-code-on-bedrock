# 3-1. Vercel 배포

이 섹션에서는 학습 트래커 앱을 GitHub에 푸시하고 Vercel에 배포합니다. Vercel은 Next.js를 만든 회사에서 제공하는 배포 플랫폼으로, Git 연동을 통해 자동 배포를 지원합니다.

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
2. `gh repo create` 명령으로 GitHub 리포지토리 생성
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
gh repo create study-tracker --public --source=. --remote=origin

# 첫 커밋 및 푸시
git add .
git commit -m "Initial commit: Study Tracker app"
git push -u origin main
```

{% hint style="warning" %}
**주의**: `.env.local` 파일에는 데이터베이스 URL, API 키 등 민감한 정보가 포함되어 있습니다. 절대 GitHub에 푸시하지 마세요. `.gitignore`에 포함되어 있는지 반드시 확인하세요.
{% endhint %}

## 2단계: Vercel 계정 연결 및 프로젝트 Import

### Vercel 가입 및 GitHub 연동

1. [vercel.com](https://vercel.com)에 접속합니다
2. **Sign Up** 클릭 후 **Continue with GitHub**를 선택합니다
3. GitHub 계정 인증을 완료합니다

### 프로젝트 Import

1. Vercel 대시보드에서 **Add New...** > **Project** 클릭
2. **Import Git Repository** 섹션에서 방금 생성한 리포지토리를 찾습니다
3. **Import** 클릭

### 프로젝트 설정

Import 화면에서 다음을 확인합니다:

| 설정 | 값 |
|------|-----|
| Framework Preset | Next.js (자동 감지) |
| Root Directory | `./` |
| Build Command | `next build` (기본값) |
| Output Directory | `.next` (기본값) |

{% hint style="info" %}
Vercel은 Next.js 프로젝트를 자동으로 감지하므로 별도의 빌드 설정 변경이 필요 없습니다.
{% endhint %}

## 3단계: 환경변수 설정

**Deploy** 버튼을 누르기 전에 환경변수를 설정해야 합니다. **Environment Variables** 섹션을 펼치고 다음 변수를 추가합니다:

| 변수명 | 값 | 설명 |
|--------|-----|------|
| `DATABASE_URL` | Neon 데이터베이스 연결 문자열 | `.env.local`의 값과 동일 |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk Publishable Key | Clerk 대시보드에서 확인 |
| `CLERK_SECRET_KEY` | Clerk Secret Key | Clerk 대시보드에서 확인 |

각 변수를 입력한 후 **Add** 버튼을 클릭합니다.

```
# .env.local에서 값을 확인할 수 있습니다
DATABASE_URL=postgresql://user:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

{% hint style="warning" %}
**중요**: 환경변수 값을 복사할 때 앞뒤 공백이 포함되지 않도록 주의하세요. 특히 `DATABASE_URL`에 불필요한 문자가 포함되면 데이터베이스 연결에 실패합니다.
{% endhint %}

환경변수 설정이 완료되면 **Deploy** 버튼을 클릭합니다.

## 4단계: 배포 확인

배포가 완료되면 (보통 1~2분 소요) Vercel이 자동으로 생성한 URL이 표시됩니다.

```
https://study-tracker-xxxxx.vercel.app
```

다음 항목을 확인합니다:

- [ ] 배포 URL에 접속하면 앱이 정상적으로 로드되는지
- [ ] Clerk 로그인/회원가입이 작동하는지
- [ ] 로그인 후 학습 세션 생성이 가능한지
- [ ] 생성한 데이터가 정상적으로 표시되는지

### 배포 실패 시 트러블슈팅

배포가 실패하면 Vercel 대시보드에서 **Deployments** 탭을 클릭하여 빌드 로그를 확인합니다.

**흔한 오류와 해결 방법:**

| 오류 | 원인 | 해결 |
|------|------|------|
| `Module not found` | 의존성 미설치 | `package.json` 확인 후 재배포 |
| `DATABASE_URL is not set` | 환경변수 누락 | Vercel 대시보드에서 환경변수 추가 |
| `CLERK_SECRET_KEY is invalid` | 잘못된 키 값 | Clerk 대시보드에서 키 재확인 |
| TypeScript 타입 에러 | 빌드 시 타입 검증 실패 | 로컬에서 `npm run build`로 확인 후 수정 |

## 5단계: Clerk 프로덕션 도메인 추가

Clerk 인증이 프로덕션 환경에서 정상 작동하려면 배포 도메인을 Clerk에 등록해야 합니다.

1. [Clerk 대시보드](https://dashboard.clerk.com)에 접속합니다
2. 해당 애플리케이션을 선택합니다
3. 좌측 메뉴에서 **Domains** 클릭
4. Vercel이 할당한 도메인을 추가합니다:
   ```
   study-tracker-xxxxx.vercel.app
   ```

{% hint style="info" %}
Clerk의 개발 모드(Development instance)에서는 localhost 외의 도메인도 허용되는 경우가 있습니다. 만약 인증에 문제가 있다면 Clerk의 프로덕션 인스턴스(Production instance) 전환을 고려하세요.
{% endhint %}

## 6단계: 자동 배포 확인

Vercel은 GitHub 리포지토리와 연동되어 **main 브랜치에 push할 때마다 자동으로 배포**됩니다.

### 자동 배포 동작 방식

```
코드 수정 → git push origin main → Vercel 자동 빌드 → 배포 완료
```

- **프로덕션 배포**: `main` 브랜치에 push하면 프로덕션 URL에 배포
- **프리뷰 배포**: PR을 생성하면 별도의 프리뷰 URL에 배포 (PR 코멘트에 URL 표시)

### 테스트

간단한 변경을 push하여 자동 배포를 확인합니다:

```bash
# 사소한 변경 (예: 페이지 제목 수정)
# 변경 후:
git add .
git commit -m "Test auto deployment"
git push origin main
```

Vercel 대시보드에서 새로운 배포가 시작되는 것을 확인할 수 있습니다.

{% hint style="success" %}
**완료!** 학습 트래커 앱이 Vercel에 배포되었습니다. 다음 섹션에서는 GitHub Actions와 Claude Code를 설정하여 Issue 기반 자동 개발 환경을 구축합니다.
{% endhint %}

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Vercel 배포 URL에서 앱 접속 확인
- [ ] 환경변수 설정 완료 (DATABASE_URL, Clerk 키 등)
- [ ] Clerk 로그인/회원가입이 정상 작동
- [ ] main 브랜치 push 시 자동 배포 동작 확인

---

**다음**: [3-2. GitHub Actions + Claude Code 설정 (Bedrock)](02-github-actions-setup.md)
