---
layout: docs
title: "도커 & 클라우드 네이티브 2학기제 마스터 커리큘럼 (총 28주)"
---

# 🎓 도커 & 클라우드 네이티브 실무 마스터 커리큘럼

> **비전공·초보 대학생을 위한 '0부터 시작하는 로컬 개발 및 실서버 배포(1학기)'부터,**  
> **기업 현업에 즉시 투입 가능한 '엔터프라이즈 Ops, 쿠버네티스 & SRE 전문가 양성(2학기)'까지 이어지는 28주 로드맵**

---

<div class="card border-0 rounded-4 p-4 mb-4 shadow-sm" style="background: linear-gradient(135deg, #eff6ff 0%, #dbeafe 100%); border-left: 6px solid #2563eb !important;">
  <div class="d-flex flex-column flex-md-row justify-content-between align-items-md-center gap-3">
    <div>
      <span class="badge rounded-pill bg-primary px-3 py-2 fw-bold mb-2">Hands-on Project Track</span>
      <h4 class="fw-bold mb-1" style="color: #1e3a8a;">🎯 전용 실전 프로젝트: 최신 PHP 8.5 라라벨 & Nginx & PostgreSQL</h4>
      <p class="text-secondary mb-0 small">
        단순한 명령어가 아닌 <strong>Web(Nginx) - WAS(PHP 8.5 라라벨) - DB(PostgreSQL) - Cache(Redis)</strong> 4계층 아키텍처를 기반으로,<br class="d-none d-lg-inline">
        로컬 3-Tier 모놀리스 개발부터 대규모 트래픽 대응을 위한 <strong>수평 스케일아웃(Scale-Out) 및 MSA 분산 오케스트레이션</strong>까지 완주합니다.
      </p>
    </div>
    <div class="flex-shrink-0">
      <a href="/curriculum/practice/" class="btn btn-primary rounded-pill px-4 py-2 fw-bold shadow-sm">
        실전 프로젝트 상세 보기 ➔
      </a>
    </div>
  </div>
</div>

<ul class="nav nav-pills nav-fill mb-4 p-2 bg-light rounded-4 shadow-sm" id="curriculumTabs" role="tablist">
  <li class="nav-item" role="presentation">
    <button class="nav-link active rounded-pill fw-bold py-3 fs-5" id="sem1-tab" data-bs-toggle="tab" data-bs-target="#semester-1" type="button" role="tab" aria-controls="semester-1" aria-selected="true">
      🌸 1학기: 도커 활용, 로컬 개발 & 실서버 배포 (14주)
    </button>
  </li>
  <li class="nav-item" role="presentation">
    <button class="nav-link rounded-pill fw-bold py-3 fs-5" id="sem2-tab" data-bs-toggle="tab" data-bs-target="#semester-2" type="button" role="tab" aria-controls="semester-2" aria-selected="false">
      🍁 2학기: 엔터프라이즈 Ops & 쿠버네티스 심화 (14주)
    </button>
  </li>
</ul>

<div class="tab-content" id="curriculumTabContent">

  <!-- ==================== 1학기 ==================== -->
  <div class="tab-pane fade show active" id="semester-1" role="tabpanel" aria-labelledby="sem1-tab">
    
    <div class="alert alert-primary border-0 rounded-4 p-4 mb-4 shadow-sm">
      <h4 class="fw-bold mb-2">🌸 1학기 학습 목표: 0에서 시작하여 실제 살아있는 웹사이트 배포하기</h4>
      <p class="mb-0">
        리눅스나 서버 지식이 전혀 없는 학생도 친절한 일러스트와 비유를 통해 쉽게 입문합니다. 로컬 PC에서 개발한 소스코드가 즉시 컨테이너에 반영되는 개발자 경험(DX)을 체득하고, 기말에는 <strong>실제 클라우드 서버(공인 도메인 및 HTTPS)에 상용 웹사이트를 직접 배포</strong>하여 실무 포트폴리오를 완성합니다.
      </p>
    </div>

    <div class="table-responsive shadow-sm rounded-4 mb-5">
      <table class="table table-hover align-middle mb-0 bg-white">
        <thead class="table-dark">
          <tr class="text-center py-3">
            <th style="width: 8%;">주차</th>
            <th style="width: 24%;">주차별 핵심 주제</th>
            <th style="width: 38%;">학생 눈높이 학습 & 실습 내용</th>
            <th style="width: 15%;">연계 강의</th>
            <th style="width: 15%;">실전 마일스톤</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="text-center fw-bold text-primary">1주차</td>
            <td><strong>클라우드 세상과 컨테이너 이야기</strong><br><small class="text-muted">내 PC에선 됐는데 왜 서버에선 안 될까?</small></td>
            <td>• 가상머신(VM)과 컨테이너의 결정적 차이를 실생활 비유로 체득<br>• 클라우드 이주와 레거시 현대화 패러다임 이해</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/000_강의오리엔테이션_클라우드네이티브와_도커대모험/" class="badge bg-secondary text-decoration-none">000~002강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">컨테이너 개념 확립</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">2주차</td>
            <td><strong>내 PC에 도커 연구실 차리기</strong><br><small class="text-muted">Windows WSL2 / macOS / Linux 설치</small></td>
            <td>• OS별 가상화 설정 및 도커 데스크톱/엔진 설치<br>• <code>docker run hello-world</code> 첫 컨테이너 구동 검증</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/003_실습환경구축1_Windows_WSL2와_macOS_Docker_Desktop/" class="badge bg-secondary text-decoration-none">003~005, 012강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">실습 환경 구축 완료</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">3주차</td>
            <td><strong>첫 컨테이너 실행과 웹서버 접속</strong><br><small class="text-muted">대화형 셸 접속과 포트포워딩의 마법</small></td>
            <td>• <code>-it</code> 셸 접속과 <code>-d</code> 백그라운드 데몬 구동<br>• 포트포워딩(<code>-p 8080:80</code>)으로 Nginx 웹서버 띄우기</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/006_컨테이너란_무엇인가_격리와밀집의_두마리토끼/" class="badge bg-secondary text-decoration-none">006~009, 013강</a></td>
            <td><span class="badge rounded-pill bg-success">첫 웹서버 브라우저 접속</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">4주차</td>
            <td><strong>실행 중인 컨테이너 점검과 생애주기</strong><br><small class="text-muted">exec 셸 침투와 양방향 파일 복사</small></td>
            <td>• 컨테이너 내부로 들어가 실시간 디버깅(<code>exec</code>)<br>• 호스트와 컨테이너 간 파일 복사(<code>cp</code>) 및 핫패치</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/010_컨테이너_생애주기_관리와_상태모니터링/" class="badge bg-secondary text-decoration-none">010~011, 014강</a></td>
            <td><span class="badge rounded-pill bg-info text-dark">1차 실습: 웹 핫패치</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">5주차</td>
            <td><strong>나만의 이미지 레시피: Dockerfile 기본</strong><br><small class="text-muted">레이어 아키텍처와 캐시 적중률 극대화</small></td>
            <td>• FROM, WORKDIR, COPY, RUN, CMD 핵심 문법<br>• 코드 한 줄 고쳐도 1초 만에 빌드되는 캐시 전략</td>
            <td class="text-center"><a href="/Part02_도커이미지빌드와_멀티스테이지_레지스트리/015_도커허브_이미지활용과_런타임환경변수_주입/" class="badge bg-secondary text-decoration-none">015~019강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">첫 커스텀 이미지 제작</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">6주차</td>
            <td><strong>실행 인자와 멀티스테이지 초경량화</strong><br><small class="text-muted">Spring/Node/Go 이미지 90% 용량 다이어트</small></td>
            <td>• CMD vs ENTRYPOINT 신호 전달 완벽 비교<br>• 멀티스테이지 빌드로 빌드 도구는 버리고 바이너리만 패키징</td>
            <td class="text-center"><a href="/Part02_도커이미지빌드와_멀티스테이지_레지스트리/020_CMD_vs_ENTRYPOINT_실행명령과_인자전달패턴/" class="badge bg-secondary text-decoration-none">020~024강</a></td>
            <td><span class="badge rounded-pill bg-success">초경량(수십MB) 배포 완성</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">7주차</td>
            <td><strong>세계인과 공유하는 이미지: 레지스트리</strong><br><small class="text-muted">도커 허브 & GHCR 푸시, 시맨틱 버저닝</small></td>
            <td>• 내가 만든 이미지를 Docker Hub 및 GHCR에 푸시<br>• 시맨틱 태그 전략(v1.0.0)과 <code>:latest</code>의 위험성 회피</td>
            <td class="text-center"><a href="/Part02_도커이미지빌드와_멀티스테이지_레지스트리/026_레지스트리_리포지터리_이미지태그의_정확한구조/" class="badge bg-secondary text-decoration-none">026~031강</a></td>
            <td><span class="badge rounded-pill bg-info text-dark">2차 실습: GHCR 배포</span></td>
          </tr>
          <tr class="table-warning">
            <td class="text-center fw-bold text-danger">8주차</td>
            <td><strong>[1학기 중간평가] PHP 8.5 라라벨 단일 패키징 실기</strong><br><small class="text-muted">보안 하드닝 및 멀티스테이지 경량 이미지 제작</small></td>
            <td>• 최신 PHP 8.5 + Composer 의존성 분리 멀티스테이지 빌드<br>• non-root 사용자 적용, 보안 취약점 차단 및 로컬 구동 실기 평가</td>
            <td class="text-center">-</td>
            <td><span class="badge rounded-pill bg-danger">중간고사 실기 평가</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">9주차</td>
            <td><strong>데이터가 사라지지 않게: 퍼시스턴트 볼륨</strong><br><small class="text-muted">도커 볼륨으로 데이터베이스 데이터 영속화</small></td>
            <td>• 컨테이너의 휘발성 쓰기 레이어와 CoW 원리<br>• MySQL/PostgreSQL 볼륨 연결로 컨테이너 재생성 시에도 데이터 보존</td>
            <td class="text-center"><a href="/Part03_퍼시스턴트스토리지와_도커볼륨_마운트/032_컨테이너속_데이터가_사라지는이유_기록가능레이어와_CoW/" class="badge bg-secondary text-decoration-none">032~035, 040강</a></td>
            <td><span class="badge rounded-pill bg-success">데이터 영속성 확보</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">10주차</td>
            <td><strong>로컬 개발자 경험(DX) 혁신: 바인드 마운트</strong><br><small class="text-muted">코드 수정 즉시 반영되는 라이브 핫리로드</small></td>
            <td>• 내 소스코드 폴더를 컨테이너에 실시간 직결(<code>-v</code>)<br>• 재빌드 없이 VS Code에서 코딩하고 브라우저 새로고침</td>
            <td class="text-center"><a href="/Part03_퍼시스턴트스토리지와_도커볼륨_마운트/036_바인드마운트_호스트디렉터리를_컨테이너에_직결하기/" class="badge bg-secondary text-decoration-none">036~039, 041강</a></td>
            <td><span class="badge rounded-pill bg-info text-dark">3차 실습: 핫리로드 개발환경</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">11주차</td>
            <td><strong>명령어 하나로 복합 서비스 기동: 컴포즈</strong><br><small class="text-muted">compose.yaml 선언적 다중 컨테이너</small></td>
            <td>• 웹 프론트엔드 + API 백엔드 + DB 원클릭 동시 기동<br>• <code>docker compose up -d</code> 및 상태 제어</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/042_도커컴포즈_소개와_compose_yaml_선언적구조/" class="badge bg-secondary text-decoration-none">042~043, 046강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">다중 컨테이너 일괄 기동</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">12주차</td>
            <td><strong>컨테이너 간 대화: 컴포즈 네트워크 & 환경변수</strong><br><small class="text-muted">도커 내장 DNS와 .env 비밀번호 은닉</small></td>
            <td>• IP 주소 몰라도 서비스 이름으로 자동 통신(DNS)<br>• <code>.env</code> 파일로 민감한 DB 비밀번호 소스코드 분리</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/044_도커컴포즈_내장DNS_서비스디스커버리와_수평스케일링/" class="badge bg-secondary text-decoration-none">025, 044~045강</a></td>
            <td><span class="badge rounded-pill bg-success">안전한 서비스 디스커버리</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-primary">13주차</td>
            <td><strong>진짜 서버로 떠나자: 리눅스 클라우드 서버 배포</strong><br><small class="text-muted">클라우드 가상서버(VPS) 생성 및 원격 배포</small></td>
            <td>• 클라우드 가상서버 인스턴스 생성 및 방화벽 포트 개방<br>• 원격 서버 SSH 접속 후 도커 환경 세팅 & Git 배포</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/004_실습환경구축2_Linux서버_Docker_Engine과_CLI검증/" class="badge bg-secondary text-decoration-none">004, 062강</a></td>
            <td><span class="badge rounded-pill bg-primary">공인 IP 원격 배포 성공</span></td>
          </tr>
          <tr class="table-primary">
            <td class="text-center fw-bold text-primary">14주차</td>
            <td><strong>[1학기 기말 캡스톤] Nginx + 라라벨 + PostgreSQL 클라우드 배포</strong><br><small class="text-muted">공인 도메인 + Let's Encrypt SSL + pgsql 3-Tier 서비스 완성</small></td>
            <td>• 스마트폰으로 누구나 접속할 수 있는 실서비스 오픈<br>• Nginx 리버스 프록시 ↔ PHP-FPM WAS ↔ pgsql 실전 포트폴리오 발표회</td>
            <td class="text-center"><a href="/Part08_프로덕션운영_설정_로깅_리버스프록시_메시지큐/126_현업도입가이드1_우리조직의_애플리케이션으로_PoC성공시키기/" class="badge bg-secondary text-decoration-none">126~127강</a></td>
            <td><span class="badge rounded-pill bg-danger">1학기 최종 캡스톤 완성</span></td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>

  <!-- ==================== 2학기 ==================== -->
  <div class="tab-pane fade" id="semester-2" role="tabpanel" aria-labelledby="sem2-tab">

    <div class="alert alert-success border-0 rounded-4 p-4 mb-4 shadow-sm">
      <h4 class="fw-bold mb-2">🍁 2학기 학습 목표: 기업 현업에 즉시 투입 가능한 엔터프라이즈 Ops 전문가 양성</h4>
      <p class="mb-0">
        1학기에서 웹서비스를 배포해 본 역량을 바탕으로, 기업 운영팀이 요구하는 <strong>고가용성 자가치유, 풀스택 관제(Prometheus/Grafana), 컨테이너 CI/CD 파이프라인, 쿠버네티스(K8s) & 스웜 오케스트레이션, 4대 실전 트러블슈팅 런북, 엔터프라이즈 보안 하드닝</strong>을 총망라하여 실무형 DevOps/SRE 엔지니어로 도약합니다.
      </p>
    </div>

    <div class="table-responsive shadow-sm rounded-4 mb-5">
      <table class="table table-hover align-middle mb-0 bg-white">
        <thead class="table-dark">
          <tr class="text-center py-3">
            <th style="width: 8%;">주차</th>
            <th style="width: 24%;">주차별 핵심 주제</th>
            <th style="width: 38%;">기업 현업 실무 학습 & 실습 내용</th>
            <th style="width: 15%;">연계 강의</th>
            <th style="width: 15%;">실전 마일스톤</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td class="text-center fw-bold text-success">1주차</td>
            <td><strong>서버가 죽지 않는 마법: 헬스체크(Healthcheck)</strong><br><small class="text-muted">프로세스는 살아있는데 서비스는 응답이 없다?</small></td>
            <td>• 데드락 및 500 에러를 도커 데몬이 감지하는 메커니즘<br>• 커스텀 헬스체크 스크립트 작성 및 상태 천이 모니터링</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/047_헬스체크1_프로세스는_살아있는데_서비스는_죽었다/" class="badge bg-secondary text-decoration-none">047~050강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">불량 컨테이너 자동 감지</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">2주차</td>
            <td><strong>자가치유(Self-Healing) & 조건부 의존성 제어</strong><br><small class="text-muted">Fail-Safe 전략과 서비스 자동 부활</small></td>
            <td>• <code>depends_on</code> 한계 극복: <code>condition: service_healthy</code><br>• 프로세스 강제 종료 시 1초 만에 자동 재생성되는 자가치유</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/051_도커컴포즈_헬스체크와_조건부의존성_service_healthy/" class="badge bg-secondary text-decoration-none">051~052, 061강</a></td>
            <td><span class="badge rounded-pill bg-success">무인 자가치유 시스템</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">3주차</td>
            <td><strong>시스템 관제탑 1: 프로메테우스(Prometheus) 메트릭</strong><br><small class="text-muted">cAdvisor & Node Exporter 연동 시계열 수집</small></td>
            <td>• 풀(Pull) 기반 모니터링 아키텍처 및 메트릭 엔드포인트<br>• PromQL 질의문으로 CPU/메모리 병목 지점 정밀 분석</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/053_컨테이너_모니터링_기술스택_Prometheus와_Grafana/" class="badge bg-secondary text-decoration-none">053~057강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">인프라 자원 메트릭 수집</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">4주차</td>
            <td><strong>시스템 관제탑 2: 그라파나(Grafana) 대시보드 코드화</strong><br><small class="text-muted">실시간 대시보드 프로비저닝 & 슬랙 긴급 알림</small></td>
            <td>• 클릭 없는 대시보드 코드화(Provisioning YAML/JSON)<br>• 자원 임계치 초과 시 슬랙(Slack)/디스코드 긴급 Alerting 발송</td>
            <td class="text-center"><a href="/Part04_도커컴포즈와_신뢰성_헬스체크_모니터링/058_그라파나_Grafana_컨테이너구축과_실시간대시보드_시각화/" class="badge bg-secondary text-decoration-none">058~060강</a></td>
            <td><span class="badge rounded-pill bg-info text-dark">실시간 관제탑 & 경보 완성</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">5주차</td>
            <td><strong>기업형 다중 환경 설계: Dev, Test, Prod 격리</strong><br><small class="text-muted">컴포즈 오버라이드 병합 & 비밀값(Secrets) 은닉</small></td>
            <td>• 단일 코드로 개발/테스트/운영 환경 동시 파생<br>• Docker Secrets를 통한 소스코드 암호 노출 원천 차단</td>
            <td class="text-center"><a href="/Part05_다중환경설계와_컨테이너기반_CICD파이프라인/063_도커컴포즈_오버라이드파일_병합원리와_config검증/" class="badge bg-secondary text-decoration-none">063~068강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">3대 환경 오버라이드 구축</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">6주차</td>
            <td><strong>손 하나 까딱 않는 배포: 컨테이너 CI/CD 파이프라인</strong><br><small class="text-muted">DooD 소켓 러너와 Git Push 자동 빌드/테스트</small></td>
            <td>• 호스트 의존성 없는 Docker-out-of-Docker 러너 구성<br>• GitHub Actions 연동 자동 테스트 통과 시에만 배포하는 품질 게이트</td>
            <td class="text-center"><a href="/Part05_다중환경설계와_컨테이너기반_CICD파이프라인/069_도커를이용한_지속적통합_CI_원리와_컨테이너빌드인프라/" class="badge bg-secondary text-decoration-none">069~075강</a></td>
            <td><span class="badge rounded-pill bg-success">Git Push CI/CD 파이프라인</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">7주차</td>
            <td><strong>단일 서버의 한계를 넘어: 도커 스웜 클러스터</strong><br><small class="text-muted">매니저-워커 노드와 오버레이(VXLAN) 네트워크</small></td>
            <td>• 서버 3대를 묶어 하나의 거대한 슈퍼컴퓨터로 운영<br>• Raft 합의 알고리즘과 매니저 노드 고가용성(HA) 원리</td>
            <td class="text-center"><a href="/Part06_컨테이너오케스트레이션_도커스웜과_쿠버네티스/076_컨테이너_오케스트레이션이란_왜_클러스터가_필요한가/" class="badge bg-secondary text-decoration-none">076~079, 093강</a></td>
            <td><span class="badge rounded-pill bg-primary">3-노드 클러스터 기동</span></td>
          </tr>
          <tr class="table-warning">
            <td class="text-center fw-bold text-danger">8주차</td>
            <td><strong>[2학기 중간평가] 라라벨 WAS 수평 스케일아웃 & 롤백 실기</strong><br><small class="text-muted">Nginx 부하분산 + WAS 5대 스케일아웃 + 자동 롤백</small></td>
            <td>• Nginx Upstream 라운드로빈 및 Least Connection 부하 분산 시연<br>• WAS 5대 동시 가동 부하 테스트 및 불량 배포 시 1초 자동 롤백 검증</td>
            <td class="text-center"><a href="/Part06_컨테이너오케스트레이션_도커스웜과_쿠버네티스/088_애플리케이션_무중단업그레이드_프로세스와_오버라이드병합/" class="badge bg-secondary text-decoration-none">088~091, 095강</a></td>
            <td><span class="badge rounded-pill bg-danger">중간고사: 스케일아웃 실기</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">9주차</td>
            <td><strong>엔터프라이즈 스택 배포와 노드 유지보수</strong><br><small class="text-muted">docker stack deploy & 무중단 Drain 작업 대피</small></td>
            <td>• 중앙 설정(Config)과 시크릿(Secret) 주입 스택 배포<br>• 서버 점검 시 컨테이너를 다른 노드로 자동 이주시키는 Drain 모드</td>
            <td class="text-center"><a href="/Part06_컨테이너오케스트레이션_도커스웜과_쿠버네티스/082_도커스웜_스택배포_docker_stack_deploy와_운영컴포즈/" class="badge bg-secondary text-decoration-none">082~087, 092강</a></td>
            <td><span class="badge rounded-pill bg-success">프로덕션 스택 운영 완료</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">10주차</td>
            <td><strong>모놀리스에서 MSA 분리 & 쿠버네티스 전환</strong><br><small class="text-muted">Auth/Order/Web 서비스 분리 + Database-per-Service (pgsql)</small></td>
            <td>• 단일 라라벨 모놀리스를 Auth 및 Order 독립 마이크로서비스로 분리<br>• 서비스별 독립 pgsql 데이터베이스 구성 및 쿠버네티스 Pod/Service 전환</td>
            <td class="text-center"><a href="/Part06_컨테이너오케스트레이션_도커스웜과_쿠버네티스/080_도커스웜_vs_쿠버네티스_Kubernetes_아키텍처완벽비교/" class="badge bg-secondary text-decoration-none">080~081강</a></td>
            <td><span class="badge rounded-pill bg-info text-dark">라라벨 MSA 구조 전환</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">11주차</td>
            <td><strong>엔터프라이즈 보안 하드닝 & 멀티 아키텍처</strong><br><small class="text-muted">mTLS 상호 인증서, 비루트 유저, Buildx 크로스 컴파일</small></td>
            <td>• 암호화된 mTLS로 원격 도커 데몬을 안전하게 제어하기<br>• root 권한 탈취를 막는 non-root 보안 & Trivy 취약점 스캔<br>• Intel(x86_64)과 Apple/ARM 동시 빌드하는 Docker Buildx</td>
            <td class="text-center"><a href="/Part07_보안원격접근_다중아키텍처_이미지최적화/096_도커엔진_API엔드포인트와_비보안TCP포워딩의_치명적위험/" class="badge bg-secondary text-decoration-none">096~109강</a></td>
            <td><span class="badge rounded-pill bg-light text-dark border">보안 하드닝 골든 이미지</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">12주차</td>
            <td><strong>프로덕션 장애 해결 훈련: 4대 트러블슈팅 런북</strong><br><small class="text-muted">포트 충돌, 디스크 고갈, 권한 거부, 좀비 프로세스</small></td>
            <td>• 현업에서 가장 많이 터지는 4대 장애 시나리오 집중 격파<br>• 고의로 유발된 장애 상황을 30분 내 원상 복구하는 실무 훈련</td>
            <td class="text-center"><a href="/Part01_컨테이너기초와_도커엔진아키텍처/011_실행중인_컨테이너_내부진단_exec와_파일복사_cp/" class="badge bg-secondary text-decoration-none">트러블슈팅 특화랩</a></td>
            <td><span class="badge rounded-pill bg-danger">실전 장애 복구 런북 체득</span></td>
          </tr>
          <tr>
            <td class="text-center fw-bold text-success">13주차</td>
            <td><strong>대규모 트래픽 분산과 중앙 로깅 파이프라인</strong><br><small class="text-muted">Traefik 리버스 프록시 & Fluentd/Loki 로그 통합</small></td>
            <td>• 컨테이너 증설 시 자동으로 라우팅해 주는 Traefik 프록시<br>• 모든 컨테이너의 표준 출력을 중앙에서 수집·검색하는 로깅 스택</td>
            <td class="text-center"><a href="/Part08_프로덕션운영_설정_로깅_리버스프록시_메시지큐/114_도커의_컨테이너로깅모델_stdout_stderr와_로그드라이버/" class="badge bg-secondary text-decoration-none">114~125강</a></td>
            <td><span class="badge rounded-pill bg-success">L7 프록시 & 통합 로깅 구축</span></td>
          </tr>
          <tr class="table-success">
            <td class="text-center fw-bold text-success">14주차</td>
            <td><strong>[2학기 최종 캡스톤] Nginx + 라라벨 MSA 엔터프라이즈 플랫폼 운영</strong><br><small class="text-muted">스케일아웃 + 비동기 큐 + 풀스택 관제(Prometheus/Grafana) + 로그 통합</small></td>
            <td>• 대규모 동시 트래픽 부하 주입 및 노드 강제 다운 시 99.99% 무중단 서비스 시연<br>• 기업 채용 담당자가 감탄할 수준의 최종 DevOps/SRE MSA 포트폴리오 평가</td>
            <td class="text-center"><a href="/Part08_프로덕션운영_설정_로깅_리버스프록시_메시지큐/128_현업도입가이드3_프로덕션으로가는길_클라우드관리형컨테이너와_CNCF/" class="badge bg-secondary text-decoration-none">128~129강</a></td>
            <td><span class="badge rounded-pill bg-danger">2학기 최종 캡스톤 완성</span></td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>

</div>

---

### 🏆 학기별 최종 산출물 및 포트폴리오 비교

<div class="row g-4 mb-5">
  <div class="col-md-6">
    <div class="card h-100 border-0 rounded-4 p-4 shadow-sm" style="background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%); border-left: 6px solid #0284c7 !important;">
      <h4 class="fw-bold text-primary mb-3">🌸 1학기 수료 산출물</h4>
      <h6 class="fw-bold mb-3">"풀스택 웹서비스 클라우드 배포 포트폴리오"</h6>
      <ul class="text-secondary small ps-3 mb-0">
        <li class="mb-2"><strong>실제 배포 URL</strong>: <code>https://student.jiny.dev</code> (공인 도메인 및 SSL 인증서 연결)</li>
        <li class="mb-2"><strong>이미지 경량화</strong>: 멀티스테이지 빌드로 용량 80% 이상 압축한 Dockerfile</li>
        <li class="mb-2"><strong>다중 서비스 정의</strong>: <code>compose.yaml</code>로 Web + API + DB 3계층 통합 선언</li>
        <li class="mb-2"><strong>로컬 개발자 경험(DX)</strong>: 소스코드 수정 즉시 반영되는 핫리로드 환경</li>
        <li><strong>스토리지 영속화</strong>: 컨테이너 삭제 후 재생성 시에도 데이터가 보존되는 볼륨</li>
      </ul>
    </div>
  </div>

  <div class="col-md-6">
    <div class="card h-100 border-0 rounded-4 p-4 shadow-sm" style="background: linear-gradient(135deg, #f0fdf4 0%, #dcfce7 100%); border-left: 6px solid #16a34a !important;">
      <h4 class="fw-bold text-success mb-3">🍁 2학기 수료 산출물</h4>
      <h6 class="fw-bold mb-3">"엔터프라이즈 클라우드 네이티브 Ops 운영 포트폴리오"</h6>
      <ul class="text-secondary small ps-3 mb-0">
        <li class="mb-2"><strong>클러스터 오케스트레이션</strong>: 3-노드 고가용성 클러스터 및 무중단 롤링 배포</li>
        <li class="mb-2"><strong>배포 자동화</strong>: Git Push 시 자동 테스트/빌드/배포되는 컨테이너 CI/CD</li>
        <li class="mb-2"><strong>풀스택 관제탑</strong>: 프로메테우스 + 그라파나 대시보드 코드화 및 슬랙 실시간 경보</li>
        <li class="mb-2"><strong>자가치유 아키텍처</strong>: 컨테이너 불량 발생 시 1초 만에 자동 복구되는 Fail-Safe</li>
        <li class="mb-2"><strong>장애 복구 런북</strong>: 4대 핵심 장애(포트/OOM/권한/좀비) 실전 트러블슈팅 런북</li>
        <li><strong>엔터프라이즈 보안</strong>: mTLS 원격 데몬 제어, non-root 보안, 멀티 아키텍처 빌드</li>
      </ul>
    </div>
  </div>
</div>
