# 1.2 Claude Code 설치 및 Bedrock 연결

> Claude Code를 설치하고 Amazon Bedrock을 AI 제공자로 연결합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: `claude` 명령어, Bedrock 연결

> ⭐ **난이도**: 쉬움

## 개요

이 섹션에서는 Claude Code CLI를 설치하고, 환경변수를 통해 Amazon Bedrock에 연결합니다. 일반적인 Claude Code 사용과의 핵심 차이점은 `CLAUDE_CODE_USE_BEDROCK=1` 환경변수 설정입니다.

## 1단계: Claude Code 설치

운영체제에 맞는 방법을 선택하세요.

### Native Install (권장)

macOS 및 Linux (WSL 포함)에서 다음 명령어를 실행합니다:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

{% hint style="info" %}
Native Install은 백그라운드에서 자동 업데이트가 지원됩니다. 항상 최신 버전을 유지할 수 있어 권장되는 설치 방법입니다.
{% endhint %}

### Homebrew (macOS)

```bash
brew install --cask claude-code
```

{% hint style="warning" %}
Homebrew로 설치한 경우 자동 업데이트가 지원되지 않습니다. `brew upgrade claude-code` 명령으로 수동 업데이트가 필요합니다.
{% endhint %}

### 설치 확인

```bash
claude --version
```

버전 정보가 출력되면 설치가 완료된 것입니다.

## 2단계: Bedrock 연결 환경변수 설정

Claude Code가 Amazon Bedrock을 사용하도록 환경변수를 설정합니다.

### 필수 환경변수

```bash
# Bedrock 통합 활성화
export CLAUDE_CODE_USE_BEDROCK=1

# AWS 리전 설정 (필수)
export AWS_REGION=us-east-1
```

{% hint style="danger" %}
**중요**: `AWS_REGION`은 반드시 환경변수로 설정해야 합니다. Claude Code는 `.aws/config` 파일에서 리전 설정을 읽지 않습니다.
{% endhint %}

## 3단계: 모델 버전 고정 (권장)

모델 별칭(sonnet, haiku 등) 대신 **특정 모델 버전을 고정**하는 것을 강력히 권장합니다. 모델 별칭을 사용하면 Anthropic이 새 모델을 출시할 때 Bedrock 계정에서 아직 사용할 수 없는 모델로 전환을 시도하여 문제가 발생할 수 있습니다.

```bash
# 모델 버전 고정
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
```

{% hint style="info" %}
위 모델 ID는 교차 리전 추론 프로파일 ID입니다 (`us.` 접두사). 다른 리전 접두사나 애플리케이션 추론 프로파일을 사용하는 경우 적절히 조정하세요.
{% endhint %}

### Claude Code 기본 모델 참고

환경변수를 설정하지 않은 경우 Claude Code가 사용하는 기본 모델:

| 모델 유형 | 기본값 |
|-----------|--------|
| 기본 모델 (Primary) | `global.anthropic.claude-sonnet-4-6` |
| 소형/빠른 모델 (Small/Fast) | `us.anthropic.claude-haiku-4-5-20251001-v1:0` |

### 추가 모델 커스터마이징

더 세밀한 모델 설정이 필요한 경우:

```bash
# 추론 프로파일 ID로 직접 지정
export ANTHROPIC_MODEL='global.anthropic.claude-sonnet-4-6'
export ANTHROPIC_SMALL_FAST_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'

# 소형 모델의 리전을 별도 지정 (선택사항)
export ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION=us-west-2

# 애플리케이션 추론 프로파일 ARN 사용 (선택사항)
export ANTHROPIC_MODEL='arn:aws:bedrock:us-east-2:your-account-id:application-inference-profile/your-model-id'
```

## 4단계: 프롬프트 캐싱 설정

프롬프트 캐싱은 반복되는 대화에서 비용과 응답 시간을 줄여주는 기능입니다. 다만 일부 리전에서는 지원되지 않을 수 있습니다.

캐싱 관련 오류가 발생하면 다음 환경변수로 비활성화할 수 있습니다:

```bash
# 프롬프트 캐싱 비활성화 (필요한 경우에만)
export DISABLE_PROMPT_CACHING=1
```

{% hint style="info" %}
`us-east-1` 리전에서는 프롬프트 캐싱이 정상적으로 작동합니다. 다른 리전에서 문제가 발생하는 경우에만 비활성화하세요.
{% endhint %}

## 5단계: 환경변수 영구 설정

매번 환경변수를 설정하지 않도록 shell 프로필에 추가합니다.

{% hint style="tip" %}
어떤 셸을 사용하는지 모르겠다면 터미널에서 `echo $SHELL`을 실행해 보세요. `/bin/bash`면 Bash, `/bin/zsh`면 Zsh입니다.
{% endhint %}

### Bash 사용자 (~/.bashrc)

```bash
cat << 'EOF' >> ~/.bashrc

# Claude Code - Amazon Bedrock 설정
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
EOF

source ~/.bashrc
```

### Zsh 사용자 (~/.zshrc)

```bash
cat << 'EOF' >> ~/.zshrc

# Claude Code - Amazon Bedrock 설정
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
export ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
export ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
EOF

source ~/.zshrc
```

## 6단계: 연결 확인

모든 설정이 완료되었으면 Claude Code를 실행합니다:

```bash
claude
```

정상적으로 연결되면 Claude Code 인터페이스가 나타납니다. 간단한 프롬프트를 입력해 응답이 오는지 확인하세요:

```
Hello! Can you tell me which model you are using?
```

{% hint style="warning" %}
**Bedrock 모드 주의사항**:
- Bedrock을 사용할 때 `/login`과 `/logout` 명령은 비활성화됩니다. 인증은 AWS 자격증명을 통해 처리됩니다.
- 리전 관련 오류가 발생하면 `aws bedrock list-inference-profiles --region us-east-1` 명령으로 모델 가용성을 확인하세요.
- "on-demand throughput isn't supported" 오류가 나타나면 모델을 [추론 프로파일](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html) ID로 지정해야 합니다.
{% endhint %}

## 트러블슈팅

| 증상 | 해결 방법 |
|------|-----------|
| 리전 오류 | `aws bedrock list-inference-profiles --region us-east-1`로 확인 |
| 자격증명 오류 | `aws sts get-caller-identity`로 자격증명 유효성 확인 |
| 모델 접근 거부 | Bedrock 콘솔에서 모델 액세스 상태 확인 |
| "on-demand throughput isn't supported" | 추론 프로파일 ID 사용 (`us.anthropic.` 접두사) |
| 프롬프트 캐싱 오류 | `DISABLE_PROMPT_CACHING=1` 설정 |

## 요약

설정한 환경변수 정리:

```bash
# 필수
CLAUDE_CODE_USE_BEDROCK=1
AWS_REGION=us-east-1

# 권장 (모델 고정)
ANTHROPIC_DEFAULT_SONNET_MODEL='us.anthropic.claude-sonnet-4-6'
ANTHROPIC_DEFAULT_HAIKU_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'
```

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] `claude --version` 실행 시 버전 표시
- [ ] `claude` 실행 시 Claude Code 인터페이스 표시
- [ ] 간단한 질문에 응답 확인

다음 섹션에서는 [Claude Code를 사용하여 Next.js 프로젝트를 생성](03-project-creation.md)합니다.
