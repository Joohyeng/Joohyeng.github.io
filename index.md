---
layout: single
title: false
author_profile: false
comments: false
toc: true
toc_label: "목차"
toc_sticky: true
masthead_title: "Joohyeng's Portfolio"
---

<div class="port-hero-wrap">
  <div class="port-hero-main">
    <section class="port-intro-block">
      <h2 id="introduce">Introduce</h2>
      <p class="port-intro-text">AI 광고물 검수 흐름과 실시간 채팅·알림을 개발한 백엔드 개발자 김주형입니다. 파일 처리·n8n 검수·결과 조회와 Kafka 기반 권한 동기화를 구현했습니다. 익숙하지 않은 기술도 배우고 적용하며, 기능이 사용자 화면까지 이어지도록 개발하고 있습니다.</p>
    </section>
    <section class="port-contact-block">
      <h2 id="contact">Contact</h2>
      <div class="port-contact-list">
        <a class="port-contact-item" href="https://github.com/Joohyeng" target="_blank" rel="noopener noreferrer">
          <span class="port-contact-icon"><i class="fab fa-github"></i></span>
          <span>github.com/Joohyeng</span>
        </a>
        <span class="port-contact-item">
          <span class="port-contact-icon"><i class="fas fa-envelope"></i></span>
          <span>Email · wngudxogh@naver.com</span>
        </span>
      </div>
    </section>
  </div>
  <div class="port-hero-photo">
    <img src="/assets/images/profile.png" alt="김주형 프로필 사진">
  </div>
</div>

---

<h2 id="projects">Projects</h2>

<div class="port-proj-cards">
  <a class="port-proj-card" href="/project/callog/">
    <div class="ppc-type">AI Integration · MSA · 5인 팀</div>
    <div class="ppc-title">Callog</div>
    <div class="ppc-period">기업 간 캠페인 협업·광고물 검수 플랫폼</div>
    <div class="ppc-desc">기업들이 캠페인을 함께 관리하고 광고물을 AI로 점검하는 서비스입니다. AI 검수·결과 조회와 Kafka 기반 캠페인·권한 동기화를 담당했습니다.</div>
    <div class="ppc-tags"><span class="ppc-tag">Spring Boot</span><span class="ppc-tag">n8n</span><span class="ppc-tag">Kafka</span><span class="ppc-tag">MongoDB</span><span class="ppc-tag">Kubernetes</span></div>
  </a>
  <a class="port-proj-card" href="/project/file-in-n-out/">
    <div class="ppc-type">Realtime Collaboration · Team Project</div>
    <div class="ppc-title">FileInNOut</div>
    <div class="ppc-period">실시간 파일·문서 협업 서비스</div>
    <div class="ppc-desc">파일·문서 관리와 대화를 한곳에서 이어가는 협업 서비스입니다. 채팅·알림의 프론트엔드·백엔드와 페이징·집계 쿼리·캐시를 활용한 조회 개선을 담당했습니다.</div>
    <div class="ppc-tags"><span class="ppc-tag">Spring Boot</span><span class="ppc-tag">Vue 3</span><span class="ppc-tag">WebSocket</span><span class="ppc-tag">Redis</span></div>
  </a>
</div>

---

<h2 id="certification">Certification</h2>

<div class="port-tl-wrap">
  <div class="port-cert-col">
    <div class="port-tl-section">Certification</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2024.06</div>
        <div class="port-tl-title">정보처리기사</div>
        <div class="port-tl-sub">한국산업인력공단</div>
      </div>
    </div>
  </div>
</div>

---

<h2 id="education">Graduate / Education</h2>

<div class="port-tl-wrap port-cert-lang-wrap">
  <div class="port-cert-col">
    <div class="port-tl-section">Graduate</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-date">2018.03 ~ 2024.02</div>
        <div class="port-tl-title">한신대학교 졸업</div>
      </div>
    </div>
  </div>
  <div class="port-cert-col">
    <div class="port-tl-section">Education</div>
    <div class="port-timeline">
      <div class="port-tl-item">
        <div class="port-tl-title">한화 Beyond SW Camp</div>
        <div class="port-tl-sub">24기 수료 · FE·BE·DevOps</div>
      </div>
    </div>
  </div>
</div>

---

<h2 id="skills">Skills</h2>

<div class="port-tabs">
  <div class="port-tab-nav">
    <button class="port-tab-btn active" data-tab="tab-ai">AI Integration</button>
    <button class="port-tab-btn" data-tab="tab-backend">Backend</button>
    <button class="port-tab-btn" data-tab="tab-data">Data &amp; Messaging</button>
    <button class="port-tab-btn" data-tab="tab-collaboration">Collaboration</button>
    <button class="port-tab-btn" data-tab="tab-infra">DevOps</button>
  </div>

  <div id="tab-ai" class="port-tab-panel active">
    <div class="port-skill-grid">
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-project-diagram"></i></div><div class="si-body"><div class="si-name">n8n</div><div class="si-desc">추출 텍스트를 검수 워크플로에 전달하고 결과 저장·조회 흐름을 연결했습니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-file-alt"></i></div><div class="si-body"><div class="si-name">OCR · Layout Analysis</div><div class="si-desc">파일의 내용 추출 기능을 AI 판사 모듈에 연결하고 분석 산출물을 관리했습니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-exchange-alt"></i></div><div class="si-body"><div class="si-name">AI 결과 처리</div><div class="si-desc">작업 식별자로 결과와 파일 정보를 연결하고 오류 정보·상세 조회를 구성했습니다.</div></div></div>
    </div>
  </div>

  <div id="tab-backend" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item"><div class="si-icon"><i class="fab fa-java"></i></div><div class="si-body"><div class="si-name">Java · Spring Boot</div><div class="si-desc">도메인 중심 REST API와 서비스 모듈을 설계하고 구현합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-shield-alt"></i></div><div class="si-body"><div class="si-name">Spring Security</div><div class="si-desc">인증·인가 흐름과 JWT 기반 보안 구성을 다룹니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-database"></i></div><div class="si-body"><div class="si-name">JPA · MariaDB</div><div class="si-desc">엔티티 관계와 트랜잭션을 고려한 데이터 접근 계층을 구현합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-file-code"></i></div><div class="si-body"><div class="si-name">OpenAPI · Swagger</div><div class="si-desc">프론트엔드·백엔드가 함께 사용할 API 계약을 문서화합니다.</div></div></div>
    </div>
  </div>

  <div id="tab-data" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-stream"></i></div><div class="si-body"><div class="si-name">Kafka</div><div class="si-desc">서비스 간 비동기 이벤트 전달과 후속 처리 흐름을 구성합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-database"></i></div><div class="si-body"><div class="si-name">MongoDB · Redis</div><div class="si-desc">사용 목적에 맞는 데이터 저장소를 선택하고 연동합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-bolt"></i></div><div class="si-body"><div class="si-name">Event-driven Flow</div><div class="si-desc">요청 흐름과 이벤트 처리의 책임을 분리하는 방식을 학습·적용합니다.</div></div></div>
    </div>
  </div>

  <div id="tab-collaboration" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-comments"></i></div><div class="si-body"><div class="si-name">WebSocket · STOMP</div><div class="si-desc">채팅·협업 상황에 맞춘 실시간 메시지 전달 구조를 구현합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-bell"></i></div><div class="si-body"><div class="si-name">SSE</div><div class="si-desc">서버 이벤트를 사용자 화면에 즉시 전달하는 알림 채널을 구성합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-file-alt"></i></div><div class="si-body"><div class="si-name">Web Push · Service Worker</div><div class="si-desc">백그라운드 알림과 브라우저의 Push 구독 흐름을 연결했습니다.</div></div></div>
    </div>
  </div>

  <div id="tab-infra" class="port-tab-panel">
    <div class="port-skill-grid">
      <div class="port-skill-item"><div class="si-icon"><i class="fab fa-docker"></i></div><div class="si-body"><div class="si-name">Docker</div><div class="si-desc">개발·배포 환경을 컨테이너로 일관되게 구성합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fas fa-cubes"></i></div><div class="si-body"><div class="si-name">Kubernetes</div><div class="si-desc">서비스 배포와 운영 환경의 기본 흐름을 이해하고 적용합니다.</div></div></div>
      <div class="port-skill-item"><div class="si-icon"><i class="fab fa-jenkins"></i></div><div class="si-body"><div class="si-name">Jenkins · CI/CD</div><div class="si-desc">빌드·테스트·배포 과정을 자동화하는 파이프라인을 구성합니다.</div></div></div>
    </div>
  </div>
</div>

<script>
(function () {
  function initTabs() {
    document.querySelectorAll('.port-tabs').forEach(function (wrap) {
      wrap.querySelectorAll('.port-tab-btn').forEach(function (btn) {
        btn.addEventListener('click', function () {
          wrap.querySelectorAll('.port-tab-btn').forEach(function (b) { b.classList.remove('active'); });
          wrap.querySelectorAll('.port-tab-panel').forEach(function (p) { p.classList.remove('active'); });
          btn.classList.add('active');
          var panel = wrap.querySelector('#' + btn.dataset.tab);
          if (panel) panel.classList.add('active');
        });
      });
    });
  }
  if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', initTabs);
  else initTabs();
})();
</script>
