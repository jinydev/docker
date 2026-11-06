---
layout: docs
title: "Lab 03. PostgreSQL 연동 CRUD 웹 애플리케이션 완성"
---

# 📝 Lab 03. PostgreSQL 연동 CRUD 웹 애플리케이션 완성
## 라라벨 Eloquent ORM, 마이그레이션 및 도커 볼륨 데이터 영속성 검증

> **"도커 컨테이너를 껐다 켜면 데이터가 다 날아간다던데 정말인가요? 아니요, 도커 볼륨이 있으면 영원히 안전합니다!"**  
> 이번 실습에서는 라라벨의 **Eloquent ORM**과 **데이터베이스 마이그레이션**을 활용하여 실제 데이터가 저장되고 조회되는 **'도커 원정대 탐험 게시판(Expedition Board)'** 웹 애플리케이션을 완성합니다. 또한 컨테이너를 완전히 파괴(`down`)했다가 재생성해도 데이터가 완벽히 보존되는 도커 볼륨의 영속성을 직접 확인합니다.

---

## 🎯 학습 목표

1. 컨테이너 내부에서 라라벨 Artisan CLI(`php artisan migrate`)를 실행하여 PostgreSQL에 테이블 스키마를 프로비저닝합니다.
2. **Eloquent 모델**과 **라라벨 라우팅/컨트롤러**를 구현하여 데이터 등록(Create), 목록 조회(Read), 삭제(Delete) 기능을 완성합니다.
3. 게시글이 저장될 때 현재 요청을 처리한 **WAS 컨테이너의 고유 호스트네임(`gethostname()`)**을 함께 DB에 기록하도록 설계하여, 이후 스케일아웃 시 어떤 컨테이너가 일했는지 추적할 수 있도록 만듭니다.
4. `docker compose down`으로 컨테이너를 삭제한 뒤 다시 띄워도 데이터가 안전하게 유지되는 **볼륨의 영속성(Persistence)**을 증명합니다.

---

## 🏗️ 1. 라라벨 프로젝트 기본 파일 구성

호스트 머신에 PHP나 Composer가 없어도 도커 컨테이너를 통해 라라벨 프로젝트의 핵심 파일들을 구성할 수 있습니다. `src` 디렉토리에 라라벨 표준 구조의 필수 파일들을 생성합니다.

### 1) 데이터베이스 마이그레이션 파일 (`src/database/migrations/2026_01_01_000000_create_expedition_posts_table.php`)

```bash
mkdir -p src/database/migrations
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('expedition_posts', function (Blueprint $table) {
            $table->id();
            $table->string('author', 50);             // 작성자 (도로시, 지니 등)
            $table->string('character_role', 50);     // 원정대 역할
            $table->text('message');                  // 탐험 일지 내용
            $table->string('container_id', 50);       // 처리한 WAS 컨테이너 호스트명
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('expedition_posts');
    }
};
```

---

### 2) Eloquent 모델 생성 (`src/app/Models/ExpeditionPost.php`)

```bash
mkdir -p src/app/Models
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class ExpeditionPost extends Model
{
    protected $table = 'expedition_posts';

    protected $fillable = [
        'author',
        'character_role',
        'message',
        'container_id',
    ];
}
```

---

### 3) 웹 라우트 작성 (`src/routes/web.php`)

게시판 조회, 등록, 삭제를 담당하는 라우트를 정의합니다:

```bash
mkdir -p src/routes
```

```php
<?php

use Illuminate\Support\Facades\Route;
use Illuminate\Http\Request;
use App\Models\ExpeditionPost;

// 1. 방명록 목록 및 등록 화면
Route::get('/', function () {
    $posts = ExpeditionPost::orderBy('created_at', 'desc')->get();
    $currentHost = gethostname();
    return view('expedition', compact('posts', 'currentHost'));
});

// 2. 새 탐험일지 등록
Route::post('/posts', function (Request $request) {
    $request->validate([
        'author' => 'required|max:50',
        'character_role' => 'required|max:50',
        'message' => 'required|max:1000',
    ]);

    ExpeditionPost::create([
        'author' => $request->author,
        'character_role' => $request->character_role,
        'message' => $request->message,
        'container_id' => gethostname(), // [핵심] 현재 요청을 처리한 WAS 호스트명 기록
    ]);

    return redirect('/')->with('success', '새 탐험일지가 등록되었습니다!');
});

// 3. 탐험일지 삭제
Route::delete('/posts/{id}', function ($id) {
    ExpeditionPost::destroy($id);
    return redirect('/')->with('success', '탐험일지가 삭제되었습니다.');
});
```

---

### 4) 블레이드 뷰 템플릿 작성 (`src/resources/views/expedition.blade.php`)

사용자가 브라우저에서 볼 아름다운 부트스트랩 UI를 작성합니다:

```bash
mkdir -p src/resources/views
```

{% raw %}
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>도커 원정대 커뮤니티 보드</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background-color: #f8fafc; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        .hero-banner { background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%); color: white; border-radius: 1.5rem; }
        .post-card { border-radius: 1rem; border: 1px solid #e2e8f0; transition: transform 0.2s; }
        .post-card:hover { transform: translateY(-3px); box-shadow: 0 10px 15px -3px rgba(0,0,0,0.05); }
    </style>
</head>
<body class="py-5">
    <div class="container" style="max-width: 800px;">
        
        <!-- 상단 헤더 -->
        <div class="hero-banner p-4 p-md-5 mb-4 shadow">
            <span class="badge bg-primary px-3 py-2 rounded-pill fw-bold mb-2">Docker + Laravel + PostgreSQL</span>
            <h2 class="fw-bold mb-2">🧭 도커 원정대 탐험 일지 게시판</h2>
            <p class="text-light opacity-75 mb-3">Nginx가 FastCGI로 분기하고 PostgreSQL이 안전하게 영속 저장하는 실전 웹사이트</p>
            
            <div class="d-flex align-items-center gap-2 bg-dark bg-opacity-50 p-2 px-3 rounded-pill d-inline-flex border border-secondary">
                <span class="badge bg-success rounded-circle" style="width: 10px; height: 10px; padding: 0;">&nbsp;</span>
                <span class="small">현재 응답 컨테이너 ID: <strong>{{ $currentHost }}</strong></span>
            </div>
        </div>

        @if(session('success'))
            <div class="alert alert-success rounded-4 alert-dismissible fade show" role="alert">
                {{ session('success') }}
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        @endif

        <!-- 새 일지 등록 폼 -->
        <div class="card border-0 rounded-4 p-4 mb-4 shadow-sm bg-white">
            <h5 class="fw-bold mb-3" style="color: #1e293b;">✍️ 새 탐험일지 남기기</h5>
            <form action="/posts" method="POST">
                <div class="row g-3 mb-3">
                    <div class="col-md-6">
                        <label class="form-label small fw-bold">작성자 이름</label>
                        <input type="text" name="author" class="form-control rounded-3" placeholder="예: 도로시, 지니" required>
                    </div>
                    <div class="col-md-6">
                        <label class="form-label small fw-bold">원정대 역할</label>
                        <select name="character_role" class="form-select rounded-3" required>
                            <option value="주니어 메이커">주니어 메이커 (도로시)</option>
                            <option value="연구실 조교">연구실 수석 조교 (지니)</option>
                            <option value="총괄 멘토">총괄 멘토 (이호진 교수님)</option>
                            <option value="긍정 비타민">긍정 비타민 (토토)</option>
                            <option value="인프라 로봇">인프라 로봇 (티니)</option>
                        </select>
                    </div>
                </div>
                <div class="mb-3">
                    <label class="form-label small fw-bold">탐험 소감 및 메시지</label>
                    <textarea name="message" class="form-control rounded-3" rows="3" placeholder="도커 실습 소감을 자유롭게 공유해 주세요!" required></textarea>
                </div>
                <button type="submit" class="btn btn-primary rounded-pill px-4 py-2 fw-bold w-100">
                    🚀 탐험일지 등록하기
                </button>
            </form>
        </div>

        <!-- 일지 목록 피드 -->
        <h5 class="fw-bold mb-3" style="color: #1e293b;">📋 실시간 탐험 피드 (총 {{ count($posts) }}건)</h5>
        @forelse($posts as $post)
            <div class="card post-card p-3 mb-3 bg-white">
                <div class="d-flex justify-content-between align-items-center mb-2">
                    <div>
                        <strong class="fs-6 me-2">{{ $post->author }}</strong>
                        <span class="badge bg-secondary rounded-pill small">{{ $post->character_role }}</span>
                    </div>
                    <div class="text-end">
                        <span class="badge bg-light text-dark border me-2 small">기록 노드: {{ $post->container_id }}</span>
                        <small class="text-muted">{{ $post->created_at->diffForHumans() }}</small>
                    </div>
                </div>
                <p class="mb-2 text-secondary">{{ $post->message }}</p>
                <div class="text-end">
                    <form action="/posts/{{ $post->id }}" method="POST" onsubmit="return confirm('정말 삭제하시겠습니까?');">
                        <input type="hidden" name="_method" value="DELETE">
                        <button type="submit" class="btn btn-outline-danger btn-sm rounded-pill px-3">삭제</button>
                    </form>
                </div>
            </div>
        @empty
            <div class="text-center py-5 text-muted">
                아직 등록된 탐험일지가 없습니다. 첫 번째 주인공이 되어 보세요!
            </div>
        @endforelse

    </div>
</body>
</html>
```
{% endraw %}

---

## ⚡ 2. 데이터베이스 마이그레이션 실행

이제 컨테이너 내부에서 Artisan 명령어로 PostgreSQL 테이블을 생성합니다. 로컬 호스트 터미널에서 아래 명령을 실행합니다:

```bash
docker compose exec was php artisan migrate --force
```

**예상 출력 결과:**
```
INFO  Preparing database.
Creating migration table ................................... 18.25ms DONE
INFO  Running migrations.
2026_01_01_000000_create_expedition_posts_table ............ 24.11ms DONE
```

PostgreSQL 내부에 `expedition_posts` 테이블이 성공적으로 생성되었습니다!

---

## 🧪 3. 기능 테스트 및 데이터 영속성 검증

1. 웹 브라우저에서 **`http://localhost:8080`**에 접속합니다.
2. 새 탐험일지를 2~3개 작성해 등록합니다:
   - 도로시: *"Nginx와 PostgreSQL이 도커 내부 네트워크로 너무 잘 연결돼요!"*
   - 지니: *"호스트네임 배지에 현재 컨테이너 ID가 찍히는 게 신기해요."*
3. 화면에 등록된 글과 함께 `기록 노드: expedition_was`가 정상 출력되는지 확인합니다.

---

### 🔥 결정적 검증: 컨테이너 완전 파괴 후 재생성

이제 도커의 가장 강력한 기능인 **볼륨 영속성**을 시험해 봅니다. 컨테이너를 모조리 삭제(`down`)합니다:

```bash
# 모든 컨테이너와 네트워크 중지 및 삭제
docker compose down
```

`docker compose ps`를 쳐보면 실행 중인 컨테이너가 하나도 없습니다.  
이제 다시 서비스를 기동합니다:

```bash
docker compose up -d
```

웹 브라우저를 새로고침(`F5`)해 보세요!  
**방금 등록했던 게시글들이 단 1자도 유실되지 않고 그대로 화면에 출력됩니다.**  
`pgsql_data` 도커 볼륨이 호스트의 안전한 스토리지 영역에 저장되어 있기 때문에, 컨테이너가 소멸되어도 데이터는 영구히 살아남습니다.

---

## 🏁 Lab 03 요약 및 다음 단계

이제 여러분은 **실제로 데이터를 쓰고, 읽고, 지우는 완전한 풀스택 웹 애플리케이션**을 도커 환경에서 구동하는 데 성공했습니다!

하지만 실제 서비스에서 수천 명의 사용자가 동시에 글을 쓰면, 환영 이메일을 보내거나 알림을 처리하느라 WAS 컨테이너의 CPU와 메모리가 고갈될 수 있습니다.  
다음 실습에서는 이를 해결하기 위해 **Redis를 연동하고 백그라운드 큐 워커(Queue Worker) 컨테이너를 독립 분리**하여 웹 요청 처리 속도를 10배 이상 끌어올리는 아키텍처를 학습하겠습니다!

👉 **[다음 단계: Lab 04. Redis 캐시 & 비동기 큐 워커 분리](../04_redis_queue_worker/)**
