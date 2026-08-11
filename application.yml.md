```
# =============================================================================
# APPLICATION CONFIGURATION — ENTERPRISE REFERENCE SCAFFOLD
# =============================================================================
# Usage:
#   - This is the BASE config. Profile-specific files OVERRIDE values here.
#   - All secrets MUST come from environment variables or a secrets manager.
#   - Never commit real credentials. Use ${ENV_VAR:default} syntax.
#   - Profile files: application-dev.yml / application-staging.yml / application-prod.yml
#
# Secret Management Options (pick one per environment):
#   Dev      → .env file + spring-dotenv or IDE env vars
#   Staging  → AWS Secrets Manager / HashiCorp Vault
#   Prod     → AWS Secrets Manager / HashiCorp Vault / GCP Secret Manager
# =============================================================================

# =============================================================================
# SPRING CORE
# =============================================================================
spring:
  application:
    name: ${APP_NAME:my-service}                        # used in tracing, actuator, logs

  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}               # override via env var in CI/CD

  # ---------------------------------------------------------------------------
  # BANNER
  # ---------------------------------------------------------------------------
  main:
    banner-mode: log                                    # off | console | log
    lazy-initialization: false                          # true speeds up dev startup only
    allow-bean-definition-overriding: false             # catch duplicate beans early

  # ---------------------------------------------------------------------------
  # DATASOURCE — PostgreSQL (Primary)
  # ---------------------------------------------------------------------------
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/myapp}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:changeme}
    driver-class-name: org.postgresql.Driver

    hikari:
      # Pool sizing formula: connections = (core_count * 2) + effective_spindle_count
      # For a 4-core host with SSD: ~10 is often optimal. Tune with load testing.
      pool-name: HikariPool-Primary
      maximum-pool-size: ${DB_POOL_MAX:20}
      minimum-idle: ${DB_POOL_MIN_IDLE:5}
      idle-timeout: 600000                              # 10 min — remove idle connections
      max-lifetime: 1800000                             # 30 min — must be < DB wait_timeout
      connection-timeout: 30000                         # 30s — fail fast if pool exhausted
      keepalive-time: 300000                            # 5 min — prevent firewall timeouts
      leak-detection-threshold: 60000                  # warn if connection held > 60s

      # Connection validation
      connection-test-query: SELECT 1
      validation-timeout: 5000

      # PostgreSQL-specific optimizations
      data-source-properties:
        reWriteBatchedInserts: true                     # batch INSERT performance
        preparedStatementCacheQueries: 256
        preparedStatementCacheSizeMiB: 5
        socketTimeout: 60                               # seconds — prevent hung queries
        connectTimeout: 10
        ApplicationName: ${spring.application.name}    # visible in pg_stat_activity

  # ---------------------------------------------------------------------------
  # READ REPLICA DATASOURCE (optional — uncomment and wire to AbstractRoutingDataSource)
  # ---------------------------------------------------------------------------
  # datasource-replica:
  #   url: ${DB_REPLICA_URL:jdbc:postgresql://replica:5432/myapp}
  #   username: ${DB_REPLICA_USERNAME:readonly}
  #   password: ${DB_REPLICA_PASSWORD:changeme}
  #   hikari:
  #     pool-name: HikariPool-Replica
  #     maximum-pool-size: 10
  #     read-only: true

  # ---------------------------------------------------------------------------
  # JPA / HIBERNATE
  # ---------------------------------------------------------------------------
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    open-in-view: false                                 # CRITICAL — prevents lazy-loading through HTTP thread
    show-sql: false                                     # override to true in dev profile only
    hibernate:
      ddl-auto: validate                                # PROD: validate | DEV: update | CI: create-drop
                                                        # Use Flyway for schema changes — never rely on ddl-auto in prod
    properties:
      hibernate:
        format_sql: false
        use_sql_comments: false
        jdbc:
          batch_size: 50                                # batch inserts/updates
          fetch_size: 100                               # JDBC cursor fetch size
          order_inserts: true
          order_updates: true
        cache:
          use_second_level_cache: true
          use_query_cache: true
          region:
            factory_class: org.hibernate.cache.jcache.JCacheCacheRegionFactory
        generate_statistics: false                      # enable only when profiling
        connection:
          provider_disables_autocommit: true            # perf: avoid extra round-trips
        query:
          in_clause_parameter_padding: true             # better query plan caching

  # ---------------------------------------------------------------------------
  # FLYWAY — Database Migration
  # ---------------------------------------------------------------------------
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: false                          # true only on first migration of existing DB
    validate-on-migrate: true
    out-of-order: false                                 # never allow in prod
    clean-disabled: true                                # CRITICAL — never allow flyway:clean in prod
    placeholders:
      app_schema: public

  # ---------------------------------------------------------------------------
  # REDIS — Cache + Sessions + Rate Limiting + Pub/Sub
  # ---------------------------------------------------------------------------
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}                      # empty = no auth (dev only)
      database: 0
      timeout: 2000ms
      connect-timeout: 2000ms
      client-name: ${spring.application.name}

      lettuce:
        pool:
          max-active: 16                                # max connections
          max-idle: 16
          min-idle: 4
          max-wait: 1000ms                              # wait before "pool exhausted" error
        shutdown-timeout: 200ms

  # ---------------------------------------------------------------------------
  # CACHE (Spring Cache abstraction over Redis)
  # ---------------------------------------------------------------------------
  cache:
    type: redis
    redis:
      time-to-live: 3600000                            # 1 hour default TTL (ms)
      cache-null-values: false
      use-key-prefix: true
      key-prefix: "${spring.application.name}:cache:"

  # ---------------------------------------------------------------------------
  # SESSION (Redis-backed — enables horizontal scaling)
  # ---------------------------------------------------------------------------
  session:
    store-type: redis
    timeout: 1800                                       # 30 min (seconds)
    redis:
      namespace: "${spring.application.name}:session"
      flush-mode: on-save

  # ---------------------------------------------------------------------------
  # SECURITY — OAuth2
  # ---------------------------------------------------------------------------
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - openid
              - email
              - profile
            redirect-uri: "${app.frontend.base-url}/login/oauth2/code/google"
            authorization-grant-type: authorization_code
            client-name: Google

          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope:
              - read:user
              - user:email
            redirect-uri: "${app.frontend.base-url}/login/oauth2/code/github"
            authorization-grant-type: authorization_code
            client-name: GitHub

          # Example: Microsoft Entra ID (Azure AD) — common in enterprise
          # microsoft:
          #   client-id: ${AZURE_CLIENT_ID}
          #   client-secret: ${AZURE_CLIENT_SECRET}
          #   scope: openid, profile, email
          #   redirect-uri: "${app.frontend.base-url}/login/oauth2/code/microsoft"
          #   authorization-grant-type: authorization_code
          #   provider: microsoft

        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
            token-uri: https://oauth2.googleapis.com/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
            jwk-set-uri: https://www.googleapis.com/oauth2/v3/certs
            user-name-attribute: sub
            issuer-uri: https://accounts.google.com

          # GitHub is auto-configured by Spring Security — no provider block needed
          # microsoft:
          #   issuer-uri: https://login.microsoftonline.com/${AZURE_TENANT_ID}/v2.0

      # Resource server (if this service also validates tokens from another auth server)
      # resourceserver:
      #   jwt:
      #     issuer-uri: ${AUTH_SERVER_URL}
      #     jwk-set-uri: ${AUTH_SERVER_URL}/.well-known/jwks.json

  # ---------------------------------------------------------------------------
  # MAIL (SMTP)
  # ---------------------------------------------------------------------------
  mail:
    host: ${MAIL_HOST:smtp.gmail.com}
    port: ${MAIL_PORT:587}
    username: ${MAIL_USERNAME}
    password: ${MAIL_PASSWORD}                          # use App Password for Gmail
    default-encoding: UTF-8
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true
          connectiontimeout: 10000
          timeout: 10000
          writetimeout: 10000
          ssl:
            trust: smtp.gmail.com

  # ---------------------------------------------------------------------------
  # RABBITMQ
  # ---------------------------------------------------------------------------
  rabbitmq:
    host: ${RABBITMQ_HOST:localhost}
    port: ${RABBITMQ_PORT:5672}
    username: ${RABBITMQ_USERNAME:guest}
    password: ${RABBITMQ_PASSWORD:guest}
    virtual-host: ${RABBITMQ_VHOST:/}

    # Connection factory
    connection-timeout: 10000ms
    requested-heartbeat: 30s
    channel-rpc-timeout: 10s

    # Publisher confirms (reliability)
    publisher-confirm-type: correlated               # none | simple | correlated
    publisher-returns: true

    template:
      mandatory: true
      receive-timeout: 5000ms
      reply-timeout: 5000ms
      retry:
        enabled: true
        initial-interval: 1000ms
        max-attempts: 3
        multiplier: 2.0
        max-interval: 10000ms

    listener:
      simple:
        acknowledge-mode: manual                     # CRITICAL — manual ack prevents message loss
        default-requeue-rejected: false              # send to DLQ, don't requeue poison messages
        concurrency: ${RABBITMQ_CONCURRENCY:5}
        max-concurrency: ${RABBITMQ_MAX_CONCURRENCY:20}
        prefetch: 10
        retry:
          enabled: true
          initial-interval: 1000ms
          max-attempts: 3
          multiplier: 2.0
          max-interval: 10000ms

  # ---------------------------------------------------------------------------
  # HTTP CLIENT (WebClient / RestClient defaults)
  # ---------------------------------------------------------------------------
  # (Spring 6.x RestClient bean should be configured in @Configuration)
  # Reference timeouts used in @Bean RestClient:
  # connect-timeout: 5s, read-timeout: 30s

  # ---------------------------------------------------------------------------
  # ASYNC / TASK EXECUTION
  # ---------------------------------------------------------------------------
  task:
    execution:
      pool:
        core-size: ${TASK_POOL_CORE:10}
        max-size: ${TASK_POOL_MAX:50}
        queue-capacity: 1000
        keep-alive: 60s
      thread-name-prefix: "async-exec-"
    scheduling:
      pool:
        size: 5
      thread-name-prefix: "scheduler-"

  # ---------------------------------------------------------------------------
  # JACKSON (JSON Serialization)
  # ---------------------------------------------------------------------------
  jackson:
    serialization:
      write-dates-as-timestamps: false
      fail-on-empty-beans: false
    deserialization:
      fail-on-unknown-properties: false
      accept-single-value-as-array: true
    default-property-inclusion: non_null               # omit null fields from responses
    time-zone: UTC
    date-format: "yyyy-MM-dd'T'HH:mm:ss.SSSZ"

  # ---------------------------------------------------------------------------
  # MVC / WEB
  # ---------------------------------------------------------------------------
  mvc:
    format:
      date-time: iso
    pathmatch:
      use-suffix-pattern: false
    throw-exception-if-no-handler-found: true          # return 404 JSON, not Whitelabel error

  web:
    resources:
      add-mappings: false                              # disable static resource handler if pure API

  # ---------------------------------------------------------------------------
  # MULTIPART (File Upload)
  # ---------------------------------------------------------------------------
  servlet:
    multipart:
      enabled: true
      max-file-size: ${MAX_FILE_SIZE:50MB}
      max-request-size: ${MAX_REQUEST_SIZE:100MB}
      file-size-threshold: 2MB                        # write to disk after this size

  # ---------------------------------------------------------------------------
  # TRANSACTIONS
  # ---------------------------------------------------------------------------
  transaction:
    default-timeout: 30                               # seconds — global TX timeout
    rollback-on-commit-failure: true


# =============================================================================
# SERVER
# =============================================================================
server:
  port: ${SERVER_PORT:8080}
  shutdown: graceful                                  # wait for in-flight requests before shutdown

  servlet:
    context-path: /
    session:
      timeout: 30m
      cookie:
        name: SESSION
        http-only: true
        secure: ${SERVER_SECURE_COOKIE:false}         # true in prod (HTTPS only)
        same-site: strict

  # Tomcat tuning
  tomcat:
    threads:
      max: ${TOMCAT_MAX_THREADS:200}
      min-spare: ${TOMCAT_MIN_THREADS:20}
    accept-count: 100                                 # queue before refusing connections
    connection-timeout: 60000
    max-connections: 8192
    keep-alive-timeout: 75000
    accesslog:
      enabled: true
      pattern: '%h %l %u %t "%r" %s %b %D ms'        # D = duration in ms
      rotate: true
      rename-on-rotate: true
      max-days: 30

  # HTTP/2 support
  http2:
    enabled: true

  # Compression
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
    min-response-size: 2048

  # Forward headers (behind reverse proxy / load balancer)
  forward-headers-strategy: native                   # or: framework


# =============================================================================
# ACTUATOR — Health, Metrics, Observability
# =============================================================================
management:
  server:
    port: ${MANAGEMENT_PORT:8081}                    # expose actuator on a separate port
    # address: 127.0.0.1                             # restrict to localhost in prod

  endpoints:
    web:
      exposure:
        include: ${ACTUATOR_ENDPOINTS:health,info,metrics,prometheus,loggers,env,flyway,liquibase,threaddump,heapdump,caches}
        # prod: restrict to: health,info,prometheus
      base-path: /actuator
    enabled-by-default: false                        # opt-in only

  endpoint:
    health:
      enabled: true
      show-details: ${ACTUATOR_HEALTH_DETAILS:when-authorized}   # never | when-authorized | always
      show-components: when-authorized
      probes:
        enabled: true                                # /actuator/health/liveness + /readiness
      group:
        liveness:
          include: livenessState,diskSpace
        readiness:
          include: readinessState,db,redis,rabbit
    info:
      enabled: true
    metrics:
      enabled: true
    prometheus:
      enabled: true
    loggers:
      enabled: true
    env:
      enabled: true
      show-values: ${ACTUATOR_ENV_SHOW_VALUES:never} # never in prod
    threaddump:
      enabled: true
    heapdump:
      enabled: ${ACTUATOR_HEAPDUMP_ENABLED:false}    # enable only when debugging
    flyway:
      enabled: true
    caches:
      enabled: true
    shutdown:
      enabled: false                                 # never expose in prod without auth

  # Liveness / readiness for Kubernetes
  health:
    livenessstate:
      enabled: true
    readinessstate:
      enabled: true
    redis:
      enabled: true
    rabbit:
      enabled: true
    db:
      enabled: true
    diskspace:
      enabled: true
      threshold: 10GB

  # Metrics
  metrics:
    export:
      prometheus:
        enabled: true
        step: 60s
        descriptions: true
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
      version: ${app.version:unknown}
      region: ${APP_REGION:local}
    distribution:
      percentiles-histogram:
        http.server.requests: true                  # enable histograms for P95/P99
        spring.data.repository.invocations: true
      percentiles:
        http.server.requests: 0.5,0.90,0.95,0.99
      slo:
        http.server.requests: 50ms,100ms,200ms,500ms,1s,2s

  # Distributed Tracing
  tracing:
    sampling:
      probability: ${TRACING_SAMPLE_RATE:0.1}       # 10% in prod; 1.0 in dev
    propagation:
      type: w3c,b3                                  # W3C standard + Zipkin B3

  zipkin:
    tracing:
      endpoint: ${ZIPKIN_URL:http://localhost:9411/api/v2/spans}
      connect-timeout: 5s
      read-timeout: 10s

  # Info endpoint content
  info:
    env:
      enabled: true
    java:
      enabled: true
    os:
      enabled: true
    git:
      enabled: true
      mode: full                                    # requires git-commit-id-maven-plugin


# =============================================================================
# LOGGING
# =============================================================================
logging:
  level:
    root: INFO
    # Application packages
    com.example: ${LOG_LEVEL_APP:INFO}
    com.example.security: DEBUG                     # override per-package as needed

    # Framework noise reduction
    org.springframework: WARN
    org.springframework.security: INFO
    org.springframework.web: WARN
    org.springframework.cache: WARN
    org.springframework.data: WARN
    org.springframework.transaction: WARN
    org.hibernate: WARN
    org.hibernate.SQL: ${LOG_LEVEL_SQL:WARN}        # set to DEBUG to see SQL in dev
    org.hibernate.type.descriptor.sql: WARN         # set to TRACE to see bind parameters
    org.flywaydb: INFO
    com.zaxxer.hikari: WARN
    io.lettuce: WARN
    org.apache.kafka: WARN
    com.rabbitmq: WARN

  pattern:
    # Structured console pattern — includes trace/span IDs for correlation
    console: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId},%X{spanId}] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{traceId},%X{spanId}] %-5level %logger{36} - %msg%n"

  file:
    name: ${LOG_FILE_PATH:logs}/${spring.application.name}.log
    max-size: 100MB
    max-history: 30
    total-size-cap: 3GB

  # Use logstash-logback-encoder for JSON logs in prod
  # Configure via logback-spring.xml for full JSON structured logging


# =============================================================================
# SPRINGDOC — OpenAPI 3 / Swagger UI
# =============================================================================
springdoc:
  api-docs:
    path: /v3/api-docs
    enabled: ${SWAGGER_ENABLED:true}               # disable in prod or restrict with security
  swagger-ui:
    path: /swagger-ui.html
    enabled: ${SWAGGER_ENABLED:true}
    operations-sorter: method
    tags-sorter: alpha
    display-request-duration: true
    filter: true
    try-it-out-enabled: ${SWAGGER_TRY_IT_OUT:false}
    oauth:
      use-pkce-with-authorization-code-grant: true
  show-actuator: false
  packages-to-scan: com.example
  cache:
    disabled: false


# =============================================================================
# JWT
# =============================================================================
jwt:
  secret: ${JWT_SECRET}                            # NEVER hardcode. Min 256-bit base64 key.
  issuer: ${spring.application.name}
  audience: ${APP_JWT_AUDIENCE:my-service-clients}
  expiration: ${JWT_EXPIRY_MS:3600000}             # 1 hour
  refresh-expiration: ${JWT_REFRESH_EXPIRY_MS:604800000}  # 7 days
  token-prefix: "Bearer "
  header-name: Authorization


# =============================================================================
# RATE LIMITING
# =============================================================================
rate-limiter:
  enabled: ${RATE_LIMITER_ENABLED:true}
  type: ${RATE_LIMITER_TYPE:redis}                 # redis (prod) | memory (dev)
  max-requests: ${RATE_LIMITER_MAX:100}
  window-seconds: ${RATE_LIMITER_WINDOW:60}
  # Per-endpoint overrides (map in @Configuration)
  routes:
    /api/auth/login:
      max-requests: 5
      window-seconds: 60
    /api/auth/register:
      max-requests: 3
      window-seconds: 300
    /api/auth/password-reset:
      max-requests: 3
      window-seconds: 300
    /api/v1/public:
      max-requests: 300
      window-seconds: 60


# =============================================================================
# APPLICATION — Domain / Business Config
# =============================================================================
app:
  version: ${APP_VERSION:local}
  environment: ${spring.profiles.active}

  # Base URLs
  frontend:
    base-url: ${FRONTEND_URL:http://localhost:3000}
    cors-origins:
      - ${FRONTEND_URL:http://localhost:3000}
      - http://localhost:8080

  backend:
    base-url: ${BACKEND_URL:http://localhost:8080}

  # CORS (read by your WebMvcConfigurer)
  cors:
    allowed-origins: ${CORS_ALLOWED_ORIGINS:http://localhost:3000}
    allowed-methods: GET,POST,PUT,PATCH,DELETE,OPTIONS
    allowed-headers: "*"
    allow-credentials: true
    max-age: 3600

  # Security
  security:
    whitelist-paths:
      - /actuator/health
      - /actuator/info
      - /v3/api-docs/**
      - /swagger-ui/**
      - /api/auth/**
      - /api/v1/public/**
    admin-paths:
      - /actuator/**
      - /api/admin/**

  # Pagination defaults
  pagination:
    default-page-size: 20
    max-size: 100

  # File Storage
  storage:
    type: ${STORAGE_TYPE:local}                    # local | s3 | gcs
    local-path: ${STORAGE_LOCAL_PATH:/tmp/uploads}
    s3:
      bucket: ${AWS_S3_BUCKET:}
      region: ${AWS_REGION:us-east-1}
      prefix: uploads/
      max-file-size-bytes: 52428800               # 50 MB
      allowed-content-types:
        - image/jpeg
        - image/png
        - image/webp
        - application/pdf

  # Email
  email:
    from: ${MAIL_FROM:noreply@example.com}
    from-name: "${APP_NAME:My App} Team"
    reply-to: ${MAIL_REPLY_TO:support@example.com}
    template-prefix: classpath:/templates/email/
    template-suffix: .html

  # Auth
  auth:
    oauth2:
      success-redirect-uri: ${OAUTH2_SUCCESS_REDIRECT:${app.frontend.base-url}/oauth2/success}
      failure-redirect-uri: ${OAUTH2_FAILURE_REDIRECT:${app.frontend.base-url}/oauth2/failure}
      allowed-redirect-hosts:
        - localhost
        - example.com
        - www.example.com

  # Password Policy
  password:
    min-length: 8
    require-uppercase: true
    require-number: true
    require-special-char: true
    bcrypt-strength: ${BCRYPT_STRENGTH:12}         # 10 dev, 12 prod

  # Password Reset
  password-reset:
    expiry-minutes: 15
    token-length: 64

  # Email Verification
  email-verification:
    expiry-hours: 24
    token-length: 64

  # Scheduled Jobs
  cleanup:
    interval-ms: ${CLEANUP_INTERVAL_MS:3600000}   # 1 hour
    expired-token-retention-days: 1
    soft-delete-retention-days: 30

  # Multi-tenancy (uncomment if needed)
  # tenancy:
  #   mode: schema                                  # schema | database | discriminator
  #   default-schema: public

  # Feature Flags (simple — use LaunchDarkly/Unleash for advanced)
  features:
    email-verification-required: ${FEATURE_EMAIL_VERIFY:true}
    oauth2-signup-enabled: ${FEATURE_OAUTH2_SIGNUP:true}
    maintenance-mode: ${FEATURE_MAINTENANCE:false}
    new-dashboard-enabled: ${FEATURE_NEW_DASHBOARD:false}

  # API Versioning
  api:
    current-version: v1
    deprecated-versions:
      - v0
    sunset-date: "2026-12-31"                      # added to response headers

  # Circuit Breaker / Resilience (used with Resilience4j)
  resilience:
    external-service:
      failure-rate-threshold: 50
      wait-duration-in-open-state: 30s
      sliding-window-size: 10

  # Notifications (Slack webhook, PagerDuty, etc.)
  notifications:
    slack:
      webhook-url: ${SLACK_WEBHOOK_URL:}
      channel: ${SLACK_ALERT_CHANNEL:#alerts}
    pagerduty:
      integration-key: ${PAGERDUTY_KEY:}


# =============================================================================
# RESILIENCE4J — Circuit Breaker, Retry, Rate Limiter, Bulkhead
# =============================================================================
resilience4j:
  circuitbreaker:
    configs:
      default:
        sliding-window-type: count-based
        sliding-window-size: 10
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
        record-exceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
          - org.springframework.web.client.HttpServerErrorException
    instances:
      externalPaymentService:
        base-config: default
        wait-duration-in-open-state: 60s
      externalNotificationService:
        base-config: default

  retry:
    configs:
      default:
        max-attempts: 3
        wait-duration: 1s
        exponential-backoff-multiplier: 2.0
        retry-exceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
        ignore-exceptions:
          - com.example.exception.ValidationException
    instances:
      externalPaymentService:
        base-config: default
        max-attempts: 2

  ratelimiter:
    configs:
      default:
        limit-for-period: 100
        limit-refresh-period: 1s
        timeout-duration: 0s
    instances:
      externalApiService:
        base-config: default

  bulkhead:
    configs:
      default:
        max-concurrent-calls: 20
        max-wait-duration: 0ms
    instances:
      externalPaymentService:
        max-concurrent-calls: 5

  timelimiter:
    configs:
      default:
        timeout-duration: 10s
        cancel-running-future: true
    instances:
      externalPaymentService:
        timeout-duration: 30s


# =============================================================================
# KAFKA (uncomment if using Kafka instead of / alongside RabbitMQ)
# =============================================================================
# spring:
#   kafka:
#     bootstrap-servers: ${KAFKA_BROKERS:localhost:9092}
#     properties:
#       security.protocol: ${KAFKA_SECURITY_PROTOCOL:PLAINTEXT}
#       # For SASL_SSL (prod):
#       # security.protocol: SASL_SSL
#       # sasl.mechanism: PLAIN
#       # sasl.jaas.config: "org.apache.kafka.common.security.plain.PlainLoginModule required username='${KAFKA_USERNAME}' password='${KAFKA_PASSWORD}';"
#       schema.registry.url: ${KAFKA_SCHEMA_REGISTRY:http://localhost:8081}
#
#     producer:
#       key-serializer: org.apache.kafka.common.serialization.StringSerializer
#       value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
#       acks: all                       # strongest durability guarantee
#       retries: 3
#       properties:
#         enable.idempotence: true      # exactly-once semantics
#         max.in.flight.requests.per.connection: 5
#         compression.type: snappy
#
#     consumer:
#       group-id: ${spring.application.name}
#       auto-offset-reset: earliest
#       enable-auto-commit: false       # CRITICAL — manual commit for reliability
#       key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
#       value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
#       properties:
#         spring.json.trusted.packages: "com.example.*"
#         max.poll.records: 500
#         max.poll.interval.ms: 300000
#
#     listener:
#       ack-mode: manual_immediate
#       concurrency: ${KAFKA_LISTENER_CONCURRENCY:3}
#       poll-timeout: 3000


# =============================================================================
# AWS (uncomment if using AWS services)
# =============================================================================
# cloud:
#   aws:
#     region:
#       static: ${AWS_REGION:us-east-1}
#     credentials:
#       access-key: ${AWS_ACCESS_KEY_ID:}          # prefer IAM roles over keys
#       secret-key: ${AWS_SECRET_ACCESS_KEY:}
#     s3:
#       bucket: ${AWS_S3_BUCKET}
#     sqs:
#       listener:
#         auto-startup: true
#         max-number-of-messages: 10
#         visibility-timeout: 30
#         wait-time-out: 20


# =============================================================================
# FEIGN CLIENT (OpenFeign — for inter-service HTTP calls)
# =============================================================================
feign:
  client:
    config:
      default:
        connect-timeout: 5000
        read-timeout: 30000
        logger-level: basic                        # none | basic | headers | full
        retryer: com.example.config.FeignRetryer
  compression:
    request:
      enabled: true
      mime-types: application/json
      min-request-size: 2048
    response:
      enabled: true
  circuitbreaker:
    enabled: true


# =============================================================================
# OPEN FEIGN / WEBCLIENT — External Service Base URLs
# (Define your own external service URLs here for reference)
# =============================================================================
external:
  payment-service:
    base-url: ${PAYMENT_SERVICE_URL:http://payment-service}
    timeout-seconds: 30
    api-key: ${PAYMENT_SERVICE_API_KEY}
  notification-service:
    base-url: ${NOTIFICATION_SERVICE_URL:http://notification-service}
    timeout-seconds: 10
  analytics-service:
    base-url: ${ANALYTICS_SERVICE_URL:http://analytics-service}
    timeout-seconds: 5


# =============================================================================
# NOTES ON PROFILE-SPECIFIC OVERRIDES
# =============================================================================
# application-dev.yml should override:
#   spring.jpa.show-sql: true
#   spring.jpa.hibernate.ddl-auto: update
#   logging.level.com.example: DEBUG
#   logging.level.org.hibernate.SQL: DEBUG
#   management.endpoints.web.exposure.include: "*"
#   springdoc.swagger-ui.try-it-out-enabled: true
#   rate-limiter.type: memory
#
# application-staging.yml should override:
#   spring.jpa.hibernate.ddl-auto: validate
#   management.endpoint.env.show-values: when-authorized
#   app.features.maintenance-mode: false
#
# application-prod.yml should override:
#   spring.jpa.show-sql: false
#   management.endpoints.web.exposure.include: health,info,prometheus
#   management.endpoint.health.show-details: never
#   management.endpoint.env.show-values: never
#   springdoc.api-docs.enabled: false
#   springdoc.swagger-ui.enabled: false
#   server.servlet.session.cookie.secure: true
#   app.features.maintenance-mode: false
#   logging.level.root: WARN
```