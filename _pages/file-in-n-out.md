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
| 이미지·파일 공유 | HTTP 업로드·STOMP 메시지 | 객체 스토리지에 파일을 저장한 뒤 URL과 파일 정보를 메시지에 포함 |
| 과거 대화로 스크롤 | IntersectionObserver | 이전 메시지 20개씩 조회하고 추가 전후 높이 차이로 위치 보정 |
| 메시지 읽음 처리 | REST 요청·STOMP 이벤트 | 마지막 읽은 메시지를 기록하고 방 구독자에게 읽음 상태 반영 |
| 새 활동과 채팅방 미리보기 확인 | SSE·Web Push | 인앱 상태 갱신과 백그라운드 알림 전달 |
| 현재 채팅방을 보고 있는 상태 유지 | Redis·heartbeat | 입장 상태의 TTL을 갱신하고 불필요한 알림 대상에서 제외 |

아래 기술 의사결정에는 이 흐름을 구현한 코드와 조회 개선 근거를 연결했습니다.

## 그룹챗 성능 테스트 비교

### 테스트 대상과 조건

**로그인한 가상 사용자들이 그룹챗의 조회·생성·삭제 작업을 수행하도록 구성한 nGrinder 부하 테스트**입니다. `WaffleBearGroupChatScenario`는 팀 위키에서 이 테스트를 구분하는 스크립트 이름입니다. 위키에 설명된 범위와 원본 보고서에서 확인되는 조건은 다음과 같습니다.

| 구분 | 테스트 내용 |
| --- | --- |
| 인증 | 로그인 후 발급받은 Bearer 토큰으로 기능 API 호출 |
| 대상 작업 | 그룹챗의 조회·생성·삭제가 섞인 흐름 — 팀 위키 설명 기준 |
| 부하 조건 | 개선 전후 모두 **가상 사용자 100명(100 VUser)** 설정 |
| 측정 지표 | 기록 대상 테스트의 초당 처리량(TPS)과 1회 평균 소요 시간 |

### 개선 전후 결과

같은 100 VUser 설정에서 TPS는 **314.7 → 829.2**, 평균 테스트 시간은 **305.52 → 84.28ms**로 기록됐습니다. 그룹챗 관련 작업이 섞인 부하에서, 기록 대상 테스트의 처리량은 늘고 평균 소요 시간은 줄었습니다.

| 지표 | 개선 전 | 개선 후 | 변화 |
| --- | --- | --- | --- |
| 총 VUser | 100 | 100 | 동일 |
| TPS | 314.7 | 829.2 | 약 **2.63배** |
| 최고 TPS | 407.0 | 981.0 | 약 2.41배 |
| 평균 테스트 시간 | 305.52ms | 84.28ms | 약 **72.4% 감소** |
| 총 실행 테스트 | 5,044 | 5,004 | 원본 기록 |
| 성공한 테스트 | 5,044 | 5,004 | 원본 기록 |
| 에러 | 0 | 0 | 두 실행 모두 0건 |
| 동작 시간 | 23초 | 13초 | 원본 기록 |

위키는 그룹챗 개선 항목으로 **메시지 목록의 프로필 이미지 배치 조회와 조회 컬럼의 인덱스 보강**을 정리합니다. 현재 메시지 목록 코드에서도 메시지 식별자별 미읽음 수를 묶어서 집계하고, 같은 발신자의 프로필 이미지 조회 결과를 요청 안에서 재사용하는 구조를 확인할 수 있습니다.

**해석 범위:** 공개된 자료에는 해당 시나리오의 원본 스크립트가 없어, 개별 API의 호출 순서·비중과 테스트 1회로 집계되는 범위까지는 확인되지 않습니다. 따라서 이 수치는 **팀 위키의 그룹챗 부하 테스트 결과**로 제시하며, 메시지 이력 API 하나의 응답시간이나 WebSocket 메시지의 초당 전달 건수로 해석할 수는 없습니다. 두 실행의 총 테스트 횟수와 동작 시간도 다릅니다.

[성능 테스트 원문과 개선 항목](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/성능-테스트#4-7-그룹-챗) · [메시지 목록의 묶음 집계·프로필 결과 재사용 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageService.java#L55-L105)

<details>
  <summary>nGrinder 원본 보고서 — 개선 전·후 화면</summary>
  <figure>
    <a href="/assets/images/projects/fileinnout-group-chat-before.png" target="_blank" rel="noopener noreferrer">
      <img src="/assets/images/projects/fileinnout-group-chat-before.png" alt="그룹챗 개선 전 nGrinder 보고서: 100 VUser, TPS 314.7, 평균 테스트 시간 305.52ms, 에러 0건" loading="lazy" />
    </a>
    <figcaption>개선 전 — 팀 위키 성능 테스트 원본 보고서</figcaption>
  </figure>
  <figure>
    <a href="/assets/images/projects/fileinnout-group-chat-after.png" target="_blank" rel="noopener noreferrer">
      <img src="/assets/images/projects/fileinnout-group-chat-after.png" alt="그룹챗 개선 후 nGrinder 보고서: 100 VUser, TPS 829.2, 평균 테스트 시간 84.28ms, 에러 0건" loading="lazy" />
    </a>
    <figcaption>개선 후 — 팀 위키 성능 테스트 원본 보고서</figcaption>
  </figure>
</details>

## 기술 스택

### 직접 사용한 기술

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span><span class="port-badge pb-b">JavaScript</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Security</span><span class="port-badge pb-b">JPA</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Real-time &amp; Notification</div><div class="ptg-badges"><span class="port-badge pb-p">WebSocket · STOMP</span><span class="port-badge pb-p">SockJS</span><span class="port-badge pb-p">SSE</span><span class="port-badge pb-p">Web Push</span></div></div>
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
메시지 이력·업로드·읽음 처리 요청은 REST API로 처리하고, 새 메시지와 읽음 상태 변경의 전파는 WebSocket/STOMP로 연결했습니다. 채팅방마다 목적지를 구독해 같은 방의 이벤트를 받을 수 있도록 했으며, 연결에는 SockJS를 사용했습니다. 전송 직후에는 임시 식별자를 가진 메시지를 먼저 보여 주고, 서버의 실제 메시지로 교체해 응답 대기 중의 공백과 중복 표시를 줄였습니다.
</div>

예를 들어 읽음 처리는 `POST /chat/{roomIdx}/read`로 요청하고, 처리 결과는 `READ_UPDATE` 이벤트로 같은 방 구독자에게 전달합니다. 메시지의 발행 경로와 구독 경로는 각각 `/pub/chat/{roomIdx}`, `/sub/chat/room/{roomIdx}`로 구분했습니다.

첨부 파일은 `POST /chat/{roomIdx}/upload`로 먼저 업로드하고, 반환된 파일 URL과 정보를 포함한 메시지를 WebSocket으로 전달합니다. 파일 전송과 채팅 이벤트를 나누어 처리했습니다.

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
서버에서는 알림 저장 후 SSE와 Push를 전달하고, SSE의 `chat-preview-update`·`new-message` 이벤트로 채팅방 목록과 미읽음 상태를 갱신했습니다. Service Worker는 화면이 보이는 클라이언트가 있으면 화면에 이벤트를 전달하고, 보이는 화면이 없을 때 OS 알림을 표시하도록 분기했습니다. 알림 클릭 시 해당 채팅방으로 이동하는 흐름도 연결했습니다.
</div>

SSE는 앱 내부의 단방향 상태 갱신, WebSocket/STOMP는 채팅방의 양방향 메시지, Web Push는 서비스 화면 밖의 알림 수신을 담당합니다.

[알림 서비스 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/notification/NotificationService.java) · [Web Push 구독 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/utils/pushNotification.js) · [Service Worker의 화면 상태 분기](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/public/sw.js)

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

### 5. Redis 입장 상태로 현재 대화를 보고 있는 사용자의 알림 제외

같은 채팅방을 보고 있는 사용자에게 새 메시지 알림까지 반복해서 전달하면 대화 중 불필요한 알림이 발생합니다. 현재 입장 여부는 자주 바뀌는 일시적인 상태이므로, 영구 데이터와 분리해 관리했습니다.

<div class="port-decision">
입장 시 방·사용자별 Redis 키를 5분 TTL로 저장하고, heartbeat로 갱신하며 퇴장 시 삭제하도록 구성했습니다. 메시지를 저장한 뒤에는 이 입장 상태를 확인해 현재 방을 보고 있는 사용자를 알림 대상에서 제외했습니다. 입장 상태와 10초 목록 캐시는 목적과 수명을 나누어 관리했습니다.
</div>

[입장 상태·TTL 관리 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatPresenceService.java) · [실시간 채팅 위키의 설계 설명](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/실시간-채팅-기능)

## 구현 근거와 프로젝트 문서

- [개인 저장소 — 코드와 프로젝트 문서](https://github.com/Joohyeng/File-In-N-Out)
- [팀 프로젝트 소개·시스템 아키텍처·배포 구성](https://github.com/Joohyeng/File-In-N-Out/blob/main/README.md)
- [원본 위키 — 성능 테스트와 개선 전후 보고서](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/성능-테스트)
- [원본 위키 — 실시간 채팅의 기술별 역할과 메시지 흐름](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/실시간-채팅-기능)
- [Swagger·OpenAPI 문서](https://github.com/Joohyeng/File-In-N-Out/tree/main/backend/docs)
- [메시지 이력 조회 부하 테스트 스크립트](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/ngrinder/chat/history.groovy)
- [팀 원본 저장소](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File)

위의 `history.groovy`는 메시지 이력 단독 조회를 검증하는 자료이며, 비교 표의 그룹챗 시나리오와는 구분했습니다.
