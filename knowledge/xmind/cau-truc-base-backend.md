# ElectroStore Backend Base (EPIC-00)

## Stack đã chốt

- Java 21 + Spring Boot 3.5.3 (Maven wrapper ./mvnw)
- PostgreSQL 16 — database chính
- Redis 7 — cache, guest cart sau này
- Spring Data JPA + Hibernate — ORM
- Flyway — schema migration
- springdoc-openapi 2.8.6 — Swagger UI
- Lombok + MapStruct 1.6.3 — boilerplate, mapping
- JUnit 5 + Mockito + Testcontainers — test
- Actuator + Micrometer/Prometheus — observability
- Spotless + Checkstyle + JaCoCo — quality
- GitHub Actions — CI

## Kiến trúc Modular Monolith

- Một app Spring Boot duy nhất, chia module theo domain
- Nguyên tắc: modules first, services later
- 11 module domain
  - iam — User, Role, Auth, Address
  - catalog — Category, Brand, Product, Variant
  - attribute — EAV có kiểm soát
  - inventory — tồn kho, reserve/release
  - cart — giỏ hàng
  - promotion — Coupon, PricingEngine
  - order — Order, state machine
  - payment — strategy, gateway adapter
  - search — PostgreSQL FTS, sau này Elasticsearch
  - admin — orchestration trang quản trị
  - notification — email/SMS Phase 2
- Layout 4 tầng trong mỗi module
  - api — Controller, Request/Response DTO
  - app — Application Service, transaction boundary
  - domain — Entity, Value Object, domain rule
  - infra — Repository, adapter ngoài
- Boundary rule bắt buộc
  - Module A không inject repository của module B
  - Muốn dùng B thì gọi qua application service của B
  - Hiện tại mỗi module chỉ có package-info.java, sub-package tạo khi có class đầu tiên

## Common layer (com.electrostore.common)

- response
  - ApiResponse — envelope success/data/error/meta
  - PageResponse — content, page, size, totalElements, totalPages
  - ErrorResponse — code, message, details theo field
- exception
  - ErrorCode enum — 12 code nền tảng, gắn sẵn HTTP status
  - BusinessException — exception nghiệp vụ chung
  - NotFoundException — 404 RESOURCE_NOT_FOUND
  - GlobalExceptionHandler — map mọi exception ra envelope, không lộ stacktrace
- audit
  - BaseEntity — id BIGINT identity, createdAt, updatedAt, createdBy, updatedBy
  - AuditorAwareImpl — tạm return "system", TODO lấy từ SecurityContext ở ECM-015
- logging
  - CorrelationIdFilter — nhận hoặc sinh X-Correlation-ID, đưa vào MDC
  - CorrelationIdHolder — hằng số + truy cập MDC
- util
  - SlugUtil — slug tiếng Việt có dấu thành ascii
- config
  - OpenApiConfig — Swagger info, sẵn chỗ GroupedOpenApi theo module
  - JpaAuditingConfig — bật JPA Auditing
  - WebConfig — CORS theo profile
  - RedisConfig — RedisTemplate key String, value JSON

## API conventions

- Base path /api/v1, admin dùng /api/v1/admin
- Envelope thống nhất mọi response
  - Success: success true, data, error null, meta
  - Error: success false, data null, error có code/message/details
- error.code là SCREAMING_SNAKE_CASE ổn định, FE map theo code
- HTTP status chuẩn: 200/201/204, 400/401/403/404/405/409/422, 500
- Validation lỗi trả 422 kèm details theo field
- Phân trang: page 0-based, items trong data, thông tin trang trong meta
- Nguồn sự thật: docs/main/api-conventions.md

## Config và profiles

- application.yml — config chung, default chỉ cho local
- local (default) — credential trùng docker-compose, log DEBUG + SQL
- dev — DB/Redis bắt buộc qua env, fail fast nếu thiếu
- staging — như dev, log INFO, Swagger vẫn bật cho UAT
- prod — Swagger tắt, Actuator chỉ health + prometheus
- Không commit secret, mọi credential ngoài local qua env variable

## Hạ tầng local (docker-compose.yml)

- postgres:16-alpine — db electrostore, healthcheck pg_isready
- redis:7-alpine — appendonly, healthcheck redis-cli ping
- Host port override qua .env (gitignored)
  - Máy hiện tại: POSTGRES_HOST_PORT=5435 vì 5432 bị dự án khác chiếm
- OrbStack cần api.version=1.44 trong ~/.docker-java.properties cho Testcontainers

## Flyway migration

- Thư mục src/main/resources/db/migration
- Tên file VyyyyMMddNNNN__mo_ta_ngan.sql
- Baseline: V202607030001__baseline.sql, chưa có business table
- Luật
  - Migration đã merge không được sửa
  - Đổi schema thì tạo migration mới
  - PR đổi schema phải kèm migration + entity + test
  - ddl-auto validate, Hibernate không tự sửa schema

## Logging và correlation id

- X-Correlation-ID: nhận từ client nếu hợp lệ, không thì sinh UUID
- Chặn header không hợp lệ chống log injection
- Correlation id nằm trong MDC của mọi dòng log, trả lại ở response header
- local log text dễ đọc, môi trường khác log JSON (logstash encoder)
- Không log password, token, thẻ, PII

## Observability (Actuator)

- /actuator/health kèm liveness + readiness probes
- /actuator/prometheus cho metrics (local/dev/staging)
- prod chỉ expose health + prometheus

## Test foundation

- Unit test *Test — surefire, không cần Docker, chạy bằng ./mvnw test
- Integration test *IT — failsafe, cần Docker, chạy bằng ./mvnw verify
- TestcontainersConfig — PG16 + Redis 7, @ServiceConnection tự nối config
- AbstractIntegrationTest — base class, các IT dùng chung context/container
- EnvelopeTestController — controller chỉ trong test sources để verify envelope
- Đã pass: 7 unit + 8 integration

## Code quality và CI

- Spotless palantir-java-format — ./mvnw spotless:apply
- Checkstyle rule cơ bản — config/checkstyle/checkstyle.xml
- JaCoCo coverage report — Sonar-ready
- CI GitHub Actions
  - Job build: JDK 21 + cache Maven + ./mvnw verify + upload JaCoCo
  - Job security: Trivy scan, fail khi CVE CRITICAL/HIGH có fix

## Mapping EPIC-00 (đã xong cả 11 task)

- ECM-001 — project + cấu trúc package
- ECM-002 — profiles + externalized config
- ECM-003 — docker-compose PG16 + Redis 7
- ECM-004 — Flyway baseline + quy ước
- ECM-005 — envelope + ErrorCode + handler
- ECM-006 — Swagger (GroupedOpenApi chờ controller thật)
- ECM-007 — structured logging + correlation id
- ECM-008 — CI GitHub Actions
- ECM-009 — BaseEntity + JPA Auditing
- ECM-010 — Spotless + Checkstyle + JaCoCo
- ECM-011 — Actuator health/liveness/readiness

## Bước tiếp theo

- ECM-012 — entity User, Role, UserProfile + migration (mở đầu EPIC-01 Auth)
- Khi thêm module mới: package-info trước, sub-package khi có class
- Khi thêm endpoint: DTO validate, ApiResponse.ok, logic ở app layer
- AuditorAware lấy user thật sau khi có JWT (ECM-015)
