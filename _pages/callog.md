---
title: "Callog — AI 기반 마케팅 캠페인 협업 플랫폼"
permalink: /project/callog/
layout: single
live_demo_pending: true
author_profile: false
sidebar:
  nav: "projects_nav"
toc: true
toc_label: "목차"
toc_sticky: true
---

<div class="port-meta-bar">
  <div class="port-meta-item"><div class="port-meta-label">유형</div><div class="port-meta-value">AI 기반 마케팅 협업 플랫폼</div></div>
  <div class="port-meta-item"><div class="port-meta-label">팀 구성</div><div class="port-meta-value">5인 팀 프로젝트</div></div>
  <div class="port-meta-item"><div class="port-meta-label">아키텍처</div><div class="port-meta-value">MSA · Event-driven</div></div>
  <div class="port-meta-item"><div class="port-meta-label">GitHub</div><div class="port-meta-value"><a href="https://github.com/beyond-sw-camp/be24-fin-magamFairy-callog" target="_blank" rel="noopener noreferrer">Repository</a></div></div>
</div>

## 프로젝트 개요

- 마케팅 캠페인, 콘텐츠, 파트너, KPI를 하나의 플랫폼에서 관리하기 위한 MSA 기반 협업 서비스
- 캠페인 생성 이후 필요한 메타데이터·권리 처리와 AI 콘텐츠 리뷰를 각 서비스의 책임에 맞게 연결
- Gateway와 서비스 디스커버리를 바탕으로 도메인별 서비스를 분리하고, 이벤트로 후속 작업을 처리

## 담당 기능

팀 개발에서 인증·조직 관리부터 AI 검수 기능의 MSA 전환까지, 사용자의 요청이 서비스 경계를 넘어 처리되는 흐름을 구현했습니다.

- **인증·계정 흐름** — 로그인·로그아웃·회원가입·비밀번호 재설정을 프론트엔드와 백엔드로 연결하고, JWT 리프레시 토큰과 비로그인 사용자 라우팅을 조정
- **조직·권한 관리** — 사용자·관리자 계정 생성, 조직 정보 연결, General Manager 역할과 구성원 관리 기능을 구현
- **AI 검수 흐름** — 광고물 검수 요청, 파일 업로드, OCR·레이아웃 분석, 결과 메타데이터 저장과 상세 조회 화면을 연결
- **AI 판사 MSA 전환** — 기존 애플리케이션에 있던 AI 판사 기능을 별도 모듈로 분리하고, 캠페인 생성·참여자 권한 변경 이벤트를 Kafka로 전달해 AI 판사 서버가 MongoDB의 동기화된 권한 정보를 바탕으로 검수 요청을 처리하도록 구성

## 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Cloud Gateway</span><span class="port-badge pb-b">Eureka</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Messaging</div><div class="ptg-badges"><span class="port-badge pb-p">Kafka</span><span class="port-badge pb-p">Event-driven Architecture</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Data</div><div class="ptg-badges"><span class="port-badge pb-g">MariaDB</span><span class="port-badge pb-g">MongoDB</span><span class="port-badge pb-g">Redis · Valkey</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">AI &amp; Automation</div><div class="ptg-badges"><span class="port-badge pb-r">OCR</span><span class="port-badge pb-r">n8n</span><span class="port-badge pb-r">Layout Analysis</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Infra</div><div class="ptg-badges"><span class="port-badge pb-o">Docker</span><span class="port-badge pb-o">Kubernetes</span><span class="port-badge pb-o">Jenkins</span></div></div>
</div>

---

## 기술 의사결정

### 1. 인증 상태와 화면 접근 제어를 함께 관리

로그인·회원가입·로그아웃 API를 화면과 연결하고, JWT 액세스 토큰과 리프레시 토큰의 갱신 흐름을 조정했습니다. 비로그인 사용자가 접근할 수 있는 화면은 라우터 가드로 구분했습니다.

<div class="port-decision">
인증 실패를 각 화면에서 개별 처리하면 상태가 쉽게 달라질 수 있습니다. 토큰 저장·갱신과 라우팅 기준을 공통 흐름으로 두어 사용자가 로그인 상태를 예측할 수 있도록 했습니다.
</div>

### 2. 조직과 역할을 계정 생성 시점부터 연결

관리자가 사용자를 생성할 때 조직 정보가 함께 연결되도록 수정하고, General Manager 역할과 조직 구성원 관리 기능을 추가했습니다.

<div class="port-decision">
캠페인 참여와 관리 기능은 개인 계정만으로 판단하기보다 소속 조직과 역할을 함께 알아야 합니다. 이후 권한 확인과 캠페인 참여 흐름에서 사용할 기준을 계정 생성 단계부터 갖추도록 구현했습니다.
</div>

### 3. AI 판사 기능을 별도 모듈로 분리

기존 애플리케이션에서 처리하던 파일 업로드, OCR, 레이아웃 분석, 자동화 도구 호출을 AI 판사 모듈로 옮기고, 기존 애플리케이션과 요청·응답 흐름을 연결했습니다.

<div class="port-decision">
검수 처리 로직을 별도 모듈에 두면 AI 기능의 변경이 캠페인·계정 기능에 미치는 영향을 줄일 수 있습니다. 다만 분리된 서버가 검수에 필요한 권한 정보를 어떻게 확보할지를 함께 해결해야 했습니다.
</div>

### 4. Kafka 이벤트로 AI 판사 서버의 권한 정보를 동기화

캠페인이 생성되거나 참여자 권한이 변경될 때 Kafka 이벤트를 발행하고, AI 판사 서버가 이를 소비해 MongoDB에 캠페인·권한 정보를 저장하도록 구성했습니다.

<div class="port-decision">
검수 요청마다 기존 애플리케이션에 권한 정보를 동기 호출하지 않고, AI 판사 서버가 동기화된 정보를 바탕으로 검수 요청을 처리할 수 있게 했습니다. 서비스 분리에서 데이터 전달 방식과 정합성 기준을 함께 설계한 경험입니다.
</div>

### 5. AI 분석 결과는 메타데이터와 상세 결과를 나누어 조회

검수 결과의 메타데이터를 저장하고, OCR·레이아웃 분석을 포함한 상세 결과는 MongoDB에서 조회할 수 있도록 API와 검수 화면을 연결했습니다.

<div class="port-decision">
목록에서 필요한 처리 상태와 상세 분석 결과의 조회 요구가 다르므로, 결과를 저장한 뒤 요청 식별자와 캠페인 정보를 기준으로 상세 화면까지 연결해 사용자가 검수 과정을 추적할 수 있도록 했습니다.
</div>

---
