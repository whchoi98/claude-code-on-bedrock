# Chapter 3: GitHub 저장소 설정 및 GitHub Actions 자동화

이 챕터에서는 Todo 앱의 코드를 GitHub에 푸시하고, GitHub Actions와 Claude Code를 연동하여 Issue 기반 자동 개발 워크플로를 구축합니다.

## 학습 목표

- GitHub 저장소를 생성하고 코드를 푸시
- GitHub Actions에서 Claude Code를 Amazon Bedrock과 연동하여 사용
- Issue와 PR 코멘트로 Claude에게 자동 개발 요청
- CLAUDE.md를 활용한 프로젝트 규칙 자동 적용

## 사전 준비

- Chapter 2에서 완성한 Todo 앱
- GitHub 계정
- AWS 계정 (Amazon Bedrock 접근 권한 필요)

## 챕터 구성

| 순서 | 제목 | 내용 |
|------|------|------|
| 3-1 | [GitHub 저장소 설정](01-vercel-deployment.md) | GitHub 리포지토리 생성, 코드 푸시, 개발 서버 실행 확인 |
| 3-2 | [GitHub Actions + Claude Code 설정](02-github-actions-setup.md) | GitHub App 생성, AWS OIDC 설정, IAM Role 구성, 워크플로 파일 작성 |
| 3-3 | [Issue 기반 자동 개발](03-issue-based-development.md) | Issue로 기능 요청, Claude 자동 구현, PR 리뷰 및 수정 요청 |

### 이 챕터에서 배우는 Claude Code 기능

| 기능 | 배우는 섹션 | 설명 |
|------|-----------|------|
| Git 커밋/푸시 | 3.1 | Claude Code로 Git 명령 실행 |
| GitHub Actions 연동 | 3.2 | @claude로 자동 코드 생성 |
| Issue 기반 개발 | 3.3 | Issue에서 자동으로 PR 생성 |

## 아키텍처 개요

```
GitHub Issue → GitHub Actions → AWS 인증 → Claude Code (Bedrock) → PR 생성
```

<details>
<summary>상세 아키텍처 보기</summary>

```
GitHub Issue/PR 코멘트 (@claude)
        |
        v
GitHub Actions 워크플로 트리거
        |
        v
AWS OIDC 인증 (임시 자격증명)
        |
        v
Claude Code Action (Amazon Bedrock)
        |
        v
코드 구현 & PR 생성/업데이트
```

</details>

## 핵심 기술

- **Claude Code GitHub Actions**: `@claude` 멘션으로 Issue 분석, 코드 구현, PR 생성을 자동화 ([공식 문서](https://code.claude.com/docs/en/github-actions))
- **Amazon Bedrock**: AWS 관리형 서비스로 Claude 모델 호출. OIDC를 통한 안전한 인증
- **CLAUDE.md**: 프로젝트 규칙과 코딩 컨벤션을 정의. GitHub Actions에서도 자동으로 읽어 적용

### 다음 챕터 미리보기

Chapter 3를 완료하면, [Chapter 4: 고급 기능](../chapter-04/README.md)에서 이미지 기반 UI 피드백, 사용자 범위 Skills, 커스텀 에이전트, Hooks, CLI 파이프 등 Claude Code의 고급 기능을 학습합니다.

## 예상 소요 시간

약 45~60분
