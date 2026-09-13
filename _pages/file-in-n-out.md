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
  <div class="port-meta-item"><div class="port-meta-label">핵심 경험</div><div class="port-meta-value">실시간 채팅 · 알림 · 조회 흐름</div></div>
  <div class="port-meta-item"><div class="port-meta-label">구성</div><div class="port-meta-value">Frontend · Backend · Infra</div></div>
  <div class="port-meta-item"><div class="port-meta-label">GitHub</div><div class="port-meta-value"><a href="https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File" target="_blank" rel="noopener noreferrer">Repository</a></div></div>
</div>

## 프로젝트 개요

- 팀이 파일과 문서를 한곳에서 관리하고, 구성원과 즉시 소통할 수 있도록 만든 협업 플랫폼
- 폴더·파일 관리, 공유 문서 공동 편집, 채팅, 알림을 하나의 흐름으로 연결
- 파일의 바이너리 데이터와 서비스 메타데이터를 분리하고, 실시간 상태 변경을 화면에 반영하는 구조를 설계

## 담당 영역

팀 개발에서 채팅·알림 기능의 프론트엔드·백엔드 연결을 담당했습니다. 메시지 전송뿐 아니라 읽음 상태, 첨부 파일, 알림이 사용자 화면에서 함께 동작하도록 구현했습니다.

- **실시간 채팅 API·이벤트** — 채팅방·참여자·메시지 API를 구현하고, WebSocket/STOMP로 메시지와 읽음 상태를 실시간 전달
- **채팅 사용자 경험** — 이전 메시지를 위로 불러오는 무한 스크롤에 스크롤 위치 보정을 적용하고, 임시 메시지·중복 방지·이미지·파일 메시지 UI를 구현
- **알림 흐름** — 서비스 화면에서는 SSE 기반 인앱 알림을, 백그라운드에서는 Web Push 알림을 연결하고 읽지 않은 메시지 상태를 화면에 반영
- **조회 흐름 개선** — 채팅방 목록의 마지막 메시지·읽지 않은 수·참여자 정보를 함께 조회하는 과정에서 반복 조회를 줄이기 위해 페이지네이션과 조회 쿼리 개선을 적용

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

### 1. REST API와 WebSocket/STOMP의 역할 분리

채팅방에 처음 진입할 때 필요한 메시지 이력은 REST API로 조회하고, 접속 이후 새로 발생하는 메시지와 읽음 상태는 WebSocket/STOMP로 전달하도록 구현했습니다.

<div class="port-decision">
조회는 요청·응답 형태로 페이지 단위를 명확하게 관리하고, 실시간 이벤트는 연결을 유지한 채 즉시 전달해야 합니다. 두 흐름을 분리해 채팅 이력 조회와 실시간 수신의 책임을 구분했습니다.
</div>

### 2. IntersectionObserver·DB 페이지네이션 기반 위쪽 무한 스크롤

이전 메시지를 불러올 때 기존 메시지 위에 데이터를 추가하면 화면이 밀려 사용자가 읽던 위치를 잃을 수 있습니다. 메시지를 불러오기 전후의 스크롤 높이를 비교해 `scrollTop`을 보정하고, DB 페이지네이션으로 과거 메시지를 나누어 조회했습니다.

<div class="port-decision">
메시지 전체를 한 번에 가져오는 방식보다 초기 로딩 부담을 줄일 수 있고, 사용자는 과거 대화를 확인한 뒤에도 현재 읽던 문맥을 유지할 수 있습니다.
</div>

### 3. 낙관적 UI와 임시 식별자 기반 메시지 정합

메시지를 보낸 뒤 서버 응답을 기다리기만 하면 입력 직후 화면이 비어 보일 수 있습니다. 클라이언트에서는 임시 식별자를 가진 메시지를 먼저 표시하고, 서버에서 전달한 메시지를 받으면 이를 실제 메시지로 교체하도록 구현했습니다.

<div class="port-decision">
빠른 연속 전송에서 같은 메시지가 중복되거나 순서가 어긋날 수 있어, 임시 메시지와 서버 메시지를 구분하고 서버 식별자를 기준으로 화면 상태를 정리했습니다.
</div>

### 4. SSE·Service Worker·Web Push 기반 알림 분기

서비스 화면을 보고 있을 때는 SSE로 인앱 알림을 전달하고, 브라우저가 백그라운드에 있을 때는 Service Worker와 Web Push를 통해 알림을 받을 수 있도록 연결했습니다.

<div class="port-decision">
동일한 알림을 무조건 노출하기보다 사용자가 현재 서비스를 보고 있는지에 따라 전달 방식을 구분해, 메시지를 놓치지 않으면서도 불필요한 알림을 줄이고자 했습니다.
</div>

---
