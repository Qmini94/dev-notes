**1. ResponseEntity는 뭐고 왜 사용하는지**

HTTP 응답을 세밀하게 제어하는 객체예요. 상태 코드 + 헤더 + 바디를 담아요.

java

```java
// ResponseEntity 없이 → 항상 200만 나감
@GetMapping
public GameResponse getGame() { ... }

// ResponseEntity 사용 → 상태 코드 구분 가능
@PostMapping
public ResponseEntity<ApiResponse<Void>> create() {
    return ResponseEntity.status(HttpStatus.CREATED).body(...); // 201
}
@DeleteMapping
public ResponseEntity<ApiResponse<Void>> delete() {
    return ResponseEntity.status(HttpStatus.NO_CONTENT).body(...); // 204
}
```

면접 답변: **"RESTful API에서 각 요청에 맞는 HTTP 상태 코드를 명시적으로 반환하기 위해 사용합니다. 클라이언트가 응답의 의미를 상태 코드만으로 구분할 수 있어 협업에 유리합니다."**

---

**2. 생성자 주입에서 final 왜 사용하는지**

java

```java
// final 있음 → 재할당 불가, 안전
private final GameRepository gameRepository; // ✅

// final 없음 → 누군가 바꿀 수 있음
private GameRepository gameRepository; // ❌ 위험
```

final이 하는 일 세 가지:

- **불변 보장** → 한번 주입되면 변경 불가
- **컴파일 체크** → 생성자에서 초기화 안 하면 컴파일 에러
- **Thread-safe** → 싱글톤 Bean에서 안전

면접 답변: **"불변성을 보장하여 싱글톤 환경에서 thread-safety를 확보하고, 생성자에서 반드시 초기화하도록 컴파일 시점에 강제합니다."**

---

**3. 왜 싱글톤에서 상태값이 있으면 안 되는지**

java

````java
@Service
public class GameService {
    private int count = 0; // ❌ 상태값 — 모든 스레드가 공유

    public void createGame() {
        count++; // 스레드 A가 1로 만들고, 동시에 B도 1로 만듦 → 버그
    }
}
```
```
유저A 요청 ──→ ┐
                ├── 같은 GameService 인스턴스 ── count 공유!
유저B 요청 ──→ ┘
````

final 필드(의존성 주입)는 **불변**이라 안전하고, 지역 변수는 **스레드 스택에 독립적**이라 안전해요. 문제는 **변경 가능한 인스턴스 필드**예요.

면접 답변: **"싱글톤은 모든 스레드가 하나의 인스턴스를 공유하므로, 변경 가능한 상태값이 있으면 동시성 문제가 발생합니다. 상태가 필요하면 Redis나 DB에 저장합니다."**

---

**4. 생성자, 오버라이딩, 오버로드**

java

````java
// 생성자 — 객체 생성 시 초기화
public Game(String title, Integer price) {
    this.title = title;
    this.price = price;
}

// 오버로딩 — 같은 이름, 다른 파라미터
public Game(String title) { ... }
public Game(String title, Integer price) { ... }
public Game(String title, Integer price, Platform platform) { ... }

// 오버라이딩 — 부모 메서드를 재정의
public class ReturnClaim extends AbstractClaim {
    @Override
    void adjust() {
        // 반품에 맞게 재정의
    }
}
```

면접 답변: **"오버로딩은 같은 이름의 메서드를 파라미터를 다르게 정의하는 것이고, 오버라이딩은 상속받은 메서드를 하위 클래스에서 재정의하는 것입니다."**

---

**5. 더티체킹, 스냅샷, save 메서드**
```
트랜잭션 시작
  → findById(1) → 엔티티 조회
  → JPA가 원본 복사본(스냅샷) 저장
  → entity.update("새이름")
  → 트랜잭션 끝 → 스냅샷이랑 비교
  → 바뀐 거 있으면 → UPDATE 자동 실행
````

save()를 쓰는 이유:

java

```java
// save() 없어도 더티체킹이 UPDATE 해줌
// 그럼에도 쓰는 이유 → 명시성과 가독성
game.update(...);
gameRepository.save(game); // "여기서 저장한다"는 의도 표현
```

면접 답변: **"더티체킹은 트랜잭션 내에서 엔티티의 변경을 스냅샷과 비교하여 자동으로 UPDATE하는 JPA 기능입니다. save()는 더티체킹으로 불필요하지만, 코드의 명시성을 위해 팀 컨벤션에 따라 사용합니다."**

---

**6. 테스트케이스는 왜 필드 주입하는가**

java

```java
// 실제 코드 — 생성자 주입
@RequiredArgsConstructor
public class GameService {
    private final GameRepository gameRepository;
}

// 테스트 — 필드 주입
@SpringBootTest
class GameServiceTest {
    @Autowired
    private GameService gameService;
}
```

면접 답변: **"테스트 클래스는 Spring이 관리하는 Bean이 아니라 생성자 주입이 불필요하고, 테스트는 간결함이 우선이라 필드 주입을 사용합니다."**

---

**7. @Valid와 @Validated 차이**

java

```java
// @Valid — 자바 표준 (jakarta.validation)
// RequestBody에서 DTO 검증
@PostMapping
public void create(@Valid @RequestBody GameWriteRequest req) { }

// @Validated — Spring 제공
// PathVariable, RequestParam 같은 단일 파라미터 검증
@Validated  // 클래스 레벨에 붙여야 동작
@RestController
public class GameController {
    @GetMapping("/{id}")
    public void get(@Positive @PathVariable Long id) { }
}
```

면접 답변: **"@Valid는 자바 표준으로 RequestBody DTO 검증에 사용하고, @Validated는 Spring 제공으로 PathVariable 같은 단일 파라미터 검증 시 클래스 레벨에 선언하여 사용합니다."**

---

**8. Checked vs Unchecked Exception**

java

````java
// Checked — 컴파일러가 처리 강제 (try-catch 필수)
public void readFile() throws IOException {  // 안 잡으면 컴파일 에러
    FileReader reader = new FileReader("file.txt");
}

// Unchecked — RuntimeException 상속, 처리 안 해도 됨
public void getGame(Long id) {
    throw new GameNotFoundException(id);  // try-catch 안 해도 됨
}
```
```
Exception
├── Checked (IOException, SQLException)
│   → 외부 시스템 오류, 복구 가능성 있음
└── RuntimeException (Unchecked)
    ├── GameNotFoundException
    ├── IllegalArgumentException
    └── NullPointerException
    → 프로그래밍 오류, 복구보다는 수정 필요
````

면접 답변: **"Checked는 컴파일러가 처리를 강제하는 외부 시스템 관련 예외이고, Unchecked는 RuntimeException을 상속하여 프로그래밍 오류를 나타냅니다. Spring에서는 Unchecked 기반 커스텀 예외 + GlobalExceptionHandler 조합이 일반적입니다."**

---

**9. JPA N+1 문제**

java

```java
// 게임 목록 조회 — 1번 쿼리
List<Game> games = gameRepository.findAll();

// 각 게임의 리뷰를 접근 — N번 추가 쿼리 발생!
for (Game game : games) {
    game.getReviews().size(); // 여기서 매번 SELECT 발생
}
// 게임 100개면 → 1 + 100 = 101번 쿼리 실행!
```

해결 방법:

java

```java
// fetch join — 한 번에 가져오기
@Query("SELECT g FROM Game g JOIN FETCH g.reviews")
List<Game> findAllWithReviews();
// → 쿼리 1번으로 해결
```

면접 답변: **"연관 엔티티를 지연 로딩으로 조회할 때 부모 N건에 대해 자식 조회가 N번 추가 발생하는 문제입니다. fetch join이나 @EntityGraph로 한 번에 조회하여 해결합니다."**

---

**10. @Transactional 전파 속성**

java

````java
@Transactional
public void createGame() {
    gameRepository.save(game);
    logService.saveLog(); // 이 메서드에도 @Transactional 있으면?
}
```

주요 전파 속성:
```
REQUIRED (기본값)
→ 기존 트랜잭션 있으면 참여, 없으면 새로 생성
→ createGame 트랜잭션에 saveLog가 합류

REQUIRES_NEW
→ 항상 새 트랜잭션 생성
→ saveLog가 실패해도 createGame은 커밋됨

// 예: 감사 로깅은 실패해도 본 로직은 살려야 할 때
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveLog() { ... }
````

면접 답변: **"트랜잭션이 다른 트랜잭션을 만났을 때 어떻게 동작할지 결정하는 속성입니다. 기본값 REQUIRED는 기존 트랜잭션에 참여하고, REQUIRES_NEW는 독립 트랜잭션을 생성합니다. 예를 들어 감사 로깅처럼 실패해도 본 로직에 영향을 주면 안 되는 경우 REQUIRES_NEW를 사용합니다."**