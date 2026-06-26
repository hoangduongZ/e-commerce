# 01 — Kiến Trúc Hệ Thống & Tech Stack

---

## 1. Lựa chọn kiến trúc tổng thể

### Quyết định: **Modular Monolith** (không phải microservices ngay từ đầu)

Hệ thống `system-design.md` mô tả nhiều "service" theo domain. Tuy nhiên ở giai đoạn MVP, triển khai **microservices thật** (mỗi service 1 DB, network call, saga phân tán) sẽ làm chậm tiến độ và tăng chi phí vận hành không cần thiết.

**Chiến lược:** Xây **Modular Monolith** — một ứng dụng Spring Boot, nhưng chia **module nội bộ rõ ràng theo domain**, mỗi module có ranh giới (package boundary), giao tiếp qua interface/application service, không gọi chéo repository của nhau. Khi một domain cần scale riêng (vd. Search, Notification), ta tách thành service độc lập mà không phải viết lại nghiệp vụ.

> Nguyên tắc: "Modules first, services later." Ranh giới domain đúng quan trọng hơn việc tách process sớm.

### Phân tầng (Layered)
```
┌─────────────────────────────────────────────┐
│  API Layer (Controller, DTO, exception map)  │  REST /api/v1
├─────────────────────────────────────────────┤
│  Application Layer (Service, orchestration,  │  @Transactional, use-case
│   pricing engine, saga/outbox)               │
├─────────────────────────────────────────────┤
│  Domain Layer (Entity, domain rules, VO)     │  business invariants
├─────────────────────────────────────────────┤
│  Infrastructure (Repository/JPA, Redis,      │
│   mail, payment gateway, search client)      │
└─────────────────────────────────────────────┘
```

---

## 2. Tech Stack

| Lớp | Công nghệ | Lý do |
|---|---|---|
| Ngôn ngữ | **Java 21** (LTS) | Records, pattern matching, virtual threads |
| Framework | **Spring Boot 3.3+** | Chuẩn doanh nghiệp, hệ sinh thái đầy đủ |
| Web | Spring Web (MVC) | REST API |
| Bảo mật | Spring Security + **JWT** (jjwt/nimbus) | Auth + RBAC |
| Truy cập DB | Spring Data JPA + Hibernate | ORM, repository |
| Query động | JPA Specification / Criteria | Filter sản phẩm theo thuộc tính |
| Migration | **Flyway** | Versioned migration, audit DB |
| CSDL chính | **PostgreSQL 16** | JSONB (specs/variant), full-text, ACID |
| Cache / Guest cart | **Redis 7** | Cache catalog, giỏ guest, rate-limit token |
| Tìm kiếm | PostgreSQL FTS (MVP) → **Elasticsearch 8** (Phase 2) | Full-text + facet |
| Message Queue | **RabbitMQ** (Phase 2) | Async: email, index, outbox |
| Mapping | MapStruct | Entity ↔ DTO |
| Boilerplate | Lombok | Giảm code lặp |
| API Docs | **springdoc-openapi** (Swagger UI) | Hợp đồng API cho FE |
| Validation | Jakarta Bean Validation | Ràng buộc input |
| Lưu trữ file | MinIO (S3-compatible) / local (dev) | Ảnh sản phẩm |
| Test | JUnit 5, Mockito, **Testcontainers**, RestAssured | Unit + integration |
| Build | **Maven** | Phổ biến trong enterprise Spring |
| Container | Docker + docker-compose | Môi trường nhất quán |
| CI/CD | GitHub Actions | Build/test/scan/deploy |
| Observability | Actuator + Micrometer + Prometheus + Grafana | Metrics, health |
| Chất lượng | Spotless + Checkstyle, SonarQube, OWASP Dependency-Check | Chuẩn code, bảo mật |

---

## 3. Cấu trúc package (theo module domain)

```
com.electrostore
├── EcommerceApplication.java
├── common/                      # cross-cutting
│   ├── config/                  # SecurityConfig, OpenApiConfig, RedisConfig...
│   ├── exception/               # GlobalExceptionHandler, BusinessException, ErrorCode
│   ├── response/                # ApiResponse<T>, PageResponse<T>
│   ├── audit/                   # BaseEntity (createdAt, updatedAt, createdBy)
│   └── util/                    # SlugUtil, IdGenerator
├── iam/                         # Identity & Access (User, Auth, Role, Address)
│   ├── api/  domain/  app/  infra/
├── catalog/                     # Category, Brand, Product, Image, Variant
├── attribute/                   # Attribute, AttributeTemplate, Spec, ProductAttributeValue
├── inventory/                   # InventoryItem, StockMovement, reserve/release
├── cart/                        # Cart, CartItem (Redis + DB)
├── promotion/                   # Coupon, Promotion, PricingEngine
├── order/                       # Order, OrderItem, OrderStatusHistory, state machine
├── payment/                     # Payment, PaymentMethod strategy, gateway adapters
├── search/                      # search/filter service (JPA → ES)
├── admin/                       # admin-facing orchestration + dashboard metrics
└── notification/                # email/sms (Phase 2), outbox dispatcher
```

Mỗi module con dùng layout `api / app / domain / infra`:
- `api`: Controller + Request/Response DTO.
- `app`: Application Service (use-case), giao dịch.
- `domain`: Entity, domain service, value object, rule nghiệp vụ.
- `infra`: Repository (JPA), adapter ngoài.

**Rule ranh giới:** module A không được `@Autowired` repository của module B. Muốn dùng dữ liệu B → gọi qua application service / interface công khai của B.

---

## 4. Hợp đồng API (API contract conventions)

- Base path: `/api/v1`. Tăng version khi breaking change.
- Envelope thống nhất:
  ```json
  { "success": true, "data": { ... }, "error": null, "meta": { "page": 1, "totalPages": 5 } }
  ```
  Lỗi:
  ```json
  { "success": false, "data": null, "error": { "code": "PRODUCT_NOT_FOUND", "message": "..." } }
  ```
- HTTP status chuẩn: 200/201/204, 400/401/403/404/409/422, 500.
- Phân trang: `?page=0&size=20&sort=price,asc`.
- Mọi endpoint admin yêu cầu role (`ADMIN`/`MANAGER`/`SUPPORT`) qua `@PreAuthorize`.
- Idempotency key cho `POST /orders` (header `Idempotency-Key`).

---

## 5. Architecture Decision Records (ADR) tóm tắt

| ID | Quyết định | Lý do | Hệ quả |
|---|---|---|---|
| **ADR-01** | Modular monolith thay vì microservices | Tốc độ MVP, giảm chi phí vận hành | Phải giữ ranh giới module nghiêm ngặt |
| **ADR-02** | PostgreSQL làm CSDL chính | ACID, JSONB cho spec/variant, FTS sẵn có | Hạn chế phụ thuộc DB engine khác |
| **ADR-03** | Attribute Template động (EAV có kiểm soát) | Ngành điện tử có thuộc tính đa dạng theo category | Cần index/cache để truy vấn nhanh |
| **ADR-04** | Reserve tồn kho + optimistic locking (`@Version`) | Chống oversell mà vẫn throughput cao | Phải xử lý retry khi xung đột version |
| **ADR-05** | Payment theo Strategy + Outbox | Cắm thêm gateway (VNPay/MoMo) không sửa Order | Cần bảng outbox + dispatcher |
| **ADR-06** | JWT access (ngắn) + refresh token (Redis) | Stateless, thu hồi được token | Quản lý vòng đời refresh token |
| **ADR-07** | MVP search bằng PostgreSQL, ES ở Phase 2 | Tránh hạ tầng phức tạp sớm | Có abstraction `SearchService` để thay backend |
| **ADR-08** | Flyway versioned migration | Audit & rollback schema có kiểm soát | Cấm sửa migration đã merge |

> Mỗi ADR sẽ có file chi tiết riêng `docs/backend-plan/adr/ADR-NN.md` khi đi vào hiện thực (tham chiếu trong task tương ứng).

---

## 6. Môi trường

| Môi trường | Mục đích | DB | Triển khai |
|---|---|---|---|
| `local` | Dev máy cá nhân | docker-compose (PG+Redis) | `mvn spring-boot:run` |
| `dev` | Tích hợp liên tục | shared dev DB | auto-deploy nhánh `develop` |
| `staging` | UAT, giống prod | staging DB | deploy từ `release/*` |
| `prod` | Production | managed PostgreSQL | deploy từ tag `v*` |

Cấu hình tách theo profile (`application-{env}.yml`), secret qua biến môi trường / vault — **không commit secret**.
