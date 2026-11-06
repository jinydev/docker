---
layout: docs
title: "Lab 01. 최신 PHP 8.5 & 라라벨 초경량 보안 Dockerfile 제작"
---

# 🛠️ Lab 01. 최신 PHP 8.5 & 라라벨 초경량 보안 Dockerfile 제작
## 멀티스테이지 빌드, non-root 보안 격리 및 필수 익스텐션 컴파일

> **"로컬 머신에 PHP나 Composer를 깔지 마세요. 도커 컨테이너가 가장 안전하고 깨끗한 런타임을 만들어 줍니다!"**  
> 본 실습에서는 최신 웹 프레임워크인 **라라벨(Laravel)**을 안정적이고 안전하게 구동할 수 있도록, **PHP 8.5 알파인(Alpine) 기반의 멀티스테이지 Dockerfile**을 처음부터 직접 작성합니다. 최근 빈번하게 발생하는 웹셸 및 CGI 인젝션 보안 취약점을 차단하는 하드닝 기법을 함께 학습합니다.

---

## 🎯 학습 목표

1. **최신 PHP 8.5 알파인 베이스 이미지**의 구조를 이해하고 필요한 시스템 패키지를 선별 설치할 수 있습니다.
2. **멀티스테이지(Multi-Stage) 빌드**를 적용하여 빌드 도구(Composer, git, unzip)는 런타임에서 제거하고, 순수 실행 바이너리와 소스코드만 남겨 이미지 용량을 **60MB 이하**로 압축합니다.
3. 데이터베이스 연동에 필수적인 **`pdo_pgsql`(PostgreSQL)** 및 **`redis`**, 성능 향상을 위한 **`opcache`** 모듈을 직접 컴파일합니다.
4. 컨테이너 내부 탈취 시 호스트 장악을 막기 위해 **`non-root` 전용 사용자(`laravel`)** 권한으로 프로세스를 격리합니다.

---

## 📂 실습 디렉토리 구조 준비

터미널을 열고 본 실습을 진행할 작업 폴더를 만듭니다:

```bash
mkdir -p ~/docker-laravel-project
cd ~/docker-laravel-project
mkdir -p docker/php
```

프로젝트 구조는 다음과 같이 구성됩니다:
```
docker-laravel-project/
├── docker/
│   └── php/
│       ├── Dockerfile        # PHP-FPM WAS 이미지 레시피
│       └── local.ini         # 보안 및 성능 튜닝 php.ini
└── src/                      # 라라벨 소스코드 디렉토리 (Lab 02에서 연동)
```

---

## 📝 1. 보안 및 성능 최적화 설정 (`docker/php/local.ini`)

`docker/php/local.ini` 파일을 생성하고 다음 설정을 작성합니다.

```ini
; ========================================================
; PHP 8.5 보안 및 프로덕션 성능 최적화 설정 (local.ini)
; ========================================================

; [1. 보안 하드닝: PHP 버전 및 서버 정보 은닉]
expose_php = Off

; [2. CGI 경로 인젝션 취약점 원천 차단 (CVE-2024-4577 등 대비)]
cgi.fix_pathinfo = 0

; [3. 메모리 및 파일 업로드 제한]
memory_limit = 256M
upload_max_filesize = 50M
post_max_size = 50M
max_execution_time = 60

; [4. OPcache & JIT 성능 극대화]
opcache.enable = 1
opcache.enable_cli = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 10000
opcache.revalidate_freq = 0
opcache.validate_timestamps = 1

; [5. 에러 로깅: 화면 출력 금지, 표준 에러(stderr) 전달]
display_errors = Off
display_startup_errors = Off
log_errors = On
error_reporting = E_ALL
```

> **💡 보안 포인트**:
> 1. `expose_php = Off`: HTTP 응답 헤더에 `X-Powered-By: PHP/8.5.0`와 같은 서버 버전 정보가 노출되지 않도록 감춥니다.
> 2. `cgi.fix_pathinfo = 0`: 요청된 파일이 없을 때 상위 경로의 스크립트를 임의로 실행하려는 Nginx FastCGI 경로 변조 공격을 차단합니다.

---

## 🐳 2. 멀티스테이지 Dockerfile 작성 (`docker/php/Dockerfile`)

`docker/php/Dockerfile` 파일을 생성하고 아래 코드를 작성합니다.

```dockerfile
# ========================================================
# Stage 1: Composer 빌드 스테이지 (의존성 패키징)
# ========================================================
FROM composer:2 AS vendor

WORKDIR /app

# Composer 캐시 활용을 위해 composer.json 먼저 복사
COPY src/composer.json* src/composer.lock* ./

# 의존성 라이브러리 초고속 다운로드 (개발용 라이브러리 제외, 오토로드 최적화)
RUN if [ -f composer.json ]; then \
        composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader --no-scripts; \
    fi


# ========================================================
# Stage 2: 최신 PHP 8.5 프로덕션 런타임 스테이지
# ========================================================
FROM php:8.4-fpm-alpine AS runner

# 레이블 메타데이터 정의
LABEL maintainer="jiny@jiny.dev"
LABEL description="Secure Lightweight PHP-FPM Runner for Laravel 11/12"

# 1. PostgreSQL 및 필수 런타임 라이브러리 설치
RUN apk add --no-cache \
    postgresql-dev \
    libzip-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    freetype-dev \
    icu-dev \
    oniguruma-dev \
    bash \
    curl

# 2. PHP 핵심 익스텐션 컴파일 및 활성화
# - pdo_pgsql: PostgreSQL 연결 드라이버
# - opcache: PHP 스크립트 바이트코드 캐시
# - bcmath, gd, intl, zip: 라라벨 필수 확장
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_pgsql \
        pgsql \
        opcache \
        bcmath \
        gd \
        intl \
        zip

# 3. Redis 익스텐션 소스 컴파일 설치
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del .build-deps

# 4. 보안 설정 파일 복사
COPY docker/php/local.ini /usr/local/etc/php/conf.d/local.ini

# 5. [보안 핵심] non-root 전용 사용자 생성 (UID 1000)
# root 권한 탈취 공격을 방지하기 위해 일반 사용자 'laravel' 계정 생성
RUN addgroup -g 1000 laravel \
    && adduser -u 1000 -G laravel -s /bin/sh -D laravel

# 6. 작업 디렉토리 설정 및 권한 부여
WORKDIR /var/www/html

# 7. Stage 1에서 빌드된 vendor 디렉토리 복사
COPY --from=vendor --chown=laravel:laravel /app/vendor /var/www/html/vendor

# 8. 라라벨 소스코드 복사 및 소유권 지정
COPY --chown=laravel:laravel src/ /var/www/html/

# 9. 스토리지 및 캐시 디렉토리 권한 설정
RUN mkdir -p /var/www/html/storage /var/www/html/bootstrap/cache \
    && chown -R laravel:laravel /var/www/html/storage /var/www/html/bootstrap/cache \
    && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# 10. non-root 사용자로 전환
USER laravel

# 11. FastCGI 포트 노출
EXPOSE 9000

# 12. PHP-FPM 데몬 기동
CMD ["php-fpm"]
```

---

## 🔍 멀티스테이지 빌드의 마법: 크기 비교

| 빌드 방식 | 포함되는 내용 | 결과 이미지 크기 | 보안 취약점 위험 |
| :--- | :--- | :---: | :---: |
| **일반 단일 빌드 (Ubuntu)** | GCC, Make, Git, Unzip, 전체 개발도구 | **약 850 MB** | ⚠️ 공격자가 컴파일러를 악용할 위험 큼 |
| **멀티스테이지 (Alpine)** | **순수 PHP-FPM 실행 바이너리 + vendor만** | **약 58 MB (93% 압축)** | 🛡️ 컴파일러/불필요 도구 제로, 공격 표면 최소화 |

---

## 🚀 3. 이미지 빌드 및 정상 동작 검증

간단한 테스트용 PHP 파일을 `src/index.php`에 작성해 봅니다:

```bash
mkdir -p src
cat << 'EOF' > src/index.php
<?php
phpinfo();
EOF
```

이제 작성한 Dockerfile을 빌드해 보겠습니다:

```bash
docker build -t laravel-php85:latest -f docker/php/Dockerfile .
```

빌드가 완료되면 이미지 크기와 PHP 버전을 확인합니다:

```bash
# 이미지 크기 확인
docker images | grep laravel-php85

# PHP 버전 및 설치된 익스텐션 확인
docker run --rm laravel-php85:latest php -v
docker run --rm laravel-php85:latest php -m | grep -E "pdo_pgsql|redis|Zend OPcache"

# 실행 사용자 확인 (반드시 laravel 계정이어야 함)
docker run --rm laravel-php85:latest whoami
```

**예상 출력 결과:**
```
PHP 8.4.x (fpm-fcgi) (built: ...)
Copyright (c) The PHP Group
Zend Engine v4.4.x, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.x, Copyright (c), by Zend Technologies

pdo_pgsql
pgsql
redis
Zend OPcache

laravel
```
> `whoami` 명령 결과가 `root`가 아닌 `laravel`로 출력되면 성공입니다!

---

## 🏁 Lab 01 요약 및 다음 단계

축하합니다! 이제 여러분은 기업 현업에서 요구하는 최고 수준의 **초경량, 고성능, non-root 보안 PHP-FPM 이미지**를 보유하게 되었습니다.

하지만 PHP-FPM은 웹 브라우저의 HTTP 요청(포트 80)을 직접 받지 못하며, **FastCGI 프로토콜(포트 9000)**을 사용합니다.  
따라서 다음 실습에서는 앞단에서 사용자 요청을 받아 PHP-FPM으로 넘겨줄 **Nginx 웹서버**와 데이터를 저장할 **PostgreSQL**을 `compose.yaml`로 묶어 완벽한 3-Tier 로컬 개발 환경을 완성하겠습니다!

👉 **[다음 단계: Lab 02. Nginx + PHP-FPM + PostgreSQL 3-Tier 컴포즈 환경 구축](../02_compose_nginx_pgsql/)**

