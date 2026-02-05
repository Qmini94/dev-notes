# Story 1.3: 통합 검색 API

Status: review

## Story

As a 사용자,
I want 단일 키워드로 CMS 전체 콘텐츠(게시글, 페이지, 메뉴)를 검색하여,
So that 원하는 정보를 빠르게 찾을 수 있다.

## Acceptance Criteria

1. **AC1**: `GET /back-api/search?keyword=소상공인&page=0&size=10` 요청 시 BM25 기반 검색 결과가 `ApiResponse<T>` 형식으로 반환된다 (title, snippet, contentType, sourceUrl 포함)
2. **AC2**: keyword가 빈 문자열이거나 null이면 HTTP 400과 안내 메시지를 반환한다
3. **AC3**: 검색 결과가 없으면 빈 결과 목록과 재검색 안내 메시지를 반환한다
4. **AC4**: `page=1, size=10` 요청 시 11-20번째 결과가 반환되고 `totalCount`로 전체 건수가 포함된다
5. **AC5**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: 검색 DTO 클래스 생성 (AC: #1, #4)
  - [x] 1.1 SearchRequest (keyword, page, size)
  - [x] 1.2 SearchResponse (items, totalCount, page, size)
  - [x] 1.3 SearchItemResponse (documentId, title, snippet, contentType, sourceUrl, createdDate, updatedDate)

- [x] Task 2: SearchService 구현 (AC: #1, #3)
  - [x] 2.1 SearchService 인터페이스
  - [x] 2.2 SearchServiceImpl - BM25 multi_match 검색, highlighting, pagination

- [x] Task 3: SearchController 구현 (AC: #1, #2, #4)
  - [x] 3.1 GET /back-api/search 엔드포인트
  - [x] 3.2 keyword 빈 문자열 검증 → 400
  - [x] 3.3 ApiResponse 래핑

- [x] Task 4: 테스트 작성 (AC: #5)
  - [x] 4.1 DTO 단위 테스트
  - [x] 4.2 SearchController 단위 테스트
  - [x] 4.3 gradle compileJava + test 성공 확인

## Dev Notes

### 기존 프로젝트 패턴 (반드시 준수)

**Controller 패턴:**
- `@RestController` + `@RequiredArgsConstructor` + `@RequestMapping("/back-api/...")`
- 반환: `ResponseEntity<ApiResponse<T>>`
- ApiResponse: `ApiResponse.success(data)`, `ApiResponse.error(code, message)`

**ES 검색 패턴:**
- Read Alias: `cms_content_read` (SearchProperties.alias.read)
- ContentDocument: title(Text), body(Text), contentType(Keyword), visibility(Keyword)
- BM25 multi_match: title^2 boost, body 기본 가중치

### 주의사항

- **금지**: Permission 기반 필터링 (Story 2.2 범위)
- **금지**: JWT 인증 연동 (Story 2.3 범위)
- **금지**: 프론트엔드 UI (Story 1.4 범위)
- Story 1.3에서는 visibility=PUBLIC 필터만 적용

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradle compileJava: BUILD SUCCESSFUL (UP-TO-DATE)
- gradle test --tests "kr.co.itid.cms.search.*": BUILD SUCCESSFUL (8 test classes 통과)

### Completion Notes List

- SearchRequest: keyword, page(default 0), size(default 10) DTO
- SearchResponse: items, totalCount, page, size Builder 패턴
- SearchItemResponse: documentId, title, snippet, contentType, sourceUrl, createdDate, updatedDate + @JsonInclude(NON_NULL)
- SearchService: 인터페이스 정의 (search 메서드)
- SearchServiceImpl: BM25 multi_match (title^2, body), highlighting(<strong> 태그), visibility=PUBLIC 필터, Read Alias 사용
- SearchController: GET /back-api/search, keyword 빈 문자열 400, size 제한(1-100), ApiResponse 래핑
- SearchControllerTest: 7개 테스트 (null/empty/whitespace keyword 400, valid 200, empty results, negative page, oversized size)
- SearchDtoTest: 5개 테스트 (Request defaults/setters, ItemResponse builder, Response builder, empty response)

### Change Log

- 2026-02-05: Story 1.3 구현 완료 - 통합 검색 API (세션 중단 후 재개하여 빌드 검증 완료)

### File List

**신규 생성:**
- cms_backend/src/main/java/kr/co/itid/cms/search/dto/request/SearchRequest.java
- cms_backend/src/main/java/kr/co/itid/cms/search/dto/response/SearchResponse.java
- cms_backend/src/main/java/kr/co/itid/cms/search/dto/response/SearchItemResponse.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/SearchService.java
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/SearchServiceImpl.java
- cms_backend/src/main/java/kr/co/itid/cms/search/controller/SearchController.java
- cms_backend/src/test/java/kr/co/itid/cms/search/controller/SearchControllerTest.java
- cms_backend/src/test/java/kr/co/itid/cms/search/dto/SearchDtoTest.java

**수정:**
- (없음 - 기존 CMS 코드 수정 없음)
