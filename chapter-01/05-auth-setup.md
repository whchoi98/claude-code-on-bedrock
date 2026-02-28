# 1.5 PostgreSQL 설치 및 설정

> EC2 인스턴스에 PostgreSQL을 설치하고 Todo 앱을 위한 데이터베이스를 생성합니다.

> 🎯 **이 섹션에서 배울 Claude Code 기능**: 자연어로 시스템 패키지 설치 및 설정

> ⭐⭐ **난이도**: 보통

## 개요

Todo 앱의 데이터를 저장할 PostgreSQL 데이터베이스를 EC2 인스턴스에 직접 설치합니다. 클라우드 DB 서비스 대신 로컬 PostgreSQL을 사용하여 외부 의존성을 최소화합니다.

{% hint style="info" %}
**용어 설명**
- **PostgreSQL**: 오픈소스 관계형 데이터베이스입니다. 안정성과 확장성이 뛰어나 널리 사용됩니다.
- **로컬 PostgreSQL**: 클라우드 서비스가 아닌, EC2 인스턴스에 직접 설치하여 사용하는 방식입니다. 외부 네트워크 의존성 없이 동작합니다.
{% endhint %}

## 1단계: PostgreSQL 설치

EC2(Amazon Linux 2023) 인스턴스에 PostgreSQL 15를 설치합니다.

Claude Code에 다음과 같이 요청할 수 있습니다:

> *이 프롬프트는 EC2 인스턴스에 PostgreSQL을 설치하기 위한 것입니다:*

```
EC2 Amazon Linux 2023에 PostgreSQL 15를 설치해줘.
```

또는 터미널에서 직접 실행합니다:

```bash
sudo dnf install postgresql15-server postgresql15 -y
```

{% hint style="info" %}
Amazon Linux 2023에는 `dnf` 패키지 매니저가 기본 제공됩니다. `yum` 대신 `dnf`를 사용합니다.
{% endhint %}

### 설치 확인

```bash
psql --version
```

PostgreSQL 15.x 버전이 출력되면 설치가 완료된 것입니다.

## 2단계: PostgreSQL 초기화 및 시작

데이터베이스를 초기화하고 서비스를 시작합니다:

```bash
sudo postgresql-setup --initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

{% hint style="info" %}
`systemctl enable` 명령은 EC2 인스턴스가 재시작될 때 PostgreSQL이 자동으로 시작되도록 설정합니다.
{% endhint %}

### 서비스 상태 확인

```bash
sudo systemctl status postgresql
```

`active (running)` 상태가 표시되면 정상적으로 실행 중입니다.

## 3단계: 데이터베이스 및 사용자 생성

Todo 앱 전용 데이터베이스와 사용자를 생성합니다. Claude Code에 요청할 수 있습니다:

> *이 프롬프트는 PostgreSQL에 애플리케이션 전용 데이터베이스와 사용자를 생성하기 위한 것입니다:*

```
PostgreSQL에 todo_app 데이터베이스를 생성하고, todouser 사용자를 비밀번호 'todopass'로 생성해줘.
해당 사용자에게 데이터베이스에 대한 모든 권한을 부여해줘.
```

또는 터미널에서 직접 실행합니다:

```bash
sudo -u postgres createdb todo_app
sudo -u postgres psql -c "CREATE USER todouser WITH PASSWORD 'todopass';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE todo_app TO todouser;"
sudo -u postgres psql -c "ALTER DATABASE todo_app OWNER TO todouser;"
```

{% hint style="warning" %}
워크샵 환경에서는 간단한 비밀번호를 사용하지만, 프로덕션 환경에서는 반드시 강력한 비밀번호를 사용하세요.
{% endhint %}

### 추가 권한 설정

Drizzle ORM이 스키마를 생성할 수 있도록 public 스키마에 대한 권한도 부여합니다:

```bash
sudo -u postgres psql -d todo_app -c "GRANT ALL ON SCHEMA public TO todouser;"
```

## 4단계: 인증 방식 설정

PostgreSQL의 기본 인증 방식을 `ident`에서 `md5`(비밀번호 인증)로 변경합니다:

```bash
sudo sed -i 's/ident/md5/g' /var/lib/pgsql/data/pg_hba.conf
sudo systemctl restart postgresql
```

{% hint style="info" %}
`ident` 인증은 OS 사용자와 PostgreSQL 사용자가 일치해야 합니다. `md5`로 변경하면 비밀번호를 사용한 인증이 가능해져, 애플리케이션에서 연결할 수 있습니다.
{% endhint %}

## 5단계: .env.local 환경변수 설정

프로젝트 루트에 `.env.local` 파일을 생성하여 데이터베이스 연결 정보를 설정합니다.

> *이 프롬프트는 데이터베이스 연결 환경변수를 설정하기 위한 것입니다:*

```
.env.local 파일을 생성하고 다음 내용을 추가해줘:

DATABASE_URL=postgresql://todouser:todopass@localhost:5432/todo_app
```

{% hint style="danger" %}
**중요**: `.env.local` 파일이 `.gitignore`에 포함되어 있는지 확인하세요. Next.js 프로젝트에서는 기본적으로 `.env.local`이 `.gitignore`에 포함됩니다.
{% endhint %}

현재 `.env.local` 파일의 전체 내용:

```env
# Database
DATABASE_URL=postgresql://todouser:todopass@localhost:5432/todo_app
```

## 6단계: 연결 테스트

데이터베이스에 정상적으로 연결되는지 확인합니다:

```bash
psql -U todouser -d todo_app -h localhost -c "SELECT 1;"
```

비밀번호를 입력하라는 프롬프트가 나타나면 `todopass`를 입력합니다.

```
 ?column?
----------
        1
(1 row)
```

위와 같은 결과가 출력되면 연결이 정상적으로 동작하는 것입니다.

{% hint style="info" %}
연결 오류가 발생하면 다음을 확인하세요:
- PostgreSQL 서비스가 실행 중인지: `sudo systemctl status postgresql`
- `pg_hba.conf` 변경 후 서비스를 재시작했는지: `sudo systemctl restart postgresql`
- 사용자와 데이터베이스가 생성되었는지: `sudo -u postgres psql -c "\du"` 및 `sudo -u postgres psql -c "\l"`
{% endhint %}

## 7단계: CLAUDE.md 업데이트

데이터베이스 설정 정보를 CLAUDE.md에 추가합니다:

> *이 프롬프트는 CLAUDE.md에 데이터베이스 설정 정보를 기록하여 이후 세션에서도 참조할 수 있게 하기 위한 것입니다:*

```
CLAUDE.md에 다음 데이터베이스 관련 정보를 추가해줘:

## 데이터베이스
- PostgreSQL 15 (EC2 로컬 설치)
- 데이터베이스: todo_app
- 사용자: todouser
- 연결 문자열: DATABASE_URL 환경변수 (.env.local)
```

## 요약

| 항목 | 상태 |
|------|------|
| PostgreSQL 15 설치 | &#x2705; |
| PostgreSQL 초기화 및 시작 | &#x2705; |
| todo_app 데이터베이스 생성 | &#x2705; |
| todouser 사용자 생성 및 권한 부여 | &#x2705; |
| 인증 방식 (md5) 설정 | &#x2705; |
| .env.local 환경변수 설정 | &#x2705; |
| 연결 테스트 | &#x2705; |

---

## ✅ 체크포인트

이 섹션을 완료하면 다음을 확인하세요:

- [ ] PostgreSQL 15가 설치되고 서비스가 실행 중 (`sudo systemctl status postgresql`)
- [ ] `todo_app` 데이터베이스가 생성됨
- [ ] `todouser` 사용자로 데이터베이스에 연결 가능 (`psql -U todouser -d todo_app -h localhost -c "SELECT 1;"`)
- [ ] `.env.local`에 `DATABASE_URL`이 설정됨
- [ ] CLAUDE.md에 데이터베이스 정보가 추가됨

다음 섹션에서는 [컨텍스트 창의 개념과 관리 방법](06-context-window.md)을 학습합니다.
