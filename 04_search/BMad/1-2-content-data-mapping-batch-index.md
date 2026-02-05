# Story 1.2: 콘텐츠 데이터 매핑 및 초기 배치 색인

Status: review

## Story

As a 시스템 운영자,
I want 기존 CMS 콘텐츠(게시글, 페이지, 메뉴)를 검색 인덱스에 초기 적재하여,
So that 검색 서비스가 기존 데이터를 대상으로 동작할 수 있다.

## Acceptance Criteria

1. **AC1**: CMS의 게시판 게시글이 ContentDocument(contentType=POST)로 변환되어 ES에 색인된다
2. **AC2**: 페이지 콘텐츠가 ContentDocument(contentType=PAGE)로 변환되어 ES에 색인된다
3. **AC3**: 메뉴명/메뉴 경로가 ContentDocument(contentType=MENU)로 변환되어 ES에 색인된다
4. **AC4**: 각 Document에 documentId, contentType, title, body, menuId, boardId, visibility, createdDate, updatedDate 필드가 포함된다
5. **AC5**: cms_content_read alias가 정상 동작하고 색인된 문서 수가 원본 데이터 수와 일치한다
6. **AC6**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: ContentDocument 클래스 생성 (AC: #4)
  - [x] 1.1 `kr.co.itid.cms.search.document.ContentDocument` @Document 클래스 생성
  - [x] 1.2 ES 필드 매핑 정의 (title: Text, body: Text, contentType: Keyword, visibility: Keyword 등)
  - [x] 1.3 documentId 형식: `{contentType}_{sourceIdx}` (예: post_123, page_456, menu_789)

- [x] Task 2: ES 인덱스 설정 서비스 (AC: #5)
  - [x] 2.1 IndexSetupService 생성 - 인덱스 생성 및 alias 관리
  - [x] 2.2 cms_content_v1 인덱스 생성 (매핑 + 설정)
  - [x] 2.3 cms_content_read / cms_content_write alias 설정

- [x] Task 3: 배치 색인용 데이터 조회 DAO (AC: #1, #2, #3)
  - [x] 3.1 BatchIndexDao - 동적 게시판 테이블에서 전체 게시글 조회 (JDBC)
  - [x] 3.2 ContentRepository 활용 - 활성 페이지 콘텐츠 조회 (JPA)
  - [x] 3.3 MenuRepository 활용 - 표시 메뉴 조회 (JPA)

- [x] Task 4: IndexingService 구현 (AC: #1, #2, #3, #4, #5)
  - [x] 4.1 IndexingService 인터페이스 정의
  - [x] 4.2 IndexingServiceImpl - 초기 배치 색인 로직
  - [x] 4.3 Bulk API로 배치 색인 (500건 단위)
  - [x] 4.4 색인 결과 로깅 및 카운트 검증

- [x] Task 5: 테스트 작성 (AC: #6)
  - [x] 5.1 ContentDocument 단위 테스트
  - [x] 5.2 BatchIndexDao 보안 검증 테스트
  - [x] 5.3 IndexingResult 단위 테스트
  - [x] 5.4 gradle compileJava + test 성공 확인

## Dev Notes

### 기존 프로젝트 패턴 (반드시 준수)

**CMS 엔티티 구조:**
- `Content` 엔티티 (`cms_content`): idx, parentId, isUse, isMain, title, content, hostname, createdDate, updatedDate
- `Menu` 엔티티 (`cms_menu`): id, parentId, title, name, type, value, isShow, pathUrl, pathString, level
- `BoardMaster` 엔티티 (`cms_board_master`): idx, boardId, boardName, boardType, isUse
- 동적 게시판 테이블 (`board_{boardId}`): idx, board_id, menu_id, title, content, is_deleted, created_date, updated_date

**Content-Menu 관계:**
- `ContentMasterMenu` (cms_content_master_menu): content_idx ↔ menu_id 매핑
- `BoardMasterMenu` (cms_board_master_menu): board_master_idx ↔ menu_id 매핑
- Menu.type='board' → Menu.value = BoardMaster.idx (CAST AS UNSIGNED)
- Menu.type='content' → Menu.value = Content.idx

**동적 게시판 접근 패턴:**
- `DynamicBoardDaoImpl`이 `NamedParameterJdbcTemplate`으로 `board_{boardId}` 테이블 접근
- `resolveBoardIdByMenuId()`: menuId → boardId 변환 (cms_menu JOIN cms_board_master)
- 모든 동적 테이블은 `is_deleted=0` 조건으로 soft-delete 필터링

**Visibility 결정 (Story 1.2 기본 정책):**
- Menu.isShow=true → PUBLIC (기본)
- Menu.isShow=false → ADMIN
- Story 2.1에서 Permission 기반 정밀 visibility 구현 예정

**검색 패키지 구조 (Story 1.1에서 생성):**
```
kr.co.itid.cms.search/
├── config/
│   ├── ElasticsearchConfig.java (AbstractElasticsearchConfiguration 상속)
│   ├── SearchProperties.java (search.index prefix, alias 설정)
│   └── CircuitBreakerConfig.java
├── enums/
│   ├── ContentType.java (POST, PAGE, MENU)
│   ├── Visibility.java (PUBLIC, MEMBER, ADMIN)
│   ├── IndexAction.java
│   └── OutboxStatus.java
├── document/   ← Story 1.2에서 생성
├── es/         ← Story 1.2에서 생성
├── service/    ← Story 1.2에서 생성
└── repository/ ← Story 1.2에서 생성
```

### 구체적 구현 가이드

#### Task 1: ContentDocument

```java
package kr.co.itid.cms.search.document;

@Document(indexName = "#{@searchProperties.alias.write}")
@Setting(shards = 1, replicas = 0)
public class ContentDocument {
    @Id
    private String documentId;           // post_123, page_456, menu_789

    @Field(type = FieldType.Keyword)
    private String contentType;          // POST, PAGE, MENU

    @Field(type = FieldType.Text, analyzer = "standard")
    private String title;

    @Field(type = FieldType.Text, analyzer = "standard")
    private String body;

    @Field(type = FieldType.Long)
    private Long menuId;

    @Field(type = FieldType.Keyword)
    private String boardId;              // POST만 해당, 나머지 null

    @Field(type = FieldType.Keyword)
    private String visibility;           // PUBLIC, MEMBER, ADMIN

    @Field(type = FieldType.Date, format = DateFormat.date_hour_minute_second_millis)
    private LocalDateTime createdDate;

    @Field(type = FieldType.Date, format = DateFormat.date_hour_minute_second_millis)
    private LocalDateTime updatedDate;
}
```

#### Task 2: IndexSetupService

- RestHighLevelClient의 indices API 사용
- 인덱스 존재 여부 확인 후 생성
- Read/Write alias 동시 설정
- SearchProperties에서 인덱스명, alias명 참조

#### Task 3: 배치 데이터 조회

**게시글 (board posts):**
```sql
-- 1단계: 활성 BoardMaster 목록 조회
SELECT idx, board_id FROM cms_board_master WHERE is_use = 1

-- 2단계: 각 board_{boardId}에서 게시글 조회
SELECT idx, board_id, menu_id, title, content, created_date, updated_date
FROM board_{boardId}
WHERE is_deleted = 0
```

**페이지 콘텐츠:**
```java
// ContentRepository: isMain=true인 활성 콘텐츠 + ContentMasterMenu로 menuId 매핑
contentRepository.findAll() → filter(isMain && isUse)
contentMasterMenuRepository로 menuId 조회
```

**메뉴:**
```java
// MenuRepository: isShow=true인 메뉴
menuRepository.findAll() → filter(isShow)
```

#### Task 4: IndexingService

```java
public interface IndexingService {
    IndexingResult performFullIndexing();
}

public class IndexingResult {
    private long postCount;
    private long pageCount;
    private long menuCount;
    private long totalIndexed;
    private long failedCount;
}
```

- ElasticsearchRestTemplate의 bulkIndex() 사용
- 500건 단위 배치 처리
- 실패 건수 로깅

### 주의사항 (Anti-Patterns)

- **금지**: 기존 CMS 엔티티/서비스 수정
- **금지**: Document에 사용자 개인 권한 정보 저장 (Story 2.1 범위)
- **금지**: ElasticsearchRepository에 복잡한 검색 쿼리 추가 (Story 1.3 범위)
- **금지**: 실시간 이벤트 기반 색인 (Story 4.x 범위)
- **금지**: ES 8.x API 사용 (Spring Boot 2.7.12 호환 RestHighLevelClient만 사용)

### References

- [Source: architecture.md#ES Document Mappings]
- [Source: architecture.md#Batch Indexing Process]
- [Source: architecture.md#Alias Strategy]
- [Source: epics.md#Story 1.2 Acceptance Criteria]
- [Source: Story 1.1 - ElasticsearchConfig, SearchProperties]
- [Source: DynamicBoardDaoImpl - 동적 게시판 JDBC 접근 패턴]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradle compileJava: BUILD SUCCESSFUL (deprecated API note - RestHighLevelClient, 기존과 동일)
- gradle test --tests "kr.co.itid.cms.search.*": BUILD SUCCESSFUL (6 test classes 통과)

### Completion Notes List

- ContentDocument: @Document 클래스, 9개 필드 (documentId, contentType, title, body, menuId, boardId, visibility, createdDate, updatedDate)
- ContentDocumentRepository: ElasticsearchRepository 확장, countByContentType 메서드
- IndexSetupService/Impl: 인덱스 생성 + Read/Write alias 설정 (RestHighLevelClient indices API)
- BatchIndexDao/Impl: 동적 게시판 JDBC 조회, boardId 보안 검증 (SQL injection 방지)
- IndexingService/Impl: 전체 배치 색인 - POST(동적 게시판), PAGE(cms_content), MENU(cms_menu)
- IndexingResult: 색인 결과 DTO (postCount, pageCount, menuCount, totalIndexed, failedCount)
- Visibility 기본 정책: Menu.isShow=true → PUBLIC, isShow=false → ADMIN (Story 2.1에서 Permission 기반 정밀화 예정)
- Bulk 색인: ElasticsearchRestTemplate.bulkIndex() 500건 단위 배치 처리

### Change Log

- 2026-02-05: Story 1.2 구현 완료 - 콘텐츠 데이터 매핑 및 초기 배치 색인

### File List

**신규 생성:**
- cms_backend/src/main/java/kr/co/itid/cms/search/document/ContentDocument.java
- cms_backend/src/main/java/kr/co/itid/cms/search/es/ContentDocumentRepository.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/IndexSetupService.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/IndexSetupServiceImpl.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/IndexingService.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/IndexingServiceImpl.java
- cms_backend/src/main/java/kr/co/itid/cms/search/repository/BatchIndexDao.java
- cms_backend/src/main/java/kr/co/itid/cms/search/repository/impl/BatchIndexDaoImpl.java
- cms_backend/src/main/java/kr/co/itid/cms/search/dto/IndexingResult.java
- cms_backend/src/test/java/kr/co/itid/cms/search/document/ContentDocumentTest.java
- cms_backend/src/test/java/kr/co/itid/cms/search/dto/IndexingResultTest.java
- cms_backend/src/test/java/kr/co/itid/cms/search/repository/BatchIndexDaoTest.java

**수정:**
- (없음 - 기존 CMS 코드 수정 없음)
