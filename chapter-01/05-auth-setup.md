# 1.5 Clerk 인증 설정

> Clerk를 사용하여 학습 트래커에 사용자 인증(로그인/회원가입) 기능을 추가합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 자연어로 패키지 설치 및 코드 생성

> ⭐⭐ **난이도**: 보통

**Clerk**는 로그인/회원가입 기능을 쉽게 추가할 수 있는 인증 서비스입니다. 직접 인증 시스템을 만들지 않아도 됩니다.

## 개요

학습 트래커는 사용자별 데이터를 관리하므로 인증이 필수입니다. [Clerk](https://clerk.com)는 Next.js와 긴밀하게 통합되는 인증 서비스로, 미들웨어 기반 라우트 보호, 사전 구축된 UI 컴포넌트, 서버/클라이언트 양쪽에서의 사용자 정보 접근을 제공합니다.

## 1단계: Clerk 계정 및 애플리케이션 생성

### Clerk 계정 생성

1. [clerk.com](https://clerk.com)에 접속합니다.
2. **Sign up**을 클릭하여 계정을 생성합니다.
3. 대시보드에서 **Create application**을 클릭합니다.

### 애플리케이션 설정

1. **Application name**: `Study Tracker` (또는 원하는 이름)
2. **Sign-in options**: Email, Google 등 원하는 인증 방식을 선택합니다.
3. **Create application**을 클릭합니다.

## 2단계: API 키 복사

애플리케이션이 생성되면 **API Keys** 페이지로 이동합니다.

다음 두 개의 키를 복사합니다:
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`: 클라이언트에서 사용되는 공개 키
- `CLERK_SECRET_KEY`: 서버에서 사용되는 비밀 키

## 3단계: 환경변수 파일 생성

프로젝트 루트에 `.env.local` 파일을 생성합니다. Claude Code에 다음과 같이 요청할 수 있습니다:

→ *이 프롬프트는 인증에 필요한 환경변수 파일을 생성하기 위한 것입니다:*

```
다음 내용으로 .env.local 파일을 생성해줘:

NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_your-key-here
CLERK_SECRET_KEY=sk_test_your-key-here
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
```

{% hint style="danger" %}
**중요**: `.env.local` 파일에 실제 API 키를 입력한 후 반드시 `.gitignore`에 포함되어 있는지 확인하세요. Next.js 프로젝트에서는 기본적으로 `.env.local`이 `.gitignore`에 포함됩니다.
{% endhint %}

실제 키 값으로 교체하는 것을 잊지 마세요.

## 4단계: Clerk 설치 및 설정

Claude Code에 다음 프롬프트를 입력합니다:

→ *이 프롬프트는 Clerk 패키지를 설치하고 인증 관련 파일들을 자동 생성하기 위한 것입니다:*

```
Next.js 인증을 위해 Clerk를 설치해줘. middleware, 로그인/회원가입 페이지를 설정하고,
앱 라우트를 보호해줘. @clerk/nextjs 패키지를 사용해줘.

요구사항:
1. @clerk/nextjs 설치
2. 앱 레이아웃을 ClerkProvider로 감싸기
3. app/sign-in/[[...sign-in]]/page.tsx에 로그인 페이지 생성
4. app/sign-up/[[...sign-up]]/page.tsx에 회원가입 페이지 생성
5. 로그인/회원가입을 제외한 모든 라우트를 보호하는 middleware.ts 생성
```

### Claude Code가 수행하는 작업

Claude Code는 다음 작업을 자동으로 수행합니다:

1. **패키지 설치**: `pnpm add @clerk/nextjs` 실행
2. **레이아웃 수정**: `app/layout.tsx`에 `ClerkProvider` 추가
3. **로그인 페이지 생성**: `app/sign-in/[[...sign-in]]/page.tsx`
4. **회원가입 페이지 생성**: `app/sign-up/[[...sign-up]]/page.tsx`
5. **미들웨어 생성**: 프로젝트 루트에 `middleware.ts`

## 5단계: 생성된 파일 확인

Claude Code가 작업을 완료하면 주요 파일을 확인합니다.

### middleware.ts

→ *이 프롬프트는 생성된 미들웨어 파일의 내용을 확인하기 위한 것입니다:*

```
middleware.ts의 내용을 보여줘
```

미들웨어는 다음과 같은 구조여야 합니다:

```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/sign-in(.*)',
  '/sign-up(.*)',
]);

export default clerkMiddleware(async (auth, request) => {
  if (!isPublicRoute(request)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    '/(api|trpc)(.*)',
  ],
};
```

### app/layout.tsx

→ *이 프롬프트는 레이아웃에 ClerkProvider가 올바르게 추가되었는지 확인하기 위한 것입니다:*

```
app/layout.tsx의 내용을 보여줘
```

`ClerkProvider`로 전체 앱이 감싸져 있는지 확인합니다:

```typescript
import { ClerkProvider } from '@clerk/nextjs';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <ClerkProvider>
      <html lang="ko">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

### 로그인/회원가입 페이지

→ *이 프롬프트는 로그인/회원가입 페이지가 올바르게 생성되었는지 확인하기 위한 것입니다:*

```
로그인과 회원가입 페이지 파일을 보여줘
```

## 6단계: 인증 테스트

개발 서버를 실행하고 인증을 테스트합니다:

→ *이 프롬프트는 개발 서버를 실행하여 인증 기능이 정상 동작하는지 테스트하기 위한 것입니다:*

```
pnpm dev를 실행하고 서버가 준비되면 알려줘
```

또는 터미널에서 직접:

```bash
pnpm dev
```

### 테스트 순서

1. [http://localhost:3000](http://localhost:3000)에 접속합니다.
2. 미들웨어에 의해 `/sign-in` 페이지로 리다이렉트되는지 확인합니다.
3. 회원가입을 진행합니다 (이메일 또는 Google 등).
4. 로그인 후 메인 페이지(`/`)가 정상적으로 표시되는지 확인합니다.

{% hint style="info" %}
Clerk의 개발 모드에서는 테스트용 계정을 자유롭게 생성할 수 있습니다. 프로덕션 환경에서는 Clerk 대시보드에서 도메인 설정 등 추가 구성이 필요합니다.
{% endhint %}

## 7단계: 사용자 정보 활용 준비

나중에 데이터베이스 스키마에서 `userId` 필드를 사용하게 됩니다. Clerk에서 사용자 ID를 가져오는 방법을 미리 확인합니다.

### 서버 컴포넌트에서

```typescript
import { auth } from '@clerk/nextjs/server';

export default async function Page() {
  const { userId } = await auth();
  // userId를 사용하여 사용자별 데이터 조회
}
```

### 클라이언트 컴포넌트에서

```typescript
'use client';
import { useUser } from '@clerk/nextjs';

export default function Component() {
  const { user } = useUser();
  // user.id를 사용
}
```

## 8단계: CLAUDE.md 업데이트

인증 관련 정보를 CLAUDE.md에 추가합니다:

→ *이 프롬프트는 CLAUDE.md에 인증 설정 정보를 기록하여 이후 세션에서도 참조할 수 있게 하기 위한 것입니다:*

```
CLAUDE.md에 다음 인증 관련 정보를 추가해줘:

## 인증
- Clerk (@clerk/nextjs)를 사용한 인증
- middleware가 /sign-in과 /sign-up을 제외한 모든 라우트를 보호
- Server Components에서 userId를 가져오려면 @clerk/nextjs/server의 `auth()` 사용
- Client Components에서는 @clerk/nextjs의 `useUser()` 사용
- Clerk의 userId를 모든 사용자 관련 데이터베이스 테이블의 외래 키로 사용
```

## 요약

| 항목 | 상태 |
|------|------|
| Clerk 계정 및 애플리케이션 생성 | &#x2705; |
| API 키 설정 (.env.local) | &#x2705; |
| @clerk/nextjs 설치 | &#x2705; |
| 미들웨어 구성 | &#x2705; |
| 로그인/회원가입 페이지 생성 | &#x2705; |
| 인증 테스트 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] Clerk 계정이 생성되고 API 키가 `.env.local`에 설정됨
- [ ] `@clerk/nextjs` 패키지가 설치됨
- [ ] `middleware.ts`가 생성되어 라우트를 보호함
- [ ] 로그인/회원가입 페이지가 정상적으로 표시됨
- [ ] 로그인 후 메인 페이지에 접근 가능

다음 섹션에서는 [컨텍스트 창의 개념과 관리 방법](06-context-window.md)을 학습합니다.
