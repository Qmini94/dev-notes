# Story 3.2: 정렬 전환 및 관리자 편집 링크

Status: review

## Story

As a 사용자,
I want 검색 결과를 정확도순 또는 최신순으로 정렬하고, 관리자는 편집 화면으로 바로 이동하여,
So that 상황에 맞는 방식으로 결과를 탐색할 수 있다.

## Acceptance Criteria

1. **AC1**: 검색 결과가 표시된 상태에서 정렬을 "최신순"으로 변경하면 결과가 updatedDate 기준 내림차순으로 재정렬된다 (Story 3.1에서 구현됨)
2. **AC2**: 정렬을 "정확도순"(기본)으로 복원할 때 BM25 관련도 기준으로 다시 정렬된다 (Story 3.1에서 구현됨)
3. **AC3**: 관리자 권한을 가진 사용자가 검색 결과를 볼 때 해당 콘텐츠의 관리 화면(편집)으로 이동하는 링크가 표시된다
4. **AC4**: gradle compileJava, test가 성공한다
5. **AC5**: 프론트엔드 빌드(nuxt build)가 성공한다

## Tasks / Subtasks

- [x] Task 1: AC1, AC2 - 정렬 기능 (Story 3.1에서 완료)

- [x] Task 2: Backend - 관리자 편집 URL 생성 (AC: #3)
  - [x] 2.1 SearchItemResponse에 adminEditUrl 필드 추가
  - [x] 2.2 SearchServiceImpl에 buildAdminEditUrl() 메서드 구현
  - [x] 2.3 toSearchItemResponse()에서 isAdmin 체크 후 URL 포함

- [x] Task 3: Backend - 테스트 작성 (AC: #4)
  - [x] 3.1 기존 SearchControllerTest 통과 (13 tests)
  - [x] 3.2 gradle compileJava + test 성공

- [x] Task 4: Frontend - 편집 링크 UI (AC: #3)
  - [x] 4.1 SearchItem 타입에 adminEditUrl 추가
  - [x] 4.2 SearchResultItem에서 관리자 편집 링크 표시 (조건부)

- [x] Task 5: Frontend - 빌드 검증 (AC: #5)
  - [x] 5.1 nuxt build 성공

## Dev Notes

### Architecture 결정사항

**관리자 편집 URL 생성 로직:**
- POST: `/business/board/{boardIdx}/post/{postIdx}/edit`
- PAGE: `/business/content/{contentIdx}/edit`
- MENU: `/business/menu/{menuIdx}/edit`

**권한 체크:**
- JwtAuthenticatedUser.isAdmin()이 true인 경우에만 adminEditUrl 포함
- 비로그인/일반 사용자는 adminEditUrl = null

**보안:**
- 프론트엔드에서 링크를 숨겨도 백엔드에서 권한 체크 필수
- 관리자 권한이 없으면 adminEditUrl 자체를 응답에서 제외
