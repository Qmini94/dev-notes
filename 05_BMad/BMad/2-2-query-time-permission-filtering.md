# Story 2.2: Query-time 권한 필터링 및 존재 유추 불가 보장

Status: review

## Story

As a 사용자,
I want 내 권한 범위 내의 콘텐츠만 검색 결과에 보이고, 비공개 콘텐츠는 존재조차 알 수 없게 하여,
So that 보안이 보장된 안전한 검색을 이용할 수 있다.

## Acceptance Criteria

1. **AC1**: 비로그인 사용자가 검색할 때 visibility=PUBLIC 문서만 결과에 포함된다
2. **AC2**: MEMBER/ADMIN 문서의 존재 여부가 totalCount에 반영되지 않는다
3. **AC3**: 로그인 사용자가 검색할 때 ES에서 공개 범위 기반 1차 필터 후 상위 N건에 대해 CMS PermissionService로 최종 권한 재검증을 수행한다
4. **AC4**: totalCount는 Query-time 권한 필터링이 반영된 결과 기준으로 계산한다
5. **AC5**: 인덱스 동기화가 지연된 상태에서 방금 비공개로 전환된 콘텐츠는 Query-time 재검증에서 필터링되어 결과에 포함되지 않는다
6. **AC6**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: SearchService 인터페이스 확장 (AC: #1, #3)
  - [x] 1.1 search(keyword, page, size, user) 오버로드 추가

- [x] Task 2: SearchServiceImpl에 사용자 인식 검색 구현 (AC: #1, #2, #3, #4)
  - [x] 2.1 비로그인(guest/null): visibility=PUBLIC 필터만
  - [x] 2.2 로그인: visibility 기반 1차 필터 + PermissionResolverService로 2차 검증
  - [x] 2.3 권한 없는 문서 제외 후 totalCount 추정 계산

- [x] Task 3: SearchController 수정 (AC: #1, #3)
  - [x] 3.1 SecurityContextHolder에서 현재 사용자 획득
  - [x] 3.2 user 정보를 SearchService.search(keyword, page, size, user)에 전달

- [x] Task 4: PermissionResolverService 확장 (AC: #3, #5)
  - [x] 4.1 hasPermissionForMenu(user, menuId, permission) 추가

- [x] Task 5: 테스트 작성 (AC: #6)
  - [x] 5.1 SearchControllerTest 업데이트 (10개 테스트)
  - [x] 5.2 gradle compileJava + test 성공

## Dev Notes

### Architecture 결정사항

**2단계 권한 필터링 전략:**

```
1차 필터 (ES Query):
├── Guest/null: visibility=PUBLIC only
├── Member (userLevel > 1): visibility IN (PUBLIC, MEMBER)
└── Admin (userLevel = 1): 모든 visibility

2차 필터 (Application):
├── ES 결과의 각 menuId에 대해 VIEW 권한 확인
├── PermissionResolverService.hasPermissionForMenu(user, menuId, VIEW)
├── 권한 없는 항목 제외
└── totalCount 추정 계산 (passRate 기반)
```

**성능 최적화:**
- PERMISSION_FETCH_BUFFER = 2: size * 2배 가져와서 권한 필터링
- PermissionResolverService는 Redis 캐시 사용 (MenuPermissionData)
- menuId가 null인 경우 권한 확인 생략 (PUBLIC으로 간주)

### 주요 코드 변경

**SearchService.java:**
```java
SearchResponse search(String keyword, int page, int size, JwtAuthenticatedUser user);
```

**SearchServiceImpl.java:**
```java
@Override
@CircuitBreaker(name = "searchService", fallbackMethod = "searchFallbackWithUser")
public SearchResponse search(String keyword, int page, int size, JwtAuthenticatedUser user) {
    // 1. null/guest → PUBLIC only 검색
    if (user == null || user.isGuest()) {
        return search(keyword, page, size);
    }

    // 2. 1차 필터: visibility 기반
    // Admin: 모든 visibility
    // Member: PUBLIC, MEMBER

    // 3. 2차 필터: PermissionResolverService.hasPermissionForMenu()
    // 각 결과의 menuId에 대해 VIEW 권한 확인

    // 4. totalCount 추정 계산
}
```

**SearchController.java:**
```java
JwtAuthenticatedUser currentUser = getCurrentUser();
SearchResponse response = searchService.search(keyword.trim(), page, size, currentUser);
```

**PermissionResolverService.java:**
```java
boolean hasPermissionForMenu(JwtAuthenticatedUser user, Long menuId, String permission);
```

### 테스트 결과

- SearchControllerTest: 10 tests, 0 failures
- 전체 search 패키지: BUILD SUCCESSFUL

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradlew compileJava: BUILD SUCCESSFUL
- gradlew test --tests 'kr.co.itid.cms.search.*': BUILD SUCCESSFUL
- SearchControllerTest: 10 tests, 0 failures (기존 9개 + 신규 1개)

### Completion Notes List

- **SearchService.java**: search(keyword, page, size, user) 오버로드 추가
- **SearchServiceImpl.java**: 2단계 권한 필터링 구현, estimateTotalCount() 추정 로직, searchFallbackWithUser() 추가
- **SearchController.java**: SecurityContextHolder에서 사용자 획득, search(keyword, page, size, user) 호출
- **PermissionResolverService.java**: hasPermissionForMenu(user, menuId, permission) 인터페이스 추가
- **PermissionResolverServiceImpl.java**: hasPermissionForMenu() 구현 (Redis 캐시 활용)
- **SearchControllerTest.java**: 10개 테스트로 업데이트 (새 API 시그니처 반영)

### Change Log

- 2026-02-05: Story 2.2 구현 완료 - Query-time 권한 필터링

### File List

**수정:**
- cms_backend/src/main/java/kr/co/itid/cms/search/service/SearchService.java (+오버로드)
- cms_backend/src/main/java/kr/co/itid/cms/search/service/impl/SearchServiceImpl.java (+사용자 인식 검색)
- cms_backend/src/main/java/kr/co/itid/cms/search/controller/SearchController.java (+사용자 획득)
- cms_backend/src/main/java/kr/co/itid/cms/service/auth/PermissionResolverService.java (+hasPermissionForMenu)
- cms_backend/src/main/java/kr/co/itid/cms/service/auth/impl/PermissionResolverServiceImpl.java (+hasPermissionForMenu 구현)
- cms_backend/src/test/java/kr/co/itid/cms/search/controller/SearchControllerTest.java (+테스트 업데이트)
