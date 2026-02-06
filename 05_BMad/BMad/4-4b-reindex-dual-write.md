# Story 4.4B: 재색인 중 증분 이벤트 Dual-Write (운영 안정화)

Status: review

## Story

As a 시스템 운영자,
I want 재색인 진행 중에도 증분 이벤트가 기존·신규 인덱스 양쪽에 반영되어,
So that 재색인 완료 후 최신 데이터 누락이 없다.

## Acceptance Criteria

1. **AC1**: 재색인이 진행 중일 때 CMS에서 콘텐츠 CRUD가 발생하면 증분 이벤트가 기존 인덱스(Read Alias)와 신규 인덱스(Write Alias) 양쪽에 기록된다
2. **AC2**: 재색인 완료 후 Alias 전환 시 데이터 누락이 없다
3. **AC3**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: IndexingServiceImpl 수정
  - [x] 1.1 ReindexService 의존성 주입 (@Lazy로 순환 의존성 방지)
  - [x] 1.2 indexContent()에 dual-write 로직 추가
  - [x] 1.3 deleteContent()에 dual-write 로직 추가

- [x] Task 2: 빌드 검증 (AC: #3)
  - [x] 2.1 gradle compileJava + test 성공

## Dev Notes

### Dual-Write 로직

```java
// Write Alias (신규 인덱스 또는 현재 인덱스)
IndexCoordinates writeCoordinates = IndexCoordinates.of(
        searchProperties.getAlias().getWrite());
elasticsearchRestTemplate.index(indexQuery, writeCoordinates);

// Dual-write: 재색인 중이면 Read Alias (기존 인덱스)에도 쓰기
if (reindexService != null && reindexService.isReindexing()) {
    IndexCoordinates readCoordinates = IndexCoordinates.of(
            searchProperties.getAlias().getRead());
    elasticsearchRestTemplate.index(indexQuery, readCoordinates);
    log.info("[INDEXING] Dual-write index success: documentId={}", document.getDocumentId());
}
```

### 상태 흐름

```
[일반 운영]
Write Alias → v1
Read Alias → v1
CRUD → Write Alias (v1)

[재색인 중]
Write Alias → v2 (신규)
Read Alias → v1 (기존)
CRUD → Write Alias (v2) + Read Alias (v1)  ← Dual-Write

[재색인 완료]
Write Alias → v2
Read Alias → v2
CRUD → Write Alias (v2)
```

### 순환 의존성 방지

```java
// @Lazy to avoid circular dependency: ReindexService → IndexingService → ReindexService
@Autowired
@Lazy
private ReindexService reindexService;
```

- ReindexService는 IndexingService.performFullIndexing() 호출
- IndexingService는 ReindexService.isReindexing() 확인
- @Lazy로 초기화 지연하여 순환 의존성 방지

### 변경된 파일

1. `IndexingServiceImpl.java`:
   - ReindexService 의존성 추가 (@Autowired @Lazy)
   - `indexContent()`: Write Alias + Read Alias (if reindexing)
   - `deleteContent()`: Write Alias + Read Alias (if reindexing)

### 데이터 정합성 보장

재색인 중 CRUD 발생 시:
1. 신규 인덱스(Write Alias)에 먼저 기록 - 재색인 완료 후 서비스
2. 기존 인덱스(Read Alias)에도 기록 - 현재 사용자 검색에 즉시 반영
3. Alias 전환 후 데이터 누락 없음
