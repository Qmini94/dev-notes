# Story 4.1: Spring Events + Outbox 패턴 구현

Status: review

## Story

As a 시스템,
I want CRUD 이벤트를 Outbox 테이블에 저장하고 비동기로 ES에 색인하여,
So that 이벤트 유실 없이 안정적으로 색인을 동기화할 수 있다.

## Acceptance Criteria

1. **AC1**: cms_search_outbox 테이블이 생성되고, IndexingEvent 발행 시 동일 트랜잭션 내에서 SearchOutbox 레코드가 INSERT된다
2. **AC2**: Outbox payload에는 최소 식별 정보(contentType, contentIdx, action)만 저장하고, 실제 ES 문서 변환은 처리 시점에 원본 데이터를 조회하여 mapper를 통해 생성한다
3. **AC3**: IndexingEventListener가 @Async로 ES Write Alias에 색인 시도하며, 성공 시 COMPLETED, 실패 시 FAILED + retryCount 증가
4. **AC4**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: Outbox 테이블 및 엔티티 (AC: #1, #2)
  - [x] 1.1 SearchOutbox 엔티티 생성 (entity/SearchOutbox.java)
  - [x] 1.2 SearchOutboxRepository 생성 (repository/SearchOutboxRepository.java)
  - [x] 1.3 JPA 자동 테이블 생성 (@Entity + @Table)

- [x] Task 2: IndexingEvent 도메인 이벤트 (AC: #1)
  - [x] 2.1 IndexingEvent 클래스 생성 (event/IndexingEvent.java)
  - [x] 2.2 IndexAction enum 이미 존재 (enums/IndexAction.java)

- [x] Task 3: IndexingEventListener (AC: #3)
  - [x] 3.1 @TransactionalEventListener(AFTER_COMMIT)로 Outbox INSERT
  - [x] 3.2 @Async("searchIndexExecutor")로 ES 색인 처리
  - [x] 3.3 markCompleted()/markFailed()로 상태 업데이트

- [x] Task 4: IndexingService 확장
  - [x] 4.1 indexContent(ContentType, Long) 메서드 추가
  - [x] 4.2 deleteContent(ContentType, Long) 메서드 추가
  - [x] 4.3 BatchIndexDao.findPostById() 추가

- [x] Task 5: AsyncConfig (AC: #3)
  - [x] 5.1 searchIndexExecutor ThreadPoolTaskExecutor 설정

- [x] Task 6: 테스트 작성 (AC: #4)
  - [x] 6.1 gradle compileJava + test 성공

## Dev Notes

### Outbox 테이블 스키마

```sql
CREATE TABLE cms_search_outbox (
    outbox_idx BIGINT AUTO_INCREMENT PRIMARY KEY,
    content_type VARCHAR(20) NOT NULL,      -- POST, PAGE, MENU
    content_idx BIGINT NOT NULL,            -- 원본 콘텐츠 ID
    action VARCHAR(10) NOT NULL,            -- CREATE, UPDATE, DELETE
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',  -- PENDING, COMPLETED, FAILED
    retry_count INT NOT NULL DEFAULT 0,
    error_message VARCHAR(500),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    processed_at DATETIME,
    INDEX idx_status_created (status, created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 이벤트 흐름

1. CMS Service에서 CRUD 발생
2. ApplicationEventPublisher.publishEvent(IndexingEvent)
3. @TransactionalEventListener: 동일 트랜잭션에서 Outbox INSERT
4. @Async: 비동기로 ES 색인 시도
5. 성공/실패에 따라 Outbox 상태 업데이트

### 참고
- Outbox는 이벤트 유실 방지를 위한 안전장치
- 실제 ES 색인 실패 시 4.3의 폴링 서비스가 재시도
