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

- **AI 리뷰 흐름** — OCR·레이아웃 분석 결과를 활용해 콘텐츠 검토 흐름을 구성하고, 자동화 도구와 연계할 수 있는 확장 지점을 설계
- **이벤트 기반 후속 처리** — 캠페인 생성 이벤트를 발행하고, 메타데이터·권리 처리 서비스가 비동기로 후속 작업을 수행하도록 흐름 구성
- **서비스 기반 구조** — Gateway·서비스 모듈 중심의 API 흐름과 공통 설정을 정리하고, MSA 환경에서 서비스별 책임을 분리
- **배포·운영 환경** — Docker와 Kubernetes, Jenkins 기반의 빌드·배포 흐름을 프로젝트 환경에 적용

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

### 1. 도메인 단위로 서비스를 나누고 Gateway로 진입점을 통합

캠페인, 콘텐츠, 파트너, KPI처럼 변화 속도와 데이터 책임이 다른 기능을 하나의 애플리케이션에 두면 기능 간 의존이 빠르게 커집니다. Callog은 도메인 단위로 서비스를 나누고, 외부 요청은 Gateway를 통해 들어오도록 구성했습니다.

<div class="port-decision">
Gateway는 인증, 라우팅, 공통 정책처럼 모든 요청에 필요한 관심사를 한곳에서 처리하고, 각 도메인 서비스는 자신의 비즈니스 규칙과 데이터에 집중할 수 있게 합니다. 서비스 디스커버리를 함께 사용하면 배포 환경에서 개별 서비스의 위치가 바뀌어도 호출 경로를 유연하게 관리할 수 있습니다.
</div>

### 2. 캠페인 생성 뒤의 작업은 Kafka 이벤트로 연결

캠페인이 만들어진 뒤 메타데이터 생성, 권리 정보 처리, 알림 같은 작업이 필요합니다. 이 과정을 동기 API 호출로 묶으면 캠페인 생성 요청이 모든 후속 서비스의 처리 시간과 장애에 영향을 받습니다.

<div class="port-decision">
캠페인 서비스는 핵심 데이터 저장과 이벤트 발행에 집중하고, 후속 서비스는 필요한 이벤트를 구독해 자신의 작업을 수행하도록 구성했습니다. 이 방식은 생성 요청의 응답 시간을 불필요하게 늘리지 않고, 소비 서비스의 재처리·확장도 독립적으로 검토할 수 있게 합니다.
</div>

### 3. 데이터 성격에 따라 저장소를 분리

관계와 트랜잭션이 중요한 운영 데이터는 관계형 DB로, 유연한 구조의 콘텐츠·분석성 데이터는 문서형 DB로 다루는 방식을 적용했습니다. 빠른 조회가 필요한 상태성 데이터는 Redis·Valkey 사용을 고려했습니다.

<div class="port-decision">
하나의 저장소에 모든 종류의 데이터를 맞추기보다, 각 데이터의 조회 패턴과 일관성 요구사항에 맞는 저장소를 선택하면 모델이 단순해집니다. 대신 서비스 경계를 넘는 데이터 정합성은 이벤트와 재처리 전략으로 관리해야 하므로, 데이터 소유 서비스가 명확해야 합니다.
</div>

### 4. AI 리뷰는 결과를 서비스 흐름에 연결하는 방식으로 설계

AI 기능은 모델 호출 자체보다 결과를 어떤 기준으로 저장하고, 사람이 어떻게 검토하며, 실패했을 때 어떤 상태로 남길지가 서비스 품질에 더 큰 영향을 줍니다. Callog에서는 OCR·레이아웃 분석 결과를 리뷰 흐름에 연결하고, 자동화 도구를 통해 확장 가능한 처리 단계를 마련했습니다.

- 입력 콘텐츠와 분석 결과의 상태를 분리해 추적
- 비동기 처리 결과를 사용자 검토 화면으로 연결
- 실패·재시도·수동 검토가 가능한 운영 흐름을 고려

### 5. 컨테이너 환경에서 일관된 빌드·배포 흐름 유지

각 마이크로서비스를 컨테이너 이미지로 분리하고, Jenkins를 통해 빌드·테스트·배포 과정을 연결했습니다. Kubernetes 환경에서는 서비스별 배포 단위와 설정을 분리하여 운영 변경의 범위를 줄일 수 있도록 구성했습니다.

---
