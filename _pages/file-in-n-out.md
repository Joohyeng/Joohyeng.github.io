---
title: "FileInNOut — 실시간 파일·문서 협업 플랫폼"
permalink: /project/file-in-n-out/
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
  <div class="port-meta-item"><div class="port-meta-label">유형</div><div class="port-meta-value">팀 파일 협업 웹 서비스</div></div>
  <div class="port-meta-item"><div class="port-meta-label">핵심 경험</div><div class="port-meta-value">실시간 동기화 · 파일 관리</div></div>
  <div class="port-meta-item"><div class="port-meta-label">구성</div><div class="port-meta-value">Frontend · Backend · Infra</div></div>
  <div class="port-meta-item"><div class="port-meta-label">GitHub</div><div class="port-meta-value"><a href="https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File" target="_blank" rel="noopener noreferrer">Repository</a></div></div>
</div>

## 프로젝트 개요

- 팀이 파일과 문서를 한곳에서 관리하고, 구성원과 즉시 소통할 수 있도록 만든 협업 플랫폼
- 폴더·파일 관리, 공유 문서 공동 편집, 채팅, 알림을 하나의 흐름으로 연결
- 파일의 바이너리 데이터와 서비스 메타데이터를 분리하고, 실시간 상태 변경을 화면에 반영하는 구조를 설계

## 담당 영역

현재는 프로젝트 전체 구조와 기술 의사결정 중심으로 정리했습니다. 개인별 세부 기여 기능, 화면 캡처, 시연 영상은 포트폴리오 제출 전 확인한 뒤 이 섹션에 추가할 예정입니다.

- **파일 관리** — 파일·폴더의 생성, 이동, 공유 상태를 서비스 데이터와 연결
- **실시간 협업** — 문서 변경 사항과 채팅·알림을 사용자 화면에 전달하는 흐름 구성
- **배포 환경** — 컨테이너 기반 빌드·배포 흐름과 서비스 운영 환경을 구성

## 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span><span class="port-badge pb-b">TypeScript</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Security</span><span class="port-badge pb-b">JPA</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Real-time</div><div class="ptg-badges"><span class="port-badge pb-p">WebSocket · STOMP</span><span class="port-badge pb-p">SSE</span><span class="port-badge pb-p">Yjs · CRDT</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Data &amp; Storage</div><div class="ptg-badges"><span class="port-badge pb-g">MariaDB</span><span class="port-badge pb-g">Redis</span><span class="port-badge pb-g">MinIO</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Infra</div><div class="ptg-badges"><span class="port-badge pb-o">Docker</span><span class="port-badge pb-o">Kubernetes</span><span class="port-badge pb-o">Jenkins</span><span class="port-badge pb-o">Helm</span></div></div>
</div>

---

## 기술 의사결정

### 1. 공동 편집에는 CRDT 기반 동기화 모델을 사용

여러 사용자가 같은 문서를 동시에 수정하면 단순한 마지막 저장 방식은 이전 변경을 덮어쓰게 됩니다. FileInNOut은 문서의 변경을 작은 연산 단위로 동기화하고, 각 사용자가 같은 상태로 수렴하도록 하는 CRDT 모델을 사용했습니다.

<div class="port-decision">
Yjs는 문서 변경을 공유 가능한 업데이트로 다루고, 네트워크 지연이나 일시적인 연결 끊김이 있어도 변경 이력을 병합할 수 있습니다. 단일 편집 서버가 "최신 문서"를 판정하는 방식보다 협업 상황에 자연스럽고, WebSocket 연결을 통해 변경을 빠르게 전달할 수 있습니다.
</div>

### 2. REST API와 실시간 채널의 책임을 분리

파일 목록 조회, 권한 확인, 파일 메타데이터 수정처럼 요청-응답이 명확한 작업은 REST API로 처리했습니다. 반대로 채팅 메시지, 문서 편집 상태, 알림처럼 서버에서 즉시 전달해야 하는 정보는 WebSocket·STOMP 또는 SSE 채널로 분리했습니다.

<div class="port-decision">
HTTP는 자원 조회·변경의 명확한 계약에 적합하고, WebSocket은 연결을 유지하면서 양방향 이벤트를 전달하기에 적합합니다. 역할을 구분하면 각 기능의 오류 처리와 확장 방식을 독립적으로 설계할 수 있습니다.
</div>

### 3. 파일 바이너리와 서비스 메타데이터를 분리

업로드한 파일 자체는 객체 스토리지에 두고, 파일명·경로·소유자·권한·버전 같은 서비스 정보는 데이터베이스에 저장하는 구조를 선택했습니다.

<div class="port-decision">
대용량 바이너리를 관계형 데이터베이스에 직접 저장하면 백업·조회·확장 부담이 커집니다. MinIO 같은 객체 스토리지는 파일 저장과 전송에 집중하고, 서비스 DB는 검색·권한·관계 데이터에 집중하도록 역할을 나누어 운영할 수 있습니다.
</div>

### 4. 컨테이너 기반 배포 흐름 구성

개발 환경과 배포 환경의 차이에서 발생할 수 있는 문제를 줄이기 위해 서비스를 컨테이너 이미지로 만들고, CI/CD 파이프라인을 통해 동일한 이미지를 배포하는 흐름을 구성했습니다.

- Docker로 서비스 실행 환경을 표준화
- Jenkins에서 빌드·테스트·이미지 생성 단계를 자동화
- Kubernetes와 Helm으로 서비스 설정 및 배포 단위를 관리

---
