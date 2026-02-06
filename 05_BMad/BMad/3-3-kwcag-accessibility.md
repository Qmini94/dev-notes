# Story 3.3: KWCAG 2.2 접근성 적용

Status: review

## Story

As a 키보드/스크린리더 사용자,
I want 검색의 모든 기능을 키보드와 보조 기술로 사용하여,
So that 장애 유무와 관계없이 동등한 검색 경험을 얻는다.

## Acceptance Criteria

1. **AC1**: 키보드만 사용할 때 Tab/Enter/Arrow 키로 검색 입력, 결과 탐색, 필터 조작, 정렬 변경을 수행할 수 있다
2. **AC2**: 스크린리더 사용 시 검색 결과 수, 필터 상태, 정렬 상태, 현재 페이지 정보가 음성으로 전달된다 (aria-live, aria-label 적용)
3. **AC3**: 검색 실행 후 포커스가 결과 영역으로 이동한다
4. **AC4**: 검색 결과 페이지에 noindex 메타 태그가 적용된다 (Story 1.4에서 이미 적용됨)
5. **AC5**: 프론트엔드 빌드(nuxt build)가 성공한다

## Tasks / Subtasks

- [x] Task 1: 키보드 접근성 (AC: #1)
  - [x] 1.1 검색 입력창, 버튼 포커스 스타일 (outline: 2px solid #409eff)
  - [x] 1.2 필터 버튼 키보드 탐색 (Tab) + focus 스타일
  - [x] 1.3 정렬 드롭다운 키보드 조작 (기본 select)
  - [x] 1.4 페이지네이션 키보드 탐색 + focus 스타일

- [x] Task 2: 스크린리더 접근성 (AC: #2)
  - [x] 2.1 검색 결과 영역에 aria-live="polite", role="region" 적용
  - [x] 2.2 총 결과 수 + 필터/정렬 상태 aria-label (resultsStatusLabel)
  - [x] 2.3 필터 버튼 role="radiogroup", aria-checked 적용
  - [x] 2.4 페이지네이션 nav role="navigation", aria-current="page" 적용
  - [x] 2.5 SearchResultItem에 aria-label 추가 (링크, 콘텐츠 유형)

- [x] Task 3: 포커스 관리 (AC: #3)
  - [x] 3.1 검색 실행 후 결과 영역으로 포커스 이동 (focusResultsRegion)
  - [x] 3.2 resultsRegionRef + tabindex="-1" 포커스 가능

- [x] Task 4: noindex 확인 (AC: #4)
  - [x] 4.1 useHead에 noindex, nofollow 메타 태그 확인됨

- [x] Task 5: 빌드 검증 (AC: #5)
  - [x] 5.1 nuxt build 성공

## Dev Notes

### KWCAG 2.2 / WCAG 2.1 준수 항목

**키보드 접근성:**
- 모든 상호작용 요소에 키보드로 접근 가능
- 포커스 표시가 시각적으로 명확 (outline)
- Tab 순서가 논리적

**스크린리더 호환:**
- aria-live="polite" - 동적 콘텐츠 변경 알림
- aria-label - 요소의 접근 가능한 이름
- role="region" - 랜드마크 역할
- aria-current="page" - 현재 페이지 표시

**포커스 관리:**
- 검색 실행 후 결과 영역으로 포커스 이동 (ref + focus())
