---
layout: docs
title: "Lab 02. Nginx + PHP-FPM + PostgreSQL 3-Tier 컴포즈 환경 구축"
---

# 🌐 Lab 02. Nginx + PHP-FPM + PostgreSQL 3-Tier 컴포즈 환경 구축
## 도커 컴포즈 선언적 오케스트레이션, FastCGI 프록시 및 소스코드 핫리로드

> **"웹서버(Nginx), WAS(PHP-FPM), 데이터베이스(PostgreSQL)를 명령어 단 한 줄로 동시에 띄우고 실시간 코딩을 시작합니다!"**  
> Lab 01에서 제작한 PHP 8.5 FPM 컨테이너는 웹 요청을 직접 받지 못하고 FastCGI 프로토콜(포트 9000)을 사용합니다. 본 실습에서는 앞단에 **Nginx 웹서버**를 배치하여 정적 파일을 초고속 서빙하고 동적 PHP 요청을 WAS로 프록시 패스하며, 뒷단에는 영구 데이터를 저장할 **PostgreSQL** 컨테이너를 연결하는 **3-Tier 웹 아키텍처**를 구축합니다.

---

## 🎯 학습 목표

1. **Nginx 가상 호스트 설정 파일(`default.conf`)**을 작성하여 라라벨 라우팅 규칙(`try_files`)과 FastCGI(`fastcgi_pass was:9000`)를 연동할 수 있습니다.
2. `compose.yaml`을 통해 3개 서비스(`web`, `was`, `pgsql`)를 **격리된 도커 내부 브리지 네트워크**로 묶고 서비스 이름(DNS)으로 통신합니다.
3. 호스트 PC의 소스코드 디렉토리를 **바인드 마운트(Bind Mount)**하여, 코드를 저장하면 컨테이너 재빌드 없이 웹 브라우저에 즉시 반영되는 핫리로드 개발자 경험(DX)을 구현합니다.
4. **도커 볼륨(Named Volume)**을 지정하여 PostgreSQL 컨테이너를 삭제하고 다시 띄워도 데이터가 보존되는 영속성을 검증합니다.

---

## 📂 프로젝트 디렉토리 구성

작업 디렉토리(`~/docker-laravel-project`)에 Nginx 설정 폴더를 추가합니다:

```bash
mkdir -p docker/nginx
```

전체 구조:
```
docker-laravel-project/
├── compose.yaml              # 3-Tier 복합 컨테이너 선언 파일
├── .env                      # 데이터베이스 암호 및 환경 변수
├── docker/
│   ├── nginx/
│   │   └── default.conf      # Nginx FastCGI 라우팅 설정
│   └── php/
│       ├── Dockerfile        # (Lab 01에서 작성)
│       └── local.ini         # (Lab 01에서 작성)
└── src/                      # 라라벨 소스코드 (호스트 ↔ 컨테이너 바인드 마운트)
    └── public/
        └── index.php         # 웹 진입점
```

---

## 📝 1. Nginx 가상 호스트 설정 (`docker/nginx/default.conf`)

`docker/nginx/default.conf` 파일을 작성합니다. 라라벨의 단일 진입점인 `public/index.php`로 모든 트래픽을 집중시키고, PHP 요청을 WAS 컨테이너의 9000 포트로 포워딩합니다.

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;

    index index.php index.html;
    charset utf-8;

    # 1. 정적 파일 캐싱 및 로그 기록 제외 (성능 향상)
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    # 2. 라라벨 프론트 컨트롤러 라우팅 규칙
    # 요청된 파일이 존재하면 직접 서빙, 없으면 index.php로 전달
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # 3. PHP-FPM FastCGI 프록시 패스
    location ~ \.php$ {
        # 도커 내장 DNS에 의해 'was'라는 서비스명이 PHP 컨테이너의 내부 IP로 자동 해석됩니다.
        fastcgi_pass was:9000;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;

        # FastCGI 버퍼 최적화
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_read_timeout 60s;
    }

    # 4. 숨김 파일(.env, .git 등) 외부 접근 절대 차단
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

## 📄 2. 환경 변수 파일 작성 (`.env`)

민감한 데이터베이스 비밀번호를 코드에 하드코딩하지 않고 `.env` 파일로 분리합니다.

```env
# 애플리케이션 기본 설정
APP_NAME="도커 원정대 커뮤니티"
APP_ENV=local
APP_KEY=base64:YOUR_GENERATED_KEY_HERE
APP_DEBUG=true
APP_URL=http://localhost:8080

# PostgreSQL 연결 정보 (컨테이너 서비스명 'pgsql'이 DB 호스트가 됨)
DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=laravel_expedition
DB_USERNAME=dorothy
DB_PASSWORD=secret_docker_pass

# 외부 노출 포트
HTTP_PORT=8080
PGSQL_PORT=5432
```

---

## 🐳 3. `compose.yaml` 파일 작성

루트 디렉토리에 `compose.yaml`을 생성하고 Nginx, PHP-FPM, PostgreSQL 서비스를 선언합니다.

```yaml
services:
  # ========================================================
  # 1. Web Tier: Nginx 웹서버 (리버스 프록시)
  # ========================================================
  web:
    image: nginx:alpine
    container_name: expedition_web
    restart: unless-stopped
    ports:
      - "${HTTP_PORT:-8080}:80"
    volumes:
      # 라라벨 정적 파일 직접 서빙을 위한 소스코드 마운트
      - ./src:/var/www/html:ro
      # Nginx 가상호스트 설정 파일 마운트
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - was
    networks:
      - expedition_net

  # ========================================================
  # 2. App Tier: PHP 8.5 라라벨 WAS (PHP-FPM)
  # ========================================================
  was:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: expedition_was
    restart: unless-stopped
    environment:
      DB_CONNECTION: ${DB_CONNECTION}
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      DB_DATABASE: ${DB_DATABASE}
      DB_USERNAME: ${DB_USERNAME}
      DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      # [핵심] 호스트 코드를 컨테이너에 직결하여 핫리로드 구현
      - ./src:/var/www/html
    depends_on:
      pgsql:
        condition: service_healthy
    networks:
      - expedition_net

  # ========================================================
  # 3. Data Tier: PostgreSQL 16 데이터베이스
  # ========================================================
  pgsql:
    image: postgres:16-alpine
    container_name: expedition_pgsql
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_DATABASE:-laravel_expedition}
      POSTGRES_USER: ${DB_USERNAME:-dorothy}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret_docker_pass}
    ports:
      # 로컬 DBeaver나 DataGrip 등 DB 툴 접속용 포트 개방
      - "${PGSQL_PORT:-5432}:5432"
    volumes:
      # [핵심] 컨테이너가 죽어도 DB 데이터가 보존되는 영속 볼륨
      - pgsql_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME:-dorothy} -d ${DB_DATABASE:-laravel_expedition}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - expedition_net

# ========================================================
# 볼륨 및 네트워크 정의
# ========================================================
volumes:
  pgsql_data:
    driver: local

networks:
  expedition_net:
    driver: bridge
```

---

## 🚀 4. 테스트 페이지 작성 및 전체 인프라 기동

라라벨 기본 라우팅 구조를 흉내 낸 테스트 진입점을 `src/public/index.php`에 작성해 봅니다:

```bash
mkdir -p src/public
cat << 'EOF' > src/public/index.php
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>도커 원정대 3-Tier 환경 테스트</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light p-5">
    <div class="container text-center bg-white p-5 rounded-4 shadow-sm" style="max-width: 600px;">
        <h1 class="fw-bold mb-3">🧭 도커 원정대 인프라 가동 성공!</h1>
        <p class="text-secondary">Nginx와 PHP-FPM, PostgreSQL 컨테이너가 정상적으로 연결되었습니다.</p>
        <hr class="my-4">
        <ul class="list-group list-group-flush text-start mb-4">
            <li class="list-group-item d-flex justify-content-between">
                <span>서버 소프트웨어:</span> <strong><?= $_SERVER['SERVER_SOFTWARE'] ?? 'Nginx' ?></strong>
            </li>
            <li class="list-group-item d-flex justify-content-between">
                <span>PHP 버전:</span> <strong><?= phpversion() ?></strong>
            </li>
            <li class="list-group-item d-flex justify-content-between">
                <span>현재 WAS 컨테이너 호스트명:</span> <span class="badge bg-primary fs-6"><?= gethostname() ?></span>
            </li>
        </ul>
        <div class="alert alert-success fw-bold">
            🎉 축하합니다! 3-Tier 웹 환경이 성공적으로 동작하고 있습니다.
        </div>
    </div>
</body>
</html>
EOF
```

이제 도커 컴포즈 명령어로 3개 서비스를 백그라운드(`-d`)에서 일괄 실행합니다:

```bash
docker compose up -d --build
```

실행 상태를 확인합니다:

```bash
docker compose ps
```

**예상 출력 결과:**
```
NAME               IMAGE                     COMMAND                  SERVICE   CREATED         STATUS                   PORTS
expedition_pgsql   postgres:16-alpine        "docker-entrypoint.s…"   pgsql     2 seconds ago   Up 2 seconds (healthy)   0.0.0.0:5432->5432/tcp
expedition_was     docker-laravel-was        "docker-php-entrypoi…"   was       1 second ago    Up 1 second              9000/tcp
expedition_web     nginx:alpine              "/docker-entrypoint.…"   web       1 second ago    Up 1 second              0.0.0.0:8080->80/tcp
```

웹 브라우저를 열고 `http://localhost:8080`에 접속하면, 멋진 축하 메시지와 함께 현재 PHP 버전 및 WAS 컨테이너의 고유 호스트네임이 표시됩니다!

---

## ⚡ 5. 핫리로드(Hot-Reload) 개발자 경험 직접 체험하기

내 PC의 에디터에서 `src/public/index.php` 파일의 제목 문구를 수정하고 저장해 보세요:
- 예: `<h1>🧭 도로시와 지니의 도커 원정대 실습실!</h1>`

컨테이너를 재시작(`docker compose restart`)하거나 이미지를 다시 빌드(`build`)할 필요가 없습니다!  
웹 브라우저에서 **`F5`(새로고침)**만 누르면 변경 사항이 0.1초 만에 즉시 반영됩니다. 이것이 바로 도커 바인드 마운트가 주는 강력한 개발 생산성입니다.

---

## 🏁 Lab 02 요약 및 다음 단계

이제 웹 브라우저 ➔ **Nginx (포트 8080)** ➔ **PHP-FPM WAS (포트 9000)** 통신 파이프라인이 완벽히 구축되었습니다!

다음 실습에서는 이 인프라 위에 **라라벨 정식 프로젝트를 탑재하고, PostgreSQL 테이블 마이그레이션과 CRUD(방명록 등록·조회·삭제) 기능**을 직접 코딩하여 실제로 데이터가 영구 보존되는 완전한 웹사이트를 완성해 보겠습니다!

👉 **[다음 단계: Lab 03. PostgreSQL 연동 CRUD 웹 애플리케이션 완성](../03_crud_and_database/)**

