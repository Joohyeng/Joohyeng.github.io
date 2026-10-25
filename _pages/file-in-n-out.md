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

## 채팅 기능 시연

기존 Frontend README에 등록된 채팅 시연입니다. 제가 담당한 채팅 화면과 메시지를 주고받는 흐름을 볼 수 있습니다.

<figure>
  <a href="/assets/images/projects/fileinnout-chat-demo.gif" target="_blank" rel="noopener noreferrer">
    <img src="/assets/images/projects/fileinnout-chat-demo.gif" alt="FileInNOut 채팅 화면과 메시지 송수신을 보여주는 기존 기능 시연" loading="lazy" />
  </a>
  <figcaption>팀 프로젝트의 기존 채팅 시연. 조회 구조의 변경 전후는 아래 작업 커밋을 근거로 설명합니다.</figcaption>
</figure>

[시연 출처 — Frontend README의 채팅 기능](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/README.md) · [채팅 화면 구현 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/components/ChatRoom.vue)

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

아래에는 조회 구조의 변경 전후와, 이 흐름에서 통신 기술의 역할을 나눈 이유를 정리했습니다.

## 문제 해결과 기술 의사결정

### 1. DB 페이지네이션·집계 쿼리·Redis로 채팅 조회의 반복 처리 개선

초기 채팅방 목록은 사용자가 참여한 방을 모두 조회한 뒤, 방마다 읽지 않은 메시지 수와 마지막 메시지를 개별 조회했습니다. 페이지 구분도 전체 목록을 가져온 후 Java에서 잘라내는 방식이어서, 한 페이지를 요청해도 모든 방의 부가 정보를 조회하는 구조였습니다. 메시지 이력에서도 각 메시지의 미읽음 수와 발신자 프로필 이미지를 반복 조회했습니다.

<div class="port-decision">
목록을 DB 페이지네이션으로 나눈 뒤 해당 페이지의 채팅방 식별자를 모았습니다. 참여자 수와 읽지 않은 메시지 수는 이 식별자 묶음을 기준으로 집계하고, 마지막 메시지는 채팅방에 저장된 요약 정보를 활용했습니다. 반복 요청에는 사용자·페이지·크기별 Redis 캐시를 사용하고, 메시지 전송·읽음 처리 등 목록에 영향을 주는 변경이 생기면 해당 참여자의 캐시를 무효화하도록 연결했습니다.
</div>

메시지 이력도 현재 페이지의 메시지 식별자를 모아 미읽음 수를 한 번에 집계하도록 바꿨습니다. 발신자 프로필 이미지 URL은 요청 내 Map에 저장해, 같은 발신자의 URL을 다시 사용할 수 있도록 했습니다.

아래는 직접 작성한 커밋(`772b934`, 작성자 `kjh`)에서 확인할 수 있는 변경 전후입니다.

| 처리 대상 | 변경 전 | 변경 후 |
| --- | --- | --- |
| 채팅방 목록의 페이지 구분 | 전체 목록과 부가 정보를 조회한 뒤 `subList`로 자름 | `PageRequest`로 DB에서 페이지 조회 후 해당 방의 정보 조합 |
| 채팅방별 미읽음 수 | 방마다 개별 count 조회 | 현재 페이지의 `roomIds`를 묶어 집계 |
| 채팅방별 마지막 메시지 | 방마다 마지막 메시지 개별 조회 | 채팅방의 `lastMessage`·`lastMessageTime` 요약 필드 활용 |
| 메시지별 미읽음 수 | 메시지마다 개별 count 조회 | 현재 페이지의 `messageIds`를 묶어 집계 |
| 발신자 프로필 이미지 | 메시지마다 URL 조회 | 요청 내 Map으로 같은 발신자의 URL 재사용 |
| 같은 목록의 반복 요청 | 요청마다 목록 조합 | 사용자·페이지·크기별 10초 TTL 캐시와 변경 시 무효화 추가 |

**전체 조회 후 페이지 구분 → DB 페이지 조회**

기존 코드는 전체 목록을 기준으로 미읽음 수와 마지막 메시지를 계산한 뒤, 아래처럼 Java에서 필요한 구간을 잘랐습니다.

```java
int start = page * size;
int end = Math.min(start + size, sorted.size());
List<ChatParticipants> paged = sorted.subList(start, end);
```

변경 후에는 조회 단계에 페이지 조건을 전달하고, 반환된 페이지의 채팅방 식별자로 부가 정보를 조합했습니다.

```java
Pageable pageable = PageRequest.of(page, size);
Page<ChatParticipants> result =
        participantsRepository.findAllByUsersIdx(userIdx, pageable);
```

**메시지별 미읽음 수 조회 → 미리 집계한 결과 사용**

메시지 목록도 현재 페이지의 `messageIds`로 미읽음 수를 집계한 Map을 만든 뒤, 각 메시지를 응답으로 변환할 때 해당 값을 사용하도록 바꿨습니다. 아래는 같은 커밋의 핵심 변경입니다.

```diff
- int messageUnreadCount = chatMessageRepository.countUnreadParticipants(
-         roomIdx, msg.getIdx(), msg.getSender().getIdx()
- );
+ int messageUnreadCount = unreadCountMap.getOrDefault(msg.getIdx(), 0);
```

**구현 결과:** 전체 목록을 가져와 나누던 처리를 DB 페이지 조회로 옮겼고, 방별·메시지별 미읽음 수를 반복 조회하던 코드를 식별자 묶음 단위의 집계로 변경했습니다. 목록 캐시와 데이터 변경 시의 무효화도 함께 구성했습니다.

[직접 작성한 변경 전후 커밋 — `kjh`, 772b934](https://github.com/Joohyeng/File-In-N-Out/commit/772b9340fe5e002396254b70157035f70209376d) · [메시지 목록의 묶음 집계·프로필 URL 재사용 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageService.java#L55-L105)

[채팅방 목록 조합 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatRoomService.java#L171-L238) · [읽지 않은 수 집계 쿼리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageRepository.java#L62-L77) · [참여자 수 집계 쿼리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ParticipantsRepository.java#L39-L50) · [캐시 키·TTL·무효화 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatListCacheService.java)

### 2. REST·STOMP·SSE·Web Push로 조회·대화·알림의 역할을 구분

채팅방에 처음 들어왔을 때 필요한 것은 이전 메시지 목록, 참여자 정보, 읽지 않은 메시지 수처럼 현재 상태를 조회하는 일입니다. 반면 대화 중에는 새 메시지와 읽음 상태가 요청 없이도 즉시 화면에 도착해야 합니다. 모든 동작을 하나의 통신 방식으로 처리하면 이력 조회의 페이지 처리와 실시간 이벤트의 재연결·중복 처리가 서로 얽힐 수 있다고 판단했습니다.

<div class="port-decision">
메시지 이력·업로드·읽음 처리 요청은 REST API로 처리하고, 새 메시지와 읽음 상태 변경의 전파는 WebSocket/STOMP로 연결했습니다. 채팅방마다 목적지를 구독해 같은 방의 이벤트를 받을 수 있도록 했으며, 연결에는 SockJS를 사용했습니다. 전송 직후에는 임시 식별자를 가진 메시지를 먼저 보여 주고, 서버의 실제 메시지로 교체해 응답 대기 중의 공백과 중복 표시를 줄였습니다.
</div>

예를 들어 읽음 처리는 `POST /chat/{roomIdx}/read`로 요청하고, 처리 결과는 `READ_UPDATE` 이벤트로 같은 방 구독자에게 전달합니다. 메시지의 발행 경로와 구독 경로는 각각 `/pub/chat/{roomIdx}`, `/sub/chat/room/{roomIdx}`로 구분했습니다.

첨부 파일은 `POST /chat/{roomIdx}/upload`로 먼저 업로드하고, 반환된 파일 URL과 정보를 포함한 메시지를 WebSocket으로 전달합니다. 파일 전송과 채팅 이벤트를 나누어 처리했습니다.

이렇게 역할을 나누면서, 이전 메시지를 가져오는 페이지 조회와 현재 대화의 실시간 구독을 각각 관리했습니다. 프론트에서는 서버 응답 전의 임시 메시지와 서버가 반환한 메시지를 구분해 처리했습니다.

[채팅 메시지 처리 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatMessageService.java) · [화면의 메시지 상태 처리](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/components/ChatRoom.vue)

#### 화면 상태에 따른 알림 전달

사용자가 서비스 화면을 보고 있을 때와 브라우저를 닫거나 다른 탭을 보고 있을 때는 알림을 받는 목적이 다릅니다. 전자는 화면 안에서 어떤 채팅방에 변화가 생겼는지 빠르게 확인하면 되고, 후자는 서비스 밖에서도 새 활동을 놓치지 않아야 합니다. 하나의 채널로 처리하면 화면을 보고 있는 중에도 브라우저 알림이 반복되거나, 백그라운드에서 알림을 놓칠 수 있습니다.

<div class="port-decision">
서버에서는 알림 저장 후 SSE와 Push를 전달하고, SSE의 `chat-preview-update`·`new-message` 이벤트로 채팅방 목록과 미읽음 상태를 갱신했습니다. Service Worker는 화면이 보이는 클라이언트가 있으면 화면에 이벤트를 전달하고, 보이는 화면이 없을 때 OS 알림을 표시하도록 분기했습니다. 알림 클릭 시 해당 채팅방으로 이동하는 흐름도 연결했습니다.
</div>

SSE는 앱 내부의 단방향 상태 갱신, WebSocket/STOMP는 채팅방의 양방향 메시지, Web Push는 서비스 화면 밖의 알림 수신을 담당합니다.

예를 들어 채팅방 목록의 미리보기 갱신은 서버에서 화면으로 전달하는 이벤트이므로 SSE로 처리했습니다. 메시지 본문은 방 단위 STOMP 구독으로 전달하고, Service Worker에서는 보이는 화면이 있는지 확인해 알림 표시를 분기했습니다. 이 구분은 팀 위키의 통신 기술별 선택 이유와 실제 코드에서 함께 확인할 수 있습니다.

[알림 서비스 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/notification/NotificationService.java) · [Web Push 구독 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/utils/pushNotification.js) · [Service Worker의 화면 상태 분기](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/public/sw.js)

#### Redis TTL로 현재 채팅방 입장 상태 관리

같은 채팅방을 보고 있는 사용자에게 새 메시지 알림까지 반복해서 전달하면 대화 중 불필요한 알림이 발생합니다. 현재 입장 여부는 자주 바뀌는 일시적인 상태이므로, 영구 데이터와 분리해 관리했습니다.

<div class="port-decision">
입장 시 방·사용자별 Redis 키를 5분 TTL로 저장하고, heartbeat로 갱신하며 퇴장 시 삭제하도록 구성했습니다. 메시지를 저장한 뒤에는 이 입장 상태를 확인해 현재 방을 보고 있는 사용자를 알림 대상에서 제외했습니다. 입장 상태와 10초 목록 캐시는 목적과 수명을 나누어 관리했습니다.
</div>

입장 상태는 heartbeat가 끊겨도 TTL이 지나면 만료되도록 했습니다. 목록 캐시는 메시지 전송·읽음 처리·초대 등 데이터 변경 시 관련 사용자의 페이지 키를 삭제합니다. 두 데이터 모두 Redis에 저장하지만, 만료와 갱신을 처리하는 기준은 다르게 구성했습니다.

[입장 상태·TTL 관리 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/src/main/java/com/example/WaffleBear/chat/ChatPresenceService.java) · [실시간 채팅 위키의 설계 설명](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/실시간-채팅-기능)

### 3. IntersectionObserver·DB 페이지네이션으로 긴 대화에서도 읽던 위치를 유지

메시지가 누적된 채팅방에서 전체 이력을 한 번에 가져오면 초기 로딩이 길어지고, 사용자가 과거 대화로 이동할수록 화면 부담도 커집니다. 단순히 위쪽에 이전 메시지를 추가하는 방식도 화면을 아래로 밀어 사용자가 읽던 문맥을 잃게 만드는 문제가 있었습니다.

<div class="port-decision">
상단 감지에는 IntersectionObserver를 사용하고, 서버에서는 DB 페이지네이션으로 일정 수의 이전 메시지만 조회했습니다. 데이터를 추가하기 전후의 스크롤 높이 차이만큼 `scrollTop`을 보정해, 사용자는 위로 스크롤해 과거 메시지를 불러와도 방금 읽던 위치를 유지할 수 있도록 했습니다.
</div>

**구현 결과:** 메시지를 페이지 단위로 불러오고, 위쪽에 이전 메시지가 추가된 뒤에도 읽던 위치를 이어갈 수 있도록 처리했습니다. 요청 중 중복 호출과 마지막 페이지 이후의 추가 조회도 상태값으로 제어했습니다.

[무한 스크롤·위치 보정 코드](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/frontend/src/components/ChatRoom.vue#L189-L245)

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

## 구현 근거와 프로젝트 문서

- [개인 저장소 — 코드와 프로젝트 문서](https://github.com/Joohyeng/File-In-N-Out)
- [팀 프로젝트 소개·시스템 아키텍처·배포 구성](https://github.com/Joohyeng/File-In-N-Out/blob/main/README.md)
- [원본 위키 — 실시간 채팅의 기술별 역할과 메시지 흐름](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File/wiki/실시간-채팅-기능)
- [Swagger·OpenAPI 문서](https://github.com/Joohyeng/File-In-N-Out/tree/main/backend/docs)
- [메시지 이력 조회용 nGrinder 스크립트](https://github.com/Joohyeng/File-In-N-Out/blob/cfde24f3db34e079fb5a45fecb8edb6eacc9c2cd/backend/ngrinder/chat/history.groovy)
- [팀 원본 저장소](https://github.com/beyond-sw-camp/be24-4th-ShakeShackFile-In-N-Out-File)
