# 1.1 AWS 환경 설정

> Amazon Bedrock에서 Claude 모델을 사용하기 위한 AWS 환경을 구성합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: AWS 환경 설정 (Claude Code 기능 없음 - 준비 단계)

> ⭐ **난이도**: 쉬움

## 개요

Claude Code는 기본적으로 Anthropic API를 사용하지만, Amazon Bedrock을 통해서도 사용할 수 있습니다. 이 워크샵에서는 **Amazon Bedrock**을 AI 제공자로 사용합니다. 이 섹션에서는 Bedrock에서 Claude 모델에 접근하기 위한 AWS 환경을 설정합니다.

{% hint style="info" %}
이 워크샵은 **사전 구성된 EC2 인스턴스**에서 진행됩니다. AWS 자격증명은 인스턴스 역할(Instance Role)로 이미 구성되어 있을 수 있습니다. 이 경우 3단계(AWS 자격증명 설정)를 건너뛸 수 있습니다.
{% endhint %}

## 1단계: Amazon Bedrock 모델 액세스 활성화

### Bedrock 콘솔 접속

1. [AWS Management Console](https://console.aws.amazon.com/bedrock/)에 로그인합니다.
2. 리전을 **US East (N. Virginia) - us-east-1**로 선택합니다.
3. **Amazon Bedrock** 서비스로 이동합니다.

### 모델 액세스 요청

1. 좌측 메뉴에서 **Model access**를 클릭합니다.
2. **Modify model access**를 클릭합니다.
3. **Anthropic** 섹션에서 다음 모델을 활성화합니다:
   - `Claude Sonnet 4` (claude-sonnet-4-20250514)
   - `Claude Haiku` (claude-haiku-4-5-20251001)
4. **Next** -> **Submit**을 클릭합니다.

{% hint style="info" %}
처음 Anthropic 모델을 사용하는 경우 **use case details**를 제출해야 합니다. Bedrock 콘솔의 **Chat/Text playground**에서 Anthropic 모델을 선택하면 자동으로 양식 작성 화면이 나타납니다. 이 과정은 계정당 한 번만 필요합니다.
{% endhint %}

### 모델 액세스 확인

모델 액세스 상태가 **Access granted**로 표시되는지 확인합니다. 일반적으로 즉시 승인되지만, 경우에 따라 몇 분이 소요될 수 있습니다.

## 2단계: IAM 정책 구성

Claude Code가 Bedrock API를 호출하려면 적절한 IAM 권한이 필요합니다.

### IAM 정책 생성

1. [IAM 콘솔](https://console.aws.amazon.com/iam/)로 이동합니다.
2. **Policies** -> **Create policy**를 클릭합니다.
3. **JSON** 탭을 선택하고 다음 정책을 입력합니다:

이 정책을 아래 JSON 블록에서 그대로 복사하여 붙여넣으세요.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowModelAndInferenceProfileAccess",
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
    },
    {
      "Sid": "AllowMarketplaceSubscription",
      "Effect": "Allow",
      "Action": [
        "aws-marketplace:ViewSubscriptions",
        "aws-marketplace:Subscribe"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:CalledViaLast": "bedrock.amazonaws.com"
        }
      }
    }
  ]
}
```

4. 정책 이름을 `ClaudeCodeBedrockAccess`로 지정하고 **Create policy**를 클릭합니다.

{% hint style="warning" %}
위 정책은 모든 리전의 모든 추론 프로파일과 파운데이션 모델에 대한 접근을 허용합니다. 프로덕션 환경에서는 특정 추론 프로파일 ARN으로 Resource를 제한하는 것을 권장합니다.
{% endhint %}

### IAM 사용자/역할에 정책 연결

생성한 `ClaudeCodeBedrockAccess` 정책을 Claude Code를 사용할 IAM 사용자 또는 역할에 연결합니다:

1. **IAM** -> **Users** (또는 **Roles**)로 이동합니다.
2. 해당 사용자/역할을 선택합니다.
3. **Add permissions** -> **Attach policies directly**를 클릭합니다.
4. `ClaudeCodeBedrockAccess`를 검색하여 연결합니다.

{% hint style="info" %}
EC2 인스턴스에서 실행하는 경우, 인스턴스에 연결된 IAM 역할(Instance Profile)에 이 정책을 연결하면 됩니다. 별도의 Access Key 설정이 필요 없습니다.
{% endhint %}

## 3단계: AWS 자격증명 설정

Claude Code는 AWS SDK의 기본 자격증명 체인을 사용합니다. 다음 옵션 중 하나를 선택하세요.

{% hint style="info" %}
EC2 인스턴스에 IAM 역할(Instance Role)이 이미 연결되어 있다면 이 단계를 건너뛸 수 있습니다. `aws sts get-caller-identity` 명령으로 자격증명이 유효한지 확인하세요.
{% endhint %}

### Option A: AWS CLI 구성 (권장)

가장 간단한 방법입니다. AWS CLI가 설치되어 있다면:

```bash
aws configure
```

프롬프트에 따라 Access Key ID, Secret Access Key, 기본 리전을 입력합니다.

### Option B: 환경변수 (Access Key)

환경변수를 직접 설정합니다:

```bash
export AWS_ACCESS_KEY_ID=your-access-key-id
export AWS_SECRET_ACCESS_KEY=your-secret-access-key
export AWS_SESSION_TOKEN=your-session-token  # 임시 자격증명 사용 시
```

### Option C: SSO 프로필

AWS SSO를 사용하는 조직 환경인 경우:

```bash
aws sso login --profile=your-profile-name

export AWS_PROFILE=your-profile-name
```

### Option D: AWS Management Console 자격증명

```bash
aws login
```

자세한 내용은 [AWS 로그인 문서](https://docs.aws.amazon.com/signin/latest/userguide/command-line-sign-in.html)를 참고하세요.

### Option E: Bedrock API Keys

AWS 전체 자격증명 없이 간편하게 인증할 수 있는 방법입니다:

```bash
export AWS_BEARER_TOKEN_BEDROCK=your-bedrock-api-key
```

Bedrock API Keys에 대한 자세한 내용은 [AWS 블로그](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/)를 참고하세요.

## 4단계: 리전 선택

이 워크샵에서는 **us-east-1** 리전을 사용합니다.

```bash
export AWS_REGION=us-east-1
```

{% hint style="danger" %}
**중요**: Claude Code는 `AWS_REGION` 환경변수를 **반드시** 설정해야 합니다. `.aws/config` 파일에서 리전 설정을 읽지 않습니다.
{% endhint %}

### 자격증명 테스트

설정이 올바른지 확인합니다:

```bash
aws bedrock list-inference-profiles --region us-east-1
```

Claude 모델이 포함된 추론 프로파일 목록이 출력되면 설정이 완료된 것입니다.

## 요약

| 항목 | 상태 |
|------|------|
| Bedrock 모델 액세스 활성화 | &#x2705; |
| IAM 정책 생성 및 연결 | &#x2705; |
| AWS 자격증명 설정 | &#x2705; |
| 리전 설정 (us-east-1) | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] AWS 자격증명 테스트 성공 (`aws bedrock list-inference-profiles --region us-east-1` 실행 시 프로파일 목록 출력)
- [ ] Bedrock 모델 액세스 활성화 확인 (Bedrock 콘솔에서 Claude 모델 상태가 Access granted)

다음 섹션에서는 [Claude Code를 설치하고 Bedrock에 연결](02-claude-code-install.md)합니다.
