# Story 1.5: 장애 격리 및 Graceful Degradation

Status: review

## Story

As a CMS 사용자,
I want 검색 서비스 장애 시에도 CMS 기본 기능을 정상 이용하여,
So that 검색 장애가 전체 서비스에 영향을 주지 않는다.

## Acceptance Criteria

1. **AC1**: ES 다운 시 검색 API가 Circuit Breaker를 통해 503 + "검색 서비스가 일시적으로 사용할 수 없습니다" 메시지를 반환한다
2. **AC2**: CMS 게시판, 페이지, 메뉴 등 기본 기능은 검색 장애와 무관하게 정상 동작한다
3. **AC3**: 프론트엔드에서 503 수신 시 SearchServiceDown 컴포넌트가 표시된다
4. **AC4**: ES 복구 후 Circuit Breaker half-open → 정상 검색 자동 복구
5. **AC5**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: SearchServiceException 생성 (AC: #1)
  - [x] 1.1 검색 서비스 전용 예외 클래스

- [x] Task 2: SearchServiceImpl에 @CircuitBreaker 적용 (AC: #1, #4)
  - [x] 2.1 search() 메서드에 Resilience4j @CircuitBreaker 어노테이션
  - [x] 2.2 fallback 메서드에서 SearchServiceException throw

- [x] Task 3: SearchController에 예외 처리 (AC: #1)
  - [x] 3.1 SearchServiceException catch → 503 ApiResponse

- [x] Task 4: GlobalExceptionHandler에 SearchServiceException 핸들러 (AC: #1)
  - [x] 4.1 SearchServiceException → 503 SERVICE_UNAVAILABLE

- [x] Task 5: 테스트 작성 (AC: #5)
  - [x] 5.1 SearchServiceException 테스트
  - [x] 5.2 SearchController 503 fallback 테스트
  - [x] 5.3 gradle compileJava + test 성공

## Dev Notes

### Architecture 결정사항
- Resilience4j Circuit Breaker (resilience4j-spring-boot2:1.7.1)
- application.yml: searchService 인스턴스 (sliding-window=10, failure-rate=50%, wait=30s)
- 장애 시 503 + ApiResponse 표준 형식
- 프론트엔드: searchStore.isServiceDown = true → SearchServiceDown.vue 렌더링 (Story 1.4에서 구현 완료)

### 주의사항
- **금지**: CMS 기존 코드의 예외 처리 변경
- **금지**: 검색 외 서비스에 Circuit Breaker 적용
- GlobalExceptionHandler 기존 핸들러와 충돌 없이 추가

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradlew compileJava: BUILD SUCCESSFUL
- gradlew test --tests 'kr.co.itid.cms.search.*': BUILD SUCCESSFUL
  - SearchControllerTest: 9 tests, 0 failures (7 기존 + 2 신규 503 테스트)
  - SearchServiceExceptionTest: 3 tests, 0 failures

### Completion Notes List

- **SearchServiceException.java**: RuntimeException 상속, message/cause 생성자. Circuit Breaker fallback에서 throw됨
- **SearchServiceImpl.java**: `@CircuitBreaker(name = "searchService", fallbackMethod = "searchFallback")` 적용, fallback에서 SearchServiceException throw + 경고 로그
- **SearchController.java**: try/catch로 SearchServiceException → 503 ApiResponse.error 반환
- **GlobalExceptionHandler.java**: SearchServiceException → 503 SERVICE_UNAVAILABLE 핸들러 추가 (safety net)
- **SearchServiceExceptionTest.java**: 3개 테스트 (메시지 생성, cause 포함 생성, RuntimeException 상속 확인)
- **SearchControllerTest.java**: 2개 테스트 추가 (503 반환, cause 포함 503 반환)

### Change Log

- 2026-02-05: Story 1.5 구현 완료 - 장애 격리 및 Graceful Degradation

### File List

**신규 생성:**
- cms_backend/src/main/java/kr/co/itid/cms/search/exception/SearchServiceException.java
- cms_backend/src/test/java/kr/co/itid/cms/search/exception/SearchServiceExceptionTest.java

**수정:**
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/SearchServiceImpl.java (+@CircuitBreaker, +fallback)
- cms_backend/src/main/java/kr/co/itid/cms/search/controller/SearchController.java (+try/catch SearchServiceException)
- cms_backend/src/main/java/kr/co/itid/cms/config/exception/GlobalExceptionHandler.java (+SearchServiceException handler)
- cms_backend/src/test/java/kr/co/itid/cms/search/controller/SearchControllerTest.java (+2 tests)
