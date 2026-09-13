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

### 직접 사용한 기술

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span><span class="port-badge pb-b">JavaScript</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Security</span><span class="port-badge pb-b">JPA</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Real-time &amp; Notification</div><div class="ptg-badges"><span class="port-badge pb-p">WebSocket · STOMP</span><span class="port-badge pb-p">SSE</span><span class="port-badge pb-p">Web Push</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Data</div><div class="ptg-badges"><span class="port-badge pb-g">MariaDB</span><span class="port-badge pb-g">Redis</span></div></div>
</div>

### 프로젝트 공통 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Security</span><span class="port-badge pb-b">JPA</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Real-time Collaboration</div><div class="ptg-badges"><span class="port-badge pb-p">WebSocket · STOMP</span><span class="port-badge pb-p">SSE</span><span class="port-badge pb-p">Yjs · CRDT</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Data &amp; Storage</div><div class="ptg-badges"><span class="port-badge pb-g">MariaDB</span><span class="port-badge pb-g">Redis</span><span class="port-badge pb-g">MinIO</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Infra</div><div class="ptg-badges"><span class="port-badge pb-o">Docker</span><span class="port-badge pb-o">Kubernetes</span><span class="port-badge pb-o">Jenkins</span><span class="port-badge pb-o">Helm</span></div></div>
</div>

---

## 기술 의사결정

### 1. REST API와 STOMP를 함께 사용해 채팅의 조회·전송 책임을 분리

채팅방에 처음 들어왔을 때 필요한 것은 이전 메시지 목록, 참여자 정보, 읽지 않은 메시지 수처럼 현재 상태를 조회하는 일입니다. 반면 대화 중에는 새 메시지와 읽음 상태가 요청 없이도 즉시 화면에 도착해야 합니다. 모든 동작을 하나의 통신 방식으로 처리하면 이력 조회의 페이지 처리와 실시간 이벤트의 재연결·중복 처리가 서로 얽힐 수 있다고 판단했습니다.

<div class="port-decision">
메시지 이력은 REST API로 페이지 단위 조회하고, 새 메시지·읽음 상태는 WebSocket/STOMP로 전달했습니다. 전송 직후에는 임시 식별자를 가진 메시지를 먼저 보여 주고, 서버에서 전달한 실제 메시지로 교체해 응답을 기다리는 공백과 빠른 연속 전송에서 발생할 수 있는 중복 표시를 줄였습니다.
</div>

### 2. IntersectionObserver·DB 페이지네이션으로 긴 대화에서도 읽던 위치를 유지

메시지가 누적된 채팅방에서 전체 이력을 한 번에 가져오면 초기 로딩이 길어지고, 사용자가 과거 대화로 이동할수록 화면 부담도 커집니다. 단순히 위쪽에 이전 메시지를 추가하는 방식도 화면을 아래로 밀어 사용자가 읽던 문맥을 잃게 만드는 문제가 있었습니다.

<div class="port-decision">
상단 감지에는 IntersectionObserver를 사용하고, 서버에서는 DB 페이지네이션으로 일정 수의 이전 메시지만 조회했습니다. 데이터를 추가하기 전후의 스크롤 높이 차이만큼 `scrollTop`을 보정해, 사용자는 위로 스크롤해 과거 메시지를 불러와도 방금 읽던 위치를 유지할 수 있도록 했습니다.
</div>

### 3. SSE·Service Worker·Web Push를 조합해 사용자의 화면 상태에 맞게 알림 전달

사용자가 서비스 화면을 보고 있을 때와 브라우저를 닫거나 다른 탭을 보고 있을 때는 알림을 받는 목적이 다릅니다. 전자는 화면 안에서 어떤 채팅방에 변화가 생겼는지 빠르게 확인하면 되고, 후자는 서비스 밖에서도 새 활동을 놓치지 않아야 합니다. 하나의 채널로 처리하면 화면을 보고 있는 중에도 브라우저 알림이 반복되거나, 백그라운드에서 알림을 놓칠 수 있습니다.

<div class="port-decision">
서비스 화면에서는 SSE로 인앱 알림과 읽지 않은 상태를 갱신하고, 백그라운드에서는 Service Worker와 Web Push로 알림을 표시하도록 연결했습니다. 같은 이벤트라도 사용자 상태에 따라 전달 방식을 분기해, 즉시성은 유지하면서 불필요한 중복 알림을 줄이고자 했습니다.
</div>

---
