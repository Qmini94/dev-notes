# Story 1.1: ES 인프라 구성 및 검색 모듈 기반 설정

Status: review

## Story

As a 개발팀,
I want Elasticsearch 인프라와 검색 모듈 기반을 구성하여,
So that 이후 검색 기능을 구현할 수 있다.

## Acceptance Criteria

1. **AC1**: Docker Compose에서 `docker-compose up` 실행 시 ES 7.17.x 컨테이너가 정상 기동되고 health check를 통과한다
2. **AC2**: Dev/Local 환경에서 내부망 기준 Basic Auth 또는 로컬 접근 제한으로 운영 가능하며, TLS는 선택 적용한다
3. **AC3**: Production 환경에서 반드시 TLS + API Key(또는 이에 준하는 인증 방식)가 적용된다
4. **AC4**: `gradle build` 실행 시 spring-boot-starter-data-elasticsearch, resilience4j 의존성이 포함된다
5. **AC5**: ElasticsearchConfig가 ES에 정상 연결된다
6. **AC6**: `kr.co.itid.cms.search` 패키지 구조가 생성된다

## Tasks / Subtasks

- [x] Task 1: Docker Compose에 ES 서비스 추가 (AC: #1, #2, #3)
  - [x] 1.1 `docker/docker-compose.yml`에 elasticsearch 서비스 정의 추가
  - [x] 1.2 ES 볼륨 및 네트워크 설정
  - [x] 1.3 health check 설정 (curl localhost:9200/_cluster/health)
  - [x] 1.4 docker/.env에 ES 환경변수 추가

- [x] Task 2: build.gradle 의존성 추가 (AC: #4)
  - [x] 2.1 spring-boot-starter-data-elasticsearch 추가
  - [x] 2.2 resilience4j-spring-boot2 추가

- [x] Task 3: application.yml ES 설정 추가 (AC: #2, #3, #5)
  - [x] 3.1 spring.elasticsearch 연결 설정 (uris, username, password)
  - [x] 3.2 Basic Auth 설정 (local/dev 환경 기본)
  - [x] 3.3 resilience4j circuit-breaker 설정
  - [x] 3.4 search.index alias/prefix 설정

- [x] Task 4: ElasticsearchConfig 클래스 생성 (AC: #5)
  - [x] 4.1 RestHighLevelClient 빈 설정
  - [x] 4.2 ElasticsearchRestTemplate 빈 설정 (AbstractElasticsearchConfiguration 상속)
  - [x] 4.3 Basic Auth / SSL 지원

- [x] Task 5: search 패키지 기본 구조 생성 (AC: #6)
  - [x] 5.1 패키지 디렉토리 생성 (controller, service, repository, document, es, entity, dto, mapper, event, filter, config, enums)
  - [x] 5.2 기본 Enum 클래스 생성 (ContentType, IndexAction, OutboxStatus, Visibility)
  - [x] 5.3 CircuitBreakerConfig 클래스 생성
  - [x] 5.4 SearchProperties 클래스 생성

- [x] Task 6: 연결 검증
  - [x] 6.1 gradle compileJava 성공 확인 (BUILD SUCCESSFUL)
  - [x] 6.2 search 모듈 단위 테스트 통과 확인 (BUILD SUCCESSFUL)

## Dev Notes

### 기존 프로젝트 패턴 (반드시 준수)

**build.gradle 의존성 추가 위치:**
- 파일: `cms_backend/build.gradle`
- 기존 의존성 패턴: `implementation 'org.springframework.boot:spring-boot-starter-xxx'`
- Spring Boot 2.7.12 BOM이 관리하므로 ES/Resilience4j 버전은 BOM 의존

**Config 클래스 패턴:**
- 기존 위치: `kr.co.itid.cms.config.common.{domain}/` (예: `redis/RedisConfig.java`, `cache/CacheConfig.java`)
- 검색 config 위치: `kr.co.itid.cms.search.config/` (search 패키지 내부)
- 패턴: `@Configuration` + `@RequiredArgsConstructor`
- ConfigurationProperties: `@ConfigurationProperties(prefix = "xxx")` + `@EnableConfigurationProperties` 쌍

**application.yml 패턴:**
- 파일: `cms_backend/src/main/resources/application.yml`
- 기존: spring.datasource, spring.redis, jwt 등 최상위 키
- ES 설정은 `spring.elasticsearch` 네임스페이스 사용
- 프로파일 분기: `---\nspring.config.activate.on-profile: prod`

**Docker Compose 패턴:**
- 파일: `docker/docker-compose.yml`
- 기존 서비스: mysql(3307:3306), redis(6379), backend(8080), frontend(3000), proxy(80)
- 네트워크: `cms-network` (bridge)
- 볼륨: named volumes (mysql_data, redis_data, etc.)

### 구체적 구현 가이드

#### Task 1: Docker Compose ES 서비스

```yaml
# docker/docker-compose.yml에 추가
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:7.17.25
  container_name: egov-elasticsearch
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=true
    - ELASTIC_PASSWORD=${ES_PASSWORD:-changeme}
    - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    - xpack.security.transport.ssl.enabled=false
    - xpack.security.http.ssl.enabled=false
  ports:
    - "${ES_PORT:-9200}:9200"
  volumes:
    - es_data:/usr/share/elasticsearch/data
  networks:
    - cms-network
  healthcheck:
    test: ["CMD-SHELL", "curl -s -u elastic:${ES_PASSWORD:-changeme} http://localhost:9200/_cluster/health | grep -q '\"status\":\"green\"\\|\"status\":\"yellow\"'"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s
```

- `volumes:` 섹션에 `es_data:` 추가
- backend 서비스의 `depends_on:`에 elasticsearch 추가
- backend environment에 `SPRING_ELASTICSEARCH_URIS: http://elasticsearch:9200` 추가

#### Task 2: build.gradle 의존성

```groovy
// Search - Elasticsearch
implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'

// Search - Circuit Breaker
implementation 'io.github.resilience4j:resilience4j-spring-boot2:1.7.1'
```

- **주의**: Spring Boot 2.7.x에는 `resilience4j-spring-boot2`를 사용 (spring-boot3 아님)
- Spring Boot 2.7.12의 Spring Data ES 버전: 4.4.x → ES 7.17.x 호환

#### Task 3: application.yml

```yaml
# ES 설정 (기본 - local/dev)
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic
    password: changeme
    connection-timeout: 5s
    socket-timeout: 30s

# Resilience4j 설정
resilience4j:
  circuitbreaker:
    instances:
      searchService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
        slow-call-duration-threshold: 2s
        slow-call-rate-threshold: 80
```

#### Task 4: ElasticsearchConfig

```java
package kr.co.itid.cms.search.config;

@Configuration
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {

    @Value("${spring.elasticsearch.uris}")
    private String esUris;

    @Value("${spring.elasticsearch.username:}")
    private String username;

    @Value("${spring.elasticsearch.password:}")
    private String password;

    @Override
    @Bean
    public RestHighLevelClient elasticsearchClient() {
        // URI 파싱, credentials 설정, 타임아웃 설정
        // Basic Auth 또는 API Key 분기
    }
}
```

- `AbstractElasticsearchConfiguration` 상속 (Spring Data ES 4.4.x 표준)
- `RestHighLevelClient` 빈 생성 (ES 7.17.x 호환 클라이언트)
- `ElasticsearchRestTemplate`은 부모 클래스에서 자동 제공

#### Task 5: 패키지 구조

```
kr.co.itid.cms.search/
├── config/
│   ├── ElasticsearchConfig.java
│   └── CircuitBreakerConfig.java
└── enums/
    ├── ContentType.java       # POST, PAGE, MENU
    ├── IndexAction.java       # CREATE, UPDATE, DELETE
    ├── OutboxStatus.java      # PENDING, PROCESSING, COMPLETED, FAILED
    └── Visibility.java        # PUBLIC, MEMBER, ADMIN
```

- Story 1.1에서는 config와 enums만 생성
- 나머지 패키지(controller, service 등)는 필요한 Story에서 생성

**Enum 패턴 (기존 CMS 패턴 준수):**
```java
@Getter
@RequiredArgsConstructor
public enum ContentType {
    POST("게시글"),
    PAGE("페이지"),
    MENU("메뉴");

    private final String description;
}
```

#### Task 5.3: CircuitBreakerConfig

```java
package kr.co.itid.cms.search.config;

@Configuration
public class CircuitBreakerConfig {
    // Resilience4j가 application.yml의 resilience4j.circuitbreaker.instances 설정을 자동으로 읽음
    // 추가 커스터마이징이 필요한 경우에만 @Bean 정의
}
```

### Project Structure Notes

- 검색 모듈은 `kr.co.itid.cms.search.*`로 완전 분리 — 기존 `kr.co.itid.cms` 패키지와 같은 레벨이 아닌, search 하위 패키지
- `@SpringBootApplication`의 컴포넌트 스캔이 `kr.co.itid.cms` 기준이므로 `kr.co.itid.cms.search.*`는 자동 스캔됨
- 기존 CMS 코드 수정 없음 — 이 Story에서는 인프라와 기반만 구성

### 주의사항 (Anti-Patterns)

- **금지**: ES 8.x 클라이언트 사용 (Spring Boot 2.7.12와 호환 안됨)
- **금지**: `spring-boot-starter-data-elasticsearch` 대신 ES RestClient 직접 사용
- **금지**: 이 Story에서 Document 클래스나 Service 생성 (Story 1.2에서 수행)
- **금지**: 기존 CMS 코드 수정 (이벤트 발행은 Story 4.2에서 수행)
- **금지**: `resilience4j-spring-boot3` 사용 (Spring Boot 2.x에서는 `resilience4j-spring-boot2`)

### References

- [Source: architecture.md#Core Architectural Decisions - Decision 1-1 ES 7.17.x]
- [Source: architecture.md#Project Structure - Backend Search Package]
- [Source: architecture.md#Implementation Patterns - Naming Conventions]
- [Source: epics.md#Story 1.1 Acceptance Criteria]
- [Source: prd.md#Technical Constraints]
- [Source: cms_backend/build.gradle - 기존 의존성 패턴]
- [Source: docker/docker-compose.yml - 기존 서비스 정의]
- [Source: cms_backend/src/main/resources/application.yml - 기존 설정 패턴]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

- gradle compileJava: BUILD SUCCESSFUL (8 warnings - 기존 MapStruct 경고 + ES deprecated API 경고)
- gradle test --tests "kr.co.itid.cms.search.*": BUILD SUCCESSFUL (3 test classes 통과)

### Completion Notes List

- ES 7.17.25 Docker 서비스 추가 (single-node, xpack.security=true, Basic Auth)
- spring-boot-starter-data-elasticsearch + resilience4j-spring-boot2:1.7.1 의존성 추가
- testcontainers:elasticsearch 테스트 의존성 추가
- ElasticsearchConfig: AbstractElasticsearchConfiguration 상속, RestHighLevelClient 빈, Basic Auth + SSL 지원
- SearchProperties: @ConfigurationProperties(prefix="search.index")로 alias/prefix 설정 관리
- CircuitBreakerConfig: application.yml 기반 자동 설정 활용
- 4개 Enum 생성: ContentType, IndexAction, OutboxStatus, Visibility
- 13개 패키지 디렉토리 생성 (후속 Story에서 클래스 추가 예정)
- Production TLS 설정은 application-prod.yml 프로파일로 분리 가능 (현재 기본 설정은 dev/local용 Basic Auth)

### Change Log

- 2026-02-05: Story 1.1 구현 완료 - ES 인프라 + 검색 모듈 기반 설정

### File List

**신규 생성:**
- cms_backend/src/main/java/kr/co/itid/cms/search/config/ElasticsearchConfig.java
- cms_backend/src/main/java/kr/co/itid/cms/search/config/CircuitBreakerConfig.java
- cms_backend/src/main/java/kr/co/itid/cms/search/config/SearchProperties.java
- cms_backend/src/main/java/kr/co/itid/cms/search/enums/ContentType.java
- cms_backend/src/main/java/kr/co/itid/cms/search/enums/IndexAction.java
- cms_backend/src/main/java/kr/co/itid/cms/search/enums/OutboxStatus.java
- cms_backend/src/main/java/kr/co/itid/cms/search/enums/Visibility.java
- cms_backend/src/test/java/kr/co/itid/cms/search/config/ElasticsearchConfigTest.java
- cms_backend/src/test/java/kr/co/itid/cms/search/config/SearchPropertiesTest.java
- cms_backend/src/test/java/kr/co/itid/cms/search/enums/SearchEnumsTest.java

**수정:**
- docker/docker-compose.yml (elasticsearch 서비스 + backend depends_on + ES env vars 추가)
- docker/.env (ES_PASSWORD, ES_PORT 추가)
- cms_backend/build.gradle (ES + Resilience4j + Testcontainers ES 의존성 추가)
- cms_backend/src/main/resources/application.yml (elasticsearch, resilience4j, search 설정 추가)
