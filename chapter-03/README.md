# Chapter 3: 배포 및 GitHub Actions 자동화

이 챕터에서는 학습 트래커 앱을 프로덕션 환경에 배포하고, GitHub Actions와 Claude Code를 연동하여 Issue 기반 자동 개발 워크플로를 구축합니다.

## 학습 목표

- Vercel을 사용한 Next.js 앱 배포
- GitHub Actions에서 Claude Code를 Amazon Bedrock과 연동하여 사용
- Issue와 PR 코멘트로 Claude에게 자동 개발 요청
- CLAUDE.md를 활용한 프로젝트 규칙 자동 적용

## 사전 준비

- Chapter 2에서 완성한 학습 트래커 앱
- GitHub 계정
- Vercel 계정 ([vercel.com](https://vercel.com))
- AWS 계정 (Amazon Bedrock 접근 권한 필요)

## 챕터 구성

| 순서 | 제목 | 내용 |
|------|------|------|
| 3-1 | [Vercel 배포](01-vercel-deployment.md) | GitHub 리포지토리 생성, Vercel 프로젝트 import, 환경변수 설정, 배포 확인 |
| 3-2 | [GitHub Actions + Claude Code 설정](02-github-actions-setup.md) | GitHub App 생성, AWS OIDC 설정, IAM Role 구성, 워크플로 파일 작성 |
| 3-3 | [Issue 기반 자동 개발](03-issue-based-development.md) | Issue로 기능 요청, Claude 자동 구현, PR 리뷰 및 수정 요청 |

## 아키텍처 개요

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
        |
        v
Vercel 자동 배포 (main 브랜치 merge 시)
```

## 핵심 기술

- **Vercel**: Next.js에 최적화된 배포 플랫폼. Git push만으로 자동 배포
- **Claude Code GitHub Actions**: `@claude` 멘션으로 Issue 분석, 코드 구현, PR 생성을 자동화 ([공식 문서](https://code.claude.com/docs/en/github-actions))
- **Amazon Bedrock**: AWS 관리형 서비스로 Claude 모델 호출. OIDC를 통한 안전한 인증
- **CLAUDE.md**: 프로젝트 규칙과 코딩 컨벤션을 정의. GitHub Actions에서도 자동으로 읽어 적용

## 예상 소요 시간

약 45~60분
