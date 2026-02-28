# 사전 준비사항

이 워크샵을 원활하게 진행하기 위해 아래 항목들을 미리 준비해 주세요.

{% hint style="info" %}
이 워크샵은 **사전 구성된 Amazon EC2 인스턴스**에서 진행됩니다. 필요한 개발 도구(Node.js, Git, AWS CLI, PostgreSQL 등)가 미리 설치되어 있으며, 브라우저 기반 **VS Code Server**를 통해 접속합니다. 단, **AWS Bedrock 사용에는 토큰 기반 비용이 발생**할 수 있으므로 사전에 [비용 관리 팁](../appendix/README.md#비용-관리-팁)을 확인하시기 바랍니다.
{% endhint %}

---

## AWS 계정 요구사항

| 항목 | 설명 |
|------|------|
| AWS 계정 | Amazon Bedrock에 액세스 가능한 활성 AWS 계정 |
| IAM 권한 | IAM 사용자 또는 역할을 생성할 수 있는 관리자 수준 권한 |
| Bedrock 모델 액세스 | Claude 모델(Anthropic)에 대한 모델 액세스가 활성화되어 있어야 함 |
| 리전 | Bedrock Claude 모델이 지원되는 리전 (예: `us-east-1`, `us-west-2`) |

{% hint style="warning" %}
Bedrock 모델 액세스는 **리전별로 별도 활성화**해야 합니다. AWS 콘솔에서 **Amazon Bedrock > Model access**로 이동하여 Anthropic Claude 모델을 활성화하세요.
{% endhint %}

### IAM 정책 예시

Claude Code에서 Bedrock을 사용하려면 최소한 다음 IAM 권한이 필요합니다:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "arn:aws:bedrock:*::foundation-model/anthropic.*"
    }
  ]
}
```

---

## 개발 환경

이 워크샵은 사전 구성된 Amazon EC2 인스턴스에서 진행됩니다. VS Code Server가 설치되어 있으며, 브라우저를 통해 접속합니다.

### EC2 + VS Code Server (사전 구성)

워크샵 환경에는 다음 소프트웨어가 미리 설치되어 있습니다:

| 소프트웨어 | 설명 | 확인 명령어 |
|-----------|------|------------|
| **Node.js** | 18 이상 | `node --version` |
| **npm** | 패키지 관리자 | `npm --version` |
| **Git** | 버전 관리 | `git --version` |
| **PostgreSQL** | 로컬 데이터베이스 | `psql --version` |
| **AWS CLI v2** | AWS 서비스 접근 | `aws --version` |

### 접속 방법

1. 워크샵 진행자가 제공하는 **EC2 인스턴스 URL**을 브라우저에서 열기
2. VS Code Server 인터페이스가 로드되면 **터미널(Terminal)**을 열기
3. 터미널에서 아래 명령어로 환경이 정상적으로 구성되었는지 확인

```bash
# Node.js 버전 확인 (18 이상)
node --version

# npm 버전 확인
npm --version

# Git 버전 확인
git --version

# PostgreSQL 버전 확인
psql --version

# AWS CLI 버전 확인
aws --version
```

{% hint style="danger" %}
EC2 인스턴스에 직접 소프트웨어를 설치하거나 시스템 설정을 변경하지 마세요. 필요한 모든 도구는 사전 설치되어 있습니다.
{% endhint %}

---

## 필요한 계정

워크샵에서 사용하는 외부 서비스 계정을 미리 생성해 주세요.

### 1. GitHub 계정

- **URL**: [https://github.com](https://github.com)
- **용도**: 소스 코드 저장소, GitHub Actions CI/CD
- 이미 계정이 있다면 추가 설정 불필요

---

## 기본 지식

이 워크샵을 효과적으로 따라가기 위해 아래 영역에 대한 기본적인 이해가 필요합니다:

### TypeScript 기초

- 변수 타입 선언 (`string`, `number`, `boolean`)
- 인터페이스(interface) 및 타입(type) 정의
- 화살표 함수(arrow function), `async/await`

### React 기초

- 컴포넌트 개념 (함수형 컴포넌트)
- JSX 문법
- `useState`, `useEffect` 같은 기본 Hooks
- Props 전달

### 터미널/커맨드라인 사용 경험

- 디렉토리 이동 (`cd`), 파일 목록 확인 (`ls`)
- 패키지 설치 (`npm install`)
- 환경 변수 설정 (`export`)
- Git 기본 명령어 (`git add`, `git commit`, `git push`)

{% hint style="success" %}
**걱정하지 마세요!** TypeScript나 React에 익숙하지 않더라도 이 워크샵을 충분히 따라갈 수 있습니다. Claude Code가 코드 생성을 도와주기 때문에, 여러분은 **"무엇을 만들고 싶은지"**를 설명하는 것에 집중하면 됩니다.

워크샵을 진행하면서 자연스럽게 코드 패턴이 눈에 들어올 것입니다. 완벽히 이해하지 못하더라도 괜찮습니다 — 실습을 통해 점진적으로 익숙해집니다.
{% endhint %}

---

## 권장 사항

필수는 아니지만 아래 사항을 확인하면 워크샵 경험이 더 원활해집니다.

### AWS CLI 자격증명 확인

EC2 인스턴스에 AWS CLI가 이미 설치되어 있습니다. 자격증명이 올바르게 설정되어 있는지 확인하세요:

```bash
# 자격증명 확인
aws sts get-caller-identity

# 자격증명이 설정되지 않은 경우
aws configure
# AWS Access Key ID: [입력]
# AWS Secret Access Key: [입력]
# Default region name: us-east-1
# Default output format: json
```

### Claude Code 설치 확인

```bash
# Claude Code가 설치되어 있는지 확인
claude --version

# 설치되어 있지 않은 경우
npm install -g @anthropic-ai/claude-code
```

---

## 체크리스트

워크샵 시작 전 아래 항목을 모두 확인하세요:

- [ ] AWS 계정 준비 완료 (Bedrock 모델 액세스 활성화)
- [ ] EC2 인스턴스 접속 확인 (VS Code Server)
- [ ] Node.js 18+ 설치 확인
- [ ] Git 설치 확인
- [ ] PostgreSQL 설치 확인
- [ ] GitHub 계정 생성 완료
- [ ] AWS CLI 자격증명 설정 완료

모든 준비가 완료되면 [Chapter 1: 프로젝트 설정 & Claude Code 소개](../chapter-01/README.md)로 진행하세요.
