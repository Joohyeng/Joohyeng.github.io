---
title: "Callog — AI 기반 캠페인 협업·광고물 검수 플랫폼"
permalink: /project/callog/
layout: single
author_profile: false
sidebar:
  nav: "projects_nav"
toc: true
toc_label: "목차"
toc_sticky: true
---

<div class="port-meta-bar">
  <div class="port-meta-item"><div class="port-meta-label">유형</div><div class="port-meta-value">기업 간 캠페인 협업 플랫폼</div></div>
  <div class="port-meta-item"><div class="port-meta-label">팀 구성</div><div class="port-meta-value">5인 팀 프로젝트</div></div>
  <div class="port-meta-item"><div class="port-meta-label">핵심 경험</div><div class="port-meta-value">AI 검수 · Kafka 권한 동기화</div></div>
  <div class="port-meta-item"><div class="port-meta-label">GitHub</div><div class="port-meta-value"><a href="https://github.com/Joohyeng/Callog" target="_blank" rel="noopener noreferrer">코드·프로젝트 문서 ↗</a></div></div>
</div>

## 프로젝트 개요

Callog는 기업들이 캠페인의 일정·자료·참여자를 함께 관리하고, 제작한 광고물의 위험 요소를 AI로 점검하도록 돕는 협업 플랫폼입니다. 기업 매칭, 캠페인 운영, 광고물 검수를 하나의 서비스 안에서 연결했습니다.

저는 **인증·조직 관리와 AI 검수 기능**을 담당했습니다. 특히 파일에서 내용을 추출해 n8n 검수를 요청하고 결과를 조회하는 흐름을 연결했으며, 이후 AI 판사 서비스를 분리하면서 필요한 캠페인·참여자 정보를 Kafka로 전달하도록 구현했습니다.

## 담당 기능

- **AI 검수 처리** — 기존 파일 업로드·OCR·레이아웃 분석과 n8n 호출을 AI 판사 모듈로 이전하고 요청·응답 흐름을 연결
- **결과 저장·조회** — 분석 작업 식별자와 원본 파일·추출 결과·검수 결과를 연결하고, 메타데이터 저장과 상세 조회 API·화면을 구현
- **AI 판사 서비스 분리** — 캠페인 생성·참여자 변경 이벤트를 Kafka로 전달하고, AI 판사 서버가 MongoDB의 권한 정보를 조회해 검수 요청을 처리하도록 구성
- **인증·조직 관리** — 로그인·로그아웃·회원가입·비밀번호 재설정 화면과 API 연결, 리프레시 토큰·Router Guard 처리, 조직 연결·General Manager·구성원 관리 화면을 구현

AI 영역에서는 기존 내용 추출 기능을 AI Judge로 이전하고, n8n 검수의 요청·응답 처리와 결과 저장·조회, 캠페인 정보 동기화를 연결하는 작업을 맡았습니다. 아래 기술 의사결정은 이 담당 범위를 기준으로 설명합니다.

## AI 검수 결과 화면

광고물에서 추출한 내용을 검수한 뒤, 사용자는 **판단 등급과 조치 안내**를 먼저 확인하고 상세 화면에서 **문제 표현·판단 근거·원본 자료**를 함께 볼 수 있습니다. 아래는 기존 기능 시연 영상에서 검수가 완료된 장면을 발췌한 화면입니다.

### 결과 등급과 조치 안내

예시 광고물에 대해 반환된 ‘위험’ 등급과 수정 안내를 보여줍니다. 결과와 함께 내용 추출·OCR·AI 분석 단계의 처리 시간도 확인할 수 있도록 구성했습니다.

<figure>
  <a href="/assets/images/projects/callog-ai-verdict.png" target="_blank" rel="noopener noreferrer">
    <img src="/assets/images/projects/callog-ai-verdict.png" width="600" alt="AI 검수 완료 후 위험 등급, 조치 안내와 단계별 처리 시간을 보여주는 실제 결과 화면" loading="lazy" />
  </a>
  <figcaption>검수 완료 후 표시된 판단 등급과 처리 정보. 이미지를 누르면 원본을 확인할 수 있습니다.</figcaption>
</figure>

### 검수 근거와 원본 자료 상세

상세 조회에서는 등급만 표시하는 대신 원본 광고물과 검수 요약, 문제로 분류한 문구와 그 이유를 같은 화면에서 확인하도록 연결했습니다. 저장한 결과 메타데이터와 파일 참조를 활용하는 화면입니다.

<figure>
  <a href="/assets/images/projects/callog-ai-result-detail.png" target="_blank" rel="noopener noreferrer">
    <img src="/assets/images/projects/callog-ai-result-detail.png" width="760" alt="원본 광고물, 위험 등급, 조치 안내, 검수 요약과 문제 표현을 확인하는 AI 검수 상세 결과" loading="lazy" />
  </a>
  <figcaption>결과 상세 조회 화면. 원본 자료와 AI가 반환한 판단 근거를 함께 제공합니다.</figcaption>
</figure>

[화면 출처 — 팀 README의 AI 자료 검수 시연](https://github.com/Joohyeng/Callog/blob/main/frontend/README.md#ai-자료-검수)

### AI Judge API 시연

팀 Backend README에 등록된 Swagger API 시연입니다. 위의 사용자 화면 시연과 함께 기능 호출 자료로 제시하며, 서비스 분리 이후의 구조 변경은 아래 작업 커밋을 근거로 설명합니다.

<details>
  <summary>기존 AI Judge API 시연 영상 보기</summary>
  <video controls preload="none" playsinline style="width: 100%;" aria-label="팀 README에 등록된 AI Judge Swagger API 시연">
    <source src="https://github.com/user-attachments/assets/85381ed4-03b8-4e69-b65f-856760cfef8b" type="video/mp4" />
    <a href="https://github.com/user-attachments/assets/85381ed4-03b8-4e69-b65f-856760cfef8b">AI Judge API 시연 영상 열기</a>
  </video>
  <p><a href="https://github.com/Joohyeng/Callog/blob/main/backend/README.md">시연 출처 — Backend README의 Swagger - Ai Judge</a></p>
</details>

## AI 검수 처리 흐름

사용자는 광고물을 업로드한 뒤 검수 내역에서 처리 상태와 결과를 확인합니다. 제가 연결한 파일 처리와 결과 조회 흐름을 단계별로 정리했습니다.

| 단계 | 처리 내용 | 다음 단계에 전달하거나 보관하는 정보 |
| --- | --- | --- |
| 파일 접수 | 광고물을 업로드하고 분석 작업 식별자를 생성 | 원본 파일 위치, 파일명·형식, `analysisJobId` |
| 내용 추출 | 파일 형식에 따라 텍스트를 추출하고 OCR·레이아웃 분석 결과를 활용 | 추출 텍스트, 이미지 등 분석 산출물 |
| AI 검수 | 추출한 텍스트를 n8n 웹훅으로 전달하고 검수 응답을 수신 | 검수 상태, 판단 이유, 수정 제안 등 |
| 결과 저장·조회 | 파일·분석 산출물과 결과를 연결해 저장하고 상세 조회 API로 제공 | 결과 메타데이터, 파일 참조, 오류 정보, 단계별 처리 시간 |

검수 요청은 `POST /aijudge/check/file`, 분석 상세 조회는 `GET /aijudge/analyses/{analysisJobId}`로 연결했습니다. Gateway의 라우팅을 거쳐 AI 판사 서버로 전달하며, 캠페인 정보를 포함한 검수 요청에는 동기화된 참여자 정보로 접근 범위를 확인합니다.

[파일 검수 API와 권한 확인 코드](https://github.com/Joohyeng/Callog/blob/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8/backend/aijudge/src/main/java/com/example/adcheck/controller/AiJudgeController.java) · [팀 API 명세서 안내](https://github.com/Joohyeng/Callog/blob/main/backend/README.md)

## 문제 해결과 기술 의사결정

### 1. Kafka·MongoDB로 캠페인 정보를 전달해 검수 요청의 App 의존성 축소

AI 판사 모듈을 처음 분리했을 때는 화면의 검수 요청이 기존 App을 거쳐 AI 판사 서버로 전달됐습니다. 파일 처리 기능은 옮겼지만, App이 요청 전달과 권한 정보 확인의 중간 경로로 남아 있었습니다. 검수 요청을 AI 판사 서버에서 처리하려면 필요한 캠페인·참여자 정보를 별도로 확보해야 했습니다.

목표는 AI Judge가 자신의 검수 API에서 요청을 받고, 해당 캠페인의 참여 여부도 자체 조회용 데이터로 확인하도록 만드는 것이었습니다. 구현은 파일 처리 이전, 결과 저장·조회 연결, 요청 경로와 캠페인 정보 동기화 변경 순서로 진행했습니다.

| 개발 단계 | 담당한 변경 | 작업 근거 |
| --- | --- | --- |
| 파일 처리 기능 이전 | 업로드·OCR·레이아웃 분석·n8n 호출을 AI Judge로 이동 | [a83260e](https://github.com/Joohyeng/Callog/commit/a83260e7dba20a057992c27877a3d565742c4bcc) |
| 결과 저장·조회 연결 | AI Judge의 결과 메타데이터 저장과 상세 조회, App과의 결과 연결 추가 | [1513807](https://github.com/Joohyeng/Callog/commit/15138070d6d1bf5355731128d8b8b0a8a999af27) |
| 요청 경로·데이터 의존성 변경 | 캠페인·참여자 이벤트를 발행·수신하고 프론트 요청을 AI Judge로 전환 | [a5f2120](https://github.com/Joohyeng/Callog/commit/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8) |

<div class="port-decision">
캠페인 생성·변경과 참여자 추가·변경·제거 이벤트를 App에서 발행하고, AI 판사 서버가 Kafka로 받아 MongoDB의 캠페인 조회용 데이터에 반영하도록 구성했습니다. 화면의 검수 요청 경로도 Gateway에서 AI 판사 서버로 연결했습니다. 캠페인 검수 요청을 받으면 동기화된 참여자 정보를 조회해 접근 범위를 확인하도록 구현했습니다.
</div>

| 구분 | 초기 분리 단계 | 변경 후 |
| --- | --- | --- |
| 검수 요청 경로 | 화면 → Gateway → App → AI Judge | 화면 → Gateway → AI Judge |
| 캠페인·참여자 정보 | App의 데이터·처리 로직에 의존 | App 이벤트 → Kafka → AI Judge의 MongoDB |
| 캠페인 검수 접근 확인 | App 경유 흐름 | AI Judge가 로컬 참여자 데이터 조회 |

이벤트는 캠페인 변경, 참여자 변경, 참여자 제거로 구분했습니다. 캠페인은 식별자를 기준으로 생성·갱신하고, 참여자는 사용자 식별자를 기준으로 갱신·제거하도록 연결했습니다.

캠페인의 원본 정보는 App에서 관리하고, AI Judge에는 검수 접근 확인에 사용할 조회용 데이터를 두었습니다. 이 구조에서는 검수 요청 때마다 App에 참여 여부를 동기 조회하는 대신, Kafka로 반영된 데이터를 사용합니다. 그만큼 이벤트가 반영되기 전까지 원본과 조회용 데이터 사이에 시간 차이가 생긴다는 점을 함께 고려해야 합니다.

요청 경로를 바꾸면서 **프론트의 응답 처리도 함께 수정**했습니다. 기존 App의 공통 응답 형식과 AI Judge가 직접 반환하는 결과를 구분해 읽고, 오류 메시지도 직접 응답에서 가져오도록 변경했습니다. 서버 분리에 맞춰 API 주소와 응답 변환을 함께 조정한 작업입니다.

```diff
- await api.post('/ad/check/file/aijudge', formData, {
+ await api.post('/aijudge/check/file', formData, {
```

**구현 결과:** 검수 요청과 캠페인 참여 여부 확인 범위에서 App의 동기 호출 의존성을 줄였습니다. 이벤트 전달과 반영 사이에는 시간 차이가 있으므로, 권한 변경 직후 요청이나 초기 데이터 동기화에 대한 검증은 후속 보완 항목으로 남습니다.

[권한 동기화·직접 요청 전환 커밋](https://github.com/Joohyeng/Callog/commit/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8) · [Kafka 이벤트 수신 코드](https://github.com/Joohyeng/Callog/blob/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8/backend/aijudge/src/main/java/com/example/adcheck/campaign/CampaignProjectionKafkaListener.java) · [캠페인·참여자 조회용 데이터 관리 코드](https://github.com/Joohyeng/Callog/blob/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8/backend/aijudge/src/main/java/com/example/adcheck/campaign/CampaignProjectionMongoService.java)

[프론트 요청 경로·응답 변환 변경 코드](https://github.com/Joohyeng/Callog/blob/a5f2120655e1cadd0651451d7e9d55fc3ff2a7f8/frontend/src/api/adcheck/index.js)

### 2. 파일 처리·n8n 검수·결과 조회를 작업 식별자로 연결

광고물 검수는 파일 업로드, 내용 추출, 외부 검수 요청, 결과 저장이 이어지는 작업입니다. 사용자가 원본과 분석 결과를 다시 확인하려면 각 단계의 산출물이 어느 요청에 속하는지 추적할 수 있어야 했습니다. 또한 AI 검수 호출이 실패한 경우에도 이미 추출한 내용과 오류 정보를 확인할 수 있는 구조가 필요했습니다.

<div class="port-decision">
파일 처리와 OCR·레이아웃 분석, n8n 검수 호출을 AI 판사 모듈로 옮겼습니다. 분석 작업 식별자를 기준으로 원본, 추출 텍스트, 이미지와 결과 JSON을 연결하고, MongoDB에 결과 메타데이터를 저장해 상세 조회에 활용했습니다. 검수 호출 단계에서 예외가 발생하면 추출 결과와 오류 정보를 포함한 응답을 구성해 보관하고 실패 이벤트를 전달하도록 연결했습니다.
</div>

| 저장 대상 | 저장 방식 | 사용 목적 |
| --- | --- | --- |
| 원본·추출 텍스트·이미지·결과 JSON | 객체 스토리지의 작업별 경로 | 원본과 분석 산출물 참조 |
| 분석 결과 메타데이터 | MongoDB, `analysisJobId` 기준 | 상세 조회와 파일·결과 연결 |
| 검수 오류·처리 시간 | 결과 데이터에 함께 보관 | 처리 단계와 실패 내용 확인 |

**구현 결과:** 업로드부터 결과 조회까지 요청 식별자로 연결했으며, 검수 호출 실패 시에도 해당 단계의 정보가 남도록 구성했습니다. 파일 처리와 결과 저장을 한 흐름으로 연결한 작업은 본인 커밋에서 확인할 수 있습니다.

[파일·OCR·n8n 분리 커밋](https://github.com/Joohyeng/Callog/commit/a83260e7dba20a057992c27877a3d565742c4bcc) · [결과 메타데이터·상세 조회 추가 커밋](https://github.com/Joohyeng/Callog/commit/15138070d6d1bf5355731128d8b8b0a8a999af27)

## 기술 스택

### 직접 사용한 기술

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span><span class="port-badge pb-b">JavaScript</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend &amp; Auth</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Security</span><span class="port-badge pb-b">JWT</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Messaging &amp; Data</div><div class="ptg-badges"><span class="port-badge pb-p">Kafka</span><span class="port-badge pb-g">MongoDB</span><span class="port-badge pb-g">MariaDB</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">AI Integration</div><div class="ptg-badges"><span class="port-badge pb-r">n8n</span><span class="port-badge pb-r">OCR</span><span class="port-badge pb-r">Layout Analysis</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Deployment</div><div class="ptg-badges"><span class="port-badge pb-o">Docker</span><span class="port-badge pb-o">Kubernetes</span></div></div>
</div>

### 프로젝트 공통 기술 스택

<div class="port-tech-groups">
  <div class="port-tech-group"><div class="ptg-label">Frontend</div><div class="ptg-badges"><span class="port-badge pb-b">Vue 3</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Backend</div><div class="ptg-badges"><span class="port-badge pb-b">Java</span><span class="port-badge pb-b">Spring Boot</span><span class="port-badge pb-b">Spring Cloud Gateway</span><span class="port-badge pb-b">Eureka</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Messaging</div><div class="ptg-badges"><span class="port-badge pb-p">Kafka</span><span class="port-badge pb-p">Event-driven Architecture</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Data</div><div class="ptg-badges"><span class="port-badge pb-g">MariaDB</span><span class="port-badge pb-g">MongoDB</span><span class="port-badge pb-g">Redis · Valkey</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">AI &amp; Automation</div><div class="ptg-badges"><span class="port-badge pb-r">OCR</span><span class="port-badge pb-r">n8n</span><span class="port-badge pb-r">Layout Analysis</span></div></div>
  <div class="port-tech-group"><div class="ptg-label">Infra</div><div class="ptg-badges"><span class="port-badge pb-o">Docker</span><span class="port-badge pb-o">Kubernetes</span><span class="port-badge pb-o">Jenkins</span></div></div>
</div>

---

## 구현 근거와 프로젝트 문서

개인 구현은 위의 작업 커밋을 기준으로 연결했습니다. 아래 README와 아키텍처는 팀 전체 프로젝트를 설명하는 자료입니다.

- [개인 저장소 — 코드와 작업 브랜치](https://github.com/Joohyeng/Callog)
- [팀 프로젝트 소개·시스템 아키텍처·DB 설계](https://github.com/Joohyeng/Callog/blob/main/README.md)
- [Frontend 화면 구성·권한별 사용 흐름](https://github.com/Joohyeng/Callog/blob/main/frontend/README.md)
- [Backend 모듈별 API 명세·기능 설명](https://github.com/Joohyeng/Callog/blob/main/backend/README.md)
- [팀 원본 저장소](https://github.com/beyond-sw-camp/be24-fin-magamFairy-callog)
