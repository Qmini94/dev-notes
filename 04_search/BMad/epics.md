---
stepsCompleted: [1, 2, 3, 4]
inputDocuments: [prd.md, architecture.md]
status: 'complete'
completedAt: '2026-02-05'
validationResults:
  frCoverage: '46/46 PASS'
  architectureCompliance: 'PASS'
  storyQuality: 'PASS'
  epicStructure: 'PASS'
  dependencyIntegrity: 'PASS'
---

# egov - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for egov 통합 검색 시스템, decomposing the requirements from the PRD and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

**검색 실행 (Search Execution):**
- FR1: 사용자는 키워드를 입력하여 CMS 전체(게시판 게시글, 페이지 콘텐츠, 메뉴명/메뉴 경로)를 대상으로 통합 검색을 실행할 수 있다
- FR2: 사용자는 검색을 실행하면 관련도 기반(BM25) 순위의 결과 목록을 받을 수 있다
- FR3: 사용자는 빈 검색어 또는 유효하지 않은 입력에 대해 안내 메시지를 받을 수 있다
- FR4: 사용자는 검색 결과가 없을 때 재검색을 유도하는 안내를 받을 수 있다

**검색 결과 표시 (Search Results Display):**
- FR5: 사용자는 각 검색 결과에서 제목, 요약 스니펫, 콘텐츠 유형, 소속 사이트 정보를 확인할 수 있다
- FR6: 사용자는 검색 결과를 클릭하여 원본 콘텐츠 페이지로 직접 이동할 수 있다
- FR7: 사용자는 검색 결과를 페이지네이션으로 탐색할 수 있다
- FR8: 관리자는 검색 결과에서 해당 콘텐츠의 관리 화면(편집 등)으로 연결할 수 있다

**검색 결과 필터링 및 정렬 (Filtering & Sorting):**
- FR9: 사용자는 검색 결과를 사이트별로 필터링할 수 있다
- FR10: 사용자는 검색 결과를 콘텐츠 유형별(게시글, 페이지, 메뉴)로 필터링할 수 있다
- FR11: 사용자는 사이트 필터와 유형 필터를 조합하여 결과를 좁힐 수 있다
- FR12: 사용자는 검색 결과를 정확도순(기본) 또는 최신순으로 정렬을 전환할 수 있다

**권한 및 접근 제어 (Permission & Access Control):**
- FR13: 시스템은 사용자의 인증 상태와 권한에 따라 접근 가능한 콘텐츠만 검색 결과에 포함한다
- FR14: 시스템은 권한이 없는 사용자에게 비공개 콘텐츠의 존재 여부를 노출하지 않는다
- FR15: 시스템은 사이트 정책(공개/비공개, IP 접근 제한 등)에 따라 검색 결과를 필터링한다
- FR16: 시스템은 인덱스 동기화 지연이 발생해도 검색 시점에 권한을 재확인하여 미인가 콘텐츠 노출을 방지한다
- FR17: 검색 API는 기존 CMS와 동일한 인증 체계를 통해 보호된다

**데이터 인덱싱 및 동기화 (Data Indexing & Synchronization):**
- FR18: 시스템은 게시판 게시글을 검색 인덱스에 수집하고 유지한다
- FR19: 시스템은 페이지 콘텐츠를 검색 인덱스에 수집하고 유지한다
- FR20: 시스템은 메뉴명 및 메뉴 경로를 검색 인덱스에 수집하고 유지한다
- FR21: 시스템은 콘텐츠가 생성·수정·삭제될 때 검색 인덱스에 변경 사항을 반영한다
- FR22: 시스템은 콘텐츠의 공개/비공개 상태 또는 권한이 변경될 때 인덱스에 반영한다
- FR23: 시스템은 신규 사이트가 추가될 때 해당 사이트의 콘텐츠를 자동으로 검색 대상에 포함한다
- FR24: 운영자는 전체 또는 부분 재색인을 수동으로 트리거하여 인덱스 정합성을 복구할 수 있다
- FR25: 시스템은 데이터 정합성 확보를 위해 주기적인 전체 재수집 및 재색인 작업을 실행할 수 있다
- FR26: 시스템은 재색인 작업 중에도 CMS 핵심 서비스가 중단되지 않는다
- FR27: 시스템은 색인 대상 데이터와 검색 인덱스 간 불일치를 감지하고 복구할 수 있다
- FR28: 시스템은 색인 작업의 성공/실패 여부를 기록하고 운영자가 확인할 수 있다
- FR29: 시스템은 색인 작업 실패 시 재시도 또는 복구 절차를 수행할 수 있다
- FR30: 시스템은 검색 인덱스 데이터 보관 정책이 원본 콘텐츠의 삭제 및 상태 변경과 동기화된다

**검색 UI 및 접근성 (Search UI & Accessibility):**
- FR31: 사용자는 CMS의 모든 페이지에서 검색 기능에 접근할 수 있다
- FR32: 사용자는 모바일, 태블릿, 데스크톱 환경에서 동일한 검색 기능을 이용할 수 있다
- FR33: 사용자는 키보드만으로 검색 입력, 결과 탐색, 필터 조작, 정렬 변경을 수행할 수 있다
- FR34: 스크린리더 사용자는 검색 결과, 필터 상태, 정렬 상태를 인식할 수 있다
- FR35: 시스템은 검색 결과 페이지에 검색 엔진 인덱싱 제외(noindex) 처리를 적용한다

**운영 및 모니터링 (Operations & Monitoring):**
- FR36: 시스템은 검색 서비스 장애 시에도 CMS 기본 기능이 정상 동작하도록 장애를 격리한다
- FR37: 시스템은 검색 서비스와 CMS 서비스 간 의존성을 최소화하여 장애 확산을 방지한다
- FR38: 시스템은 검색 서비스 불가 시 사용자에게 안내 메시지를 표시한다
- FR39: 시스템은 관리자의 검색 관련 설정 변경에 대한 감사 로그를 기록한다
- FR40: 시스템은 검색어 로그를 수집한다 (통계 목적, 개인 식별 정보 미포함)
- FR41: 시스템은 검색 로그, 감사 로그, 색인 로그를 서로 분리하여 관리한다
- FR42: 시스템은 검색어 로그에 대한 접근 통제 및 보관 기간 정책을 적용한다

**확장성 및 재사용성 (Extensibility & Reusability):**
- FR43: 시스템은 게시글·페이지·메뉴 외의 새로운 도메인 데이터를 검색 대상으로 확장할 수 있다
- FR44: 관리자는 검색 대상이 되는 데이터 유형 및 수집 기준을 설정 또는 확장할 수 있다
- FR45: 검색 인덱스 구조 또는 검색 대상 데이터 모델 변경이 기존 CMS 핵심 기능에 영향을 주지 않는다
- FR46: 검색 API는 내부 UI뿐 아니라 외부 시스템 또는 다른 CMS 모듈에서도 사용할 수 있다

### NonFunctional Requirements

- NFR-P1: 검색 API 평균 응답 시간 300ms 이하
- NFR-P2: 검색 API 최대 응답 시간 (P99) 1초 이하
- NFR-P3: 검색 결과 UI 렌더링 — API 응답 후 100ms 이내 화면 갱신
- NFR-P4: 인덱스 반영 지연 — 콘텐츠 변경 후 5분 이내 검색 가능
- NFR-P5: 동시 검색 사용자 최소 50명 시 응답 시간 기준 유지
- NFR-P6: 검색 결과 페이지네이션 — 다음 페이지 로드 시 동일 응답 시간 기준
- NFR-S1: Elasticsearch 내부망/VPC 전용, 외부 직접 접근 차단
- NFR-S2: ES 통신 TLS 적용 필수
- NFR-S3: ES API Key 또는 Basic 인증 + IP allowlist + RBAC
- NFR-S4: 검색 API는 CMS와 동일한 JWT + Redis 세션 기반 인증
- NFR-S5: 접근 불가 콘텐츠 노출 차단 100% — 존재 여부 유추 불가
- NFR-S6: 검색 로그 PII 미저장, 통계 목적 최소 데이터만 수집
- NFR-S7: 감사 로그 무결성 — 기록 후 변경/삭제 불가, 보관 기간 정책 적용
- NFR-S8: 검색 쿼리에 대한 XSS, 인젝션 방지
- NFR-SC1: 인덱스 데이터 규모 — 초기 10만 건, 100만 건까지 확장
- NFR-SC2: 신규 사이트 추가 시 인덱싱 자동 확장, 재배포 불필요
- NFR-SC3: 새로운 콘텐츠 유형 추가 시 인덱스 구조 변경 최소화
- NFR-SC4: 전체 재색인 시 서비스 중단 없이 수행
- NFR-A1: KWCAG 2.2 기준 준수
- NFR-A2: 검색 전체 키보드 운용 가능
- NFR-A3: 스크린리더 호환 — 결과 수, 필터 상태, 정렬 상태, 현재 페이지 음성 전달
- NFR-A4: 검색 실행 후 결과 영역으로 포커스 이동
- NFR-A5: 색상 대비 WCAG AA 기준 (4.5:1) 이상
- NFR-A6: 200%까지 텍스트 확대 시 레이아웃 깨짐 없음
- NFR-I1: 콘텐츠 CRUD 이벤트 기반 실시간 반영 + 주기적 전체 동기화
- NFR-I2: 기존 CMS JWT + Redis 세션 그대로 사용, 별도 인증 체계 도입 금지
- NFR-I3: RESTful API, JSON 응답, 기존 CMS API 패턴 일관성 유지
- NFR-I4: Elasticsearch 7.x 호환
- NFR-I5: CMS 도메인 엔티티와 검색 인덱스 모델 간 매핑 계층 존재
- NFR-R1: 검색 서비스 가용성 월 99.9%
- NFR-R2: 장애 격리 — 검색 장애 시 CMS 핵심 기능 100% 정상
- NFR-R3: Graceful degradation — 검색 불가 시 5초 이내 안내 메시지
- NFR-R4: 장애 후 전체 재색인으로 정합성 복구 가능
- NFR-R5: 색인 작업 실패 자동 재시도 (최대 3회), 이후 로그 기록
- NFR-R6: 장애 원인 추적을 위한 구조화된 로그 및 기본 메트릭 수집

### Additional Requirements

**Architecture에서 도출된 기술 요구사항:**

- Brownfield 프로젝트: 기존 Spring Boot 2.7.12 + Nuxt 3.17.5 스택에 검색 모듈 추가
- Elasticsearch 7.17.x + Spring Data Elasticsearch 4.4.x 사용
- Element Plus Autocomplete 기반 검색 UI
- Outbox 패턴(MySQL) + Spring Events 기반 이벤트 전파 (이벤트 유실 방지)
- Read/Write Alias 분리로 무중단 재색인
- Dual 권한 필터링: Index-time(리소스 키 + 공개 범위) + Query-time(CMS 권한 재검증)
- Circuit Breaker(Resilience4j) + Graceful Degradation 장애 격리
- 분리된 search 패키지 (kr.co.itid.cms.search.*)
- 3종 분리 로그: search-query.log, search-index.log, search-audit.log
- Docker Compose에 Elasticsearch 서비스 추가
- 기존 CMS Service에 이벤트 발행 1줄 추가 (최소 침습)
- Starter Template 없음 — 기존 프로젝트에 의존성 추가 방식

### FR Coverage Map

| FR | Epic | 설명 |
|----|------|------|
| FR1 | Epic 1 | 키워드 통합 검색 실행 |
| FR2 | Epic 1 | BM25 관련도 순위 |
| FR3 | Epic 1 | 빈 검색어 안내 |
| FR4 | Epic 1 | 결과 없음 안내 |
| FR5 | Epic 1 | 결과 표시(제목, 스니펫, 유형, 사이트) |
| FR6 | Epic 1 | 원본 페이지 이동 |
| FR7 | Epic 1 | 페이지네이션 |
| FR8 | Epic 3 | 관리자 편집 링크 |
| FR9 | Epic 3 | 사이트별 필터 |
| FR10 | Epic 3 | 유형별 필터 |
| FR11 | Epic 3 | 필터 조합 |
| FR12 | Epic 3 | 정렬 전환 |
| FR13 | Epic 2 | 권한 기반 결과 포함 |
| FR14 | Epic 2 | 존재 여부 비노출 |
| FR15 | Epic 2 | 사이트 정책 필터링 |
| FR16 | Epic 2 | 검색 시점 권한 재확인 |
| FR17 | Epic 2 | JWT 인증 보호 |
| FR18 | Epic 1 | 게시글 색인 |
| FR19 | Epic 1 | 페이지 색인 |
| FR20 | Epic 1 | 메뉴 색인 |
| FR21 | Epic 4 | CRUD 이벤트 반영 |
| FR22 | Epic 4 | 상태/권한 변경 반영 |
| FR23 | Epic 4 | 신규 사이트 자동 포함 |
| FR24 | Epic 4 | 수동 재색인 |
| FR25 | Epic 4 | 주기적 전체 재색인 |
| FR26 | Epic 4 | 재색인 중 무중단 |
| FR27 | Epic 4 | 불일치 감지/복구 |
| FR28 | Epic 4 | 색인 성공/실패 기록 |
| FR29 | Epic 4 | 색인 실패 재시도 |
| FR30 | Epic 4 | 삭제/상태 동기화 |
| FR31 | Epic 1 | 모든 페이지에서 접근 |
| FR32 | Epic 1 | 반응형 검색 |
| FR33 | Epic 3 | 키보드 접근성 |
| FR34 | Epic 3 | 스크린리더 호환 |
| FR35 | Epic 3 | noindex 처리 |
| FR36 | Epic 1 | 장애 격리 |
| FR37 | Epic 1 | 의존성 최소화 |
| FR38 | Epic 1 | 서비스 불가 안내 |
| FR39 | Epic 5 | 감사 로그 |
| FR40 | Epic 5 | 검색어 로그 수집 |
| FR41 | Epic 5 | 로그 분리 관리 |
| FR42 | Epic 5 | 로그 접근 통제 |
| FR43 | Epic 6 | 도메인 확장 |
| FR44 | Epic 6 | 관리자 검색 대상 설정 |
| FR45 | Epic 6 | CMS 무영향 |
| FR46 | Epic 6 | 외부 API |

## Epic List

### Epic 1: 기본 통합 검색
사용자가 CMS 전체 콘텐츠를 하나의 검색창에서 검색하고 결과를 확인할 수 있다.
**FRs covered:** FR1-7, FR18-20, FR31-32, FR36-38

### Epic 2: 권한 기반 검색 보안
시스템이 사용자 권한과 사이트 정책에 따라 접근 가능한 콘텐츠만 검색 결과에 포함하며, 비공개 콘텐츠의 존재 자체를 노출하지 않는다.
**FRs covered:** FR13-17

### Epic 3: 검색 필터링, 정렬 및 접근성
사용자가 검색 결과를 사이트별·유형별로 필터링하고, 정렬 방식을 변경하며, 키보드와 스크린리더로 모든 기능을 사용할 수 있다.
**FRs covered:** FR8-12, FR33-35

### Epic 4: 실시간 색인 동기화 파이프라인
콘텐츠 생성·수정·삭제 시 검색 인덱스에 자동 반영되며, 정합성 검증 및 무중단 재색인이 가능하다.
**FRs covered:** FR21-30

### Epic 5: 운영 모니터링 및 감사 로깅
운영자가 검색 시스템의 동작 상태를 확인하고, 모든 검색·색인·설정 변경 이력을 추적할 수 있다.
**FRs covered:** FR39-42

### Epic 6: 확장성 및 외부 API
새로운 도메인 데이터를 검색 대상으로 추가할 수 있고, 외부 시스템에서 검색 API를 활용할 수 있다.
**FRs covered:** FR43-46

---

## Epic 1: 기본 통합 검색

사용자가 CMS 전체 콘텐츠를 하나의 검색창에서 검색하고 결과를 확인할 수 있다.

### Story 1.1: ES 인프라 구성 및 검색 모듈 기반 설정

As a 개발팀,
I want Elasticsearch 인프라와 검색 모듈 기반을 구성하여,
So that 이후 검색 기능을 구현할 수 있다.

**Acceptance Criteria:**

**Given** Docker Compose 환경이 실행 중일 때
**When** docker-compose up을 실행하면
**Then** Elasticsearch 7.17.x 컨테이너가 정상 기동되고 health check를 통과한다

**Given** Dev/Local 환경에서
**When** ES 보안 설정을 확인하면
**Then** 내부망 기준 Basic Auth 또는 로컬 접근 제한으로 운영 가능하며, TLS는 선택 적용한다

**Given** Production 환경에서
**When** ES 보안 설정을 확인하면
**Then** 반드시 TLS + API Key(또는 이에 준하는 인증 방식)가 적용된다

**Given** cms_backend 프로젝트가 빌드될 때
**When** gradle build를 실행하면
**Then** spring-boot-starter-data-elasticsearch, resilience4j 의존성이 포함된다
**And** ElasticsearchConfig가 ES에 정상 연결된다
**And** kr.co.itid.cms.search 패키지 구조가 생성된다

### Story 1.2: 콘텐츠 데이터 매핑 및 초기 배치 색인

As a 시스템 운영자,
I want 기존 CMS 콘텐츠(게시글, 페이지, 메뉴)를 검색 인덱스에 초기 적재하여,
So that 검색 서비스가 기존 데이터를 대상으로 동작할 수 있다.

**Acceptance Criteria:**

**Given** ES가 정상 기동되어 있을 때
**When** 초기 배치 색인을 실행하면
**Then** CMS의 게시판 게시글이 ContentDocument로 변환되어 ES에 색인된다
**And** 페이지 콘텐츠가 PageDocument로 변환되어 ES에 색인된다
**And** 메뉴명/메뉴 경로가 MenuDocument로 변환되어 ES에 색인된다
**And** 각 Document에 documentId, contentType, title, body, menuId, boardId, visibility, createdDate, updatedDate 필드가 포함된다

**Given** 색인이 완료되었을 때
**When** ES 인덱스를 조회하면
**Then** cms_content_read alias가 정상 동작하고 색인된 문서 수가 원본 데이터 수와 일치한다

### Story 1.3: 통합 검색 API 구현

As a 사용자,
I want 키워드를 입력하여 CMS 전체를 대상으로 통합 검색을 실행하여,
So that 원하는 정보를 빠르게 찾을 수 있다.

**Acceptance Criteria:**

**Given** 사용자가 검색 API를 호출할 때
**When** GET /back-api/search?keyword=소상공인&page=0&size=10을 요청하면
**Then** BM25 관련도 기반 결과 목록이 ApiResponse 형식으로 반환된다
**And** 각 결과에 title, snippet, contentType, siteInfo, sourceUrl이 포함된다
**And** 응답 시간 목표는 로컬/개발 환경, 약 10만 문서 기준, P95 300ms 이하이다
**And** 캐시, 네트워크 환경, ES warm-up 여부에 따라 편차가 발생할 수 있다

**Given** 빈 검색어로 요청할 때
**When** keyword가 빈 문자열 또는 null이면
**Then** code 400과 안내 메시지가 반환된다

**Given** 결과가 없는 키워드로 검색할 때
**When** 매칭되는 문서가 없으면
**Then** 빈 결과 목록과 재검색 유도 메시지가 반환된다

**Given** 페이지네이션 요청 시
**When** page=1, size=10으로 요청하면
**Then** 11번째~20번째 결과가 반환되고 totalCount가 전체 건수를 나타낸다

### Story 1.4: 검색 UI 구현 (SearchBar + 결과 페이지)

As a 사용자,
I want CMS 모든 페이지에서 검색창을 이용하고 결과를 확인하여,
So that 어디에서든 원하는 정보를 검색할 수 있다.

**Acceptance Criteria:**

**Given** CMS 어떤 페이지에 있든
**When** 레이아웃을 확인하면
**Then** 상단에 검색 입력창(SearchBar)이 항상 표시된다

**Given** SearchBar에 키워드를 입력하고 검색을 실행할 때
**When** 엔터 키를 누르거나 검색 버튼을 클릭하면
**Then** /search 페이지로 이동하여 검색 결과 목록이 표시된다
**And** 각 결과에 제목, 요약 스니펫(키워드 하이라이팅), 콘텐츠 유형, 소속 사이트가 표시된다

**Given** 검색 결과 항목을 클릭할 때
**When** 결과 링크를 클릭하면
**Then** 원본 콘텐츠 페이지로 직접 이동한다

**Given** 결과가 10건을 초과할 때
**When** 페이지네이션을 조작하면
**Then** 다음/이전 페이지 결과가 표시된다

**Given** 모바일/태블릿 환경에서
**When** 검색 UI를 사용하면
**Then** 반응형으로 동일한 기능이 제공된다

### Story 1.5: 장애 격리 및 Graceful Degradation

As a CMS 사용자,
I want 검색 서비스 장애 시에도 CMS 기본 기능을 정상 이용하여,
So that 검색 장애가 전체 서비스에 영향을 주지 않는다.

**Acceptance Criteria:**

**Given** Elasticsearch가 다운된 상태에서
**When** 검색 API를 호출하면
**Then** Circuit Breaker가 작동하여 503 응답과 "검색 서비스가 일시적으로 사용할 수 없습니다" 메시지를 반환한다
**And** CMS의 게시판, 페이지, 메뉴 등 기본 기능은 정상 동작한다

**Given** 프론트엔드에서 503 응답을 받았을 때
**When** 검색 결과 영역을 렌더링하면
**Then** SearchServiceDown 컴포넌트가 표시되어 안내 메시지를 보여준다
**And** 다른 CMS 페이지 탐색은 영향 없이 동작한다

**Given** ES가 복구된 후
**When** Circuit Breaker가 half-open 상태로 전환되면
**Then** 정상 검색이 자동 복구된다

---

## Epic 2: 권한 기반 검색 보안

시스템이 사용자 권한과 사이트 정책에 따라 접근 가능한 콘텐츠만 검색 결과에 포함하며, 비공개 콘텐츠의 존재 자체를 노출하지 않는다.

### Story 2.1: Index-time 권한 필드 색인

As a 시스템,
I want 색인 시점에 각 문서에 리소스 키와 공개 범위 정보를 포함하여,
So that 검색 시 1차 필터링을 효율적으로 수행할 수 있다.

**Acceptance Criteria:**

**Given** 콘텐츠가 색인될 때
**When** ContentDocument/PageDocument/MenuDocument가 생성되면
**Then** menuId, boardId 등 리소스 키 필드가 포함된다
**And** visibility 필드에 PUBLIC/MEMBER/ADMIN 값이 저장된다
**And** 사용자별 개인 권한 정보는 ES에 저장되지 않는다

### Story 2.2: Query-time 권한 필터링 및 존재 유추 불가 보장

As a 사용자,
I want 내 권한 범위 내의 콘텐츠만 검색 결과에 보이고, 비공개 콘텐츠는 존재조차 알 수 없게 하여,
So that 보안이 보장된 안전한 검색을 이용할 수 있다.

**Acceptance Criteria:**

**Given** 비로그인 사용자가 검색할 때
**When** 통합 검색을 실행하면
**Then** visibility=PUBLIC 문서만 결과에 포함된다
**And** MEMBER/ADMIN 문서의 존재 여부가 totalCount에 반영되지 않는다

**Given** 로그인 사용자가 검색할 때
**When** 통합 검색을 실행하면
**Then** ES에서 공개 범위 기반 1차 필터 후 상위 N건에 대해 CMS PermissionService로 최종 권한 재검증을 수행한다
**And** totalCount는 Query-time 권한 필터링이 반영된 결과 기준으로 계산한다
**And** 정확한 totalCount 계산이 비용 과다를 유발하는 경우, "대략 N+" 방식 등 정책 기반 계산을 허용한다

**Given** 인덱스 동기화가 지연된 상태에서
**When** 방금 비공개로 전환된 콘텐츠를 검색하면
**Then** Query-time 재검증에서 필터링되어 결과에 포함되지 않는다

### Story 2.3: 검색 API JWT 인증 통합

As a 시스템,
I want 검색 API가 기존 CMS와 동일한 JWT 인증 체계로 보호되어,
So that 인증되지 않은 접근을 차단한다.

**Acceptance Criteria:**

**Given** JWT 토큰 없이 검색 API를 호출할 때
**When** /back-api/search를 요청하면
**Then** 비로그인 사용자로 처리되어 공개 콘텐츠만 반환된다 (검색 자체는 허용)

**Given** 유효한 JWT 토큰으로 검색 API를 호출할 때
**When** /back-api/search를 요청하면
**Then** 토큰에서 사용자 정보를 추출하여 권한 필터링에 활용한다

**Given** 관리자 전용 검색 관리 API를 호출할 때
**When** MANAGE 권한 없이 /back-api/search/admin/*을 요청하면
**Then** 403 Forbidden이 반환된다

---

## Epic 3: 검색 필터링, 정렬 및 접근성

사용자가 검색 결과를 사이트별·유형별로 필터링하고, 정렬 방식을 변경하며, 키보드와 스크린리더로 모든 기능을 사용할 수 있다.

### Story 3.1: 사이트별·유형별 필터 및 복합 필터링

As a 사용자,
I want 검색 결과를 사이트별·콘텐츠 유형별로 필터링하여,
So that 원하는 범위의 결과만 빠르게 확인할 수 있다.

**Acceptance Criteria:**

**Given** 검색 결과가 표시된 상태에서
**When** 사이트 필터를 선택하면
**Then** 해당 사이트의 결과만 표시되고 totalCount가 갱신된다

**Given** 검색 결과가 표시된 상태에서
**When** 유형 필터(게시글/페이지/메뉴)를 선택하면
**Then** 해당 유형의 결과만 표시된다

**Given** 사이트 필터와 유형 필터를 동시에 적용할 때
**When** 두 필터를 조합하면
**Then** 두 조건을 모두 만족하는 결과만 표시된다

### Story 3.2: 정렬 전환 및 관리자 편집 링크

As a 사용자,
I want 검색 결과를 정확도순 또는 최신순으로 정렬하고, 관리자는 편집 화면으로 바로 이동하여,
So that 상황에 맞는 방식으로 결과를 탐색할 수 있다.

**Acceptance Criteria:**

**Given** 검색 결과가 표시된 상태에서
**When** 정렬을 "최신순"으로 변경하면
**Then** 결과가 updatedDate 기준 내림차순으로 재정렬된다

**Given** 정렬을 "정확도순"(기본)으로 복원할 때
**When** 정렬 옵션을 변경하면
**Then** BM25 관련도 기준으로 다시 정렬된다

**Given** 관리자 권한을 가진 사용자가 검색 결과를 볼 때
**When** 결과 항목을 확인하면
**Then** 해당 콘텐츠의 관리 화면(편집)으로 이동하는 링크가 표시된다

### Story 3.3: KWCAG 2.2 접근성 적용

As a 키보드/스크린리더 사용자,
I want 검색의 모든 기능을 키보드와 보조 기술로 사용하여,
So that 장애 유무와 관계없이 동등한 검색 경험을 얻는다.

**Acceptance Criteria:**

**Given** 키보드만 사용할 때
**When** Tab/Enter/Arrow 키로 조작하면
**Then** 검색 입력, 결과 탐색, 필터 조작, 정렬 변경을 수행할 수 있다

**Given** 스크린리더를 사용할 때
**When** 검색을 실행하면
**Then** 검색 결과 수, 필터 상태, 정렬 상태, 현재 페이지 정보가 음성으로 전달된다
**And** 결과 영역에 적절한 aria-live, aria-label 속성이 적용된다

**Given** 검색 실행 후
**When** 결과가 표시되면
**Then** 포커스가 결과 영역으로 이동한다

**Given** 검색 결과 페이지가 로드될 때
**When** 페이지 메타 정보를 확인하면
**Then** noindex 메타 태그가 적용되어 검색 엔진 인덱싱이 제외된다

---

## Epic 4: 실시간 색인 동기화 파이프라인

콘텐츠 생성·수정·삭제 시 검색 인덱스에 자동 반영되며, 정합성 검증 및 무중단 재색인이 가능하다.

### Story 4.1: Spring Events + Outbox 패턴 구현

As a 시스템,
I want CRUD 이벤트를 Outbox 테이블에 저장하고 비동기로 ES에 색인하여,
So that 이벤트 유실 없이 안정적으로 색인을 동기화할 수 있다.

**Acceptance Criteria:**

**Given** cms_search_outbox 테이블이 생성되었을 때
**When** IndexingEvent가 발행되면
**Then** 동일 트랜잭션 내에서 SearchOutbox 레코드가 INSERT된다
**And** Outbox payload에는 최소 식별 정보(contentType, contentIdx, action)만 저장한다
**And** 실제 ES 문서 변환은 처리 시점에 원본 데이터를 조회하여 mapper를 통해 생성한다

**Given** IndexingEventListener가 이벤트를 처리할 때
**When** @Async로 ES Write Alias에 색인을 시도하면
**Then** 성공 시 Outbox status가 COMPLETED로 업데이트된다
**And** 실패 시 Outbox status가 FAILED로 업데이트되고 retryCount가 증가한다

### Story 4.2: 기존 CMS Service에 이벤트 발행 통합

As a 콘텐츠 작성자,
I want 게시글/페이지/메뉴를 생성·수정·삭제하면 검색 인덱스에 자동 반영되어,
So that 변경된 콘텐츠가 5분 이내에 검색 가능하다.

**Acceptance Criteria:**

**Given** 게시글을 생성할 때
**When** BoardServiceImpl.create가 실행되면
**Then** IndexingEvent(CREATE, POST, idx)가 발행되고 5분 이내 검색 결과에 반영된다

**Given** 페이지 콘텐츠를 수정할 때
**When** ContentServiceImpl.update가 실행되면
**Then** IndexingEvent(UPDATE, PAGE, idx)가 발행되고 변경 내용이 검색에 반영된다

**Given** 메뉴를 삭제할 때
**When** MenuServiceImpl.delete가 실행되면
**Then** IndexingEvent(DELETE, MENU, idx)가 발행되고 검색 결과에서 제거된다

**Given** 콘텐츠의 공개/비공개 상태가 변경될 때
**When** visibility가 변경되면
**Then** IndexingEvent(UPDATE)가 발행되고 ES 문서의 visibility 필드가 갱신된다

### Story 4.3: Outbox 폴링 재시도 서비스

As a 시스템 운영자,
I want 실패한 색인 작업이 자동으로 재시도되어,
So that 일시적 ES 장애 후 색인 정합성이 자동 복구된다.

**Acceptance Criteria:**

**Given** Outbox에 FAILED 또는 PENDING 상태의 레코드가 있을 때
**When** OutboxPollingService 스케줄러가 실행되면
**Then** 미처리 레코드를 순차적으로 ES에 색인 재시도한다
**And** 성공 시 status=COMPLETED, 실패 시 retryCount+1
**And** retryCount가 최대 횟수(3회)를 초과하면 FAILED로 유지하고 로그에 기록한다

### Story 4.4A: Read/Write Alias 기반 무중단 재색인 (MVP)

As a 시스템 운영자,
I want 전체 재색인을 서비스 중단 없이 수행하여,
So that 인덱스 구조 변경이나 정합성 복구를 안전하게 진행할 수 있다.

**Acceptance Criteria:**

**Given** 운영자가 전체 재색인을 트리거할 때
**When** POST /back-api/search/admin/reindex를 호출하면
**Then** 신규 인덱스(cms_content_v{n+1})가 생성되고 Write Alias가 전환된다
**And** 전체 데이터가 신규 인덱스에 색인된다
**And** 색인 중 기존 Read Alias는 기존 인덱스를 계속 서비스한다
**And** 완료 후 Read Alias가 신규 인덱스로 원자적 전환된다

**Given** 재색인 중일 때
**When** 사용자가 검색을 실행하면
**Then** Read Alias를 통해 기존 인덱스에서 정상 검색 결과가 반환된다

### Story 4.4B: 재색인 중 증분 이벤트 Dual-Write (운영 안정화)

As a 시스템 운영자,
I want 재색인 진행 중에도 증분 이벤트가 기존·신규 인덱스 양쪽에 반영되어,
So that 재색인 완료 후 최신 데이터 누락이 없다.

**Acceptance Criteria:**

**Given** 재색인이 진행 중일 때
**When** CMS에서 콘텐츠 CRUD가 발생하면
**Then** 증분 이벤트가 기존 인덱스(Read Alias)와 신규 인덱스(Write Alias) 양쪽에 기록된다
**And** 재색인 완료 후 Alias 전환 시 데이터 누락이 없다

### Story 4.5: 정합성 검증 및 삭제 동기화

As a 시스템,
I want 주기적으로 원본과 인덱스 간 불일치를 감지하고, 삭제된 콘텐츠를 인덱스에서 제거하여,
So that 검색 결과의 정확성을 보장한다.

**Acceptance Criteria:**

**Given** 주기적 배치 검증이 실행될 때
**When** MySQL 원본과 ES 인덱스를 비교하면
**Then** 누락된 문서를 색인하고 삭제된 문서를 인덱스에서 제거한다
**And** 불일치 건수를 search-index.log에 기록한다

**Given** CMS에서 콘텐츠가 삭제되었을 때
**When** 삭제 이벤트가 처리되면
**Then** ES 인덱스에서 해당 문서가 제거되고 검색 결과에 더 이상 표시되지 않는다

---

## Epic 5: 운영 모니터링 및 감사 로깅

운영자가 검색 시스템의 동작 상태를 확인하고, 모든 검색·색인·설정 변경 이력을 추적할 수 있다.

### Story 5.1: 3종 분리 로그 구성

As a 시스템 운영자,
I want 검색·색인·감사 로그가 분리된 파일로 관리되어,
So that 각 목적에 맞는 로그를 독립적으로 조회·관리할 수 있다.

**Acceptance Criteria:**

**Given** logback-spring.xml에 검색 로그 appender가 구성될 때
**When** 검색 요청이 처리되면
**Then** search-query.log에 검색어, 결과 수, 응답 시간이 기록된다 (PII 미포함)

**Given** 색인 작업이 실행될 때
**When** 색인 성공 또는 실패 시
**Then** search-index.log에 액션, 콘텐츠 유형, 대상 ID, 상태가 기록된다

**Given** 검색 로그가 수집될 때
**When** 로그 파일을 확인하면
**Then** 3종 로그가 각각 분리된 파일에 기록되어 혼합되지 않는다

### Story 5.2: 감사 로그 및 로그 접근 통제

As a 시스템 운영자,
I want 검색 설정 변경 이력이 감사 로그로 남고, 로그에 대한 접근이 통제되어,
So that 설정 변경 추적과 로그 보안이 보장된다.

**Acceptance Criteria:**

**Given** 관리자가 검색 설정(색인 정책, 검색 대상 등)을 변경할 때
**When** 변경 API가 호출되면
**Then** search-audit.log에 변경 내용, 변경 사용자 ID, 타임스탬프가 기록된다
**And** SearchAuditLog 엔티티에도 저장된다

**Given** 검색어 로그가 수집될 때
**When** 로그 보관 정책을 확인하면
**Then** logback rolling policy에 따라 보관 기간이 적용된다
**And** 로그 파일에 대한 접근 권한이 운영자로 제한된다

---

## Epic 6: 확장성 및 외부 API

새로운 도메인 데이터를 검색 대상으로 추가할 수 있고, 외부 시스템에서 검색 API를 활용할 수 있다.

### Story 6.1: 확장 가능한 검색 대상 구조

As a 개발팀,
I want 새로운 도메인 데이터를 최소 변경으로 검색 대상에 추가할 수 있어,
So that 시스템 확장이 용이하다.

**Acceptance Criteria:**

**Given** 새로운 도메인(예: 조직 정보)을 검색 대상에 추가할 때
**When** ContentType enum에 새 타입을 추가하고 새 Document 클래스를 생성하면
**Then** 기존 검색 기능이 영향 없이 동작한다
**And** 새 도메인의 문서가 통합 검색 결과에 포함된다

**Given** 검색 인덱스 구조가 변경될 때
**When** 신규 필드를 추가하거나 매핑을 변경하면
**Then** CMS 핵심 기능(게시판, 메뉴 등)에 영향을 주지 않는다

### Story 6.2: 외부 시스템 검색 API 제공

As a 외부 시스템 개발자,
I want 검색 REST API를 내부 UI 외에서도 호출하여,
So that 다른 CMS 모듈이나 외부 시스템에서 검색 기능을 활용할 수 있다.

**Acceptance Criteria:**

**Given** 외부 시스템이 검색 API를 호출할 때
**When** /back-api/search에 유효한 인증 정보와 함께 요청하면
**Then** 동일한 검색 결과가 JSON 형식으로 반환된다
**And** API 응답이 ApiResponse<T> 형식을 따른다

**Given** 관리자가 검색 대상 데이터 유형을 설정할 때
**When** 설정 API를 통해 수집 기준을 변경하면
**Then** 변경이 반영되고 감사 로그에 기록된다
