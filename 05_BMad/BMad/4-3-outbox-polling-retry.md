# Story 4.3: Outbox 폴링 재시도 서비스

Status: review

## Story

As a 시스템 운영자,
I want 실패한 색인 작업이 자동으로 재시도되어,
So that 일시적 ES 장애 후 색인 정합성이 자동 복구된다.

## Acceptance Criteria

1. **AC1**: Outbox에 FAILED 또는 PENDING 상태의 레코드가 있을 때 OutboxPollingService 스케줄러가 실행되면 미처리 레코드를 순차적으로 ES에 색인 재시도한다
2. **AC2**: 성공 시 status=COMPLETED, 실패 시 retryCount+1
3. **AC3**: retryCount가 최대 횟수(3회)를 초과하면 FAILED로 유지하고 로그에 기록한다
4. **AC4**: gradle compileJava, test가 성공한다

## Tasks / Subtasks

- [x] Task 1: OutboxPollingService 구현
  - [x] 1.1 @Scheduled로 주기적 폴링 (30초)
  - [x] 1.2 PENDING 레코드 처리
  - [x] 1.3 FAILED 레코드 중 retryCount < maxRetry 처리
  - [x] 1.4 처리 결과에 따른 상태 업데이트

- [x] Task 2: 스케줄링 설정
  - [x] 2.1 @EnableScheduling 이미 CmsApplication에 존재

- [x] Task 3: 빌드 검증 (AC: #4)
  - [x] 3.1 gradle compileJava + test 성공

## Dev Notes

### 폴링 로직

```java
@Scheduled(fixedDelay = 30000)  // 30초마다
public void pollAndProcessOutbox() {
    // 1. PENDING 레코드 처리
    List<SearchOutbox> pending = repository.findByStatus(PENDING);

    // 2. FAILED but retryable 레코드 처리
    List<SearchOutbox> retryable = repository.findRetryableRecords(FAILED, MAX_RETRY);

    // 3. 각 레코드 처리
    for (SearchOutbox outbox : records) {
        processOutbox(outbox);
    }
}
```

### 재시도 정책

- 최대 재시도 횟수: 3회
- 재시도 간격: 폴링 주기에 따라 자연스럽게 백오프 (30초)
- 최대 재시도 초과 시: FAILED 상태 유지, 경고 로그 출력
- 배치 크기 제한: 100개 (과부하 방지)

### 기존 인프라 활용

- `IndexingService.indexContent(type, idx)` - CREATE/UPDATE 처리 (upsert)
- `IndexingService.deleteContent(type, idx)` - DELETE 처리
- `SearchOutboxRepository.findRetryableRecords()` - 이미 구현됨
- `@EnableScheduling` - CmsApplication에 이미 활성화됨

### 구현 상세

`OutboxPollingService`:
- 30초 간격으로 폴링 (`@Scheduled(fixedDelay = 30000)`)
- PENDING + FAILED(retryable) 레코드 조회 후 병합
- 배치 크기 제한 (100개)
- 성공 시 `markCompleted()`, 실패 시 `markFailed(errorMessage)`
- MAX_RETRY(3) 초과 시 경고 로그 출력

### 생성된 파일

1. `search/service/OutboxPollingService.java` - 폴링 재시도 서비스
