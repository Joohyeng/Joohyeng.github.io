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

### 1. JWT·조직·역할을 함께 관리해 B2B 서비스의 접근 범위를 명확화

Callog에서는 개인 사용자뿐 아니라 관리자, General Manager, 캠페인 참여자가 서로 다른 범위의 정보를 다룹니다. 로그인 여부만 확인하면 같은 조직의 구성원인지, 어떤 역할로 캠페인에 참여하는지에 따라 달라져야 하는 기능의 기준이 모호해질 수 있었습니다.

<div class="port-decision">
로그인·회원가입·로그아웃 API를 화면과 연결하고, JWT 액세스 토큰과 리프레시 토큰의 갱신 흐름을 조정했습니다. 관리자가 계정을 생성할 때 조직 정보를 함께 연결하고 General Manager 역할과 구성원 관리 기능을 추가했으며, 프론트엔드에서는 Router Guard로 비로그인 화면 접근을 구분했습니다. 이후 캠페인 참여와 권한 확인에서 사용할 기준을 계정 생성 단계부터 갖추고자 했습니다.
</div>

### 2. AI 판사 모듈을 분리해 검수 처리의 변경 범위를 제한

AI 검수에는 파일 업로드, OCR, 레이아웃 분석, 자동화 도구 호출처럼 캠페인·계정 기능과 다른 처리 특성을 가진 작업이 포함됩니다. 이 기능을 기존 애플리케이션 안에서 계속 확장하면 검수 로직의 변경이 다른 도메인에도 영향을 주고, 처리 단계와 결과 저장 위치를 파악하기 어려워질 수 있었습니다.

<div class="port-decision">
파일 업로드·OCR·레이아웃 분석·자동화 도구 호출을 AI 판사 모듈로 옮기고, 기존 애플리케이션과 요청·응답 흐름을 연결했습니다. 검수 결과는 메타데이터와 상세 분석 결과로 나누어 저장하고, 요청 식별자와 캠페인 정보를 기준으로 상세 조회 API와 화면까지 연결했습니다. 기능을 분리하는 동시에 사용자가 검수 결과를 추적하는 흐름은 유지하는 것이 목표였습니다.
</div>

### 3. Kafka·MongoDB로 AI 판사 서버에 필요한 권한 정보를 동기화

AI 판사 모듈을 분리한 뒤에는 검수 요청자의 캠페인 접근 권한을 어떻게 확인할지가 과제가 되었습니다. 검수 요청마다 기존 애플리케이션에 권한 정보를 동기 호출하면 서비스가 분리되어도 호출 의존성이 남고, 기존 애플리케이션의 지연이나 장애가 검수 처리에도 직접 영향을 줄 수 있습니다.

<div class="port-decision">
캠페인이 생성되거나 참여자 권한이 변경될 때 Kafka 이벤트를 발행하고, AI 판사 서버가 이를 소비해 MongoDB에 캠페인·권한 정보를 저장하도록 구성했습니다. AI 판사 서버는 동기화된 정보를 바탕으로 검수 요청을 처리하므로, 권한 확인을 위해 기존 애플리케이션을 매번 호출하지 않아도 됩니다. 이 과정에서 서비스 분리는 서버를 나누는 작업만이 아니라, 필요한 데이터를 언제·어떤 방식으로 전달하고 정합성을 관리할지까지 함께 결정해야 한다는 점을 배웠습니다.
</div>

---
