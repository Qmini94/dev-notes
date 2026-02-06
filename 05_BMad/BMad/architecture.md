---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments: [prd.md, product-brief-egov-2026-02-04.md]
workflowType: 'architecture'
project_name: 'egov'
user_name: 'Q'
date: '2026-02-04'
lastStep: 8
status: 'complete'
completedAt: '2026-02-05'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
8개 기능 영역, 46개 FR로 구성된 통합 검색 시스템. 핵심은 BM25 키워드 검색, 권한 기반 필터링, CRUD 이벤트 기반 인덱스 동기화, 장애 격리 아키텍처이다.

**Non-Functional Requirements:**
- Performance: API 300ms, P99 1초, 50명 동시 사용자
- Security: ES 내부망 전용, TLS+RBAC, 권한 필터링 100%
- Scalability: 10만→100만 건 확장, 무중단 재색인
- Accessibility: KWCAG 2.2 준수
- Integration: 기존 JWT+Redis 인증 통합, CRUD 이벤트 동기화
- Reliability: 99.9% 가용성, 장애 격리 필수

**Scale & Complexity:**
- Primary domain: Full-stack (Spring Boot + Nuxt 3 + Elasticsearch)
- Complexity level: High
- Project context: Brownfield (기존 CMS 확장)
- Estimated architectural components: 6-8개 (검색 API, 인덱싱 파이프라인, 데이터 매핑 계층, 검색 UI, 권한 필터, 로깅, 모니터링, ES 클러스터)

### Technical Constraints & Dependencies

**기존 시스템:**
- Spring Boot 2.7.12 / Java 17 / e-Government Framework 4.2.0
- Nuxt 3.17.5 / Vue 3 / TypeScript
- MySQL 8.0 / Redis 7
- JWT HttpOnly 쿠키 + Redis 세션 인증

**아키텍처 제약:**
- Elasticsearch는 내부망 전용, 백엔드만 접근 가능
- 검색 인덱스는 CMS 원본 데이터의 파생 데이터
- 검색 시스템 장애가 CMS로 확산되지 않는 격리 구조 필수
- 데이터 매핑 계층을 CMS 도메인 로직과 분리하여 재사용성 확보

### Cross-Cutting Concerns Identified

1. **권한 필터링**: 인덱스 시점 + 검색 시점 이중 검증
2. **장애 격리**: 검색 API, 인덱싱 파이프라인 모두 CMS와 격리
3. **로깅 분리**: 검색 로그, 감사 로그, 색인 로그 3종 분리 관리
4. **인증 통합**: 기존 JWT+Redis 체계 재사용
5. **접근성**: 모든 검색 UI에 KWCAG 2.2 적용

## Starter Template Evaluation

### Primary Technology Domain

**Brownfield Full-stack Extension** — 기존 Spring Boot 2.7.12 + Nuxt 3.17.5 스택에 Elasticsearch 검색 계층 추가

### Brownfield Context

이 프로젝트는 신규 스타터 템플릿이 아닌, 기존 CMS에 검색 기능을 확장하는 Brownfield 프로젝트이다.

**기존 기술 스택 (변경 불가):**
- Backend: Spring Boot 2.7.12 / Java 17 / e-Government Framework 4.2.0
- Frontend: Nuxt 3.17.5 / Vue 3 / TypeScript / Element Plus
- Database: MySQL 8.0 / Redis 7
- Infrastructure: Docker Compose / Nginx

### New Components Evaluation

**Search Engine:**
- **선택: Elasticsearch 7.17.x**
- 근거: Spring Data Elasticsearch 4.4.x와 공식 호환, Spring Boot 2.7.12 지원

**Backend Integration:**
- **선택: spring-boot-starter-data-elasticsearch 2.7.x**
- 근거: Repository 패턴으로 일관된 데이터 접근, QueryDSL과 유사한 개발 경험

**Frontend Search UI:**
- **선택: Element Plus Autocomplete**
- 근거: 이미 프로젝트에서 사용 중, 추가 의존성 없음, KWCAG 2.2 접근성 지원

### Initialization Approach

신규 스타터 명령어 대신, 기존 프로젝트에 다음 의존성 추가:

**Backend (build.gradle):**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
```

**Docker Compose (docker-compose.yml):**
```yaml
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.17.x
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=true
```

### Architectural Decisions Provided by Stack

- **Language & Runtime:** Java 17 + TypeScript (기존 유지)
- **Styling Solution:** Tailwind CSS + Element Plus (기존 유지)
- **Testing Framework:** JUnit 5 + Vitest (기존 유지)
- **Code Organization:** 기존 패키지 구조에 search 모듈 추가
- **Development Experience:** 기존 개발 워크플로우 유지

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- 인덱싱 파이프라인 패턴 (Hybrid)
- 권한 필터링 구현 (Dual + 계층 분리)
- 장애 격리 패턴 (Circuit Breaker + Graceful Degradation)
- 인덱싱 이벤트 전파 (Spring Events + Outbox Hybrid)

**Important Decisions (Shape Architecture):**
- 인덱스 버저닝 전략 (Read/Write Alias 분리)
- 데이터 매핑 계층 위치 (분리 모듈)
- 검색 상태 관리 (새 searchStore)
- 로그 분리 전략 (Logger 분리)

**Deferred Decisions (Post-MVP):**
- ES 3-Node 클러스터 전환 (MVP는 Single Node)
- 시맨틱 검색 / 벡터 DB (Growth 단계)

### Data Architecture

**1-1. 인덱싱 파이프라인: Hybrid (Event-Driven + Batch)**
- 실시간: CMS CRUD 이벤트 기반 증분 색인
- 배치: 주기적 전체 동기화로 정합성 검증
- 근거: 실시간 반영 + FR43(불일치 감지/복구) 동시 충족

**1-2. 인덱스 버저닝: Alias Swap + Read/Write Alias 분리**
- Read Alias: `cms_content_read` → 현재 서비스 인덱스 (검색 API가 참조)
- Write Alias: `cms_content_write` → 신규 인덱스 (재색인 트래픽 수신)
- 재색인 완료 후 Read Alias를 신규 인덱스로 원자적 전환
- 재색인 중 증분 이벤트는 Write Alias(신규) + Read Alias(기존) 양쪽에 동시 기록 (dual-write)
- 근거: FR42(재색인 중 무중단) 충족, 쓰기 트래픽 처리 명확화

**1-3. 데이터 매핑 계층: 분리된 모듈**
- 별도 `search` 패키지에 Mapper 계층 배치
- CMS 도메인 로직과 분리하여 재사용성 확보
- 근거: 아키텍처 제약(격리, 재사용성), FR38(매핑 변경이 CMS에 영향 없음)

### Authentication & Security

**2-1. ES 접근 보안: API Key + TLS**
- ES API Key 발급, 환경변수로 관리
- TLS 암호화 통신
- 내부망 전용 (IP Allowlist 병행)
- 근거: 내부망 + TLS NFR 충족, API Key로 세분화된 권한 관리

**2-2. 권한 필터링: Dual 방식 + 계층 분리**
- Index-time: 리소스 키(menuId, boardId 등) + 공개 범위(PUBLIC/MEMBER/ADMIN) 필드만 색인. 사용자별 권한은 저장하지 않음
- Query-time: ES 쿼리에서 공개 범위 기반 1차 필터 → 상위 N건 결과에 대해 CMS 권한 시스템으로 최종 재검증
- "존재 유추 불가" 보장: 권한 없는 문서는 결과 건수/페이징에서 완전 제외
- 근거: PRD 이중 검증 요구 충족, 사용자별 권한을 인덱스에 저장하지 않아 인덱스 갱신 빈도 최소화

### API & Communication Patterns

**3-1. 검색 API: REST + DTO**
- 엔드포인트: `/back-api/search/*`
- SearchRequestDTO / SearchResponseDTO 기반
- 기존 CMS REST 패턴과 일관성 유지
- 근거: 기존 API 규칙 일관성, FR39(외부 시스템 사용 가능)

**3-2. 장애 격리: Circuit Breaker + Graceful Degradation**
- Resilience4j Circuit Breaker로 ES 장애 시 빠른 실패
- 장애 시 "검색 서비스 일시 중단" 메시지 반환
- CMS 핵심 기능에 영향 없음 보장
- 근거: FR46(장애 격리 필수), NFR 99.9% 가용성

**3-3. 인덱싱 이벤트 전파: Spring Events + Outbox 패턴 Hybrid**
- 1차: `ApplicationEventPublisher`로 동일 JVM 내 비동기 이벤트 발행
- Outbox: 이벤트 발행과 동시에 `search_outbox` 테이블(MySQL)에 이벤트 저장 (동일 트랜잭션)
- 재시도: 스케줄러가 미처리 Outbox 레코드를 주기적으로 폴링 → ES 색인 재시도
- 정합성: FR43 충족 — Outbox 기반으로 유실 이벤트 감지/복구 가능
- 근거: 이벤트 유실 방지, 트랜잭션 보장, 정합성 요구사항 대응

### Frontend Architecture

**4-1. 검색 상태 관리: 새 searchStore**
- 별도 Pinia store (`searchStore.ts`) 생성
- 검색어, 필터, 결과, 페이징, 로딩 상태 관리
- 근거: 기능 분리 원칙, 기존 store 오염 방지

**4-2. 검색 결과 렌더링: CSR Only**
- 클라이언트 사이드에서 검색 요청 및 결과 렌더링
- PRD 결정사항 (CSR 검색 결과)
- 근거: 검색은 사용자 상호작용 중심, SEO 불필요

### Infrastructure & Deployment

**5-1. ES 클러스터 구성: Single Node → 3-Node (단계적)**
- MVP: Single Node (개발/초기 운영)
- Production: 3-Node Cluster (고가용성)
- 근거: NFR 확장성(10만→100만), 99.9% 가용성

**5-2. 로그 분리: Logger 분리**
- `search-query.log`: 검색 쿼리 로그
- `search-audit.log`: 검색 설정 변경 감사 로그
- `search-index.log`: 색인 작업 성공/실패 로그
- 근거: FR49(로그 분리 관리), FR44(색인 작업 기록), FR47(감사 로그)

### Decision Impact Analysis

**Implementation Sequence:**
1. ES 클러스터 + Docker Compose 구성
2. 데이터 매핑 계층 (search 모듈) 구축
3. Outbox 테이블 + 인덱싱 파이프라인
4. Read/Write Alias 기반 인덱스 관리
5. 검색 API + Circuit Breaker
6. 권한 필터링 (Index-time 필드 + Query-time 재검증)
7. Frontend searchStore + 검색 UI
8. 로그 분리 + 모니터링

**Cross-Component Dependencies:**
- 인덱싱 파이프라인(1-1) → Outbox 패턴(3-3) → Alias 전략(1-2)에 의존
- 권한 필터링(2-2) → 데이터 매핑 계층(1-3)에서 공개 범위 필드 색인 필요
- 검색 API(3-1) → Circuit Breaker(3-2) + 권한 필터링(2-2) 통합
- Frontend searchStore(4-1) → 검색 API(3-1) 완성 후 구현 가능

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 7개 영역에서 AI Agent 간 불일치 가능성 식별

### Naming Patterns

**Database Naming Conventions (기존 CMS 관례 준수):**
- 테이블: `snake_case` + `cms_search_` 접두사 — `cms_search_outbox`, `cms_search_log`, `cms_search_audit_log`
- 컬럼: `snake_case` — `document_id`, `index_name`, `retry_count`, `is_processed`
- PK: `idx` (Long) — 기존 엔티티 관례 유지
- FK: `_idx` 접미사 — `board_master_idx`, `menu_idx`
- 인덱스: `idx_` 접두사 — `idx_outbox_status`, `idx_search_log_date`
- Boolean: `is_` 접두사 + `TINYINT(1)` — `is_processed`, `is_deleted`
- 감사 컬럼: `created_date`, `updated_date` (AuditingEntityListener)

**API Naming Conventions (기존 CMS 관례 준수):**
- 엔드포인트: `/back-api/search/*`
- REST 자원: `/back-api/search` (검색), `/back-api/search/index` (색인 관리), `/back-api/search/admin` (관리자)
- 쿼리 파라미터: `camelCase` — `keyword`, `searchKeys`, `contentType`, `startDate`, `endDate`, `page`, `size`
- 응답: 기존 `ApiResponse<T>` 래퍼 재사용 — `{code, message, data}`

**Backend Code Naming Conventions (기존 CMS 관례 준수):**
- 패키지: `kr.co.itid.cms.search.*`
- Service: `SearchService` / `SearchServiceImpl`, `IndexingService` / `IndexingServiceImpl`
- Repository: `SearchOutboxRepository`, `SearchOutboxRepositoryCustom` / `SearchOutboxRepositoryImpl`
- DTO: `SearchRequest`, `SearchResponse`, `IndexingEventRequest`
- Mapper: `SearchDocumentMapper` (MapStruct, `componentModel = "spring"`)
- Entity: `SearchOutbox`, `SearchLog`, `SearchAuditLog`

**Frontend Code Naming Conventions (기존 CMS 관례 준수):**
- Composable: `useSearchApi()`, `useSearchManage()`
- Store: `useSearchStore` (Pinia `defineStore("search", () => {...})`)
- 타입: `types/search/search.ts`, `types/search/index.ts`
- 컴포넌트: `components/search/SearchBar.vue`, `components/search/SearchResult.vue`

### Structure Patterns

**Backend 패키지 구조:**
```
kr.co.itid.cms.search
├── controller/
│   ├── SearchController.java          (/back-api/search)
│   └── SearchAdminController.java     (/back-api/search/admin)
├── service/
│   ├── SearchService.java
│   ├── SearchServiceImpl.java
│   ├── IndexingService.java
│   └── IndexingServiceImpl.java
├── repository/
│   ├── SearchOutboxRepository.java
│   └── SearchOutboxRepositoryCustom.java / Impl
├── entity/
│   ├── SearchOutbox.java
│   ├── SearchLog.java
│   └── SearchAuditLog.java
├── dto/
│   ├── request/  (SearchRequest, IndexRequest)
│   └── response/ (SearchResponse, IndexStatusResponse)
├── mapper/
│   ├── SearchDocumentMapper.java      (MapStruct)
│   └── ContentIndexMapper.java        (Entity→ES Document 변환)
├── event/
│   ├── IndexingEvent.java
│   └── IndexingEventListener.java
├── config/
│   ├── ElasticsearchConfig.java
│   └── CircuitBreakerConfig.java
└── enums/
    ├── IndexAction.java               (CREATE, UPDATE, DELETE)
    └── OutboxStatus.java              (PENDING, PROCESSING, COMPLETED, FAILED)
```

**Frontend 구조:**
```
composables/api/search/
├── useSearchApi.ts                    (검색 API 호출)
└── useSearchManage.ts                 (관리자 색인 관리)

stores/
└── searchStore.ts                     (검색 상태 관리)

components/search/
├── SearchBar.vue                      (검색 입력 + 자동완성)
├── SearchResult.vue                   (결과 목록)
├── SearchFilter.vue                   (필터 패널)
└── SearchHighlight.vue                (하이라이트 텍스트)

types/search/
├── search.ts                          (SearchRequest, SearchResponse 타입)
└── index.ts                           (IndexStatus 타입)
```

### Format Patterns

**API Response Formats (기존 ApiResponse 재사용):**
```json
{
  "code": 200,
  "message": "성공",
  "data": {
    "items": [...],
    "totalCount": 150,
    "page": 0,
    "size": 10,
    "facets": {...}
  }
}
```

**Graceful Degradation 응답:**
```json
{
  "code": 503,
  "message": "검색 서비스가 일시적으로 사용할 수 없습니다",
  "data": null
}
```

**ES Document Format:**
```json
{
  "documentId": "content_123",
  "contentType": "POST",
  "title": "...",
  "body": "...",
  "menuId": 45,
  "boardId": "notice",
  "visibility": "PUBLIC",
  "createdDate": "2026-02-04T10:00:00",
  "updatedDate": "2026-02-04T12:00:00"
}
```

**날짜/시간:** ISO 8601 (`LocalDateTime` → `"2026-02-04T10:00:00"`)

### Communication Patterns

**Spring Event 패턴:**
- 이벤트 클래스: `IndexingEvent`
- 액션 enum: `IndexAction.CREATE`, `IndexAction.UPDATE`, `IndexAction.DELETE`
- 리스너: `@EventListener` + `@Async`

**Outbox Entity 구조:**
```java
@Entity
@Table(name = "cms_search_outbox")
public class SearchOutbox {
    private Long idx;
    private String contentType;      // "POST", "PAGE", "MENU"
    private Long contentIdx;         // 원본 데이터 PK
    private String action;           // "CREATE", "UPDATE", "DELETE"
    private String payload;          // JSON 직렬화 데이터
    private String status;           // "PENDING", "PROCESSING", "COMPLETED", "FAILED"
    private Integer retryCount;
    private LocalDateTime createdDate;
    private LocalDateTime processedDate;
}
```

**Frontend State Management (Pinia 관례):**
```typescript
export const useSearchStore = defineStore("search", () => {
  const keyword = ref("");
  const results = ref<SearchItem[]>([]);
  const totalCount = ref(0);
  const page = ref(0);
  const isLoading = ref(false);
  const isServiceDown = ref(false);

  const search = async (query: string) => { /* ... */ };
  const clearResults = () => { /* ... */ };

  return { keyword, results, totalCount, page, isLoading, isServiceDown, search, clearResults };
});
```

### Process Patterns

**Error Handling (기존 GlobalExceptionHandler 확장):**
- 검색 전용 예외: `SearchServiceException extends EgovBizException`
- Circuit Breaker 발동 시: 503 반환
- ES 연결 실패: `SearchServiceException` → 503
- 색인 실패: Outbox status = FAILED, retryCount 증가, 로그 기록

**Logging (기존 LoggingUtil 확장):**
- Action enum 추가: `SEARCH`, `INDEX`, `REINDEX`, `SEARCH_CONFIG`
- 분리된 Logger 파일:
  - `search-query.log` → 검색 쿼리, 결과 수, 응답 시간
  - `search-index.log` → 색인 액션, 콘텐츠 타입, 상태
  - `search-audit.log` → 검색 설정 변경, 사용자 ID

**Frontend Error Handling (safeFetch 관례):**
```typescript
const res = await safeFetch<SearchResponse>("/search", {
  query: { keyword, page, size },
  throwOnFail: false,
  defaultValue: { code: 503, message: "검색 서비스 사용 불가", data: null }
});
if (res.code === 503) {
  searchStore.isServiceDown = true;
}
```

### Enforcement Guidelines

**모든 AI Agent 필수 준수:**
1. 검색 관련 DB 테이블은 반드시 `cms_search_` 접두사 사용
2. API 응답은 반드시 `ApiResponse<T>` 래퍼 사용 — 별도 응답 형식 금지
3. Service는 반드시 Interface + Impl 패턴 사용, `EgovAbstractServiceImpl` 상속
4. MapStruct `componentModel = "spring"` 설정 필수
5. ES 직접 접근은 `search` 패키지 내부에서만 — 다른 패키지에서 ES 직접 호출 금지
6. Frontend API 호출은 반드시 `safeFetch` 경유
7. 검색 로그는 분리된 Logger 파일에 기록 — 기존 CMS 로그와 혼합 금지

### Anti-Patterns

**금지 패턴:**
- ❌ CMS 기존 Service에서 직접 `ElasticsearchRestTemplate` 호출
- ❌ 검색 API에서 `ApiResponse` 대신 커스텀 응답 객체 사용
- ❌ ES Document에 사용자별 권한 정보 직접 저장
- ❌ 검색 실패 시 CMS 전체 에러 페이지 표시 (대신 Graceful Degradation)
- ❌ Outbox 테이블 없이 직접 ES API 호출만으로 색인 (이벤트 유실 위험)
- ❌ 검색 로그를 기존 CMS 로그 파일에 혼합 기록

## Project Structure & Boundaries

### Complete Project Directory Structure (신규 추가분)

**Backend (cms_backend/src/main/java/kr/co/itid/cms/search/):**
```
search/
├── controller/
│   ├── SearchController.java              # 통합 검색 API
│   ├── SearchSuggestController.java       # 자동완성/인기검색어 API
│   └── SearchAdminController.java         # 관리자 색인 관리 API
├── service/
│   ├── SearchService.java                 # 검색 서비스 인터페이스
│   ├── SearchServiceImpl.java             # 검색 서비스 구현
│   ├── IndexingService.java               # 색인 서비스 인터페이스
│   ├── IndexingServiceImpl.java           # 색인 서비스 구현
│   ├── OutboxPollingService.java          # Outbox 폴링/재시도
│   └── IndexAliasService.java             # Read/Write Alias 관리
├── repository/
│   ├── SearchOutboxRepository.java        # JPA Repository
│   ├── SearchOutboxRepositoryCustom.java  # QueryDSL Custom
│   ├── SearchOutboxRepositoryImpl.java    # QueryDSL Impl
│   ├── SearchLogRepository.java
│   └── SearchAuditLogRepository.java
├── document/
│   ├── ContentDocument.java               # ES @Document (게시글)
│   ├── PageDocument.java                  # ES @Document (페이지)
│   └── MenuDocument.java                  # ES @Document (메뉴)
├── es/
│   ├── ContentDocumentRepository.java     # ElasticsearchRepository
│   ├── PageDocumentRepository.java
│   ├── MenuDocumentRepository.java
│   └── CustomSearchRepository.java        # 복합 검색 쿼리
├── entity/
│   ├── SearchOutbox.java                  # Outbox 엔티티
│   ├── SearchLog.java                     # 검색 로그 엔티티
│   └── SearchAuditLog.java               # 감사 로그 엔티티
├── dto/
│   ├── request/
│   │   ├── SearchRequest.java             # 통합 검색 요청
│   │   ├── IndexRequest.java              # 수동 색인 요청
│   │   └── ReindexRequest.java            # 전체 재색인 요청
│   └── response/
│       ├── SearchResponse.java            # 검색 결과 응답
│       ├── SearchItemResponse.java        # 개별 결과 항목
│       ├── IndexStatusResponse.java       # 색인 상태 응답
│       └── SearchFacetResponse.java       # 패싯 필터 응답
├── mapper/
│   ├── ContentIndexMapper.java            # 게시글 Entity→ES Document
│   ├── PageIndexMapper.java               # 페이지 Entity→ES Document
│   └── MenuIndexMapper.java               # 메뉴 Entity→ES Document
├── event/
│   ├── IndexingEvent.java                 # 색인 이벤트 클래스
│   └── IndexingEventListener.java         # 비동기 이벤트 리스너
├── filter/
│   └── PermissionPostFilter.java          # Query-time 권한 재검증
├── config/
│   ├── ElasticsearchConfig.java           # ES 연결 설정
│   ├── CircuitBreakerConfig.java          # Resilience4j 설정
│   └── SearchLogConfig.java              # 검색 로그 분리 설정
└── enums/
    ├── IndexAction.java                   # CREATE, UPDATE, DELETE
    ├── OutboxStatus.java                  # PENDING, PROCESSING, COMPLETED, FAILED
    ├── ContentType.java                   # POST, PAGE, MENU
    └── Visibility.java                    # PUBLIC, MEMBER, ADMIN
```

**Frontend (egovframe-cms-frontend-q/egovframe-cms-frontend-q/):**
```
composables/api/search/
├── useSearchApi.ts                        # 검색 API 호출
├── useSearchSuggest.ts                    # 자동완성 API 호출
└── useSearchAdmin.ts                      # 관리자 색인 관리 API

stores/
└── searchStore.ts                         # 검색 상태 관리

components/search/
├── SearchBar.vue                          # 통합 검색바 + 자동완성
├── SearchResult.vue                       # 결과 목록 컨테이너
├── SearchResultItem.vue                   # 개별 결과 항목
├── SearchFilter.vue                       # 콘텐츠 유형/날짜 필터
├── SearchHighlight.vue                    # 키워드 하이라이팅
├── SearchPagination.vue                   # 검색 결과 페이징
└── SearchServiceDown.vue                  # 서비스 장애 안내

pages/
└── search/
    └── index.vue                          # 검색 결과 페이지

types/search/
├── search.ts                              # 검색 관련 타입 정의
└── admin.ts                               # 관리자 색인 타입 정의
```

### Existing Code Modifications (최소 침습)

```
# Backend 설정 변경
build.gradle                               # + ES, Resilience4j 의존성
src/main/resources/application.yml         # + ES 연결 설정
src/main/resources/logback-spring.xml      # + 검색 로그 appender

# 기존 CMS Service에 이벤트 발행 추가 (1줄 추가)
service/impl/BoardServiceImpl.java         # + eventPublisher.publishEvent(IndexingEvent)
service/impl/ContentServiceImpl.java       # + eventPublisher.publishEvent(IndexingEvent)
service/impl/MenuServiceImpl.java          # + eventPublisher.publishEvent(IndexingEvent)

# Docker
docker/docker-compose.yml                  # + elasticsearch 서비스

# Frontend
layouts/default-main.vue                   # + SearchBar 컴포넌트 통합
```

### Architectural Boundaries

**API Boundaries:**

| 경계 | 접근 방향 | 설명 |
|------|----------|------|
| `/back-api/search` | Frontend → Backend | 검색 쿼리 API (공개) |
| `/back-api/search/suggest` | Frontend → Backend | 자동완성 API (공개) |
| `/back-api/search/admin/*` | Admin UI → Backend | 색인 관리 (MANAGE 권한) |
| Backend → ES | 내부 전용 | search 패키지만 접근 |
| CMS → IndexingEvent | 단방향 이벤트 | 느슨한 결합 |

**Component Boundaries:**
- **CMS → Search**: `ApplicationEventPublisher` 단방향 이벤트만 허용. CMS에서 검색 서비스 직접 호출 금지
- **Search → CMS**: 권한 재검증 시 기존 `PermissionService` 호출 (읽기 전용)
- **Search → ES**: `search.es` 패키지 내 Repository만 ES 접근
- **Frontend**: `searchStore` ↔ `useSearchApi` ↔ `/back-api/search` 단일 경로

**Data Boundaries:**
- **MySQL 신규 테이블**: `cms_search_outbox`, `cms_search_log`, `cms_search_audit_log`
- **Elasticsearch**: `cms_content_v{n}` 인덱스 + `cms_content_read`/`cms_content_write` alias
- **Redis**: 인기 검색어 캐시 (선택적, Growth 단계)

### Requirements to Structure Mapping

| FR 그룹 | 주요 파일/컴포넌트 |
|---------|-------------------|
| FR1-5 (통합 검색) | `SearchController`, `SearchServiceImpl`, `useSearchApi`, `SearchBar.vue`, `SearchResult.vue` |
| FR6-10 (필터/패싯) | `SearchRequest`, `SearchFacetResponse`, `SearchFilter.vue`, `CustomSearchRepository` |
| FR11-13 (자동완성) | `SearchSuggestController`, `useSearchSuggest`, `SearchBar.vue` |
| FR14-19 (색인 동기화) | `IndexingService`, `IndexingEvent`, `IndexingEventListener`, `OutboxPollingService` |
| FR20-25 (권한 필터링) | `PermissionPostFilter`, `ContentDocument.visibility` |
| FR36-39 (확장성) | `ContentType` enum, `document/` 패키지 |
| FR40-45 (재색인/정합성) | `IndexAliasService`, `ReindexRequest`, `OutboxPollingService` |
| FR46 (장애 격리) | `CircuitBreakerConfig`, `SearchServiceException`, `SearchServiceDown.vue` |
| FR47-49 (로깅) | `SearchLogConfig`, `SearchLog`, `SearchAuditLog` |

### Data Flow

```
[CMS CRUD 작업]
    │
    ├─ 1. Service에서 eventPublisher.publishEvent(IndexingEvent)
    ├─ 2. 동일 트랜잭션에서 cms_search_outbox INSERT
    └─ 3. IndexingEventListener (@Async)
           ├─ ES Write Alias로 색인
           │   ├─ 성공 → Outbox status = COMPLETED
           │   └─ 실패 → Outbox status = FAILED
           └─ OutboxPollingService (스케줄러)
                 └─ FAILED/PENDING 레코드 재시도

[검색 요청]
    │
    ├─ 1. Frontend searchStore.search() → safeFetch("/back-api/search")
    ├─ 2. SearchController → CircuitBreaker 래핑
    │     ├─ ES 정상 → SearchServiceImpl.search()
    │     └─ ES 장애 → 503 Graceful Degradation
    ├─ 3. ES Read Alias로 BM25 검색 + visibility 1차 필터
    ├─ 4. PermissionPostFilter → 상위 N건 CMS 권한 재검증
    └─ 5. SearchResponse 반환
```

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
- Spring Boot 2.7.12 + Spring Data Elasticsearch 4.4.x + ES 7.17.x → 공식 호환
- Resilience4j → Spring Boot 2.7.x 호환
- Element Plus Autocomplete → 기존 프로젝트 사용 중
- Outbox 패턴(MySQL) + Spring Events → 동일 JVM, 동일 트랜잭션 보장
- 모든 기술 선택 간 충돌 없음

**Pattern Consistency:**
- DB 네이밍: 기존 `cms_` → `cms_search_` 확장 (일관)
- API: `/back-api/` + `ApiResponse<T>` 래퍼 재사용 (일관)
- Service: Interface + Impl + EgovAbstractServiceImpl 상속 (일관)
- Frontend: safeFetch + Pinia store + composable (일관)
- 모순되는 패턴 없음

**Structure Alignment:**
- 모든 검색 코드가 `kr.co.itid.cms.search` 패키지에 격리
- CMS → Search 단방향 이벤트만 허용 (경계 준수)
- Frontend 단일 경로: searchStore ↔ useSearchApi ↔ /back-api/search

### Requirements Coverage ✅

**FR 전수 검증 (46개 FR):**

| FR 그룹 | FR 번호 | 커버리지 | 상태 |
|---------|---------|---------|------|
| 통합 검색 | FR1-4 | SearchController, CustomSearchRepository, BM25 | ✅ |
| 결과 표시 | FR5-8 | SearchItemResponse, SearchResultItem.vue | ✅ |
| 필터/정렬 | FR9-12 | SearchRequest 필터, SearchFilter.vue | ✅ |
| 권한 필터 | FR13-17 | Dual 필터링, PermissionPostFilter, JWT 재사용 | ✅ |
| 색인 수집 | FR18-20 | ContentDocument, PageDocument, MenuDocument | ✅ |
| 색인 동기화 | FR21-23 | IndexingEvent, Outbox, 배치 동기화 | ✅ |
| 재색인/복구 | FR24-30 | IndexAliasService, OutboxPollingService, ReindexRequest | ✅ |
| 검색 UI | FR31-35 | SearchBar in layout, KWCAG 2.2, noindex | ✅ |
| 장애 격리 | FR36-38 | CircuitBreaker, Graceful Degradation | ✅ |
| 로깅/감사 | FR39-42 | 3종 분리 로그, SearchAuditLog | ✅ |
| 확장성/API | FR43-46 | ContentType enum, 분리 모듈, REST API | ✅ |

**NFR 검증:**

| NFR | 대응 | 상태 |
|-----|------|------|
| 300ms / P99 1s | ES BM25, Circuit Breaker 타임아웃 | ✅ |
| ES 내부망 + TLS | API Key + TLS, IP Allowlist | ✅ |
| 10만→100만 | Alias Swap, Single→3-Node | ✅ |
| KWCAG 2.2 | Element Plus a11y, 키보드 탐색 | ✅ |
| JWT+Redis 통합 | 기존 인증 재사용 | ✅ |
| 99.9% 가용성 | Circuit Breaker, Graceful Degradation, Outbox | ✅ |

### Implementation Readiness ✅

**Decision Completeness:**
- 모든 결정에 기술 버전 명시
- 패키지/파일/클래스명 수준 정의
- 기존 CMS 관례 기반 일관성 확보

**Structure Completeness:**
- Backend 20+ 클래스, Frontend 15+ 파일 명세
- 기존 코드 변경점 최소 침습 정의
- Data Flow 다이어그램 포함

**Pattern Completeness:**
- Anti-Pattern 목록으로 금지 사항 명시
- Enforcement Guidelines 7개 항목
- 구체적 코드 예시 포함

### Gap Analysis

**Critical Gaps:** 없음

**Minor Gaps (비차단, 구현 시 결정):**
1. FR42 검색 로그 보관 기간/접근 통제 세부사항 → logback rolling policy로 해결
2. FR44 관리자 검색 대상 설정 UI → Growth 단계 UX 설계에서 구체화
3. 모니터링 대시보드 → Growth 단계 운영 도구에서 대응

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] 프로젝트 컨텍스트 분석 완료
- [x] 규모 및 복잡도 평가 (High, Brownfield)
- [x] 기술 제약사항 식별
- [x] Cross-cutting concerns 매핑 (5개)

**✅ Architectural Decisions**
- [x] Critical decisions 4개 (파이프라인, 권한, 격리, 이벤트)
- [x] Important decisions 4개 (Alias, 매핑, Store, 로그)
- [x] 기술 버전 명시 및 호환성 검증
- [x] Cross-component dependencies 정의

**✅ Implementation Patterns**
- [x] 네이밍 규칙 (DB, API, Backend, Frontend)
- [x] 구조 패턴 (패키지, 파일)
- [x] 통신 패턴 (Event, Outbox, State)
- [x] 프로세스 패턴 (에러, 로깅)
- [x] 금지 패턴 (Anti-Patterns)

**✅ Project Structure**
- [x] 완전한 디렉토리 구조 (Backend + Frontend)
- [x] 아키텍처 경계 정의
- [x] FR → 파일 매핑 완료
- [x] Data Flow 다이어그램

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High

**Key Strengths:**
- 기존 CMS 관례 100% 준수로 일관성 보장
- Outbox + Dual-write + Alias Swap으로 데이터 정합성 확보
- Circuit Breaker + Graceful Degradation으로 장애 격리 완전
- 최소 침습 설계 (기존 코드 변경 3개 Service에 이벤트 발행 1줄씩)

**Areas for Future Enhancement:**
- 시맨틱 검색 (Growth 단계)
- 모니터링 대시보드 (Growth 단계)
- 관리자 검색 설정 UI (Growth 단계)
- Redis 인기 검색어 캐시 (Growth 단계)

### Implementation Handoff

**AI Agent Guidelines:**
- 모든 아키텍처 결정을 문서 그대로 따를 것
- Implementation Patterns의 Enforcement Guidelines 7개 항목 필수 준수
- Anti-Patterns에 명시된 금지 패턴 위반 금지
- 불확실한 결정은 이 문서를 참조

**First Implementation Priority:**
1. `build.gradle`에 ES + Resilience4j 의존성 추가
2. `docker-compose.yml`에 Elasticsearch 7.17.x 서비스 추가
3. `ElasticsearchConfig.java` 연결 설정
4. `SearchOutbox` 엔티티 + 테이블 생성
5. 첫 번째 Document (ContentDocument) + IndexMapper 구현
