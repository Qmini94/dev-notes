## 1. 전략 패턴 (Strategy Pattern)

### 💡 한줄 요약

> **인터페이스를 두고, 구현체를 갈아끼울 수 있게 하는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@Service
public class NotificationService {

    public void send(String type, String message) {
        if (type.equals("EMAIL")) {
            System.out.println("이메일 발송: " + message);
            // 이메일 전송 로직...
        } else if (type.equals("SMS")) {
            System.out.println("SMS 발송: " + message);
            // SMS 전송 로직...
        } else if (type.equals("PUSH")) {
            System.out.println("푸시 발송: " + message);
            // 푸시 전송 로직...
        }
        // 새로운 알림 방식 추가할 때마다 else if가 계속 늘어남...
    }
}
```

**문제점:**

- 새로운 알림 방식(카카오톡, 슬랙 등) 추가할 때마다 **기존 코드를 수정**해야 함
- if-else가 끝없이 늘어남
- 하나 고치다가 다른 로직이 깨질 위험
- **OCP(개방-폐쇄 원칙) 위반** — 확장에 열려있고, 수정에 닫혀있어야 하는데 매번 수정해야 함

### 🟢 전략 패턴 적용

```java
// 1. 전략 인터페이스 정의
public interface NotificationStrategy {
    void send(String message);
}

// 2. 각 전략(구현체) 구현
@Component("EMAIL")
public class EmailNotification implements NotificationStrategy {
    @Override
    public void send(String message) {
        System.out.println("이메일 발송: " + message);
    }
}

@Component("SMS")
public class SmsNotification implements NotificationStrategy {
    @Override
    public void send(String message) {
        System.out.println("SMS 발송: " + message);
    }
}

@Component("PUSH")
public class PushNotification implements NotificationStrategy {
    @Override
    public void send(String message) {
        System.out.println("푸시 발송: " + message);
    }
}

// 3. 컨텍스트 — 전략을 주입받아 사용
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final Map<String, NotificationStrategy> strategyMap;

    public void send(String type, String message) {
        NotificationStrategy strategy = strategyMap.get(type);
        if (strategy == null) throw new IllegalArgumentException("지원하지 않는 알림 타입");
        strategy.send(message);
    }
}
```

**장점:**

- 새로운 알림 방식 추가 → **새 클래스만 만들면 끝** (기존 코드 수정 없음)
- 각 전략이 독립적이라 **테스트가 쉬움**
- if-else 분기 제거 → 코드가 깔끔

### 🔗 스프링에서 어디에 쓰이나?

- **Service Interface + ServiceImpl** 구조 자체가 전략 패턴
- 스프링 DI(의존성 주입)의 근간
- Q의 프로젝트: `GameService(인터페이스)` + `GameServiceImpl(구현체)` 구조

### 🎯 면접 답변

> "전략 패턴은 인터페이스를 통해 알고리즘(비즈니스 로직)을 캡슐화하고, 런타임에 구현체를 교체할 수 있게 하는 패턴입니다. 스프링의 DI가 대표적인 전략 패턴 구현이며, OCP와 DIP를 만족합니다."

---

## 2. 프록시 패턴 (Proxy Pattern)

### 💡 한줄 요약

> **실제 객체 대신 대리 객체가 먼저 요청을 가로채서 부가 기능을 수행하는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    public void createOrder(OrderRequest request) {
        long startTime = System.currentTimeMillis(); // 시간 측정 시작
        log.info("createOrder 메서드 시작");          // 로깅

        // 권한 체크
        if (!SecurityUtil.hasRole("ADMIN")) {
            throw new UnauthorizedException("권한 없음");
        }

        // === 핵심 비즈니스 로직 ===
        Order order = Order.from(request);
        orderRepository.save(order);
        // === 핵심 비즈니스 로직 끝 ===

        long endTime = System.currentTimeMillis();    // 시간 측정 끝
        log.info("createOrder 실행시간: {}ms", endTime - startTime); // 로깅
    }

    public Order getOrder(Long id) {
        long startTime = System.currentTimeMillis(); // 또 시간 측정...
        log.info("getOrder 메서드 시작");              // 또 로깅...

        // 또 권한 체크...
        if (!SecurityUtil.hasRole("USER")) {
            throw new UnauthorizedException("권한 없음");
        }

        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new NotFoundException("주문 없음"));

        long endTime = System.currentTimeMillis();
        log.info("getOrder 실행시간: {}ms", endTime - startTime);

        return order;
    }
}
```

**문제점:**

- 모든 메서드마다 **로깅, 시간 측정, 권한 체크**가 중복
- 핵심 비즈니스 로직이 부가 기능에 묻혀서 읽기 어려움
- 부가 기능을 수정하려면 **모든 메서드를 다 고쳐야 함**

### 🟢 프록시 패턴 적용 (Spring AOP)

```java
// 1. 핵심 비즈니스 로직만 깔끔하게
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Transactional
    public void createOrder(OrderRequest request) {
        Order order = Order.from(request);
        orderRepository.save(order);
    }

    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new NotFoundException("주문 없음"));
    }
}

// 2. 부가 기능은 AOP(프록시)로 분리
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().getName();

        log.info("{} 메서드 시작", methodName);

        Object result = joinPoint.proceed(); // 실제 메서드 호출

        long endTime = System.currentTimeMillis();
        log.info("{} 실행시간: {}ms", methodName, endTime - startTime);

        return result;
    }
}

// 3. 권한 체크도 AOP로 분리
@Aspect
@Component
public class SecurityAspect {

    @Before("@annotation(requireRole)")
    public void checkRole(RequireRole requireRole) {
        if (!SecurityUtil.hasRole(requireRole.value())) {
            throw new UnauthorizedException("권한 없음");
        }
    }
}
```

**장점:**

- 핵심 로직과 부가 기능이 **완전히 분리** → 코드가 깨끗
- 부가 기능 수정 시 **한 곳만 고치면 전체 적용**
- 메서드가 늘어나도 부가 기능 코드 중복 없음

### 🔗 스프링에서 어디에 쓰이나?

- **@Transactional** → 프록시 객체가 트랜잭션 시작/커밋/롤백을 대신 처리
- **Spring AOP** → 로깅, 보안, 성능 측정 등 횡단 관심사 처리
- **Spring Security Filter** → 실제 컨트롤러 호출 전에 인증/인가를 대리 처리

### 🔍 @Transactional 동작 원리

```
[클라이언트] → [프록시 객체(트랜잭션 시작)] → [실제 OrderService] → [프록시 객체(커밋/롤백)]
```

Q가 `@Transactional`을 붙이면, 스프링이 자동으로 OrderService의 **프록시 객체**를 만들어서 Bean으로 등록한다. 클라이언트(Controller)가 호출하는 건 실제 OrderService가 아니라 이 프록시 객체이고, 프록시가 트랜잭션을 열고 → 실제 메서드를 호출하고 → 성공하면 커밋, 예외 나면 롤백한다.

### 🎯 면접 답변

> "프록시 패턴은 실제 객체를 대리하는 객체를 두어 접근 제어나 부가 기능을 수행하는 패턴입니다. 스프링에서는 AOP와 @Transactional이 대표적이며, CGLIB 또는 JDK Dynamic Proxy를 사용하여 런타임에 프록시 객체를 생성합니다."

---

## 3. 싱글톤 패턴 (Singleton Pattern)

### 💡 한줄 요약

> **객체를 딱 하나만 만들어서 모든 곳에서 공유하는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@RestController
public class GameController {

    @GetMapping("/api/games/{id}")
    public GameResponse getGame(@PathVariable Long id) {
        // 매 요청마다 Service 객체를 새로 생성
        GameService gameService = new GameServiceImpl();
        return gameService.getGame(id);
    }
}

@RestController
public class RankingController {

    @GetMapping("/api/rankings")
    public List<RankingResponse> getRankings() {
        // 또 새로 생성... 같은 객체인데 메모리를 중복 사용
        GameService gameService = new GameServiceImpl();
        return gameService.getRankings();
    }
}
```

**문제점:**

- 요청이 올 때마다 **같은 역할의 객체가 계속 생성**됨 → 메모리 낭비
- 초당 1,000건 요청이면 1,000개의 동일 객체가 생성됨
- DB 커넥션 풀 같은 자원은 여러 개 만들면 **자원 고갈** 위험

### 🟢 싱글톤 패턴 적용 (스프링 빈)

```java
// 스프링이 알아서 싱글톤으로 관리해줌
@Service  // 이 어노테이션 하나로 싱글톤 보장
public class GameServiceImpl implements GameService {

    private final GameRepository gameRepository;

    @Autowired // 생성자 주입 — 이것도 싱글톤 객체를 주입받는 것
    public GameServiceImpl(GameRepository gameRepository) {
        this.gameRepository = gameRepository;
    }

    @Override
    public GameResponse getGame(Long id) {
        Game game = gameRepository.findById(id)
                .orElseThrow(() -> new NotFoundException("게임 없음"));
        return GameResponse.from(game);
    }
}

// Controller에서는 주입받아 사용
@RestController
@RequiredArgsConstructor
public class GameController {

    private final GameService gameService; // 싱글톤 객체 주입

    @GetMapping("/api/games/{id}")
    public GameResponse getGame(@PathVariable Long id) {
        return gameService.getGame(id);
    }
}
```

**장점:**

- 애플리케이션 전체에서 **객체 1개만 생성** → 메모리 절약
- 초당 1,000건이든 10,000건이든 같은 객체 재사용
- 스프링이 Bean 생명주기를 관리해줌

### ⚠️ 주의할 점 — 상태를 가지면 안 됨

```java
// 🔴 절대 이렇게 하면 안 됨! (싱글톤에 상태 저장)
@Service
public class GameServiceImpl implements GameService {
    private Long currentGameId; // 공유 변수 → 동시성 문제 발생!

    public GameResponse getGame(Long id) {
        this.currentGameId = id; // 스레드 A가 저장
        // ... 이 사이에 스레드 B가 다른 id로 덮어씀
        return findGame(this.currentGameId); // 엉뚱한 게임 반환!
    }
}

// 🟢 올바른 방법 — 상태 없이 설계 (Stateless)
@Service
public class GameServiceImpl implements GameService {

    public GameResponse getGame(Long id) { // 파라미터로만 받아서 처리
        Game game = gameRepository.findById(id)
                .orElseThrow();
        return GameResponse.from(game);
    }
}
```

### 🔗 스프링에서 어디에 쓰이나?

- **모든 스프링 빈**이 기본적으로 싱글톤 (Controller, Service, Repository)
- `@Scope("prototype")`으로 변경 가능하지만 거의 안 씀

### 🎯 면접 답변

> "싱글톤 패턴은 인스턴스를 하나만 생성하여 공유하는 패턴입니다. 스프링은 IoC 컨테이너가 빈을 싱글톤으로 관리하며, 이때 빈은 무상태(Stateless)로 설계해야 동시성 문제를 방지할 수 있습니다."

---

## 4. 파사드 패턴 (Facade Pattern)

### 💡 한줄 요약

> **복잡한 내부 시스템을 감추고, 단순한 인터페이스 하나로 제공하는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@RestController
public class OrderController {

    @Autowired private ProductRepository productRepository;
    @Autowired private UserRepository userRepository;
    @Autowired private PaymentGateway paymentGateway;
    @Autowired private InventoryService inventoryService;
    @Autowired private EmailSender emailSender;

    @PostMapping("/api/orders")
    public ResponseEntity<?> createOrder(@RequestBody OrderRequest request) {
        // Controller가 모든 세부 로직을 직접 처리
        User user = userRepository.findById(request.getUserId())
                .orElseThrow();

        Product product = productRepository.findById(request.getProductId())
                .orElseThrow();

        // 재고 확인
        if (!inventoryService.isAvailable(product.getId(), request.getQuantity())) {
            throw new OutOfStockException("재고 부족");
        }

        // 결제 처리
        PaymentResult result = paymentGateway.pay(user, product, request.getQuantity());
        if (!result.isSuccess()) {
            throw new PaymentFailedException("결제 실패");
        }

        // 재고 차감
        inventoryService.decrease(product.getId(), request.getQuantity());

        // 이메일 발송
        emailSender.send(user.getEmail(), "주문 완료", "주문이 처리되었습니다.");

        return ResponseEntity.ok("주문 완료");
    }
}
```

**문제점:**

- Controller가 **모든 세부 로직을 알고 있어야 함** → 비대해짐
- 재고 확인 → 결제 → 재고 차감 → 이메일... 순서를 Controller가 관리
- 같은 로직을 다른 Controller에서도 쓰려면 **코드 복붙**
- Controller가 6개의 의존성을 가짐 → 테스트하기 어려움

### 🟢 파사드 패턴 적용

```java
// 1. Service가 복잡한 내부 로직을 감싸는 Facade 역할
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ProductRepository productRepository;
    private final UserRepository userRepository;
    private final PaymentGateway paymentGateway;
    private final InventoryService inventoryService;
    private final EmailSender emailSender;

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {
        User user = userRepository.findById(request.getUserId())
                .orElseThrow(() -> new NotFoundException("사용자 없음"));

        Product product = productRepository.findById(request.getProductId())
                .orElseThrow(() -> new NotFoundException("상품 없음"));

        validateStock(product.getId(), request.getQuantity());
        processPayment(user, product, request.getQuantity());
        inventoryService.decrease(product.getId(), request.getQuantity());
        notifyUser(user, "주문이 처리되었습니다.");

        return OrderResponse.success();
    }

    private void validateStock(Long productId, int quantity) {
        if (!inventoryService.isAvailable(productId, quantity)) {
            throw new OutOfStockException("재고 부족");
        }
    }

    private void processPayment(User user, Product product, int quantity) {
        PaymentResult result = paymentGateway.pay(user, product, quantity);
        if (!result.isSuccess()) {
            throw new PaymentFailedException("결제 실패");
        }
    }

    private void notifyUser(User user, String message) {
        emailSender.send(user.getEmail(), "주문 알림", message);
    }
}

// 2. Controller는 단순하게
@RestController
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService; // 의존성 1개만!

    @PostMapping("/api/orders")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        return ResponseEntity.ok(orderService.createOrder(request));
    }
}
```

**장점:**

- Controller는 **Service 메서드 하나만 호출** → 깔끔
- 복잡한 비즈니스 로직은 Service 내부에 캡슐화
- 다른 곳에서도 `orderService.createOrder()` 재사용 가능
- Controller 테스트할 때 Service 하나만 Mock하면 됨

### 🔗 스프링에서 어디에 쓰이나?

- **Service 레이어** 자체가 파사드 패턴
- Controller는 복잡한 내부를 모르고 Service의 단순한 메서드만 호출

### 🎯 면접 답변

> "파사드 패턴은 복잡한 서브시스템을 단순한 인터페이스로 감싸는 패턴입니다. 스프링의 Service 레이어가 대표적인 파사드로, Controller가 복잡한 비즈니스 로직의 세부사항을 몰라도 되게 합니다."

---

## 5. 어댑터 패턴 (Adapter Pattern)

### 💡 한줄 요약

> **서로 호환되지 않는 인터페이스를 연결해주는 변환기 패턴**

### 🔴 패턴 없이 작성한 코드

```java
// 외부 결제 API가 두 개인데 인터페이스가 다름
public class KakaoPay {
    public KakaoPayResponse requestPayment(KakaoPayRequest req) {
        // 카카오페이 고유 로직...
        return new KakaoPayResponse(/* ... */);
    }
}

public class NaverPay {
    public NaverPayResult processOrder(String orderId, int amount) {
        // 네이버페이 고유 로직... (메서드명, 파라미터, 리턴타입 전부 다름)
        return new NaverPayResult(/* ... */);
    }
}

// Service에서 각각 다르게 호출해야 함
@Service
public class PaymentService {

    public void pay(String provider, String orderId, int amount) {
        if (provider.equals("KAKAO")) {
            KakaoPay kakaoPay = new KakaoPay();
            KakaoPayRequest req = new KakaoPayRequest(orderId, amount);
            KakaoPayResponse res = kakaoPay.requestPayment(req);
            // 카카오 응답 처리...
        } else if (provider.equals("NAVER")) {
            NaverPay naverPay = new NaverPay();
            NaverPayResult res = naverPay.processOrder(orderId, amount);
            // 네이버 응답 처리... (완전 다른 코드)
        }
        // 토스페이 추가하면? 또 else if...
    }
}
```

**문제점:**

- 결제 수단 추가할 때마다 **if-else 증가**
- 외부 API마다 **메서드명, 파라미터, 리턴타입이 전부 다름**
- Service가 모든 외부 API의 세부사항을 알아야 함

### 🟢 어댑터 패턴 적용

```java
// 1. 공통 인터페이스 정의
public interface PaymentAdapter {
    PaymentResponse pay(String orderId, int amount);
    String getProvider();
}

// 2. 카카오페이 어댑터 — 카카오 API를 공통 인터페이스로 변환
@Component
public class KakaoPayAdapter implements PaymentAdapter {

    private final KakaoPay kakaoPay = new KakaoPay();

    @Override
    public PaymentResponse pay(String orderId, int amount) {
        // 카카오 고유 형식으로 변환
        KakaoPayRequest req = new KakaoPayRequest(orderId, amount);
        KakaoPayResponse res = kakaoPay.requestPayment(req);

        // 공통 응답으로 변환
        return new PaymentResponse(res.isSuccess(), res.getTransactionId());
    }

    @Override
    public String getProvider() {
        return "KAKAO";
    }
}

// 3. 네이버페이 어댑터
@Component
public class NaverPayAdapter implements PaymentAdapter {

    private final NaverPay naverPay = new NaverPay();

    @Override
    public PaymentResponse pay(String orderId, int amount) {
        NaverPayResult res = naverPay.processOrder(orderId, amount);
        return new PaymentResponse(res.getStatus().equals("OK"), res.getTxId());
    }

    @Override
    public String getProvider() {
        return "NAVER";
    }
}

// 4. Service는 공통 인터페이스만 사용
@Service
@RequiredArgsConstructor
public class PaymentService {

    private final Map<String, PaymentAdapter> adapterMap;

    public PaymentResponse pay(String provider, String orderId, int amount) {
        PaymentAdapter adapter = adapterMap.get(provider);
        if (adapter == null) throw new IllegalArgumentException("지원하지 않는 결제 수단");
        return adapter.pay(orderId, amount); // 어떤 결제든 동일한 방식으로 호출
    }
}
```

**장점:**

- 새로운 결제 수단 추가 → **Adapter 클래스 하나만 추가** (기존 코드 수정 없음)
- Service는 외부 API의 세부사항을 **전혀 모름**
- 220V → 110V 변환기처럼, 서로 다른 인터페이스를 **연결**해줌

### 🔗 스프링에서 어디에 쓰이나?

- **HandlerAdapter** — 다양한 형태의 Controller(@Controller, HttpRequestHandler 등)를 DispatcherServlet이 동일하게 처리
- **JDBC** — 각 DB 벤더(MySQL, PostgreSQL)의 다른 API를 JDBC라는 공통 인터페이스로 사용

### 🎯 면접 답변

> "어댑터 패턴은 호환되지 않는 인터페이스를 공통 인터페이스로 변환하여 연결하는 패턴입니다. 스프링의 HandlerAdapter가 대표적이며, 외부 API 연동 시 각기 다른 인터페이스를 통일된 방식으로 사용할 수 있게 합니다."

---

## 6. 템플릿 메서드 패턴 (Template Method Pattern)

### 💡 한줄 요약

> **전체 흐름(템플릿)은 부모가 정하고, 세부 구현은 자식이 채우는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
// CSV 파일 파싱
public class CsvParser {
    public List<Data> parse(String filePath) {
        File file = readFile(filePath);      // 1. 파일 읽기
        validate(file);                       // 2. 유효성 검증
        List<String> lines = splitLines(file); // 3. 라인 분리
        List<Data> result = new ArrayList<>();
        for (String line : lines) {
            String[] tokens = line.split(","); // CSV 파싱 로직
            result.add(mapToData(tokens));
        }
        cleanup(file);                        // 4. 정리
        return result;
    }
}

// JSON 파일 파싱 — 1, 2, 4번이 완전히 동일한데 복붙됨
public class JsonParser {
    public List<Data> parse(String filePath) {
        File file = readFile(filePath);       // 동일
        validate(file);                        // 동일
        String content = readContent(file);
        List<Data> result = objectMapper.readValue(content, ...); // JSON 파싱 로직
        cleanup(file);                         // 동일
        return result;
    }
}
```

**문제점:**

- 파일 읽기, 유효성 검증, 정리 로직이 **중복**
- 파일 처리 순서(흐름)를 각 클래스가 개별 관리 → 실수 가능성

### 🟢 템플릿 메서드 패턴 적용

```java
// 1. 추상 클래스 — 전체 흐름(템플릿)을 정의
public abstract class AbstractFileParser {

    // 템플릿 메서드 — 전체 흐름은 고정, final로 오버라이드 방지
    public final List<Data> parse(String filePath) {
        File file = readFile(filePath);      // 공통
        validate(file);                       // 공통
        List<Data> result = doParse(file);    // ← 이 부분만 자식이 구현
        cleanup(file);                        // 공통
        return result;
    }

    private File readFile(String filePath) { /* 공통 로직 */ }
    private void validate(File file) { /* 공통 로직 */ }
    private void cleanup(File file) { /* 공통 로직 */ }

    // 자식 클래스가 구현해야 하는 추상 메서드
    protected abstract List<Data> doParse(File file);
}

// 2. CSV 파서 — 파싱 로직만 구현
public class CsvParser extends AbstractFileParser {
    @Override
    protected List<Data> doParse(File file) {
        // CSV 전용 파싱 로직만 작성
        return lines.stream()
                .map(line -> line.split(","))
                .map(this::mapToData)
                .collect(Collectors.toList());
    }
}

// 3. JSON 파서 — 파싱 로직만 구현
public class JsonParser extends AbstractFileParser {
    @Override
    protected List<Data> doParse(File file) {
        // JSON 전용 파싱 로직만 작성
        return objectMapper.readValue(file, new TypeReference<>() {});
    }
}
```

**장점:**

- 공통 흐름(읽기 → 검증 → 파싱 → 정리)은 **한 곳에서 관리**
- 자식 클래스는 **다른 부분만 구현**하면 됨
- 새로운 파서 추가 시 `doParse()`만 구현하면 끝

### 🔗 스프링에서 어디에 쓰이나?

- **DispatcherServlet** — HTTP 요청 처리의 전체 흐름이 템플릿 메서드
- **AbstractController** — 공통 처리 후 자식 클래스의 handleRequestInternal 호출

### 🎯 면접 답변

> "템플릿 메서드 패턴은 상위 클래스에서 알고리즘의 골격을 정의하고, 하위 클래스에서 특정 단계를 오버라이딩하는 패턴입니다. 코드 중복을 줄이고 전체 흐름을 일관되게 유지할 수 있습니다."

---

## 7. 팩토리 메서드 패턴 (Factory Method Pattern)

### 💡 한줄 요약

> **객체 생성을 직접 하지 않고, 팩토리에게 위임하는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@Service
public class NotificationService {

    public void notify(String type, String message) {
        // 객체 생성을 직접 new로 처리
        if (type.equals("EMAIL")) {
            EmailSender sender = new EmailSender("smtp.gmail.com", 587, "apikey123");
            sender.send(message);
        } else if (type.equals("SMS")) {
            SmsSender sender = new SmsSender("twilio", "accountSid", "authToken");
            sender.send(message);
        } else if (type.equals("SLACK")) {
            SlackSender sender = new SlackSender("webhook-url", "#general");
            sender.send(message);
        }
    }
}
```

**문제점:**

- 객체 **생성 로직**과 **사용 로직**이 뒤섞임
- 생성 시 필요한 설정값(API key, URL 등)을 Service가 알아야 함
- 새로운 타입 추가마다 기존 코드 수정 필요

### 🟢 팩토리 메서드 패턴 적용

```java
// 1. 공통 인터페이스
public interface MessageSender {
    void send(String message);
    String getType();
}

// 2. 각 구현체 — 생성과 설정을 내부에서 관리
@Component
public class EmailSender implements MessageSender {
    @Value("${mail.host}") private String host;

    @Override
    public void send(String message) { /* 이메일 발송 */ }

    @Override
    public String getType() { return "EMAIL"; }
}

@Component
public class SmsSender implements MessageSender {
    @Value("${sms.api-key}") private String apiKey;

    @Override
    public void send(String message) { /* SMS 발송 */ }

    @Override
    public String getType() { return "SMS"; }
}

// 3. 팩토리 — 타입에 맞는 객체를 찾아서 반환
@Component
@RequiredArgsConstructor
public class MessageSenderFactory {

    private final List<MessageSender> senders;

    public MessageSender getSender(String type) {
        return senders.stream()
                .filter(s -> s.getType().equals(type))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("지원하지 않는 타입: " + type));
    }
}

// 4. Service는 팩토리에게 객체 생성을 위임
@Service
@RequiredArgsConstructor
public class NotificationService {

    private final MessageSenderFactory factory;

    public void notify(String type, String message) {
        MessageSender sender = factory.getSender(type); // 팩토리가 알아서 반환
        sender.send(message);
    }
}
```

**장점:**

- Service는 **어떤 객체가 생성되는지 몰라도 됨**
- 새로운 발송 방식 추가 → `MessageSender` 구현체만 추가 (팩토리/서비스 수정 없음)
- 객체 생성 로직이 **한 곳에 집중** → 관리 용이

### 🔗 스프링에서 어디에 쓰이나?

- **BeanFactory / ApplicationContext** — 빈 이름으로 객체를 찾아 반환하는 것 자체가 팩토리 패턴
- **FactoryBean** 인터페이스

### 🎯 면접 답변

> "팩토리 메서드 패턴은 객체 생성을 별도의 팩토리 클래스에 위임하여, 클라이언트가 구체적인 생성 로직을 몰라도 되게 하는 패턴입니다. 스프링의 BeanFactory가 대표적인 예시입니다."

---

## 8. 옵저버 패턴 (Observer Pattern)

### 💡 한줄 요약

> **이벤트가 발생하면 구독자들에게 자동으로 알려주는 패턴**

### 🔴 패턴 없이 작성한 코드

```java
@Service
public class OrderService {

    @Autowired private InventoryService inventoryService;
    @Autowired private EmailService emailService;
    @Autowired private PointService pointService;
    @Autowired private LogService logService;

    @Transactional
    public void completeOrder(Order order) {
        order.complete();
        orderRepository.save(order);

        // 주문 완료 후 해야 할 일들을 직접 호출
        inventoryService.decrease(order.getProductId(), order.getQuantity());
        emailService.sendOrderComplete(order.getUserEmail());
        pointService.addPoint(order.getUserId(), order.getAmount());
        logService.logOrderComplete(order.getId());
        // 슬랙 알림 추가해달래... 또 여기 수정...
        // 쿠폰 발급 추가해달래... 또 여기 수정...
    }
}
```

**문제점:**

- 주문 완료 후 할 일이 추가될 때마다 **OrderService를 수정**해야 함
- OrderService가 너무 많은 서비스에 **의존** → 결합도 높음
- 하나가 실패하면 전체가 롤백될 수 있음

### 🟢 옵저버 패턴 적용 (Spring Event)

```java
// 1. 이벤트 정의
public class OrderCompletedEvent {
    private final Long orderId;
    private final Long userId;
    private final String userEmail;
    private final int amount;

    public OrderCompletedEvent(Order order) {
        this.orderId = order.getId();
        this.userId = order.getUserId();
        this.userEmail = order.getUserEmail();
        this.amount = order.getAmount();
    }
}

// 2. 이벤트 발행 — OrderService는 이벤트만 던짐
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public void completeOrder(Order order) {
        order.complete();
        orderRepository.save(order);

        // 이벤트 발행 — 누가 듣고 있는지 OrderService는 모름
        eventPublisher.publishEvent(new OrderCompletedEvent(order));
    }
}

// 3. 이벤트 구독자(리스너)들 — 각자 독립적으로 처리
@Component
public class InventoryEventListener {
    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        inventoryService.decrease(event.getProductId(), event.getQuantity());
    }
}

@Component
public class EmailEventListener {
    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        emailService.sendOrderComplete(event.getUserEmail());
    }
}

@Component
public class PointEventListener {
    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        pointService.addPoint(event.getUserId(), event.getAmount());
    }
}

// 슬랙 알림 추가? → 리스너 하나만 추가하면 끝!
@Component
public class SlackEventListener {
    @EventListener
    public void handleOrderCompleted(OrderCompletedEvent event) {
        slackService.notify("주문 완료: " + event.getOrderId());
    }
}
```

**장점:**

- OrderService는 **이벤트만 던지고 끝** → 후속 처리를 모름 (결합도 ↓)
- 새로운 후속 처리 추가 → **리스너 클래스만 추가** (기존 코드 수정 없음)
- 각 리스너가 독립적 → 하나 실패해도 다른 건 정상 동작 가능

### 🔗 스프링에서 어디에 쓰이나?

- **ApplicationEventPublisher** + **@EventListener**
- **@TransactionalEventListener** — 트랜잭션 커밋 후 이벤트 처리
- 마이크로서비스에서는 Kafka, RabbitMQ 같은 메시지 큐로 확장

### 🎯 면접 답변

> "옵저버 패턴은 이벤트 발생 시 구독자들에게 자동으로 통지하는 패턴입니다. 스프링의 ApplicationEventPublisher와 @EventListener가 대표적이며, 서비스 간 결합도를 낮추고 확장성을 높입니다."

---

## 📋 한눈에 보는 정리표

|패턴|핵심 키워드|스프링에서의 적용|해결하는 문제|
|---|---|---|---|
|**전략**|인터페이스 + 구현체 교체|DI, Service Interface|if-else 분기 제거, OCP|
|**프록시**|대리 객체가 부가 기능|AOP, @Transactional|횡단 관심사 분리|
|**싱글톤**|객체 1개만 생성|스프링 빈 기본 스코프|메모리 낭비, 자원 관리|
|**파사드**|복잡한 내부를 단순하게|Service 레이어|Controller 비대화|
|**어댑터**|다른 인터페이스 연결|HandlerAdapter, JDBC|외부 API 호환|
|**템플릿 메서드**|흐름 고정 + 세부 위임|DispatcherServlet|코드 중복|
|**팩토리**|객체 생성 위임|BeanFactory|생성/사용 로직 분리|
|**옵저버**|이벤트 발행/구독|@EventListener|서비스 간 결합도|

---

> 💡 **핵심 포인트**: 디자인 패턴은 외우는 게 아니라, **"이 코드에 이런 문제가 있어서 → 이 패턴으로 해결했다"**를 설명할 수 있으면 된다. 면접에서는 패턴 이름보다 **왜 썼는지**를 물어본다.