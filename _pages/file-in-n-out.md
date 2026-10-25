---
title: "FileInNOut — 실시간 파일·문서 협업 플랫폼"
permalink: /project/file-in-n-out/
layout: single
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
  <div class="port-meta-item"><div class="port-meta-label">GitHub</div><div class="port-meta-value"><a href="https://github.com/Joohyeng/File-In-N-Out" target="_blank" rel="noopener noreferrer">코드·프로젝트 문서 ↗</a></div></div>
</div>

## 프로젝트 개요

FileInNOut는 팀이 파일과 문서를 한곳에서 관리하고, 채팅과 알림으로 소통하는 협업 플랫폼입니다. 파일 공유, 문서 공동 편집과 대화를 같은 서비스 안에서 이어갈 수 있도록 구성했습니다.

저는 **실시간 채팅·알림의 프론트엔드와 백엔드**, 채팅방 목록과 메시지 이력의 조회 흐름을 담당했습니다.

## 담당 영역

팀 개발에서 채팅·알림 기능의 프론트엔드·백엔드 연결을 담당했습니다. 메시지 전송뿐 아니라 읽음 상태, 첨부 파일, 알림이 사용자 화면에서 함께 동작하도록 구현했습니다.

- **실시간 채팅 API·이벤트** — 채팅방·참여자·메시지 API를 구현하고, WebSocket/STOMP로 메시지와 읽음 상태를 실시간 전달
- **채팅 사용자 경험** — 이전 메시지를 위로 불러오는 무한 스크롤에 스크롤 위치 보정을 적용하고, 임시 메시지·중복 방지·이미지·파일 메시지 UI를 구현
- **알림 흐름** — 서비스 화면에서는 SSE 기반 인앱 알림을, 백그라운드에서는 Web Push 알림을 연결하고 읽지 않은 메시지 상태를 화면에 반영
- **조회 흐름 개선** — 채팅방 목록의 마지막 메시지·읽지 않은 수·참여자 정보를 함께 조회하는 과정에서 반복 조회를 줄이기 위해 페이지네이션과 조회 쿼리 개선을 적용

## 메시지와 화면 상태의 처리 흐름

| 사용자가 하는 일 | 사용한 방식 | 담당한 처리 |
| --- | --- | --- |
| 채팅방에 들어가 이전 대화 확인 | REST API·DB 페이지네이션 | 메시지 이력과 참여 정보를 조회 |
| 새 메시지 전송·수신 | WebSocket·STOMP | 임시 메시지를 먼저 표시하고 서버의 실제 메시지로 교체 |
| 과거 대화로 스크롤 | IntersectionObserver | 이전 메시지 20개씩 조회하고 추가 전후 높이 차이로 위치 보정 |
| 읽음 상태와 새 활동 확인 | STOMP·SSE·Web Push | 읽음 상태 갱신, 인앱 알림과 백그라운드 알림 연결 |

아래 기술 의사결정에는 이 흐름을 구현한 코드와 조회 개선 근거를 연결했습니다.

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

[채팅 메시지 처리 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageService.java) · [화면의 메시지 상태 처리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/components/ChatRoom.vue)

### 2. IntersectionObserver·DB 페이지네이션으로 긴 대화에서도 읽던 위치를 유지

메시지가 누적된 채팅방에서 전체 이력을 한 번에 가져오면 초기 로딩이 길어지고, 사용자가 과거 대화로 이동할수록 화면 부담도 커집니다. 단순히 위쪽에 이전 메시지를 추가하는 방식도 화면을 아래로 밀어 사용자가 읽던 문맥을 잃게 만드는 문제가 있었습니다.

<div class="port-decision">
상단 감지에는 IntersectionObserver를 사용하고, 서버에서는 DB 페이지네이션으로 일정 수의 이전 메시지만 조회했습니다. 데이터를 추가하기 전후의 스크롤 높이 차이만큼 `scrollTop`을 보정해, 사용자는 위로 스크롤해 과거 메시지를 불러와도 방금 읽던 위치를 유지할 수 있도록 했습니다.
</div>

**구현 결과:** 메시지를 페이지 단위로 불러오고, 위쪽에 이전 메시지가 추가된 뒤에도 읽던 위치를 이어갈 수 있도록 처리했습니다. 요청 중 중복 호출과 마지막 페이지 이후의 추가 조회도 상태값으로 제어했습니다.

[무한 스크롤·위치 보정 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/components/ChatRoom.vue#L189-L245)

### 3. SSE·Service Worker·Web Push를 조합해 사용자의 화면 상태에 맞게 알림 전달

사용자가 서비스 화면을 보고 있을 때와 브라우저를 닫거나 다른 탭을 보고 있을 때는 알림을 받는 목적이 다릅니다. 전자는 화면 안에서 어떤 채팅방에 변화가 생겼는지 빠르게 확인하면 되고, 후자는 서비스 밖에서도 새 활동을 놓치지 않아야 합니다. 하나의 채널로 처리하면 화면을 보고 있는 중에도 브라우저 알림이 반복되거나, 백그라운드에서 알림을 놓칠 수 있습니다.

<div class="port-decision">
서비스 화면에서는 SSE로 인앱 알림과 읽지 않은 상태를 갱신하고, 백그라운드에서는 Service Worker와 Web Push로 알림을 표시하도록 연결했습니다. 같은 이벤트라도 사용자 상태에 따라 전달 방식을 분기해, 즉시성은 유지하면서 불필요한 중복 알림을 줄이고자 했습니다.
</div>

[알림 서비스 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/notification/NotificationService.java) · [Web Push 구독 연계](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/utils/pushNotification.js)

### 4. 채팅방 목록의 N+1 조회 병목을 페이지네이션·집계 쿼리·Redis로 개선

채팅방 목록에는 마지막 메시지, 읽지 않은 메시지 수, 참여자 수를 함께 표시해야 했습니다. 초기 구현에서는 채팅방을 조회한 뒤 각 채팅방의 부가 정보를 다시 조회하면서, 채팅방 수만큼 데이터베이스 접근이 반복되는 N+1 문제가 발생했습니다. 목록의 부가 정보를 방마다 개별 조회하는 구조를 개선해야 했습니다.

<div class="port-decision">
목록을 DB 페이지네이션으로 나눈 뒤 해당 페이지의 채팅방 식별자를 모았습니다. 참여자 수와 읽지 않은 메시지 수는 이 식별자 묶음을 기준으로 집계하고, 마지막 메시지는 채팅방에 저장된 요약 정보를 활용했습니다. 반복 요청에는 사용자·페이지·크기별 Redis 캐시를 사용하고, 메시지 전송·읽음 처리 등 목록에 영향을 주는 변경이 생기면 해당 참여자의 캐시를 무효화하도록 연결했습니다.
</div>


| 개선한 부분 | 실제 구현 방식 |
| --- | --- |
| 한 번에 조회하는 범위 | `PageRequest.of(page, size)`로 채팅방 목록을 페이지 단위 조회 |
| 참여자 수 | 페이지에 포함된 `roomIds`를 묶어 `IN`·`GROUP BY`로 집계 |
| 읽지 않은 메시지 수 | 사용자별 마지막 읽음 식별자와 입장 시각을 조건으로 방별 집계 |
| 마지막 메시지 | 채팅방의 `lastMessage`·`lastMessageTime` 요약 필드 활용 |
| 반복 요청과 변경 반영 | 10초 TTL 캐시를 사용하고 전송·읽음 처리 시 참여자의 캐시 무효화 |

**구현 결과:** 참여자 수와 읽지 않은 메시지 수를 방별로 반복 조회하던 부분을 묶음 집계로 변경하고, 페이지 크기와 캐시를 통해 목록 조회 범위를 제어했습니다. 캐시가 있는 경우의 조회 경로와 데이터 변경 시의 무효화를 함께 구성했습니다.

[채팅방 목록 조합 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatRoomService.java#L171-L238) · [읽지 않은 수 집계 쿼리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageRepository.java#L62-L77) · [참여자 수 집계 쿼리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ParticipantsRepository.java#L39-L50) · [캐시 키·TTL·무효화 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatListCacheService.java)

## 구현 근거와 프로젝트 문서

- [개인 저장소 — 코드와 프로젝트 문서](https://github.com/Joohyeng/File-In-N-Out)
- [팀 프로젝트 소개·시스템 아키텍처·배포 구성](https://github.com/Joohyeng/File-In-N-Out/blob/main/README.md)
- [Swagger·OpenAPI 문서](https://github.com/Joohyeng/File-In-N-Out/tree/main/backend/docs)
- [메시지 이력 조회 부하 테스트 스크립트](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/ngrinder/chat/history.groovy)
- [팀 원본 저장소](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File)

부하 테스트 스크립트에서는 메시지 이력 조회의 검증 시나리오를 확인할 수 있습니다.
