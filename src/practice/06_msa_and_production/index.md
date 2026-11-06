---
layout: docs
title: "Lab 06. 모놀리스에서 MSA 분리 & 프로덕션 배포"
---

# 🏛️ Lab 06. 모놀리스에서 MSA 분리 & 프로덕션 배포
## Database-per-Service 마이크로서비스 전환 및 클라우드 실서버 운영

> **"하나의 거대한 덩어리(Monolith)에서 각자의 데이터베이스를 가진 독립적인 마이크로서비스(MSA)로! 그리고 마침내 실제 클라우드 서버에 살아있는 사이트로 런칭합니다!"**  
> 실전 프로젝트의 마지막 관문입니다. Lab 05까지 완성한 3-Tier 모놀리스 시스템을 **API Gateway(Nginx)**를 전면에 두고, **'Auth(인증) 서비스'**와 **'Expedition(게시판) 서비스'**로 도메인 분리합니다. 각 서비스가 독립된 PostgreSQL 데이터베이스를 소유하는 **Database-per-Service 패턴**을 실현하고, 최종적으로 공인 도메인과 HTTPS(SSL)가 적용된 클라우드 실서버에 상용 배포하는 프로덕션 워크플로를 완성합니다.

---

## 🎯 학습 목표

1. **모놀리스의 한계(코드 충돌, 전체 배포 위험)**를 극복하기 위한 **마이크로서비스 아키텍처(MSA)**의 분리 원칙과 트레이드오프를 이해합니다.
2. **API 게이트웨이(Nginx)**를 구축하여 단일 진입점(포트 80/443)에서 URL 경로(`/api/auth/*`, `/*`)에 따라 서로 다른 백엔드 마이크로서비스로 라우팅합니다.
3. 데이터 결합도를 끊어내기 위해 서비스별로 독립된 데이터베이스(`pgsql_auth`, `pgsql_expedition`)를 운영하는 **Database-per-Service**를 도커로 구현합니다.
4. 실제 리눅스 클라우드 서버(Ubuntu/Debian VPS)에 코드를 배포하고, **Let's Encrypt 무료 SSL 인증서**를 발급받아 암호화된 `https://` 사이트로 정식 오픈합니다.

---

## 🏗️ MSA 분산 아키텍처 다이어그램

```
                             [ 외부 클라이언트 (웹 / 모바일) ]
                                            │
                                            ▼ (포트 80, 443)
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        [ L7 API Gateway : Nginx Alpine ]                               │
│                  • 단일 공인 IP / 도메인 (https://docker.jiny.dev)                      │
│                  • SSL/TLS Termination & 라우팅 분기                                   │
└───────────────────────────┬────────────────────────────────┬───────────────────────────┘
                            │                                │
    ┌───────────────────────┘                                └───────────────────────┐
    │ 경로: /api/auth/*                                                              │ 경로: /* (메인 웹 & 게시판)
    ▼                                                                                ▼
┌───────────────────────────────┐                                ┌───────────────────────────────┐
│     [ Auth Microservice ]     │                                │  [ Expedition Microservice ]  │
│  PHP 8.5 Laravel (was_auth)   │                                │  PHP 8.5 Laravel (was_board)  │
│  • 회원가입, 로그인, JWT 토큰   │                                │  • 탐험일지 등록/조회/삭제      │
└───────────────┬───────────────┘                                └───────────────┬───────────────┘
                │ (독립 연결)                                                    │ (독립 연결)
                ▼                                                                ▼
┌───────────────────────────────┐                                ┌───────────────────────────────┐
│   [ PostgreSQL Auth DB ]      │                                │  [ PostgreSQL Board DB ]      │
│   • Named Volume: auth_db     │                                │  • Named Volume: board_db     │
│   • 회원 테이블만 격리 저장     │                                │  • 게시글 데이터만 격리 저장   │
└───────────────────────────────┘                                └───────────────────────────────┘
```

---

## ⚙️ 1. API 게이트웨이 Nginx 라우팅 설정 (`docker/nginx/gateway.conf`)

Nginx가 똑똑하게 요청 주소를 보고 알맞은 백엔드 컨테이너로 패스하도록 설정합니다:

```nginx
resolver 127.0.0.11 valid=5s;

server {
    listen 80;
    server_name localhost;

    # 1. 인증 마이크로서비스 라우팅 (/api/auth/*)
    location /api/auth/ {
        # 'auth_was' 컨테이너의 FastCGI 포트로 전달
        set $auth_backend auth_was;
        fastcgi_pass $auth_backend:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /var/www/auth/public/index.php;
    }

    # 2. 메인 게시판 마이크로서비스 라우팅 (/*)
    location / {
        # 'board_was' 컨테이너의 FastCGI 포트로 전달
        set $board_backend board_was;
        fastcgi_pass $board_backend:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /var/www/board/public/index.php;
    }
}
```

---

## 🐳 2. MSA 전용 `compose.msa.yaml` 선언

서비스별로 WAS와 PostgreSQL을 완벽하게 분리하여 선언합니다:

```yaml
services:
  # ========================================================
  # 1. API Gateway (Nginx)
  # ========================================================
  gateway:
    image: nginx:alpine
    container_name: msa_gateway
    ports:
      - "80:80"
    volumes:
      - ./docker/nginx/gateway.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - auth_was
      - board_was
    networks:
      - msa_net

  # ========================================================
  # 2. Auth Domain (인증 서비스 & 독립 DB)
  # ========================================================
  auth_was:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: msa_auth_was
    environment:
      DB_CONNECTION: pgsql
      DB_HOST: auth_pgsql
      DB_PORT: 5432
      DB_DATABASE: auth_db
      DB_USERNAME: auth_user
      DB_PASSWORD: auth_password
    depends_on:
      auth_pgsql:
        condition: service_healthy
    networks:
      - msa_net

  auth_pgsql:
    image: postgres:16-alpine
    container_name: msa_auth_pgsql
    environment:
      POSTGRES_DB: auth_db
      POSTGRES_USER: auth_user
      POSTGRES_PASSWORD: auth_password
    volumes:
      - auth_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U auth_user -d auth_db"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - msa_net

  # ========================================================
  # 3. Board Domain (탐험일지 게시판 서비스 & 독립 DB)
  # ========================================================
  board_was:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: msa_board_was
    environment:
      DB_CONNECTION: pgsql
      DB_HOST: board_pgsql
      DB_PORT: 5432
      DB_DATABASE: board_db
      DB_USERNAME: board_user
      DB_PASSWORD: board_password
    depends_on:
      board_pgsql:
        condition: service_healthy
    networks:
      - msa_net

  board_pgsql:
    image: postgres:16-alpine
    container_name: msa_board_pgsql
    environment:
      POSTGRES_DB: board_db
      POSTGRES_USER: board_user
      POSTGRES_PASSWORD: board_password
    volumes:
      - board_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U board_user -d board_db"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - msa_net

volumes:
  auth_data:
  board_data:

networks:
  msa_net:
    driver: bridge
```

> **💡 MSA의 위력**:
> 이제 게시판 DB(`board_pgsql`)에 장애가 발생하여 뻗더라도, 사용자 인증 DB(`auth_pgsql`)는 완전히 독립된 컨테이너와 볼륨으로 격리되어 있으므로 로그인 기능은 정상 작동합니다. 이것이 바로 장애 격리(Fault Isolation)입니다!

---

## ☁️ 3. 실제 리눅스 클라우드 서버 프로덕션 배포 5단계

로컬에서 완성한 도커 프로젝트를 AWS EC2, GCP Compute Engine, 또는 저렴한 VPS(가상 개인 서버)에 상용 배포하는 표준 5단계 절차입니다.

### Step 1: 클라우드 서버 생성 및 도커 엔진 설치
Ubuntu 22.04 / 24.04 LTS 가상서버에 접속한 후 도커를 설치합니다:
```bash
# 최신 공식 Docker Engine 자동 설치 스크립트 실행
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

### Step 2: 클라우드 방화벽(보안 그룹) 개방
클라우드 콘솔의 방화벽 규칙에서 다음 포트를 오픈합니다:
- **`80` (HTTP)**: 웹 접속 및 Let's Encrypt 도메인 소유권 검증용
- **`443` (HTTPS)**: SSL 보안 암호화 통신용
- **`22` (SSH)**: 관리자 원격 접속용

### Step 3: 소스코드 복제 및 프로덕션 환경설정
```bash
git clone https://github.com/your-username/docker-laravel-project.git
cd docker-laravel-project
cp .env.example .env.production
```

### Step 4: 프로덕션 성능 캐싱 최적화
프로덕션 환경에서는 속도를 위해 라라벨의 설정, 라우트, 뷰 파일을 미리 바이트코드로 컴파일해 둡니다:
```bash
docker compose -f compose.yaml exec was php artisan config:cache
docker compose -f compose.yaml exec was php artisan route:cache
docker compose -f compose.yaml exec was php artisan view:cache
docker compose -f compose.yaml exec was php artisan migrate --force
```

### Step 5: Let's Encrypt 무료 SSL/TLS 인증서 발급
Certbot 컨테이너를 일회성으로 실행하여 무료 SSL 인증서를 자동 발급받습니다:
```bash
docker run -it --rm --name certbot \
  -v "/etc/letsencrypt:/etc/letsencrypt" \
  -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
  -p 80:80 \
  certbot/certbot certonly --standalone \
  -d yourdomain.com --agree-tos -m your-email@example.com --no-eff-email
```
발급된 `/etc/letsencrypt/live/yourdomain.com/` 인증서 경로를 Nginx 설정의 `ssl_certificate`에 바인드 마운트하면 자물쇠 표시가 뜨는 완벽한 **`https://` 보안 웹사이트**가 완성됩니다!

---

## 🏆 대단원의 완주: 수강생이 이룬 놀라운 도약!

```
[ 처음 시작할 때 ]
"도커가 뭔지, 리눅스가 뭔지도 잘 모르는 초보 대학생"
                        ▼
[ 1학기 마스터 ]
"PHP 8.5 & Nginx & PostgreSQL 3-Tier 웹사이트를 직접 구축하고 클라우드 서버에 배포한 주니어 메이커"
                        ▼
[ 2학기 최종 수료 ]
"대규모 트래픽 부하를 수평 스케일아웃하고, 모놀리스를 마이크로서비스(MSA)로 분리하며,
무중단 롤링 배포와 실시간 관제탑을 다루는 기업 현업 수준의 DevOps/SRE 엔지니어!"
```

여러분의 훌륭한 탐험을 응원합니다.  
지금 바로 **[Lab 01. PHP 8.5 라라벨 보안 Dockerfile 제작](../01_dockerfile_and_php85/)**부터 나만의 도커 원정대 웹사이트를 손수 구축해 보세요!

