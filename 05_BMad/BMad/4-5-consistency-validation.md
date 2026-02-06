# Story 4.5: 정합성 검증 및 삭제 동기화

Status: review

## Story

As a 시스템,
I want 주기적으로 원본과 인덱스 간 불일치를 감지하고, 삭제된 콘텐츠를 인덱스에서 제거하여,
So that 검색 결과의 정확성을 보장한다.

## Acceptance Criteria

1. **AC1**: 주기적 배치 검증이 실행될 때 MySQL 원본과 ES 인덱스를 비교하면 누락된 문서를 색인하고 삭제된 문서를 인덱스에서 제거한다
2. **AC2**: 불일치 건수를 search-index.log에 기록한다
3. **AC3**: CMS에서 콘텐츠가 삭제되었을 때 삭제 이벤트가 처리되면 ES 인덱스에서 해당 문서가 제거되고 검색 결과에 더 이상 표시되지 않는다 (Story 4.2에서 구현됨)
4. **AC4**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: ConsistencyValidationService 구현
  - [x] 1.1 MySQL에서 유효한 콘텐츠 ID 목록 조회
  - [x] 1.2 ES에서 색인된 문서 ID 목록 조회 (Scroll API)
  - [x] 1.3 누락된 문서 색인 (MySQL에 있고 ES에 없는)
  - [x] 1.4 고아 문서 삭제 (ES에 있고 MySQL에 없는)
  - [x] 1.5 정합성 검증 결과 로깅

- [x] Task 2: 스케줄링 설정
  - [x] 2.1 @Scheduled로 주기적 실행 (매일 새벽 3시)

- [x] Task 3: Admin API
  - [x] 3.1 POST /back-api/search/admin/validate - 검증 및 동기화 실행
  - [x] 3.2 GET /back-api/search/admin/validate - 검증만 실행 (조회)

- [x] Task 4: 빌드 검증 (AC: #4)
  - [x] 4.1 gradle compileJava + test 성공

## Dev Notes

### 정합성 검증 로직

```java
@Scheduled(cron = "0 0 3 * * *")  // 매일 새벽 3시
public void scheduledValidation() {
    // 1. MySQL에서 유효한 콘텐츠 ID 목록 조회
    Set<String> mysqlPostIds = getValidPostIds();   // post_{idx}
    Set<String> mysqlPageIds = getValidPageIds();   // page_{idx}
    Set<String> mysqlMenuIds = getValidMenuIds();   // menu_{idx}

    // 2. ES에서 색인된 문서 ID 목록 조회 (Scroll API)
    Set<String> esIds = getIndexedDocumentIds();

    // 3. 누락된 문서 색인 (MySQL - ES)
    indexMissingDocuments(mysqlIds - esIds);

    // 4. 고아 문서 삭제 (ES - MySQL)
    deleteOrphanedDocuments(esIds - mysqlIds);
}
```

### 문서 ID 형식

- POST: `post_{idx}`
- PAGE: `page_{idx}`
- MENU: `menu_{idx}`

### ES Scroll API 사용

대량의 문서 ID를 효율적으로 조회하기 위해 Scroll API 사용:
- Scroll Size: 1000
- Scroll Timeout: 5분
- `fetchSource(false)`: ID만 필요하므로 _source 제외

### API 엔드포인트

| Method | Path | 설명 |
|--------|------|------|
| POST | /back-api/search/admin/validate | 검증 + 동기화 실행 |
| GET | /back-api/search/admin/validate | 검증만 실행 (조회) |

### 생성된 파일

1. `search/dto/ConsistencyResult.java` - 검증 결과 DTO
2. `search/service/ConsistencyValidationService.java` - 서비스 인터페이스
3. `search/service/impl/ConsistencyValidationServiceImpl.java` - 서비스 구현
4. `search/controller/SearchAdminController.java` - API 엔드포인트 추가

### 사용 예시

```bash
# 검증 + 동기화 실행
curl -X POST http://localhost:8080/back-api/search/admin/validate

# 응답
{
  "code": "0",
  "data": {
    "missingPostCount": 5,
    "missingPageCount": 2,
    "missingMenuCount": 0,
    "orphanedCount": 3,
    "totalMissing": 7,
    "totalFixed": 10,
    "message": "Indexed 7 missing, deleted 3 orphaned documents"
  }
}

# 검증만 (조회)
curl http://localhost:8080/back-api/search/admin/validate
```

### 스케줄 실행

- 매일 새벽 3시에 자동 실행: `@Scheduled(cron = "0 0 3 * * *")`
- 결과는 search-index.log에 기록됨
