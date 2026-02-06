# Story 4.4A: Read/Write Alias 기반 무중단 재색인 (MVP)

Status: review

## Story

As a 시스템 운영자,
I want 전체 재색인을 서비스 중단 없이 수행하여,
So that 인덱스 구조 변경이나 정합성 복구를 안전하게 진행할 수 있다.

## Acceptance Criteria

1. **AC1**: POST /back-api/search/admin/reindex를 호출하면 신규 인덱스(cms_content_v{n+1})가 생성되고 Write Alias가 전환된다
2. **AC2**: 전체 데이터가 신규 인덱스에 색인된다
3. **AC3**: 색인 중 기존 Read Alias는 기존 인덱스를 계속 서비스한다
4. **AC4**: 완료 후 Read Alias가 신규 인덱스로 원자적 전환된다
5. **AC5**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: ReindexService 구현
  - [x] 1.1 현재 인덱스 버전 조회 (Read Alias → 인덱스명에서 추출)
  - [x] 1.2 신규 인덱스 생성 (v{n+1})
  - [x] 1.3 Write Alias 전환 (old → new)
  - [x] 1.4 전체 데이터 색인 (IndexingService.performFullIndexing())
  - [x] 1.5 Read Alias 원자적 전환

- [x] Task 2: Admin API 엔드포인트
  - [x] 2.1 POST /back-api/search/admin/reindex
  - [x] 2.2 GET /back-api/search/admin/index-version
  - [x] 2.3 GET /back-api/search/admin/reindex-status
  - [x] 2.4 ReindexResponse DTO

- [x] Task 3: 빌드 검증 (AC: #5)
  - [x] 3.1 gradle compileJava + test 성공

## Dev Notes

### 재색인 흐름

```
1. GET current version from Read Alias (cms_content_v1 → version=1)
2. CREATE cms_content_v{n+1} with ContentDocument mappings
3. SWITCH Write Alias: v{n} → v{n+1} (원자적)
4. BULK INDEX all data to Write Alias (신규 인덱스에 색인)
5. ATOMIC SWITCH Read Alias: v{n} → v{n+1}
6. (Optional) DELETE old index v{n} - 수동 처리
```

### Alias 전환 (원자적)

```java
IndicesAliasesRequest request = new IndicesAliasesRequest();
request.addAliasAction(new AliasActions(REMOVE).index(oldIndex).alias(aliasName));
request.addAliasAction(new AliasActions(ADD).index(newIndex).alias(aliasName));
client.indices().updateAliases(request, RequestOptions.DEFAULT);
```

### 동시 실행 방지

- `AtomicBoolean reindexing` 플래그로 동시 재색인 방지
- 이미 진행 중이면 REJECTED 응답 반환

### API 엔드포인트

| Method | Path | 설명 |
|--------|------|------|
| POST | /back-api/search/admin/reindex | 전체 재색인 실행 |
| GET | /back-api/search/admin/index-version | 현재 인덱스 버전 조회 |
| GET | /back-api/search/admin/reindex-status | 재색인 진행 상태 확인 |

### 생성된 파일

1. `search/dto/ReindexResponse.java` - 재색인 결과 DTO
2. `search/service/ReindexService.java` - 재색인 서비스 인터페이스
3. `search/service/impl/ReindexServiceImpl.java` - 재색인 서비스 구현
4. `search/controller/SearchAdminController.java` - 관리자 API 컨트롤러

### 사용 예시

```bash
# 재색인 실행
curl -X POST http://localhost:8080/back-api/search/admin/reindex

# 응답
{
  "code": "0",
  "data": {
    "status": "SUCCESS",
    "oldIndex": "cms_content_v1",
    "newIndex": "cms_content_v2",
    "postCount": 150,
    "pageCount": 30,
    "menuCount": 45,
    "totalIndexed": 225,
    "message": "재색인 완료"
  }
}
```
