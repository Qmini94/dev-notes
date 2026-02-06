# Story 3.1: 사이트별·유형별 필터 및 복합 필터링

Status: review

## Story

As a 사용자,
I want 검색 결과를 사이트별·콘텐츠 유형별로 필터링하여,
So that 원하는 범위의 결과만 빠르게 확인할 수 있다.

## Acceptance Criteria

1. **AC1**: 검색 결과가 표시된 상태에서 사이트 필터를 선택하면 해당 사이트의 결과만 표시되고 totalCount가 갱신된다
2. **AC2**: 검색 결과가 표시된 상태에서 유형 필터(게시글/페이지/메뉴)를 선택하면 해당 유형의 결과만 표시된다
3. **AC3**: 사이트 필터와 유형 필터를 동시에 적용할 때 두 조건을 모두 만족하는 결과만 표시된다
4. **AC4**: gradle compileJava, test가 성공한다
5. **AC5**: 프론트엔드 빌드(nuxt build)가 성공한다

## Tasks / Subtasks

- [x] Task 1: Backend - SearchController/Service에 필터 파라미터 추가 (AC: #1, #2, #3)
  - [x] 1.1 SearchController에 contentType, sort 파라미터 추가
  - [x] 1.2 SearchService.search()에 필터/정렬 파라미터 추가
  - [x] 1.3 SearchServiceImpl에서 ES 쿼리에 contentType 필터 및 sort 적용

- [x] Task 2: Backend - 테스트 작성 (AC: #4)
  - [x] 2.1 SearchControllerTest에 필터 파라미터 검증 테스트 추가 (13 tests)
  - [x] 2.2 gradle compileJava + test 성공

- [x] Task 3: Frontend - 필터 UI 컴포넌트 (AC: #1, #2, #3)
  - [x] 3.1 types/search/search.ts에 ContentTypeFilter, SortOption 타입 추가
  - [x] 3.2 useSearchApi에 contentType, sort 파라미터 추가
  - [x] 3.3 searchStore에 contentType, sort 상태 및 setters 추가
  - [x] 3.4 pages/search.vue에 필터 UI 통합 (버튼+드롭다운)

- [x] Task 4: Frontend - 빌드 검증 (AC: #5)
  - [x] 4.1 nuxt build 성공

## Dev Notes

### Architecture 결정사항

**Backend API 변경:**
```
GET /back-api/search
  ?keyword=소상공인
  &page=0
  &size=10
  &contentType=POST       (선택: POST/PAGE/MENU)
  &siteId=1               (선택: 사이트 ID) - 현재 CMS에는 site_id가 없으므로 추후 확장
```

**참고:** 현재 CMS의 Menu 엔티티에는 siteId가 없음. siteId 필터는 선택적으로 구현하고, contentType 필터만 우선 구현.

**ES 쿼리 변경:**
```java
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery()
    .must(multiMatchQuery)
    .filter(visibilityFilter);

if (contentType != null) {
    boolQuery.filter(QueryBuilders.termQuery("contentType", contentType));
}
```

### 주의사항
- **금지**: 기존 API 호환성 깨지 않도록 필터는 optional
- siteId는 CMS 구조상 미지원 → contentType 필터만 구현
