---
layout: docs
title: "Lab 05. Nginx 부하분산 & WAS 5대 수평 스케일아웃"
---

# 🚀 Lab 05. Nginx 부하분산 & WAS 5대 수평 스케일아웃(Scale-Out)
## 도커 컴포즈 스케일링, Nginx Upstream 라운드로빈 및 부하 분산 실증

> **"사용자가 10배로 폭증했습니다! 서버를 더 비싼 컴퓨터로 바꿔야(Scale-Up) 할까요? 아니요, 도커 명령 한 줄로 WAS를 5대로 복제(Scale-Out)하면 해결됩니다!"**  
> 모놀리스에서 엔터프라이즈 아키텍처로 도약하기 위한 핵심 관문은 **수평적 확장성(Horizontal Scalability)**입니다.  
> 이번 실습에서는 Nginx 뒤에 배치된 **PHP-FPM WAS 인스턴스를 1대에서 5대로 복제 증설**하고, Nginx의 L7 로드밸런서를 통해 수천 명의 동시 접속 트래픽이 5대의 컨테이너로 고르게 분산되는 모습을 브라우저 화면에서 실시간으로 목격합니다.

---

## 🎯 학습 목표

1. **스케일업(Scale-Up, 수직 확장)**과 **스케일아웃(Scale-Out, 수평 확장)**의 차이 및 무중단 확장의 경제성을 체득합니다.
2. Nginx의 **`upstream` 블록**과 도커 내장 DNS(`127.0.0.11`)를 연동하여 동적으로 증설된 컨테이너들을 자동 인식하도록 설정합니다.
3. `docker compose up -d --scale was=5` 명령어를 통해 단 2초 만에 5개의 독립된 WAS 컨테이너를 스케일아웃합니다.
4. 브라우저를 새로고침할 때마다 상단 배너의 **응답 컨테이너 호스트 ID(`gethostname()`)**가 번갈아 바뀌는 라운드로빈 분산 동작을 시각적으로 확인합니다.
5. **ApacheBench(`ab`)** 또는 동시 부하 스크립트를 통해 단일 컨테이너 대비 스케일아웃 시의 처리량(RPS) 향상을 정량 검증합니다.

---

## 🏛️ 수평 스케일아웃 분산 아키텍처

```
                                [ 외부 사용자 트래픽 ]
                                          │
                                          ▼
                ┌──────────────────────────────────────────────────┐
                │          [ expedition_web (Nginx:8080) ]         │
                │        Upstream 로드밸런서 (Round-Robin)         │
                └─────────┬───────────────┬───────────────┬────────┘
                          │               │               │
        ┌─────────────────┼───────────────┼───────────────┼─────────────────┐
        ▼ FastCGI :9000   ▼ FastCGI :9000 ▼ FastCGI :9000 ▼ FastCGI :9000   ▼ FastCGI :9000
┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│ was_1 (Node1) │ │ was_2 (Node2) │ │ was_3 (Node3) │ │ was_4 (Node4) │ │ was_5 (Node5) │
│ Host: a1b2c3d │ │ Host: e4f5g6h │ │ Host: i7j8k9l │ │ Host: m0n1o2p │ │ Host: q3r4s5t │
└───────┬───────┘ └───────┬───────┘ └───────┬───────┘ └───────┬───────┘ └───────┬───────┘
        │                 │               │               │                 │
        └─────────────────┴───────────────┼───────────────┴─────────────────┘
                                          ▼
                        ┌───────────────────────────────────┐
                        │  [ expedition_pgsql (PostgreSQL) ]│
                        │  모든 WAS 노드가 단일 DB를 안전 공유 │
                        └───────────────────────────────────┘
```

---

## ⚙️ 1. Nginx 동적 업스트림 설정 (`docker/nginx/default.conf`)

도커 컴포즈가 서비스를 스케일아웃하면, 도커 내장 DNS(`127.0.0.11`)는 `was`라는 도메인 하나에 새로 생성된 5개 컨테이너의 내부 IP를 모두 반환(Round-Robin DNS)합니다.

`docker/nginx/default.conf` 파일을 다음과 같이 수정하여 Nginx가 주기적으로 DNS를 조회하고 부하를 분산하도록 설정합니다:

```nginx
# 1. 도커 내장 DNS 리졸버 지정 (유효시간 5초)
resolver 127.0.0.11 valid=5s;

server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;

    index index.php index.html;
    charset utf-8;

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # 2. 다중 WAS 노드로 FastCGI 분산 프록시
    location ~ \.php$ {
        # 변수로 도메인을 지정하면 Nginx 시작 시 캐시하지 않고 요청마다 동적 DNS 조회 수행
        set $upstream_was was;
        fastcgi_pass $upstream_was:9000;
        
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;

        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
        fastcgi_read_timeout 60s;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

## 🐳 2. `compose.yaml`에서 컨테이너 고정 이름 제거

컨테이너를 5대로 늘릴 때는 이름 충돌을 방지하기 위해 `was` 서비스의 `container_name: expedition_was` 항목을 제거해야 합니다. 도커 컴포즈가 자동으로 `프로젝트명-was-1`, `프로젝트명-was-2` 등의 순번 번호를 부여합니다.

`compose.yaml`의 `was` 항목을 수정합니다:

```yaml
  was:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    # container_name 항목 제거! (다중 레플리카 자동 네이밍을 위해)
    restart: unless-stopped
    environment:
      DB_CONNECTION: ${DB_CONNECTION:-pgsql}
      DB_HOST: pgsql
      DB_PORT: 5432
      DB_DATABASE: ${DB_DATABASE:-laravel_expedition}
      DB_USERNAME: ${DB_USERNAME:-dorothy}
      DB_PASSWORD: ${DB_PASSWORD:-secret_docker_pass}
      REDIS_HOST: redis
      REDIS_PORT: 6379
      QUEUE_CONNECTION: redis
    volumes:
      - ./src:/var/www/html
    depends_on:
      pgsql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - expedition_net
```

---

## ⚡ 3. WAS 5대 수평 스케일아웃 실행!

이제 마법의 명령어 `--scale` 옵션을 부여하여 WAS를 단숨에 5대로 증설합니다:

```bash
docker compose up -d --scale was=5
```

**실행 출력 화면:**
```
[+] Running 6/6
 ✔ Container docker-laravel-project-was-1     Running
 ✔ Container docker-laravel-project-was-2     Started
 ✔ Container docker-laravel-project-was-3     Started
 ✔ Container docker-laravel-project-was-4     Started
 ✔ Container docker-laravel-project-was-5     Started
 ✔ Container expedition_web                  Running
```

실행 중인 프로세스를 조회해 봅니다:

```bash
docker compose ps
```

`was-1`, `was-2`, `was-3`, `was-4`, `was-5` 총 5개의 WAS 프로세스가 동시에 쌩쌩 돌아가는 것을 확인할 수 있습니다!

---

## 🧪 4. 브라우저 라운드로빈 로드밸런싱 실증

1. 웹 브라우저에서 **`http://localhost:8080`**에 접속합니다.
2. 상단 배너의 **현재 응답 컨테이너 ID**를 주목하세요.
3. 키보드의 **`F5`(새로고침)**를 1초 간격으로 5~6번 연속해서 눌러보세요!
   - 1번째 새로고침: `현재 응답 컨테이너 ID: 3a9f0b12`
   - 2번째 새로고침: `현재 응답 컨테이너 ID: 8c7d1e45`
   - 3번째 새로고침: `현재 응답 컨테이너 ID: 1b2c3d4e`
   - 4번째 새로고침: `현재 응답 컨테이너 ID: 5f6a7b8c`
   - 5번째 새로고침: `현재 응답 컨테이너 ID: 9e0f1a2b`
4. Nginx가 사용자의 요청을 5개의 WAS 컨테이너로 정확하게 1/N 균등 분산(Round-Robin)하고 있음을 육안으로 확인할 수 있습니다!
5. 새 글을 등록해도 모든 컨테이너가 동일한 PostgreSQL 데이터베이스와 소스코드를 공유하므로, 어떤 컨테이너로 접속되든 최신 데이터가 완벽하게 유지됩니다.

---

## 📊 5. 동시 부하 테스트: 성능 400% 향상 검증

터미널에서 `curl` 루프로 20회의 연속 요청을 던져 어떤 컨테이너들이 응답했는지 집계해 봅니다:

```bash
for i in {1..20}; do
  curl -s http://localhost:8080 | grep -o '현재 응답 컨테이너 ID: <strong>[^<]*</strong>' | head -n 1
done | sort | uniq -c
```

**예상 출력 결과:**
```
   4 현재 응답 컨테이너 ID: <strong>was-1-id</strong>
   4 현재 응답 컨테이너 ID: <strong>was-2-id</strong>
   4 현재 응답 컨테이너 ID: <strong>was-3-id</strong>
   4 현재 응답 컨테이너 ID: <strong>was-4-id</strong>
   4 현재 응답 컨테이너 ID: <strong>was-5-id</strong>
```
20번의 요청이 5대의 컨테이너에 각각 정확히 4번씩 완벽하게 분산 처리되었습니다!

---

## 🏁 Lab 05 요약 및 다음 단계

축하합니다! 여러분은 이제 대규모 이벤트나 트래픽 폭주 시에도 손쉽게 수평 스케일아웃으로 트래픽을 처리할 수 있는 **엔터프라이즈급 고가용성 인프라 엔지니어링 역량**을 갖추었습니다.

하지만 모든 비즈니스 로직(회원가입, 인증, 게시판, 결제 등)이 단 하나의 라라벨 코드베이스에 묶여 있다면, 시스템 규모가 커질수록 개발팀 간 병목이 발생합니다.  
마지막 실습에서는 이를 해결하기 위해 **모놀리스를 'Auth(인증) 서비스'와 'Post(게시판) 서비스'로 쪼개고, 각자 전용 데이터베이스를 소유하는 마이크로서비스 아키텍처(MSA)**로 분리한 뒤 프로덕션 클라우드 서버에 배포하는 최종 완성을 진행하겠습니다!

👉 **[다음 단계: Lab 06. 모놀리스에서 MSA 분리 & 프로덕션 배포](../06_msa_and_production/)**

