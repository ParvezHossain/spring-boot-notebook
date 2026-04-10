# ☕ Java & Spring Boot Developer Toolkit
> A curated reference for 10+ year Java / Spring Boot / MySQL / PostgreSQL / Docker developers.  
> Tools are grouped by category with usage notes, install hints, and opinionated picks. ★ = daily-driver recommendation.

---

## Table of Contents

1. [API Testing & HTTP Clients](#1-api-testing--http-clients)
2. [API Documentation](#2-api-documentation)
3. [Benchmarking & Load Testing](#3-benchmarking--load-testing)
4. [JVM Profiling & Diagnostics](#4-jvm-profiling--diagnostics)
5. [Observability — Metrics, Tracing & Logging](#5-observability--metrics-tracing--logging)
6. [Database Tools & Migrations](#6-database-tools--migrations)
7. [Docker & Container Tools](#7-docker--container-tools)
8. [Build, Static Analysis & Code Quality](#8-build-static-analysis--code-quality)
9. [Testing](#9-testing)
10. [Developer Productivity](#10-developer-productivity)
11. [Security Scanning](#11-security-scanning)
12. [Messaging & Event Streaming](#12-messaging--event-streaming)
13. [IDE Plugins Worth Installing](#13-ide-plugins-worth-installing)
14. [JVM Tuning Quick Reference](#14-jvm-tuning-quick-reference)
15. [Recommended Local Dev Stack (docker-compose)](#15-recommended-local-dev-stack-docker-compose)

---

## 1. API Testing & HTTP Clients

### ★ HTTPie
A human-friendly CLI HTTP client — far more readable than `curl` for quick API calls.

```bash
# Install
brew install httpie          # macOS
pip install httpie           # cross-platform

# Examples
http GET localhost:8080/api/users
http POST localhost:8080/api/users name="Alice" role="admin"
http PUT localhost:8080/api/users/1 Authorization:"Bearer <token>"
http --follow --print=HhBb GET https://api.example.com/data
```

**Why use it:** Automatic JSON pretty-print, sensible defaults, session support, and `--offline` mode to preview requests.

---

### ★ Bruno
A Git-native Postman alternative. Collections are stored as plain `.bru` files — version-controllable, diff-friendly, no cloud lock-in.

```bash
brew install --cask bruno   # macOS
# or download from https://www.usebruno.com
```

**Why use it:** Your API collection lives in your repo. Works offline. No account required.

---

### Postman
The industry standard. Best for teams needing shared environments, test scripts, mock servers, and monitors.

- Environments: manage `dev`, `staging`, `prod` base URLs
- Collection Runner: chain requests with test assertions
- Mock Server: simulate endpoints before backend is ready
- Newman CLI: run collections in CI pipelines

```bash
npm install -g newman
newman run MyCollection.json -e dev-environment.json
```

---

### Insomnia
Lightweight alternative to Postman with excellent support for **gRPC**, **GraphQL**, and **WebSocket** alongside REST.

```bash
brew install --cask insomnia
```

---

### curl (Advanced Usage)
You already know curl — here are the flags you actually want:

```bash
# Pretty-print JSON response
curl -s localhost:8080/api/users | jq .

# POST with JSON body
curl -X POST localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice"}' | jq .

# Follow redirects, show headers, measure timing
curl -LsI -w "@curl-timing.txt" https://api.example.com

# curl-timing.txt template
# time_namelookup:  %{time_namelookup}\n
# time_connect:     %{time_connect}\n
# time_appconnect:  %{time_appconnect}\n
# time_total:       %{time_total}\n
```

---

## 2. API Documentation

### ★ Springdoc OpenAPI (springdoc-openapi)
Auto-generates **OpenAPI 3** spec and a **Swagger UI** from your Spring annotations. Drop it in and it just works.

```xml
<!-- Maven -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

```yaml
# application.yml
springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html
    operations-sorter: method
  show-actuator: true
```

Access at: `http://localhost:8080/swagger-ui.html`

**Annotating your controllers:**
```java
@Operation(summary = "Get user by ID", description = "Returns a single user")
@ApiResponse(responseCode = "200", description = "User found")
@ApiResponse(responseCode = "404", description = "User not found")
@GetMapping("/users/{id}")
public ResponseEntity<UserDTO> getUser(@PathVariable Long id) { ... }
```

---

### Redoc
A cleaner, three-panel OpenAPI renderer — better for sharing docs externally.

```bash
npx @redocly/cli preview-docs openapi.yaml
```

Serve your `/v3/api-docs` endpoint directly into Redoc with a single HTML file.

---

### Stoplight
Design-first API platform with a visual spec editor, linting, and mock servers. Best when you want to define the contract *before* writing code.

---

## 3. Benchmarking & Load Testing

### ★ JMH (Java Microbenchmark Harness)
The only correct way to benchmark JVM code. Handles JIT warm-up, dead code elimination, and fork isolation.

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
```

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@State(Scope.Thread)
@Fork(2)
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 10, time = 1)
public class StringBenchmark {

    @Benchmark
    public String concatenation() {
        return "Hello" + " " + "World";
    }

    @Benchmark
    public String stringBuilder() {
        return new StringBuilder().append("Hello").append(" ").append("World").toString();
    }
}
```

```bash
# Run from Maven
mvn clean package
java -jar target/benchmarks.jar -prof gc
```

**Rules of thumb:**
- Always use `@Fork` (≥ 2) — no fork means JIT pollution from previous tests
- Use `@State` to avoid constant folding
- Never benchmark with `System.nanoTime()` in a loop — that's not a benchmark

---

### ★ Gatling
Code-based load testing with Scala/Java DSL and excellent HTML reports. First-class CI integration.

```scala
class BasicSimulation extends Simulation {
  val httpProtocol = http.baseUrl("http://localhost:8080")

  val scn = scenario("Load Test")
    .exec(http("Get Users")
      .get("/api/users")
      .check(status.is(200)))

  setUp(
    scn.inject(
      rampUsers(100).during(30.seconds),
      constantUsersPerSec(50).during(60.seconds)
    )
  ).protocols(httpProtocol)
    .assertions(global.responseTime.max.lt(2000))
}
```

```bash
mvn gatling:test
# Report at: target/gatling/results/*/index.html
```

---

### k6
Modern load testing tool scripted in JavaScript. Excellent for CI, cloud execution, and threshold assertions.

```bash
brew install k6

# k6 script (load-test.js)
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 50,
  duration: '30s',
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  const res = http.get('http://localhost:8080/api/users');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

```bash
k6 run load-test.js
k6 run --out influxdb=http://localhost:8086/k6 load-test.js  # stream to InfluxDB
```

---

### wrk / wrk2
Minimal, ultra-fast HTTP benchmarking from the terminal. Zero configuration.

```bash
brew install wrk

# Basic: 12 threads, 400 connections, 30s
wrk -t12 -c400 -d30s http://localhost:8080/api/users

# With Lua script for POST
wrk -t4 -c100 -d30s -s post.lua http://localhost:8080/api/users
```

---

### Apache JMeter
GUI-based enterprise standard. Heavy but extremely flexible — supports JDBC, LDAP, FTP, JMS alongside HTTP.

```bash
brew install jmeter
jmeter                     # GUI mode
jmeter -n -t test.jmx -l results.jtl   # headless CI mode
```

---

## 4. JVM Profiling & Diagnostics

### ★ async-profiler
Low-overhead CPU, allocation, and lock profiler. Generates **flame graphs**. Safe in production.

```bash
# Download
wget https://github.com/async-profiler/async-profiler/releases/latest/download/async-profiler-linux-x64.tar.gz

# Profile a running JVM (PID)
./asprof -d 30 -f flamegraph.html <PID>

# Profile from startup
java -agentpath:/path/to/libasyncProfiler.so=start,file=flamegraph.html -jar app.jar

# Profile allocation instead of CPU
./asprof -e alloc -d 30 -f alloc.html <PID>
```

**Reading flame graphs:**
- X-axis = alphabetical stack ordering (NOT time)
- Width = proportion of samples where this frame was on-stack
- Look for wide flat tops — that's where time is spent

---

### ★ JFR + JMC (Java Flight Recorder + Mission Control)
Built into the JVM since Java 11. Near-zero overhead. Safe to run in production.

```bash
# Enable JFR at startup
java -XX:+FlightRecorder \
     -XX:StartFlightRecording=duration=60s,filename=app.jfr \
     -jar app.jar

# Enable JFR at runtime (no restart needed)
jcmd <PID> JFR.start duration=60s filename=app.jfr

# Dump a running recording
jcmd <PID> JFR.dump filename=snapshot.jfr

# Open in JMC
jmc   # Then File → Open File → snapshot.jfr
```

JMC shows: CPU usage, GC events, I/O, thread contention, lock analysis, object allocation.

---

### Arthas (Alibaba)
Attach to a live JVM and diagnose without restart. Essential for production debugging.

```bash
# Download and attach
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar <PID>
```

**Most useful commands:**

| Command | What it does |
|---------|--------------|
| `trace com.example.UserService *` | Trace all methods, show call tree + timing |
| `watch com.example.UserService getUser returnObj` | Inspect return value of a live method |
| `jad com.example.UserService` | Decompile a loaded class at runtime |
| `ognl "@System@currentTimeMillis()"` | Evaluate arbitrary expressions in JVM |
| `logger --name ROOT --level DEBUG` | Change log level without restart |
| `heapdump /tmp/heap.hprof` | Take heap dump |
| `thread -b` | Find threads that are blocked/deadlocked |

---

### VisualVM
Free GUI for heap dumps, thread analysis, and GC monitoring. Good for local development.

```bash
brew install --cask visualvm
# Enable JMX in your app
java -Dcom.sun.management.jmxremote \
     -Dcom.sun.management.jmxremote.port=9090 \
     -Dcom.sun.management.jmxremote.authenticate=false \
     -jar app.jar
```

---

### Essential `jcmd` / `jmap` / `jstack` One-Liners

```bash
# List all JVM processes
jcmd

# Print all JVM flags in effect
jcmd <PID> VM.flags

# Thread dump (look for BLOCKED threads)
jstack <PID> > thread-dump.txt

# Heap summary
jcmd <PID> GC.heap_info

# Histogram of heap objects by class
jmap -histo <PID> | head -30

# Full heap dump (large file)
jmap -dump:format=b,file=heap.hprof <PID>
```

---

## 5. Observability — Metrics, Tracing & Logging

### ★ Micrometer + Prometheus + Grafana

**Micrometer** is Spring Boot's built-in metrics facade — one API, many backends.

```xml
<!-- Prometheus exporter -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus,metrics
  metrics:
    export:
      prometheus:
        enabled: true
```

Custom metrics:
```java
@Component
public class OrderMetrics {
    private final Counter ordersCreated;
    private final Timer orderProcessingTime;

    public OrderMetrics(MeterRegistry registry) {
        ordersCreated = Counter.builder("orders.created")
            .description("Total orders created")
            .tag("env", "prod")
            .register(registry);

        orderProcessingTime = Timer.builder("orders.processing.time")
            .description("Order processing duration")
            .register(registry);
    }
}
```

**Prometheus `prometheus.yml`:**
```yaml
scrape_configs:
  - job_name: 'spring-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
    scrape_interval: 15s
```

**Useful Grafana dashboard IDs to import:**
- `4701` — JVM (Micrometer)
- `11378` — Spring Boot 3.x Statistics
- `12900` — Spring Boot Hikari + DB pool

---

### ★ OpenTelemetry (OTel)
Vendor-neutral distributed tracing and metrics. Attach the agent — no code changes needed.

```bash
# Download Java agent
wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar

# Run with agent
java -javaagent:opentelemetry-javaagent.jar \
     -Dotel.service.name=my-spring-app \
     -Dotel.exporter.otlp.endpoint=http://localhost:4317 \
     -jar app.jar
```

---

### Zipkin / Jaeger
Distributed tracing UIs. Visualize call chains across microservices.

```bash
# Run Zipkin locally
docker run -d -p 9411:9411 openzipkin/zipkin

# Spring Boot integration
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
```

```yaml
management:
  tracing:
    sampling:
      probability: 1.0   # 100% in dev; use 0.1 in prod
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
```

---

### ELK Stack (Elasticsearch + Logstash + Kibana)
Centralized log aggregation and search.

**Structured logging with Logback + JSON:**
```xml
<!-- logback-spring.xml -->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
```

Always log with structured fields, not string interpolation:
```java
// Good
log.info("Order created", kv("orderId", order.getId()), kv("userId", userId));

// Bad
log.info("Order " + order.getId() + " created for user " + userId);
```

---

## 6. Database Tools & Migrations

### ★ Flyway
SQL-based migration tool. Integrates automatically with Spring Boot.

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
```

Migration file naming convention:
```
src/main/resources/db/migration/
  V1__create_users_table.sql
  V2__add_email_index.sql
  V3__create_orders_table.sql
  R__refresh_views.sql          # Repeatable migration
```

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id         BIGSERIAL PRIMARY KEY,
    name       VARCHAR(255) NOT NULL,
    email      VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
```

---

### ★ Testcontainers
Spin up real databases in Docker during tests. Eliminates H2 dialect mismatch bugs permanently.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Test
    void shouldSaveUser() { ... }
}
```

---

### ★ DBeaver
Universal database GUI — works with MySQL, PostgreSQL, Oracle, SQLite, and 80+ others.

```bash
brew install --cask dbeaver-community
```

**Most useful features:**
- ER diagram auto-generation from schema
- SQL query history with execution plans (`EXPLAIN ANALYZE` visualized)
- Data export to CSV/JSON/Excel
- SSH tunnel support for remote DBs

---

### DataGrip (JetBrains)
Best-in-class DB IDE if you're already in IntelliJ IDEA ecosystem. Paid but included in All Products Pack.

**Features over DBeaver:** Better schema diff, smarter SQL completion, refactoring support.

---

### Useful PostgreSQL Queries to Bookmark

```sql
-- Show active queries and how long they've been running
SELECT pid, now() - query_start AS duration, query, state
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

-- Find missing indexes (sequential scans on large tables)
SELECT schemaname, tablename, seq_scan, idx_scan,
       seq_scan - idx_scan AS too_much_seq
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY too_much_seq DESC;

-- Index usage stats
SELECT indexrelname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;

-- Table sizes
SELECT relname AS table, pg_size_pretty(pg_total_relation_size(relid)) AS size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;

-- Lock contention
SELECT pid, relation::regclass, mode, granted
FROM pg_locks
WHERE NOT granted;
```

---

### MySQL Useful Queries

```sql
-- Show running queries
SHOW PROCESSLIST;

-- Slow query analysis
SHOW VARIABLES LIKE 'slow_query_log%';
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- Explain a query
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';

-- Index sizes
SELECT table_name, index_name,
       ROUND(stat_value * @@innodb_page_size / 1024 / 1024, 2) AS size_mb
FROM mysql.innodb_index_stats
WHERE stat_name = 'size'
ORDER BY size_mb DESC;
```

---

## 7. Docker & Container Tools

### ★ Docker Compose — Local Dev Stack

Keep a `docker-compose.yml` in every project root. Start your full backing services with one command.

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  mysql:
    image: mysql:8.3
    environment:
      MYSQL_DATABASE: myapp
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
    ports:
      - "9092:9092"
    depends_on:
      - zookeeper

  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

volumes:
  postgres_data:
```

```bash
docker compose up -d          # start all services
docker compose down           # stop and remove containers
docker compose logs -f kafka  # tail logs for one service
```

---

### Jib (Build Docker images without Dockerfile)
Build optimized, reproducible Docker images from Maven/Gradle — no Docker daemon needed.

```xml
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>3.4.2</version>
    <configuration>
        <from>
            <image>eclipse-temurin:21-jre-alpine</image>
        </from>
        <to>
            <image>myregistry/myapp:${project.version}</image>
        </to>
        <container>
            <jvmFlags>
                <jvmFlag>-XX:+UseContainerSupport</jvmFlag>
                <jvmFlag>-XX:MaxRAMPercentage=75.0</jvmFlag>
            </jvmFlags>
        </container>
    </configuration>
</plugin>
```

```bash
mvn jib:build            # Push to registry
mvn jib:dockerBuild      # Build to local Docker daemon
```

---

### Dive — Inspect Image Layers
Find what's bloating your Docker image.

```bash
brew install dive
dive myapp:latest
```

Look for: Maven cache in image layers, duplicate files, dev dependencies included in prod image.

---

### Lazydocker
TUI dashboard for managing containers, viewing logs and resource usage — all in the terminal.

```bash
brew install jesseduffield/lazydocker/lazydocker
lazydocker
```

---

### Essential Docker Commands for Java Apps

```bash
# Check container resource usage
docker stats --no-stream

# See Java process inside container
docker exec -it <container> jps -v

# Take thread dump from container
docker exec -it <container> jstack 1

# Check JVM flags in container
docker exec -it <container> java -XX:+PrintFlagsFinal -version | grep HeapSize

# Multi-stage Dockerfile for Spring Boot
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline     # cache deps layer
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

---

## 8. Build, Static Analysis & Code Quality

### Maven Wrapper / Gradle Wrapper
Always commit the wrapper — it pins the build tool version and removes the "works on my machine" problem.

```bash
mvn wrapper:wrapper -Dmaven=3.9.6    # generate mvnw
gradle wrapper --gradle-version 8.7  # generate gradlew
```

```bash
./mvnw clean verify          # standard lifecycle
./mvnw -T 4 clean package    # parallel build (4 threads)
./mvnw dependency:tree       # inspect dependency graph
./mvnw versions:display-dependency-updates  # find outdated deps
```

---

### ★ SonarQube
Full static analysis platform — security, code smells, coverage, duplication, and tech debt tracking.

```bash
# Run locally with Docker
docker run -d --name sonar -p 9000:9000 sonarqube:community

# Analyze with Maven
./mvnw sonar:sonar \
  -Dsonar.projectKey=my-app \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=<token>
```

---

### ★ ArchUnit
Write tests that enforce your architecture rules. Fails your CI if someone violates package boundaries.

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.3.0</version>
    <scope>test</scope>
</dependency>
```

```java
@AnalyzeClasses(packages = "com.example")
class ArchitectureTest {

    @ArchTest
    static final ArchRule services_should_not_depend_on_controllers =
        noClasses().that().resideInAPackage("..service..")
                   .should().dependOnClassesThat()
                   .resideInAPackage("..controller..");

    @ArchTest
    static final ArchRule repositories_should_only_be_used_by_services =
        classes().that().resideInAPackage("..repository..")
                 .should().onlyBeAccessed().byClassesThat()
                 .resideInAnyPackage("..service..", "..repository..");

    @ArchTest
    static final ArchRule no_field_injection =
        noFields().should().beAnnotatedWith(Autowired.class);
}
```

---

### SpotBugs + PMD
Static analysis for bugs and code smells. Run in CI to catch issues before review.

```xml
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.4.0</version>
</plugin>
```

```bash
./mvnw spotbugs:check
./mvnw pmd:check
```

---

### Checkstyle
Enforce consistent formatting and style rules across the team.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <configuration>
        <configLocation>google_checks.xml</configLocation>
        <failsOnError>true</failsOnError>
    </configuration>
</plugin>
```

---

## 9. Testing

### ★ JUnit 5 + AssertJ + Mockito

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldReturnUserWhenExists() {
        // given
        var user = new User(1L, "Alice", "alice@example.com");
        given(userRepository.findById(1L)).willReturn(Optional.of(user));

        // when
        var result = userService.findById(1L);

        // then
        assertThat(result)
            .isPresent()
            .get()
            .extracting(User::getName, User::getEmail)
            .containsExactly("Alice", "alice@example.com");

        then(userRepository).should(times(1)).findById(1L);
    }

    @Test
    void shouldThrowWhenUserNotFound() {
        given(userRepository.findById(anyLong())).willReturn(Optional.empty());

        assertThatThrownBy(() -> userService.findById(99L))
            .isInstanceOf(UserNotFoundException.class)
            .hasMessageContaining("99");
    }
}
```

---

### ★ Spring Boot Slice Tests

```java
// Test only the web layer (no DB, no service beans)
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired MockMvc mockMvc;

    @MockBean UserService userService;

    @Test
    void shouldReturn200ForValidUser() throws Exception {
        given(userService.findById(1L)).willReturn(Optional.of(new UserDTO(1L, "Alice")));

        mockMvc.perform(get("/api/users/1")
                   .accept(MediaType.APPLICATION_JSON))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Alice"));
    }
}

// Test only the JPA layer (real DB via Testcontainers)
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
@Testcontainers
class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        r.add("spring.datasource.url", postgres::getJdbcUrl);
        r.add("spring.datasource.username", postgres::getUsername);
        r.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired UserRepository repository;

    @Test
    void shouldPersistUser() {
        var saved = repository.save(new User("Alice", "alice@example.com"));
        assertThat(saved.getId()).isNotNull();
    }
}
```

---

### WireMock
Stub external HTTP services in integration tests — no flaky third-party calls.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-wiremock</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@AutoConfigureWireMock(port = 0)
class PaymentServiceTest {

    @Test
    void shouldCallExternalPaymentProvider() {
        stubFor(post(urlEqualTo("/payments"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBody("""{"transactionId": "txn-123", "status": "APPROVED"}""")));

        var result = paymentService.process(new PaymentRequest(...));

        assertThat(result.getTransactionId()).isEqualTo("txn-123");
    }
}
```

---

## 10. Developer Productivity

### ★ sdkman
Switch between Java versions (11, 17, 21, GraalVM) instantly. Essential if you work on multiple projects.

```bash
# Install sdkman
curl -s "https://get.sdkman.io" | bash

# List available Java versions
sdk list java

# Install specific versions
sdk install java 21.0.3-tem
sdk install java 17.0.10-tem
sdk install java 21.0.3-graalce   # GraalVM CE

# Switch versions
sdk use java 21.0.3-tem           # current shell only
sdk default java 21.0.3-tem       # persistent default

# Pin version per project
echo "java=21.0.3-tem" > .sdkmanrc
sdk env install    # install pinned version
sdk env            # activate pinned version
```

---

### ★ Spring Boot Actuator
Already included in most projects but massively underused. Gives deep runtime insight.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"   # expose all in dev (restrict in prod)
  endpoint:
    health:
      show-details: always
    env:
      show-values: always
```

**Key endpoints:**

| Endpoint | What it shows |
|----------|---------------|
| `/actuator/health` | App health + DB + disk + custom indicators |
| `/actuator/env` | All config properties and their source (yaml, env var, etc.) |
| `/actuator/beans` | Every Spring bean wired in context |
| `/actuator/mappings` | All `@RequestMapping` routes |
| `/actuator/metrics` | Available Micrometer metrics |
| `/actuator/loggers` | Live log level control (POST to change) |
| `/actuator/threaddump` | JVM thread dump via HTTP |
| `/actuator/heapdump` | Download heap dump via HTTP |
| `/actuator/conditions` | Why each auto-configuration was applied / skipped |

Change log level at runtime (no restart):
```bash
curl -X POST localhost:8080/actuator/loggers/com.example \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

---

### ★ Spring DevTools
Fast restarts on classpath change. Live reload for templates.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

```yaml
spring:
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/java
    livereload:
      enabled: true
```

---

### IntelliJ IDEA Shortcuts Worth Memorizing

| Shortcut (Mac) | Action |
|----------------|--------|
| `⌘⇧F` | Find in all files |
| `⌘⇧R` | Replace in all files |
| `⌘⇧A` | Find any IDE action |
| `⌘B` | Go to declaration |
| `⌘⌥B` | Go to implementation |
| `⌘⌥L` | Reformat code |
| `⌃⇧R` | Run current test |
| `⌃⇧D` | Debug current test |
| `⌘⇧T` | Create/navigate to test |
| `⌥Enter` | Quick fix / intention actions |
| `⌘E` | Recent files |
| `⌘⇧E` | Recent locations |
| `⌘F12` | File structure (methods list) |
| `⌘⌥T` | Surround with (try-catch, if, etc.) |
| `⇧F6` | Rename refactoring |

---

## 11. Security Scanning

### ★ OWASP Dependency-Check
Scans your Maven/Gradle dependencies against the NVD CVE database.

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.2.0</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>   <!-- fail on High severity -->
        <format>HTML</format>
    </configuration>
</plugin>
```

```bash
./mvnw org.owasp:dependency-check-maven:check
# Report at: target/dependency-check-report.html
```

---

### Trivy
Scan Docker images, filesystems, and Git repos for CVEs.

```bash
brew install trivy

# Scan a Docker image
trivy image myapp:latest

# Scan only HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL myapp:latest

# Scan filesystem (for CI)
trivy fs --security-checks vuln,config .
```

---

### Snyk
Developer-friendly scanning with fix PR suggestions. Good CI integration.

```bash
npm install -g snyk
snyk auth
snyk test          # scan project
snyk monitor       # continuous monitoring
snyk container test myapp:latest  # scan Docker image
```

---

## 12. Messaging & Event Streaming

### ★ Offset Explorer (formerly Kafka Tool)
GUI for browsing Kafka topics, consumer groups, offsets, and messages.

Download at: [kafkatool.com](https://www.kafkatool.com)

---

### kcat (kafkacat)
CLI producer/consumer for Kafka — essential for debugging messages.

```bash
brew install kcat

# List topics
kcat -b localhost:9092 -L

# Consume latest 10 messages from topic
kcat -b localhost:9092 -t my-topic -o -10 -e

# Consume with key display
kcat -b localhost:9092 -t my-topic -f 'Key: %k\nValue: %s\n'

# Produce a message
echo '{"userId":1,"event":"ORDER_PLACED"}' | kcat -b localhost:9092 -t orders -P

# Consume from beginning, output as JSON metadata
kcat -b localhost:9092 -t my-topic -C -o beginning -J | jq .
```

---

### Spring Kafka Quick Reference

```java
// Producer
@Component
public class OrderProducer {
    private final KafkaTemplate<String, OrderEvent> kafka;

    public void publish(OrderEvent event) {
        kafka.send("orders", event.getOrderId().toString(), event)
             .whenComplete((result, ex) -> {
                 if (ex != null) log.error("Failed to send", ex);
                 else log.info("Sent to partition {} offset {}",
                     result.getRecordMetadata().partition(),
                     result.getRecordMetadata().offset());
             });
    }
}

// Consumer
@KafkaListener(topics = "orders", groupId = "order-processor",
               concurrency = "3")
public void consume(OrderEvent event,
                    @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
                    @Header(KafkaHeaders.OFFSET) long offset) {
    log.info("Received {} from partition {} offset {}", event, partition, offset);
    orderService.process(event);
}
```

---

## 13. IDE Plugins Worth Installing

### IntelliJ IDEA

| Plugin | Purpose |
|--------|---------|
| **Lombok** | Enable Lombok annotation processing |
| **SonarLint** | Real-time SonarQube issues in editor |
| **Rainbow Brackets** | Color-matched bracket pairs |
| **GitToolBox** | Inline git blame on every line |
| **String Manipulation** | Case conversion, encoding, sorting |
| **Maven Helper** | Resolve dependency conflicts visually |
| **Docker** | Manage Docker from within IDEA |
| **HTTP Client** | Built-in REST client (`.http` files) |
| **JPA Buddy** | Entity/repository/DTO scaffolding |
| **CheckStyle-IDEA** | Real-time Checkstyle violations |
| **GenerateAllSetter** | Generate all setters for a bean |
| **Grep Console** | Color-code log output in Run console |

### VS Code (as secondary editor)

```bash
code --install-extension redhat.java
code --install-extension vmware.vscode-spring-boot
code --install-extension vscjava.vscode-spring-initializr
code --install-extension ms-azuretools.vscode-docker
code --install-extension humao.rest-client
```

---

## 14. JVM Tuning Quick Reference

### Container-Aware JVM Flags (Java 11+)

```bash
# Recommended baseline for containerized Spring Boot apps
java \
  -XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75.0 \
  -XX:+UseG1GC \
  -XX:+UseStringDeduplication \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp/heap-dump.hprof \
  -XX:+ExitOnOutOfMemoryError \
  -Djava.security.egd=file:/dev/./urandom \
  -jar app.jar
```

### GC Selection Guide

| GC | Use When |
|----|----------|
| **G1GC** (default Java 9+) | General purpose, balanced latency/throughput |
| **ZGC** (`-XX:+UseZGC`) | Need sub-millisecond GC pauses, Java 15+ for production |
| **ShenandoahGC** | Low-latency alternative to ZGC (RedHat JDK) |
| **SerialGC** | Very small heaps (< 256MB), CLI tools, single-threaded |

### Useful GC Logging

```bash
java \
  -Xlog:gc*:file=/var/log/gc.log:time,uptime,level,tags:filecount=5,filesize=20m \
  -jar app.jar
```

### Hikari Connection Pool Tuning

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20       # default 10; tune with: connections = (core_count * 2) + disk_spindle_count
      minimum-idle: 5
      connection-timeout: 30000   # ms to wait for connection from pool
      idle-timeout: 600000        # ms before idle connection removed
      max-lifetime: 1800000       # ms max connection lifetime (< DB wait_timeout)
      leak-detection-threshold: 60000  # log warning if connection held > 60s
```

---

## 15. Recommended Local Dev Stack (docker-compose)

Full local environment for a typical Spring Boot microservice:

```yaml
# docker-compose.dev.yml
services:

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${APP_NAME:-myapp}
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 10s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./infra/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana

  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # Web UI

volumes:
  postgres_data:
  grafana_data:
```

**`application-local.yml`** to connect to all of the above:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp
    username: dev
    password: dev
  data:
    redis:
      host: localhost
      port: 6379
  kafka:
    bootstrap-servers: localhost:9092
  mail:
    host: localhost
    port: 1025

management:
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans
  tracing:
    sampling:
      probability: 1.0
```

---

## Quick Reference Card

```
API Testing:       Bruno ★  |  HTTPie ★  |  Postman  |  Insomnia
API Docs:          Springdoc ★  |  Swagger UI  |  Redoc
Load Testing:      Gatling ★  |  k6  |  JMH ★  |  wrk
JVM Profiling:     async-profiler ★  |  JFR+JMC ★  |  Arthas  |  VisualVM
Observability:     Micrometer+Prometheus+Grafana ★  |  OTel  |  Zipkin
DB GUI:            DBeaver ★  |  DataGrip
DB Migration:      Flyway ★  |  Liquibase
Integration Test:  Testcontainers ★  |  WireMock
Architecture:      ArchUnit ★
Static Analysis:   SonarQube ★  |  SpotBugs  |  PMD
Security Scan:     OWASP Dep-Check ★  |  Trivy  |  Snyk
Docker:            Jib ★  |  Dive  |  Lazydocker  |  Compose ★
JVM Mgmt:          sdkman ★  |  Arthas ★
Kafka CLI:         kcat ★  |  Offset Explorer ★
```

---

> **Maintained by:** [your-username](https://github.com/your-username)  
> **Last updated:** 2026  
> PRs and suggestions welcome!