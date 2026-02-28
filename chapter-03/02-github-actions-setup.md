# 3.2 GitHub Actions + Claude Code 설정 (Bedrock)

> GitHub Actions에서 Claude Code를 Amazon Bedrock과 연동하여 @claude 멘션으로 자동 코드 생성을 설정합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: GitHub Actions + Claude Code Action, AWS OIDC 인증

> ⭐⭐⭐ **난이도**: 어려움

{% hint style="info" %}
이 섹션은 설정 단계가 많아 복잡해 보일 수 있지만, **한 번만 설정하면** 이후부터 자동으로 동작합니다. 각 단계를 차근차근 따라가세요.
{% endhint %}

## 개요

[Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions)를 사용하면 GitHub Issue나 PR 코멘트에서 `@claude`를 멘션하여 코드 구현, PR 생성, 코드 리뷰 등을 자동화할 수 있습니다.

이 섹션에서는 **Amazon Bedrock**을 AI Provider로 사용하는 설정을 다룹니다. Anthropic API를 직접 사용하는 것과 달리, AWS OIDC 인증을 통해 안전하게 Bedrock에 접근합니다.

## 아키텍처

```
GitHub Issue/PR (@claude 멘션)
        │
        ▼
GitHub Actions 워크플로 트리거
        │
        ▼
┌───────────────────────────┐
│  1. Checkout repository    │
│  2. GitHub App Token 생성  │
│  3. AWS OIDC 인증          │
│  4. Claude Code Action     │
│     (use_bedrock: true)    │
└───────────────────────────┘
        │
        ▼
Amazon Bedrock (Claude 모델)
        │
        ▼
코드 구현 → PR 생성/업데이트
```

## 1단계: GitHub App 생성 (권장)

> 지금 하는 것: Claude Code Action이 리포지토리에 접근할 수 있도록 GitHub App을 생성합니다.

Bedrock 사용 시에는 수동 설정이 필요합니다. 먼저 커스텀 GitHub App을 생성합니다.

### GitHub App 생성

1. [https://github.com/settings/apps/new](https://github.com/settings/apps/new)에 접속합니다.
2. 기본 정보를 입력합니다:
   - **GitHub App name**: `StudyTracker Claude` (고유한 이름)
   - **Homepage URL**: 리포지토리 URL
3. **Webhooks**: "Active" 체크 해제 (필요 없음)
4. **Repository permissions** 설정:
   - **Contents**: Read & Write
   - **Issues**: Read & Write
   - **Pull requests**: Read & Write
5. **Create GitHub App** 클릭

### Private Key 생성

1. App 생성 후 설정 페이지에서 **Generate a private key** 클릭
2. `.pem` 파일이 다운로드됩니다
3. **App ID**를 메모합니다

### App 설치

1. 앱 설정 페이지에서 좌측 **Install App** 클릭
2. 본인의 계정/조직 선택
3. **Only select repositories** 선택 후 study-tracker 리포지토리 선택
4. **Install** 클릭

## 2단계: AWS OIDC Identity Provider 설정

> 지금 하는 것: GitHub Actions가 AWS 자격증명 없이도 Bedrock에 안전하게 접근할 수 있도록 OIDC 인증을 설정합니다.

GitHub Actions에서 AWS 자격증명 없이 안전하게 Bedrock에 접근하기 위해 OIDC를 설정합니다.

### OIDC Provider 생성

AWS 콘솔에서 **IAM > Identity providers > Add provider**로 이동합니다:

| 설정 | 값 |
|------|-----|
| Provider type | OpenID Connect |
| Provider URL | `https://token.actions.githubusercontent.com` |
| Audience | `sts.amazonaws.com` |

### IAM Role 생성

**IAM > Roles > Create role**에서 다음과 같이 설정합니다:

1. **Trusted entity type**: Web identity
2. **Identity provider**: `token.actions.githubusercontent.com`
3. **Audience**: `sts.amazonaws.com`
4. **Permissions**: 다음 커스텀 정책을 생성하여 연결

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream",
        "bedrock:ListInferenceProfiles"
      ],
      "Resource": [
        "arn:aws:bedrock:*:*:inference-profile/*",
        "arn:aws:bedrock:*:*:application-inference-profile/*",
        "arn:aws:bedrock:*:*:foundation-model/*"
      ]
    }
  ]
}
```

5. **Trust policy**에서 리포지토리를 제한합니다:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/study-tracker:*"
        }
      }
    }
  ]
}
```

{% hint style="warning" %}
`YOUR_ACCOUNT_ID`와 `YOUR_GITHUB_USERNAME/study-tracker`를 실제 값으로 교체하세요.
{% endhint %}

## 3단계: GitHub Secrets 설정

> 지금 하는 것: 워크플로에서 사용할 비밀 값(AWS Role ARN, GitHub App 키)을 리포지토리에 등록합니다.

리포지토리의 **Settings > Secrets and variables > Actions**에서 다음 시크릿을 추가합니다:

| Secret Name | 값 |
|-------------|-----|
| `AWS_ROLE_TO_ASSUME` | IAM Role ARN (예: `arn:aws:iam::123456789012:role/GitHubActionsBedrock`) |
| `APP_ID` | GitHub App ID (숫자) |
| `APP_PRIVATE_KEY` | `.pem` 파일의 전체 내용 |

## 4단계: 워크플로 파일 생성

> 지금 하는 것: @claude 멘션 시 자동으로 Claude Code를 실행하는 GitHub Actions 워크플로 파일을 생성합니다.

`.github/workflows/claude.yml` 파일을 생성합니다. Claude Code에 다음과 같이 요청할 수 있습니다:

→ *이 프롬프트는 GitHub Actions 워크플로 YAML 파일을 자동 생성하기 위한 것입니다:*

```
Amazon Bedrock과 연동되는 Claude Code GitHub Actions를 위한 .github/workflows/claude.yml을 생성해줘.
```

### 워크플로 파일 내용

```yaml
name: Claude Code

permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned]

jobs:
  claude:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    runs-on: ubuntu-latest
    env:
      AWS_REGION: us-east-1
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Generate GitHub App token
        id: app-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
          aws-region: us-east-1

      - name: Run Claude Code
        uses: anthropics/claude-code-action@v1
        with:
          github_token: ${{ steps.app-token.outputs.token }}
          use_bedrock: "true"
          claude_args: '--model us.anthropic.claude-sonnet-4-6 --max-turns 10'
```

### 핵심 설정 설명

| 설정 | 설명 |
|------|------|
| `permissions.id-token: write` | OIDC 토큰 요청에 필요 |
| `use_bedrock: "true"` | Bedrock을 AI Provider로 사용 |
| `--model us.anthropic.claude-sonnet-4-6` | Bedrock의 Cross-region inference profile ID |
| `--max-turns 10` | 최대 대화 턴 수 (비용 제어) |

## 5단계: 테스트

> 지금 하는 것: 설정이 올바르게 되었는지 실제로 @claude 멘션을 통해 테스트합니다.

워크플로 파일을 커밋하고 푸시합니다:

```bash
git add .github/workflows/claude.yml
git commit -m "Add Claude Code GitHub Actions workflow"
git push origin main
```

### 테스트 방법

1. GitHub에서 Issue를 생성합니다:
   - 제목: "테스트: Claude Code 연동 확인"
   - 본문: `@claude 이 리포지토리의 README.md를 읽고 프로젝트 구조를 요약해줘.`

2. 또는 기존 Issue/PR에 코멘트를 추가합니다:
   ```
   @claude 현재 프로젝트의 기술 스택을 분석해줘.
   ```

3. GitHub Actions 탭에서 워크플로가 트리거되는지 확인합니다.

{% hint style="info" %}
**첫 실행 시 시간이 소요될 수 있습니다.** Claude Code Action이 Node.js 환경을 설정하고 Claude Code를 설치해야 합니다.
{% endhint %}

## 트러블슈팅

### 워크플로가 트리거되지 않음

- 코멘트에 `@claude`가 포함되어 있는지 확인 (`/claude`가 아닌 `@claude`)
- GitHub App이 리포지토리에 설치되어 있는지 확인
- 워크플로 파일이 `main` 브랜치에 있는지 확인

### AWS 인증 오류

```
Error: Not authorized to perform sts:AssumeRoleWithWebIdentity
```

- Trust policy의 `sub` 조건에서 리포지토리 이름이 정확한지 확인
- OIDC Provider가 올바르게 설정되었는지 확인

### Bedrock 모델 액세스 오류

- AWS 계정에서 Bedrock 모델 액세스가 활성화되어 있는지 확인
- 워크플로의 `aws-region`이 모델이 활성화된 리전과 일치하는지 확인

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] GitHub App 생성 및 리포지토리에 설치 완료
- [ ] AWS OIDC Identity Provider 및 IAM Role 설정 완료
- [ ] GitHub Secrets (AWS_ROLE_TO_ASSUME, APP_ID, APP_PRIVATE_KEY) 등록 완료
- [ ] `.github/workflows/claude.yml` 파일 커밋 및 푸시 완료
- [ ] 테스트 Issue 생성 후 GitHub Actions 워크플로가 트리거되는지 확인

---

다음 섹션에서는 [Issue 기반 자동 개발](03-issue-based-development.md)을 실습합니다.
