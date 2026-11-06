---
layout: docs
title: "도커를 정복하기 위한 원정대 (Docker Conquest Expedition)"
permalink: /character/
---

# 🧭 도커를 정복하기 위한 원정대 (Docker Expedition)

> **"명령어 암기를 넘어, 리눅스 커널 격리부터 마이크로서비스(MSA), 스웜과 쿠버네티스 오케스트레이션에 이르는 거대한 클라우드 대양을 정복하라!"**

도커(Docker)와 클라우드 네이티브의 세계는 리눅스 커널 네임스페이스부터 멀티스테이지 빌드, 도커 컴포즈, 고가용성 클러스터링에 이르기까지 깊고 방대한 시스템 공학 지식을 다룹니다.

본 강좌는 자칫 딱딱하고 복잡할 수 있는 인프라 엔지니어링 과정을 **이호진 교수님**, 호기심 넘치는 주니어 메이커 **도로시**, 츤데레 천재 수석 조교 냥이 **지니**, 지치지 않는 긍정 비타민 반려견 **토토**, 든든한 피지컬 AI 협동 로봇 **틴맨(Tinny)**, 그리고 푸른 바다를 누비는 수호 고래 **모비(Moby Dock)**가 함께 항해하는 흥미진진한 모험 서사로 풀어냅니다.

<div class="my-4 text-center">
  <div class="p-3 bg-white rounded-4 shadow-sm border">
    <img src="{{ '/img/docker_expedition_team.png' | relative_url }}" alt="도커 정복 원정대 단체 일러스트" class="img-fluid rounded-3" style="max-height: 480px; width: auto;" />
    <div class="mt-3">
      <span class="badge bg-primary px-3 py-2 rounded-pill me-1">지브리 2D 애니메이션 화풍</span>
      <span class="badge bg-dark px-3 py-2 rounded-pill me-1">도커 130강 마스터 클래스</span>
      <span class="badge bg-success px-3 py-2 rounded-pill">클라우드 네이티브 대항해</span>
    </div>
    <p class="text-secondary small mt-2 mb-0">▲ 클라우드 대양을 향해 출항하는 도커 정복 원정대 5인과 수호 고래 모비(Moby Dock)</p>
  </div>
</div>

---

## 🎨 세계관 및 비주얼 아트 디렉션 (Art Direction)

도로시 유니버스의 모든 강의용 일러스트와 교재 삽화는 [`dorothy/prompt.md`](file:///Users/hojin9/dev/projects/dorothy/prompt.md) 마스터 규격을 기반으로 제작되었습니다:

* **스튜디오 지브리 2D 애니메이션 감성**: 인위적인 3D CG나 과도한 금속성 광택을 배제하고, 따뜻하고 정감 있는 2~3단계 플랫 셀 채색(Flat Cel Shading)을 적용합니다.
* **다크 에스프레소 브라운 라인아트 (`#26211E`)**: 차가운 순수 검정선 대신 부드럽고 깊이감 있는 에스프레소 브라운 윤곽선으로 따스한 동화책 질감을 완성합니다.
* **무결점 순백색 배경 (`#FFFFFF`)**: 슬라이드와 마크다운 교재 어디에나 자연스럽게 녹아들도록 테두리 프레임이나 불필요한 UI 박스 없이 인물 발밑에만 부드러운 접지 그림자(`soft contact shadow`)를 둔 1:1 정방형 컷아웃(Cutout)으로 완성됩니다.
* **철저한 한국어/영어 중심 레이블**: 뭉개지는 외계어 대신 직관적인 시각적 은유(컨테이너 박스, 회로, 나침반, 배관)와 표준 코드/수학 기호만 사용합니다.

---

## 👥 원정대원 5인 + 수호 고래 모비 상세 프로필

### 1. 👨‍🏫 이호진 교수님 (Prof. Hojin) — 원리와 본질의 나침반

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #f1f5f9; border-left: 6px solid #1e3a8a;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_hojin.png' | relative_url }}" alt="이호진 교수님" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-primary px-3 py-1 rounded-pill">총괄 아키텍트 & 멘토</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">이호진 교수님 (Prof. Hojin)</h4>
    <p class="text-primary fw-semibold mb-2">"명령어를 외우지 말고, 커널이 프로세스를 어떻게 격리하는지 원리를 생각해보렴!"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 원정대의 총괄 지휘관이자 아키텍처 나침반 (Master Architect & Navigator)</li>
      <li><strong>연령 & 인상:</strong> <strong>40대 초반의 생기 있고 스마트한 동안</strong> (주름 없이 매끄러운 피부, 인자하고 총명한 갈색 눈, 격려하는 따뜻한 미소)</li>
      <li><strong>헤어 & 안경:</strong> 단정하게 가르마를 탄 세련된 숏헤어 (다크 차콜 흑발에 자연스러운 은빛 솔트앤페퍼), 스티브 잡스 스타일의 얇은 실버 라운드 메탈 와이어테 안경</li>
      <li><strong>시그니처 착장:</strong> 단정한 <strong>다크 차콜 네이비 단색 카라 반팔 폴로 셔츠</strong> (이름표나 로고 없는 깔끔한 솔리드), 중청 스트레이트 데님, 그레이 스웨이드 레트로 러닝화</li>
      <li><strong>주요 특기 & 역할:</strong> 시스템의 근본 원리를 꿰뚫어 보는 대가입니다. "왜 컨테이너가 가상머신보다 가벼운가?", "cgroups와 네임스페이스는 어떻게 자원을 묶는가?"와 같은 핵심 질문을 던지며 학생들을 진정한 클라우드 네이티브 아키텍트로 육성합니다.</li>
    </ul>
    <details class="small">
      <summary class="text-primary cursor-pointer fw-semibold">🎨 AI 마스터 프롬프트 보기</summary>
      <pre class="bg-white p-2 rounded-3 border mt-2"><code>Professor Hojin, a youthful and friendly Korean engineering professor in his early 40s (smooth skin, no wrinkles, energetic kind face), neatly parted stylish salt-and-pepper short black hair with subtle silver streaks, thin minimalist round metal wire-rim glasses, kind intelligent warm brown eyes, encouraging warm smile. Dressed in a neat solid dark charcoal navy short-sleeve collared polo shirt with NO name tag, NO badges, NO SNU text, classic medium-wash blue straight denim jeans, and iconic grey suede retro running sneakers with white soles</code></pre>
    </details>
  </div>
</div>

---

### 2. 👧 도로시 (Dorothy) — 열정 가득한 주니어 빌더

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #fff1f2; border-left: 6px solid #e11d48;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_dorothy.png' | relative_url }}" alt="도로시" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-danger px-3 py-1 rounded-pill">주니어 메이커 & 학습자</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">도로시 (Dorothy)</h4>
    <p class="text-danger fw-semibold mb-2">"교수님, 제 라라벨 웹사이트가 컨테이너 5대로 무중단 분산 배포됐어요!"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 두려움 없이 실습 코드를 작성하고 배포하는 행동 대장 (Lead Practitioner & Maker)</li>
      <li><strong>비율 & 연령:</strong> <strong>정확한 4.5등신 치비/주니어 비율</strong> (초등 저학년 약 8~9세), 통통하고 발그레한 피치 핑크 볼살, 반짝이는 호박 갈색(Warm Amber) 눈망울</li>
      <li><strong>헤어 & 의상:</strong> 밤색 웨이브 양갈래 머리(알록달록 구슬 방울 머리끈과 단정한 앞머리), 클래식 블루 데님 멜빵바지 (가슴 중앙에 원형 <strong>'STEM HERO'</strong> 뱃지), 주황색과 노란색 가로 줄무늬 반팔 티셔츠</li>
      <li><strong>상징 소품:</strong> <strong>새빨간 캔버스 운동화 (Ruby Red Sneakers)</strong> — 가상머신의 무거운 짐을 벗어던지고 가볍게 대양을 누비는 클라우드 민첩성의 상징</li>
      <li><strong>주요 특기 & 역할:</strong> 실패를 두려워하지 않는 호기심 대장입니다. 포트 충돌, 볼륨 마운트 에러로 좌충우돌하면서도 지니와 교수님의 도움을 받아 직접 Dockerfile을 빌드하고 MSA 클러스터를 완성하는 본 강좌의 주인공입니다.</li>
    </ul>
    <details class="small">
      <summary class="text-danger cursor-pointer fw-semibold">🎨 AI 마스터 프롬프트 보기</summary>
      <pre class="bg-white p-2 rounded-3 border mt-2"><code>Dorothy, an adorable 9-year-old anime girl, 4.5 heads tall junior proportions, wavy chestnut brown hair in twin pigtails tied with colorful round beads, cute bangs, warm amber-brown eyes, rosy cheeks, cheerful bright smile. Wearing classic medium blue denim overalls with rolled cuffs and a 'STEM HERO' round patch on the chest, orange and yellow horizontal striped t-shirt underneath, white ankle socks, vibrant ruby red canvas sneakers, clean dark brown line art, soft warm cel shading</code></pre>
    </details>
  </div>
</div>

---

### 3. 🐱 수석 조교 지니 (Jiny) — 츤데레 팩트 폭격 디버거

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #ecfdf5; border-left: 6px solid #059669;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_jiny.png' | relative_url }}" alt="조교 지니" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-success px-3 py-1 rounded-pill">수석 연구실 조교 & AI 튜터</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">조교 지니 (Jiny)</h4>
    <p class="text-success fw-semibold mb-2">"또 80번 포트 충돌이냐옹! `docker inspect`로 IP 바인딩부터 확인하라옹!"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 시스템 로그를 매의 눈으로 분석하는 수석 트러블슈터 (Chief Debugger & Code Reviewer)</li>
      <li><strong>품종 & 모색:</strong> 한국 토종 코리안 숏헤어 젖소/턱시도 바이컬러 (검은 마스크, 흰 이마 가르마선, 순백의 가슴과 배, 앙증맞은 흰 양말 젤리발, 블랙 롱 테일)</li>
      <li><strong>얼굴 & 소품 (절대 불변):</strong> 신비롭고 맑은 <strong>올리브 그린 눈동자</strong>, 사랑스러운 <strong>베이비 핑크 단추 코</strong>, 콧등에 걸친 <strong>미니 원형 조교 안경</strong>, 작은 황금 방울이 달린 <strong>선명한 빨간 가죽 목걸이</strong></li>
      <li><strong>주요 특기 & 역할:</strong> 츤데레 성격의 천재 수석 조교입니다. 도로시가 레이어 캐시를 깨뜨리거나 루트 권한으로 컨테이너를 띄우면 냥냥 잔소리를 늘어놓지만, 솜방망이 젤리발로 터미널 명령어를 뚝딱 쳐서 치명적인 OOM과 포트 충돌을 1초 만에 해결해 줍니다.</li>
    </ul>
    <details class="small">
      <summary class="text-success cursor-pointer fw-semibold">🎨 AI 마스터 프롬프트 보기</summary>
      <pre class="bg-white p-2 rounded-3 border mt-2"><code>Jiny the lab assistant cat, an adorable slightly chubby black-and-white bicolor tuxedo cat with cow pattern fur, black mask over ears and forehead, pure white muzzle with a cute baby pink button nose, white chest and white paws, intelligent olive-green eyes, wearing a charming red collar with a tiny gold bell and miniature round wire-rim scholar glasses perched on his nose</code></pre>
    </details>
  </div>
</div>

---

### 4. 🐶 파트너견 토토 (Toto) — 긍정 비타민 & 헬스체크 보초병

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #f0f9ff; border-left: 6px solid #0284c7;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_toto.png' | relative_url }}" alt="토토" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-info text-dark px-3 py-1 rounded-pill">긍정 비타민 반려견 & 메이커 조수</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">토토 (Toto)</h4>
    <p class="text-info text-dark fw-semibold mb-2">"멍멍! (빌드 성공 축하 꼬리콥터 가동! 실패하면 캐시부터 날리자멍!)"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 원정대의 사기를 책임지는 무한 엔도르핀 공급원이자 서비스 상태 감시견 (Morale Booster & Health Sentinel)</li>
      <li><strong>품종 & 모색:</strong> 비숑 프리제와 말티즈 믹스 아기 강아지, **반점과 얼룩이 전혀 없는 단색 웜 밀크 화이트 / 크림 아이보리 (`#FBF6EB`)** 구름 같은 소프트 웨이브 털, 뺨을 감싸는 둥근 드룹 귀</li>
      <li><strong>얼굴 & 소품:</strong> 반질반질 윤기 나는 <strong>검은 단추 코 (핑크/브라운 코 절대 금지)</strong>, 동그란 흑진주 눈망울, 핑크빛 혀를 살짝 내민 해맑은 미소, 목에 예쁘게 묶은 <strong>선명한 스카이 블루 네커치프 반다나</strong></li>
      <li><strong>주요 특기 & 역할:</strong> 도로시의 곁을 지키는 든든한 동반자이자 메이커 조수입니다. 빌드가 통과하면 도커 고래 인형을 물고 신나게 달려오고, 서버 에러가 발생해 도로시가 지칠 때는 손등을 따뜻하게 핥아주며 엔도르핀을 충전해 줍니다. 서비스 헬스체크 프로브(`HEALTHCHECK`)의 상징이기도 합니다.</li>
    </ul>
    <details class="small">
      <summary class="text-info text-dark cursor-pointer fw-semibold">🎨 AI 마스터 프롬프트 보기</summary>
      <pre class="bg-white p-2 rounded-3 border mt-2"><code>Toto the puppy, Dorothy's beloved companion, an adorable fluffy cream-white puppy (bichon-maltese mix) with a uniform solid warm milk-white coat and NO spots or patches, cloud-like soft wavy fur, soft floppy ears, big round black button eyes, glossy black button nose, cheerful happy open-mouthed dog smile with pink tongue, wearing a vibrant sky-blue neckerchief bandana tied around his neck with a side knot, curled plumey tail wagging cheerfully</code></pre>
    </details>
  </div>
</div>

---

### 5. 🤖 로봇 틴맨 (Tinny / OzBot) — 피지컬 AI & 인프라 수호자

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #fffbeb; border-left: 6px solid #d97706;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_tinny.png' | relative_url }}" alt="로봇 틴맨" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-warning text-dark px-3 py-1 rounded-pill">피지컬 AI & 인프라 협동 로봇</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">로봇 틴맨 (Tinny / OzBot)</h4>
    <p class="text-warning text-dark fw-semibold mb-2">"인프라 자가 치유 완료! 좀비 프로세스는 제가 Tini Init으로 즉시 수거하겠습니다!"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 무거운 컨테이너 볼륨을 나르고 장애를 자동 복구하는 데브옵스 집행관 (Infrastructure Heavy-Lifter & Self-Healing Agent)</li>
      <li><strong>외형 & 신체 비율:</strong> 오즈의 양철 나무꾼 모티브, 3.5등신 소형 2족 보행 휴머노이드 로봇 (신장 65cm), 웜 매트 실버 틴 섀시 + 화이트 세라믹 에나멜 장갑 패널 + 황동 볼트 관절</li>
      <li><strong>시각적 특징:</strong> 레트로 브라운관 모니터 헤드와 **빛나는 시안 블루 LED 도트 매트릭스 눈** (`^^`, `?`, `@_@`), 머리 위의 황동 스프링 전구 안테나, 가슴 창 속에서 회전하는 황동 기어와 **은은하게 맥동하는 앰버 골드/루비 레드 하트 코어**</li>
      <li><strong>주요 특기 & 역할:</strong> 무거운 컨테이너 스토리지를 번쩍 들어 호스트에 마운트하고, 서버 랙 케이블을 연결하는 든든한 데브옵스 일꾼 로봇입니다. 컨테이너 내부에서 발생한 좀비 프로세스를 쓸어 담는 Tini Init 프로세스의 수호자 역할을 완벽히 수행합니다.</li>
    </ul>
    <details class="small">
      <summary class="text-warning text-dark cursor-pointer fw-semibold">🎨 AI 마스터 프롬프트 보기</summary>
      <pre class="bg-white p-2 rounded-3 border mt-2"><code>Tinny the AI maker robot, a cute friendly small bipedal humanoid robot with a warm matte silver tin chassis, smooth white ceramic enamel armor panels, rotational brass servo joints, round retro monitor head with glowing cyan blue LED dot-matrix crescent eyes, brass spring antenna with yellow bulb, circular chest window showing rotating brass cogs and a pulsing warm amber heart core, 3-finger mechanical hands, cute robotic feet</code></pre>
    </details>
  </div>
</div>

---

### 6. 🐳 수호 고래 모비 (Moby Dock) — 클라우드 대양의 컨테이너 기선

<div class="d-flex flex-column flex-md-row align-items-center align-items-md-start gap-4 p-4 rounded-4 mb-4" style="background-color: #eff6ff; border-left: 6px solid #2563eb;">
  <div class="text-center flex-shrink-0">
    <img src="{{ '/img/character_moby.png' | relative_url }}" alt="도커 수호 고래 모비" class="img-fluid rounded-4 shadow-sm bg-white p-2" style="width: 170px; height: 170px; object-fit: contain;" />
    <div class="mt-2">
      <span class="badge bg-primary px-3 py-1 rounded-pill">클라우드 대양의 수호신</span>
    </div>
  </div>
  <div class="flex-grow-1">
    <h4 class="fw-bold text-dark mb-1">도커 수호 고래 모비 (Moby Dock)</h4>
    <p class="text-primary fw-semibold mb-2">"어떤 파도와 폭풍 속에서도, 너의 컨테이너는 흔들림 없이 안전하게 목적지에 닿을 것이다!"</p>
    <ul class="text-secondary small mb-3">
      <li><strong>원정대 포지션:</strong> 모든 소프트웨어와 데이터 볼륨을 등에 싣고 클라우드 대양을 항해하는 거대한 선박 (Flagship of the Expedition)</li>
      <li><strong>외형적 특징:</strong> 온화하고 지혜로운 눈망울을 지닌 푸른빛 대양 고래, 등에 가지런히 실린 알록달록한 표준 규격 화물 컨테이너 박스들, 머리 위로 기분 좋게 뿜어져 나오는 하트 모양 물줄기</li>
      <li><strong>공학적 상징:</strong> Linux, Windows, macOS, AWS, GCP, On-Premise 등 호스트 환경이 아무리 달라도 애플리케이션을 불변의 표준 패키지(`Immutability`)로 격리하여 100% 동일하게 실행하는 컨테이너 가상화의 정수</li>
    </ul>
  </div>
</div>

---

## 🤝 원정대 협력 시너지 매트릭스 (Character Chemistry)

```
        ┌────────────────────────────────────────────────────────┐
        │              👨‍🏫 이호진 교수님 (총괄 아키텍트)          │
        │          - "왜 가상머신 대신 컨테이너인가?" 원리 전수  │
        └───────────────┬────────────────────────┬───────────────┘
                        │ 근본 원리 전수         │
                        ▼                        ▼
 ┌──────────────────────────────┐         ┌──────────────────────────────┐
 │     👧 도로시 (주니어 빌더)  │◄───────►│    🐱 조교 지니 (수석 디버거)│
 │  - 지치지 않는 학습 열정     │ 실전    │  - 츤데레 팩트 폭격          │
 │  - Dockerfile 직접 빌드      │ 트러블  │  - 포트 충돌 & OOM 1초 컷     │
 │  - 라라벨 4계층 스택 구축    │ 슈팅    │  - 레이어 캐시 최적화 지도   │
 └──────────────┬───────────────┘         └──────────────┬───────────────┘
                │                                        │
                │ 힐링 파트너                            │ 인프라 제어
                ▼                                        ▼
 ┌──────────────────────────────┐         ┌──────────────────────────────┐
 │     🐶 토토 (긍정 비타민)    │         │     🤖 틴맨 (인프라 협동 로봇)│
 │  - 실패해도 용기 북돋우기    │         │  - 무거운 볼륨 번쩍 운반     │
 │  - 헬스체크 프로브 보초병    │         │  - Tini 좀비 프로세스 청소   │
 │  - 도커 고래 인형 물어오기   │         │  - 무중단 자가 치유(Healing) │
 └──────────────┬───────────────┘         └──────────────┬───────────────┘
                │                                        │
                └───────────────────┬────────────────────┘
                                    │
                                    ▼
                ┌────────────────────────────────────────┐
                │        🐳 수호 고래 모비 (Moby Dock)   │
                │   - 컨테이너 대함대를 싣고 대양 항해   │
                │   - 어디서든 똑같이 실행되는 불변 런타임│
                └────────────────────────────────────────┘
```

### 💬 생생한 현장 대화로 배우는 트러블슈팅
1. **이호진 교수님 ➔ 도로시**: *"도로시, `-p 8080:80`을 줄 때 왼쪽이 호스트 포트이고 오른쪽이 컨테이너 내부 포트란다. 커널이 iptables NAT 테이블을 어떻게 조작하는지 머릿속에 그려보렴."*
2. **조교 지니 ➔ 도로시**: *"또 `npm install`이나 `composer install`을 소스코드 복사 뒤에 썼다옹! 소스코드가 1바이트만 바뀌어도 의존성 레이어 캐시가 몽땅 깨진다옹! 배치 순서를 바꾸라옹!"*
3. **도로시 ➔ 틴맨**: *"틴맨, 이번 실습에서 Nginx, Laravel, PostgreSQL, Redis 4대 컨테이너를 한꺼번에 스케일아웃으로 기동해줘!"*
4. **틴맨 ➔ 도로시**: *"삐리릭! `docker compose up -d --scale was=3` 명령 수신 완료! 부하 분산 헬스체크 정상 확인했습니다!"*
5. **토토 ➔ 도로시**: *"멍멍! (빌드가 100% 성공했으니 시원한 물 한잔 마시고 다음 장으로 넘어가자멍!)"*

---

## 🗺️ 원정대가 정복할 8대 대양 로드맵 (The 8 Oceans)

| 항해 단계 | 대양 명칭 | 핵심 정복 기술 및 실습 목표 |
| :---: | :--- | :--- |
| **Part 01** | 🌊 **컨테이너 기초와 아키텍처** | 가상머신 한계 극복, 리눅스 커널 격리(Namespace & Cgroups), 첫 Hello-World 실행 |
| **Part 02** | 📦 **이미지 빌드와 멀티스테이지** | Dockerfile 8대 문법, 레이어 캐시 불변성, 초경량 Alpine/Scratch 빌드, 레지스트리 유통 |
| **Part 03** | 💾 **영속 스토리지와 심화 네트워킹** | 컨테이너 휘발성 극복 볼륨 마운트, 실시간 바인드 마운트, 도커 내장 DNS 브리지 네트워크 |
| **Part 04** | 🎼 **도커 컴포즈와 헬스체크** | `compose.yaml` 선언적 분산 스택, 헬스체크 프로브, cAdvisor/프로메테우스 모니터링 |
| **Part 05** | 🚀 **다중 환경 설계와 CI/CD 자동화** | Dev/Test/Prod 오버라이드 병합, 시크릿(Secrets) 은닉, GitHub Actions 자동 빌드/배포 |
| **Part 06** | 👑 **클러스터 오케스트레이션** | 도커 스웜 매니저/워커 노드, Raft 쿼럼 합의, 인그레스 라우팅 메시, 쿠버네티스 비교 |
| **Part 07** | 🛡️ **보안 하드닝과 멀티 아키텍처** | 상호 TLS(mTLS) 인증, Rootless 비루트 보안 모드, QEMU/Buildx 멀티아키텍처(ARM/AMD) |
| **Part 08** | 🚢 **프로덕션 운영 & 최종 캡스톤** | Fluentd 중앙 집중 로깅, Nginx/Traefik 리버스 프록시, Redis 비동기 큐, 풀스택 완성 |

---

## ⛵ 원정대 승선 선서 (Expedition Oath)

> **"나는 복잡한 명령어에 주눅 들지 않고, 시스템의 본질과 원리를 탐구하는 자랑스러운 도커 원정대원이다.**  
> **포트 충돌이 나도 당황하지 않고 로그를 볼 것이며, 빌드가 깨져도 캐시를 다듬으며 다시 도전할 것이다.**  
> **교수님의 멘토링, 도로시의 열정, 지니의 꼼꼼함, 토토의 긍정, 틴맨의 묵묵한 성실함과 함께 마침내 나만의 클라우드 네이티브 서비스를 전 세계에 배포해낼 것이다!"**

<div class="my-5 text-center">
  <a href="{{ '/Part01_컨테이너기초와_도커엔진아키텍처/000_강의오리엔테이션_클라우드네이티브와_도커대모험/' | relative_url }}" class="btn btn-hero-primary rounded-pill px-5 py-3 fw-bold fs-5 shadow">
    🧭 도커 대항해 첫 번째 닻을 올리기 (Part 01 시작) ➔
  </a>
</div>
