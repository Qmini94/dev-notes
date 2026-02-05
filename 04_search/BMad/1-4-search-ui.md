# Story 1.4: 검색 UI 구현 (SearchBar + 결과 페이지)

Status: review

## Story

As a 사용자,
I want CMS 모든 페이지에서 검색창을 이용하고 결과를 확인하여,
So that 어디에서든 원하는 정보를 검색할 수 있다.

## Acceptance Criteria

1. **AC1**: CMS 어떤 페이지에서든 상단에 검색 입력창(SearchBar)이 항상 표시된다
2. **AC2**: 키워드 입력 후 엔터 또는 검색 버튼 클릭 시 /search 페이지로 이동하여 검색 결과가 표시된다
3. **AC3**: 각 결과에 제목, 요약 스니펫(키워드 하이라이팅), 콘텐츠 유형, 소속 사이트가 표시된다
4. **AC4**: 검색 결과 클릭 시 원본 콘텐츠 페이지로 직접 이동한다
5. **AC5**: 결과가 10건 초과 시 페이지네이션이 동작한다
6. **AC6**: 모바일/태블릿 환경에서 반응형으로 동일 기능이 제공된다
7. **AC7**: 프론트엔드 빌드(nuxt build 또는 nuxt generate)가 성공한다

## Tasks / Subtasks

- [x] Task 1: 검색 TypeScript 타입 정의 (AC: #3)
  - [x] 1.1 types/search/search.ts 생성

- [x] Task 2: 검색 API Composable 생성 (AC: #2, #3, #5)
  - [x] 2.1 composables/api/search/useSearchApi.ts 생성

- [x] Task 3: 검색 상태 Store 생성 (AC: #2, #5)
  - [x] 3.1 stores/searchStore.ts 생성

- [x] Task 4: 검색 컴포넌트 생성 (AC: #1, #3, #4, #6)
  - [x] 4.1 components/search/SearchBar.vue (검색 입력 + 이동)
  - [x] 4.2 components/search/SearchResultItem.vue (개별 결과)
  - [x] 4.3 components/search/SearchServiceDown.vue (서비스 불가 안내)

- [x] Task 5: 검색 결과 페이지 생성 (AC: #2, #3, #4, #5, #6)
  - [x] 5.1 pages/search.vue (검색 결과 페이지)

- [x] Task 6: 헤더 컴포넌트에 SearchBar 통합 (AC: #1)
  - [x] 6.1 components/default/main/Header.vue 수정
  - [x] 6.2 components/www/main/Header.vue 수정
  - [x] 6.3 components/business/Header.vue 수정

- [x] Task 7: 빌드 검증 (AC: #7)
  - [x] 7.1 nuxt build 성공 확인

## Dev Notes

### 기존 프로젝트 패턴 (반드시 준수)

**API 호출 패턴:**
- `safeFetch()` 유틸리티 사용 (utils/safeFetch.ts)
- 응답 형식: `{ code, message, data }`
- 503 응답 시 isServiceDown 처리

**Store 패턴:**
- `defineStore("name", () => { ... })` Composition API 스타일
- `ref()` for state, `computed()` for derived

**컴포넌트 패턴:**
- `<script setup lang="ts">` 필수
- Scoped CSS for public-facing components (전통 CSS 구조 호환)
- Tailwind CSS for business/admin components

**레이아웃:**
- 공개 사이트: default/www 헤더 (전통 CSS)
- 관리자: business 헤더 (Tailwind CSS)
- 검색 결과 페이지: layout: false (독립 페이지, noindex 적용)

### 주의사항

- **금지**: 필터링/정렬 UI (Story 3.1, 3.2 범위)
- **금지**: 접근성 ARIA 속성 상세 구현 (Story 3.3 범위)
- **금지**: 관리자 편집 링크 (Story 3.2 범위)
- Story 1.4에서는 기본 검색 UI만 구현 (검색, 결과 표시, 페이지네이션, 원본 이동)

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- nuxt build: BUILD SUCCESSFUL (.output/server/index.mjs 생성 확인, Total size: 6.18 MB)
- nuxt typecheck: 기존 프로젝트 전체에 auto-import 관련 TS 에러 존재 (신규 코드도 동일 패턴, 런타임 정상)

### Completion Notes List

- **types/search/search.ts**: SearchItem, SearchResult 인터페이스 정의
- **useSearchApi.ts**: safeFetch 기반 검색 API 호출, 503 기본값 처리, timeout 5초
- **searchStore.ts**: Pinia Composition API, keyword/results/totalCount/page/isLoading/isServiceDown 상태, search/clearResults 액션, totalPages/hasResults computed
- **SearchBar.vue**: 검색 입력 + 검색 버튼, form submit → /search?keyword= 라우팅, 반응형 (768px 브레이크포인트), scoped CSS
- **SearchResultItem.vue**: 콘텐츠 유형 라벨(게시글/페이지/메뉴), v-html로 하이라이팅 렌더링, sourceUrl 링크, 날짜 포맷
- **SearchServiceDown.vue**: 경고 아이콘 + 안내 메시지
- **pages/search.vue**: 독립 레이아웃(layout: false), noindex 메타, 검색 입력 + 결과 목록 + 페이지네이션(최대 5페이지 표시), route.query 기반 watch로 검색 실행, 반응형
- **헤더 통합**: default/main/Header.vue (right_utill 영역), www/main/Header.vue (GNB 영역), business/Header.vue (우측 도구 영역) — 3개 헤더 모두에 SearchSearchBar 통합

### Change Log

- 2026-02-05: Story 1.4 구현 완료 - 검색 UI (SearchBar + 결과 페이지)

### File List

**신규 생성:**
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/types/search/search.ts
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/composables/api/search/useSearchApi.ts
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/stores/searchStore.ts
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/search/SearchBar.vue
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/search/SearchResultItem.vue
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/search/SearchServiceDown.vue
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/pages/search.vue

**수정:**
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/default/main/Header.vue (+SearchBar)
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/www/main/Header.vue (+SearchBar)
- egovframe-cms-frontend-q/egovframe-cms-frontend-q/components/business/Header.vue (+SearchBar)
