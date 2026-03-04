# NHN Commerce 기술면접 예상 질문 & 답변

---

## 1. 어노테이션

### @RestControllerAdvice 왜 쓰나요?
전역 예외 처리를 한 곳에서 관리하기 위해서입니다.
각 Controller마다 try-catch를 쓰면 중복 코드가 생기는데, 한 클래스에 모을 수 있습니다.
- `@ControllerAdvice`와 차이: `@RestControllerAdvice`는 `@ResponseBody` 포함 → JSON 응답

### @Transactional(readOnly = true) 왜 쓰나요?
조회 전용이라는 걸 JPA에 알려줘서 dirty checking, flush를 생략합니다. 성능이 좋아집니다.
- dirty checking: 트랜잭션 내에서 엔티티 변경 감지 → 커밋 시 자동 UPDATE

### @EntityListeners(AuditingEntityListener.class)
@CreatedDate 같은 감사 필드를 자동으로 채워주기 위해서입니다.
메인 클래스에 @EnableJpaAuditing도 필요합니다.

### @Enumerated(EnumType.STRING) 왜 쓰나요?
기본값 ORDINAL은 0, 1, 2로 저장 → enum 순서 바뀌면 데이터 꼬임.
STRING으로 해야 "SALE", "SOLD_OUT" 문자열로 안전하게 저장됩니다.

### @RequiredArgsConstructor 왜 쓰나요?
final 필드의 생성자를 자동 생성 → Spring이 생성자 주입으로 DI 해줍니다.
- 필드 주입(@Autowired) 안 쓰는 이유: 테스트 시 Mock 주입이 어렵고, 불변성 보장 불가

### @Valid vs @Validated
- @Valid: @RequestBody DTO 검증 → 혼자서 동작
- @Validated: @PathVariable, @RequestParam에 직접 붙인 검증 → 클래스에 선언 필요

---

## 2. 설계 패턴

### Read/Write Service 분리 (CQRS)
읽기와 쓰기의 특성이 다르기 때문.
- 읽기: readOnly=true로 성능 최적화
- 쓰기: 트랜잭션 관리가 중요
- 대규모 트래픽 시 읽기/쓰기 DB 분리에 유리

### DTO ↔ Entity 분리하는 이유
엔티티 직접 노출 → DB 구조 노출 + 엔티티 변경이 API 스펙에 영향.
DTO로 분리하면 API 스펙과 DB 구조를 독립적으로 변경 가능합니다.

### Entity에 비즈니스 메서드 (update, deleted)
setter 대신 update 메서드로 변경 포인트를 제한합니다.
→ 도메인 모델 패턴 (Rich Entity)

### Soft Delete
실제 삭제 대신 status를 HIDDEN으로 변경.
- 데이터 복구 가능
- 다른 테이블에서 참조 중일 때 안전

### 인터페이스를 쓰는 이유
SOLID 원칙 중 OCP(개방-폐쇄)와 DIP(의존성 역전)를 지키기 위해서입니다.
런타임에 구현체를 교체할 수 있고, 새 요구사항에도 기존 코드 수정 없이 구현체 추가 가능합니다.

---

## 3. JPA 핵심 질문

### N+1 문제가 뭐예요?
주문 10건 조회 → 쿼리 1번, 각 주문의 상품 조회 → 쿼리 10번 = 총 11번.
LAZY 로딩으로 연관 엔티티 접근할 때마다 쿼리가 추가 발생하는 문제입니다.
- 해결: JOIN FETCH → 한 방 쿼리

### JOIN vs JOIN FETCH 차이
- JOIN: 조건은 같이 걸지만 데이터는 나중에 가져옴 (LAZY 유지)
- JOIN FETCH: 조건 + 데이터를 한번에 가져옴 (추가 쿼리 없음)

### LAZY vs EAGER 차이
- LAZY: 접근 시점에 로딩 (기본 권장)
- EAGER: 즉시 로딩 (불필요한 데이터도 항상 가져옴)

### save() 안 해도 수정되는 이유 (Dirty Checking)
@Transactional 안에서 엔티티 필드를 바꾸면 JPA가 변경을 감지해서
커밋 시점에 자동으로 UPDATE 쿼리를 날립니다.

---

## 4. 예외 처리

### GlobalExceptionHandler 동작 원리
예외는 발생 지점에서 catch할 때까지 호출 스택을 타고 위로 전파됩니다.
Service → Controller → @RestControllerAdvice 순으로 올라갑니다.
try-catch가 필요한 경우: 예외 발생 시 로그, 세션 삭제 등 추가 작업이 필요할 때.
