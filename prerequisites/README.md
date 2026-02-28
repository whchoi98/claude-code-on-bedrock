# 사전 준비사항

이 워크샵을 원활하게 진행하기 위해 아래 항목들을 미리 준비해 주세요.

{% hint style="info" %}
모든 외부 서비스 계정은 **무료 티어(Free Tier)**로 진행 가능합니다. 단, **AWS Bedrock 사용에는 토큰 기반 비용이 발생**할 수 있으므로 사전에 [비용 관리 팁](../appendix/README.md#비용-관리-팁)을 확인하시기 바랍니다.
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

### 필수 소프트웨어

| 소프트웨어 | 최소 버전 | 확인 명령어 |
|-----------|----------|------------|
| **Node.js** | 18 이상 | `node --version` |
| **npm** 또는 **pnpm** | npm 9+ / pnpm 8+ | `npm --version` 또는 `pnpm --version` |
| **Git** | 2.x | `git --version` |

### 운영체제 및 터미널

Claude Code는 다음 환경에서 동작합니다:

- **macOS** - Terminal 또는 iTerm2
- **Linux** - 기본 터미널
- **Windows** - WSL2 (Windows Subsystem for Linux) 필수

{% hint style="danger" %}
Windows의 cmd.exe 또는 PowerShell에서는 Claude Code가 직접 지원되지 않습니다. 반드시 **WSL2** 환경을 사용하세요.
{% endhint %}

### 설치 확인

아래 명령어로 필수 소프트웨어가 올바르게 설치되어 있는지 확인하세요:

```bash
# Node.js 버전 확인 (18 이상)
node --version

# npm 버전 확인
npm --version

# Git 버전 확인
git --version

# (선택) pnpm 설치 및 확인
npm install -g pnpm
pnpm --version
```

---

## 필요한 계정

워크샵에서 사용하는 외부 서비스 계정을 미리 생성해 주세요.

### 1. GitHub 계정

- **URL**: [https://github.com](https://github.com)
- **용도**: 소스 코드 저장소, GitHub Actions CI/CD
- 이미 계정이 있다면 추가 설정 불필요

### 2. Clerk 계정

- **URL**: [https://clerk.com](https://clerk.com)
- **용도**: 사용자 인증 (로그인/회원가입)
- **무료 티어**: 월 10,000 MAU (Monthly Active Users) 포함
- 가입 후 새 애플리케이션을 생성하고 API 키를 확인해 두세요

### 3. Neon 데이터베이스 계정

- **URL**: [https://neon.tech](https://neon.tech)
- **용도**: PostgreSQL 데이터베이스 (서버리스)
- **무료 티어**: 프로젝트 1개, 스토리지 0.5GB 포함
- 가입 후 새 프로젝트를 생성하고 Connection String을 확인해 두세요

### 4. Vercel 계정

- **URL**: [https://vercel.com](https://vercel.com)
- **용도**: Next.js 애플리케이션 배포
- **무료 티어**: Hobby 플랜으로 개인 프로젝트 배포 가능
- GitHub 계정으로 가입하면 저장소 연동이 편리합니다

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

{% hint style="info" %}
TypeScript나 React에 익숙하지 않더라도 Claude Code가 코드 생성을 도와주므로 워크샵을 따라갈 수 있습니다. 다만, 생성된 코드를 이해하는 데 기본 지식이 있으면 더 효과적입니다.
{% endhint %}

---

## 권장 사항

필수는 아니지만 아래 도구를 설치하면 워크샵 경험이 더 원활해집니다.

### VS Code + Claude Code 확장

Claude Code는 터미널에서 독립적으로 사용할 수 있지만, **VS Code 확장**을 함께 사용하면 에디터 내에서 바로 Claude Code를 실행할 수 있어 편리합니다.

```bash
# VS Code 설치 후 확장 마켓플레이스에서 "Claude Code" 검색하여 설치
# 또는 CLI로 설치
code --install-extension anthropic.claude-code
```

### AWS CLI v2 설치

AWS 자격증명 설정 및 Bedrock 연결 테스트에 AWS CLI가 필요합니다.

```bash
# macOS (Homebrew)
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# 설치 확인
aws --version
```

설치 후 자격증명을 설정합니다:

```bash
aws configure
# AWS Access Key ID: [입력]
# AWS Secret Access Key: [입력]
# Default region name: us-east-1
# Default output format: json
```

### Claude Code 설치

```bash
# npm으로 전역 설치
npm install -g @anthropic-ai/claude-code

# 설치 확인
claude --version
```

---

## 체크리스트

워크샵 시작 전 아래 항목을 모두 확인하세요:

- [ ] AWS 계정 준비 완료 (Bedrock 모델 액세스 활성화)
- [ ] Node.js 18+ 설치 완료
- [ ] Git 설치 완료
- [ ] GitHub 계정 생성 완료
- [ ] Clerk 계정 생성 완료
- [ ] Neon 데이터베이스 계정 생성 완료
- [ ] Vercel 계정 생성 완료
- [ ] (권장) AWS CLI v2 설치 완료
- [ ] (권장) VS Code + Claude Code 확장 설치 완료

모든 준비가 완료되면 [Chapter 1: 프로젝트 설정 & Claude Code 소개](../chapter-01/README.md)로 진행하세요.
