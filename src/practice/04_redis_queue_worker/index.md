---
layout: docs
title: "Lab 04. Redis 캐시 & 비동기 큐 워커 컨테이너 분리"
---

# ⚡ Lab 04. Redis 캐시 & 비동기 큐 워커(Queue Worker) 컨테이너 분리
## 동기식 HTTP 지연 해소, Redis 브로커 및 백그라운드 워커 독립 구동

> **"사용자가 글을 등록할 때마다 이메일을 발송하고 대용량 로그를 분석하느라 화면이 3초간 멈춘다면? 사용자는 떠나갑니다!"**  
> 모놀리스 애플리케이션이 트래픽을 견디지 못하는 가장 큰 원인은 '웹 요청을 처리하는 스레드에서 무거운 작업까지 동기(Synchronous)로 처리'하기 때문입니다.  
> 이번 실습에서는 **Redis**를 작업 큐 브로커로 추가하고, 웹 요청 처리용 컨테이너(`was`)와 백그라운드 작업 처리용 컨테이너(`worker`)를 **동일한 이미지에서 역할만 다르게 분리 실행**하는 엔터프라이즈 비동기 처리 아키텍처를 구축합니다.

---

## 🎯 학습 목표

1. **동기식(Synchronous) 처리의 한계**와 **비동기 큐(Message Queue) 기반 아키텍처**의 원리를 이해합니다.
2. `compose.yaml`에 **Redis 컨테이너**를 추가하고 라라벨 큐 드라이버(`QUEUE_CONNECTION=redis`)를 연동합니다.
3. **'동일 이미지, 상이한 역할(Same Image, Different Roles)'** 패턴을 적용하여, 동일한 PHP 8.5 라라벨 이미지에서 하나는 `php-fpm`(웹 WAS), 다른 하나는 `php artisan queue:work`(백그라운드 워커)로 분리 기동합니다.
4. 게시글 등록 시 비동기 Job을 발행하고, 실시간 워커 로그(`docker compose logs -f worker`)를 통해 백그라운드에서 작업이 격리 처리되는 과정을 관찰합니다.

---

## 🏛️ 비동기 분리 전/후 아키텍처 비교

```
[ 기존: 동기식 처리 (사용자 화면 3초 대기) ]
Client ──> Nginx ──> WAS (글 저장 0.05초 + 이메일/알림 시뮬레이션 3.0초) ──> Client (응답 지연)

[ 개선: Redis 비동기 워커 분리 (사용자 화면 0.05초 즉시 완료) ]
Client ──> Nginx ──> WAS (글 저장 0.05초 + 작업 Redis 적재 0.001초) ──> Client (즉시 렌더링!)
                      │
                      ▼ (작업 큐 적재)
                   [ Redis Broker ]
                      │
                      ▼ (비동기 인출)
            [ Worker Container (php artisan queue:work) ]
            (백그라운드에서 여유롭게 3.0초 소요 작업 처리)
```

---

## 🐳 1. `compose.yaml`에 Redis 및 Worker 서비스 추가

기존 `compose.yaml` 파일에 `redis`와 `worker` 서비스를 추가합니다.

```yaml
services:
  web:
    # (Lab 02와 동일)
    image: nginx:alpine
    container_name: expedition_web
    ports:
      - "${HTTP_PORT:-8080}:80"
    volumes:
      - ./src:/var/www/html:ro
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - was
    networks:
      - expedition_net

  was:
    # (Lab 02와 동일 - 웹 요청 전담)
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: expedition_was
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

  # ========================================================
  # [신규] Background Queue Worker 컨테이너
  # - was와 완전히 동일한 이미지를 사용하지만 실행 명령(CMD)만 다름!
  # ========================================================
  worker:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    container_name: expedition_worker
    restart: unless-stopped
    command: php artisan queue:work redis --verbose --tries=3 --timeout=90
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
      redis:
        condition: service_healthy
      pgsql:
        condition: service_healthy
    networks:
      - expedition_net

  # ========================================================
  # [신규] Redis 7 인메모리 캐시 & 메시지 브로커
  # ========================================================
  redis:
    image: redis:7-alpine
    container_name: expedition_redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - expedition_net

  pgsql:
    # (Lab 02와 동일)
    image: postgres:16-alpine
    container_name: expedition_pgsql
    environment:
      POSTGRES_DB: ${DB_DATABASE:-laravel_expedition}
      POSTGRES_USER: ${DB_USERNAME:-dorothy}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret_docker_pass}
    volumes:
      - pgsql_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME:-dorothy} -d ${DB_DATABASE:-laravel_expedition}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - expedition_net

volumes:
  pgsql_data:
  redis_data:

networks:
  expedition_net:
    driver: bridge
```

---

## 📝 2. 라라벨 비동기 작업(Job) 작성

게시글 등록 시 백그라운드에서 알림을 시뮬레이션할 라라벨 Job 클래스를 만듭니다:

`src/app/Jobs/SendExpeditionAlert.php`:
```bash
mkdir -p src/app/Jobs
```

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class SendExpeditionAlert implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public $author;
    public $message;

    public function __construct(string $author, string $message)
    {
        $this->author = $author;
        $this->message = $message;
    }

    public function handle(): void
    {
        $workerHost = gethostname();
        
        // 시간이 다소 소요되는 외부 알림(Discord, Slack, Email) 전송 시뮬레이션
        Log::info("🚀 [비동기 큐 수신] {$this->author}님의 일지 접수 (처리 워커 노드: {$workerHost})");
        sleep(2); // 2초간 무거운 백그라운드 처리 시뮬레이션
        Log::info("✅ [알림 발송 완료] {$this->author}님에게 환영 메시지가 전송되었습니다.");
    }
}
```

---

## 🔄 3. 웹 라우트에서 비동기 Job 발행 연동

`src/routes/web.php`의 `POST /posts` 처리 로직에 `SendExpeditionAlert::dispatch()`를 추가합니다:

```php
use App\Jobs\SendExpeditionAlert;

Route::post('/posts', function (Request $request) {
    $request->validate([
        'author' => 'required|max:50',
        'character_role' => 'required|max:50',
        'message' => 'required|max:1000',
    ]);

    $post = ExpeditionPost::create([
        'author' => $request->author,
        'character_role' => $request->character_role,
        'message' => $request->message,
        'container_id' => gethostname(),
    ]);

    // [핵심] 2초 걸리는 무거운 알림 작업을 Redis 큐로 비동기 위임!
    SendExpeditionAlert::dispatch($request->author, $request->message);

    // 사용자는 2초를 기다리지 않고 0.05초 만에 즉시 응답을 받음!
    return redirect('/')->with('success', '새 탐험일지가 등록되었으며, 알림 발송 작업이 백그라운드 큐로 전달되었습니다!');
});
```

---

## 🚀 4. 전체 서비스 기동 및 비동기 동작 검증

1. 새로 추가된 `redis`와 `worker`를 포함하여 컴포즈를 재기동합니다:
   ```bash
   docker compose up -d
   ```

2. 컨테이너 4개가 모두 정상 기동되었는지 확인합니다:
   ```bash
   docker compose ps
   ```
   - `expedition_web` (Up)
   - `expedition_was` (Up)
   - `expedition_worker` (Up)
   - `expedition_pgsql` (Up healthy)
   - `expedition_redis` (Up healthy)

3. 새로운 터미널 창을 열고, 백그라운드 워커의 실시간 로그를 띄워둡니다:
   ```bash
   docker compose logs -f worker
   ```

4. 웹 브라우저(`http://localhost:8080`)에서 새 글을 등록해 봅니다.
   - **사용자 경험**: "등록하기" 버튼을 누르자마자 0.1초도 걸리지 않고 즉시 화면이 새로고침됩니다!
   - **워커 터미널 창**: 워커 컨테이너가 Redis에서 작업을 인출하여 백그라운드에서 로그를 남기며 작업을 완수하는 모습이 실시간 출력됩니다:
     ```
     [2026-11-06 08:35:10] local.INFO: 🚀 [비동기 큐 수신] 도로시님의 일지 접수 (처리 워커 노드: expedition_worker)
     [2026-11-06 08:35:12] local.INFO: ✅ [알림 발송 완료] 도로시님에게 환영 메시지가 전송되었습니다.
     ```

---

## 🏁 Lab 04 요약 및 다음 단계

이제 웹 요청을 담당하는 컨테이너(`was`)와 백그라운드 큐를 처리하는 컨테이너(`worker`)가 분리되어, 아무리 많은 알림 작업이 쌓여도 웹사이트가 버벅대거나 뻗지 않는 견고한 구조를 갖추었습니다.

하지만 블랙프라이데이나 수강신청처럼 **동시에 수천 명의 사용자가 메인 웹페이지에 몰려오면** 어떻게 될까요?  
단 하나의 `was` 컨테이너로는 CPU 한계에 부딪힙니다.  
다음 실습에서는 Nginx 뒤에 **PHP-FPM WAS를 5대로 수평 스케일아웃(Scale-Out)하고, 로드밸런싱을 통해 트래픽을 완벽하게 분산 처리**하는 기술을 실습해 보겠습니다!

👉 **[다음 단계: Lab 05. Nginx 부하분산 & WAS 5대 수평 스케일아웃](../05_scale_out_load_balancing/)**

