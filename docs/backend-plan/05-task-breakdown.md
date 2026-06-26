# 05 — Chia Task Chi Tiết (Epic → Task)

> Mỗi task có ID `ECM-NNN` duy nhất. Cột: **SP** = Story Point, **Pri** = Priority, **Dep** = phụ thuộc (task ID), **Role** = vai trò đảm nhiệm.
> File này là bản đọc-hiểu; bản nhập công cụ quản lý dùng [backlog.csv](backlog.csv).
> Trạng thái khởi tạo: tất cả `Backlog`.

---

## EPIC-00 — Foundation & DevOps  _(Phase 0, P0)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-001 | Khởi tạo project Spring Boot 3 / Java 21 (Maven), cấu trúc package theo module domain | 3 | P0 | — | Lead |
| ECM-002 | Cấu hình profile `local/dev/staging/prod`, externalized config, không commit secret | 2 | P0 | ECM-001 | Lead |
| ECM-003 | Docker + docker-compose (PostgreSQL 16, Redis 7) cho môi trường local | 3 | P0 | ECM-001 | DevOps |
| ECM-004 | Tích hợp Flyway + migration baseline + quy ước đặt tên | 2 | P0 | ECM-003 | BE |
| ECM-005 | `GlobalExceptionHandler` + `ApiResponse<T>` envelope + bộ `ErrorCode` chuẩn | 3 | P0 | ECM-001 | Lead |
| ECM-006 | Cấu hình springdoc-openapi (Swagger UI) + nhóm tag theo module | 2 | P0 | ECM-005 | BE |
| ECM-007 | Structured logging (JSON) + filter correlation/trace id | 2 | P1 | ECM-001 | BE |
| ECM-008 | CI pipeline GitHub Actions: build + test + format + dependency scan | 3 | P0 | ECM-001 | DevOps |
| ECM-009 | `BaseEntity` (audit fields) + bật JPA Auditing | 2 | P1 | ECM-004 | BE |
| ECM-010 | Spotless + Checkstyle + cấu hình SonarQube quality gate | 2 | P1 | ECM-008 | DevOps |
| ECM-011 | Spring Boot Actuator: health/readiness/liveness + base metrics | 2 | P1 | ECM-001 | BE |

**Mục tiêu Epic:** skeleton chạy được, CI xanh, Swagger sống, chuẩn code & response thống nhất (M0).

---

## EPIC-01 — Auth & User Management  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-012 | Entity `User`, `Role`, `UserProfile` + migration | 3 | P1 | ECM-009 | BE |
| ECM-013 | API đăng ký `POST /auth/register` + hash mật khẩu (BCrypt/Argon2) | 3 | P1 | ECM-012 | BE |
| ECM-014 | API đăng nhập `POST /auth/login` + phát hành JWT access + refresh | 5 | P1 | ECM-013 | BE-Sr |
| ECM-015 | `SecurityConfig` + JWT filter + RBAC method security (`@PreAuthorize`) | 5 | P1 | ECM-014 | BE-Sr |
| ECM-016 | API `POST /auth/refresh` + lưu/thu hồi refresh token (Redis) + logout | 3 | P1 | ECM-014 | BE |
| ECM-017 | API hồ sơ `GET/PUT /users/me` | 2 | P1 | ECM-012 | BE |
| ECM-018 | CRUD sổ địa chỉ `addresses` (mặc định shipping/billing) | 3 | P1 | ECM-012 | BE |
| ECM-019 | Quên/đặt lại mật khẩu qua email token (`POST /auth/reset-password`) | 3 | P2 | ECM-013 | BE |
| ECM-020 | Seed role + tài khoản admin mặc định (migration/seed) | 1 | P1 | ECM-012 | BE |

**AC trọng tâm:** mật khẩu không lưu thô; JWT có exp ngắn (vd 15') + refresh dài; endpoint admin bị chặn nếu thiếu role; sai mật khẩu trả 401 không lộ thông tin.

---

## EPIC-02 — Catalog: Category & Brand  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-021 | Entity `Category` (parent_id self-ref) + `Brand` + migration | 3 | P1 | ECM-009 | BE |
| ECM-022 | CRUD Category (admin) + validate vòng lặp parent | 3 | P1 | ECM-021 | BE |
| ECM-023 | API cây danh mục lồng nhau (public) `GET /categories` | 2 | P1 | ECM-021 | BE |
| ECM-024 | CRUD Brand (admin) + API list brand (public) | 2 | P1 | ECM-021 | BE |
| ECM-025 | Tiện ích sinh slug + đảm bảo unique (Category/Brand/Product) | 2 | P1 | ECM-021 | BE |

---

## EPIC-03 — Catalog: Product / Image / Variant  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-026 | Entity `Product` + migration (giá, status, SEO fields) | 3 | P1 | ECM-021 | BE |
| ECM-027 | `ProductImage` + upload ảnh (MinIO/local) + ảnh chính + sort | 5 | P1 | ECM-026 | BE |
| ECM-028 | `ProductVariant` (JSONB attribute_values, SKU, price_override) + migration | 5 | P1 | ECM-026 | BE-Sr |
| ECM-029 | API tạo/sửa sản phẩm (admin) kèm ảnh + biến thể (transaction) | 5 | P1 | ECM-028 | BE-Sr |
| ECM-030 | API chi tiết sản phẩm (public) `GET /products/{id|slug}` gồm specs+variant | 3 | P1 | ECM-029 | BE |
| ECM-031 | API danh sách sản phẩm (public) phân trang + sort + filter cơ bản | 3 | P1 | ECM-026 | BE |
| ECM-032 | Ẩn/xoá mềm sản phẩm theo `status` (admin) | 2 | P1 | ECM-026 | BE |
| ECM-033 | SEO fields + canonical slug; validate SKU/slug unique | 2 | P2 | ECM-025 | BE |

---

## EPIC-04 — Attribute Template & Specs  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-034 | Entity `AttributeGroup`, `AttributeDefinition`, `AttributeOption` + migration | 5 | P1 | ECM-009 | BE-Sr |
| ECM-035 | Entity `AttributeTemplate` + `template_attributes` + `category_attribute_template` | 3 | P1 | ECM-034 | BE |
| ECM-036 | Entity `ProductAttributeValue` (value + value_number) + gán cho sản phẩm | 5 | P1 | ECM-034, ECM-026 | BE-Sr |
| ECM-037 | CRUD Attribute Template (admin): group, attribute, type, flags | 5 | P1 | ECM-035 | BE |
| ECM-038 | Gán template cho category (admin) | 2 | P1 | ECM-035 | BE |
| ECM-039 | API specs sản phẩm dạng group (cho trang chi tiết) | 3 | P1 | ECM-036 | BE |
| ECM-040 | API metadata filter theo category (filterable/comparable/quick_spec) | 3 | P1 | ECM-036 | BE |
| ECM-041 | Seed "Laptop Attribute Template" đầy đủ (CPU/RAM/SSD/Display/GPU/Warranty...) | 3 | P1 | ECM-037 | BE |

**Ghi chú:** đây là phần đặc thù ngành điện tử, nằm gần đường găng giá trị; làm sớm và có thể cần 1 spike thiết kế EAV trước khi code.

---

## EPIC-05 — Inventory Management  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-042 | Entity `Warehouse`, `InventoryItem` (`@Version`), `StockMovement` + migration | 3 | P1 | ECM-028 | BE |
| ECM-043 | API truy vấn tồn kho theo sản phẩm/biến thể | 2 | P1 | ECM-042 | BE |
| ECM-044 | **Reserve tồn kho atomic + optimistic lock + retry** (chống oversell) | 8 | P0 | ECM-042 | BE-Sr |
| ECM-045 | Release tồn kho (huỷ đơn / thanh toán fail / hết hạn giữ) | 3 | P1 | ECM-044 | BE-Sr |
| ECM-046 | Nhập/điều chỉnh tồn kho (admin) + ghi `StockMovement` | 3 | P1 | ECM-042 | BE |
| ECM-047 | Low-stock threshold + cờ cảnh báo sắp hết hàng | 2 | P2 | ECM-042 | BE |

**AC [ECM-044]:** test đồng thời N request mua sản phẩm còn 1 → đúng 1 thành công, còn lại nhận `409 INSUFFICIENT_STOCK`; mọi thay đổi có movement log.

---

## EPIC-06 — Cart  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-048 | Mô hình Cart/CartItem: guest (Redis + TTL) và user (DB) | 5 | P1 | ECM-030, ECM-015 | BE-Sr |
| ECM-049 | API `GET /cart` (item, tạm tính, thuế, phí ship ước tính) | 3 | P1 | ECM-048 | BE |
| ECM-050 | API thêm item `POST /cart/items` (validate tồn kho, variant) | 3 | P1 | ECM-048 | BE |
| ECM-051 | API cập nhật số lượng `PATCH /cart/items/{id}` | 2 | P1 | ECM-048 | BE |
| ECM-052 | API xoá item `DELETE /cart/items/{id}` | 1 | P1 | ECM-048 | BE |
| ECM-053 | Merge giỏ guest → user khi đăng nhập | 3 | P1 | ECM-048 | BE |

---

## EPIC-07 — Promotion & Pricing  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-054 | Entity `Coupon` + `coupon_redemptions` + migration | 3 | P1 | ECM-009 | BE |
| ECM-055 | Entity `Promotion` + `promotion_products` + migration | 3 | P2 | ECM-026 | BE |
| ECM-056 | API áp mã giảm giá `POST /coupons/apply` + bộ rule validate (min order, hạn, usage) | 5 | P1 | ECM-054 | BE-Sr |
| ECM-057 | **Pricing Engine**: tính subtotal/discount/shipping/tax/total + thứ tự ưu tiên KM | 5 | P1 | ECM-056 | BE-Sr |
| ECM-058 | API danh sách khuyến mãi hiện hành (public) | 2 | P2 | ECM-055 | BE |
| ECM-059 | CRUD coupon/promotion (admin) | 3 | P1 | ECM-054 | BE |

---

## EPIC-08 — Order Management  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-060 | Entity `Order`, `OrderItem`, `OrderStatusHistory` + migration | 3 | P1 | ECM-009, ECM-026 | BE |
| ECM-061 | **Tạo đơn từ giỏ** (transaction: reserve tồn kho + snapshot giá + áp coupon + tính total) | 8 | P0 | ECM-044, ECM-057, ECM-048 | BE-Sr |
| ECM-062 | API chi tiết đơn `GET /orders/{id}` | 2 | P1 | ECM-060 | BE |
| ECM-063 | API lịch sử đơn của khách `GET /orders?customer_id=me` | 2 | P1 | ECM-060 | BE |
| ECM-064 | Cập nhật trạng thái đơn (admin) + **state machine** hợp lệ + ghi history | 5 | P1 | ECM-060 | BE-Sr |
| ECM-065 | Huỷ đơn + release tồn kho + ghi lý do | 3 | P1 | ECM-064, ECM-045 | BE |
| ECM-066 | Bộ sinh `order_number` (unique, dễ đọc) | 2 | P1 | ECM-060 | BE |
| ECM-067 | Idempotency cho `POST /orders` (header `Idempotency-Key`) | 3 | P1 | ECM-061 | BE-Sr |

**AC [ECM-061]:** thất bại ở bất kỳ bước nào → rollback toàn bộ (không tạo đơn treo, không trừ kho); giá lưu snapshot bất biến.

---

## EPIC-09 — Payment (COD + Chuyển khoản)  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-068 | Entity `Payment` + abstraction `PaymentMethod` (Strategy) + migration | 3 | P1 | ECM-060 | BE-Sr |
| ECM-069 | Luồng thanh toán COD (tạo payment pending, xác nhận khi giao) | 2 | P1 | ECM-068 | BE |
| ECM-070 | Luồng chuyển khoản (hiển thị STK/nội dung/QR, hạn giữ đơn) | 3 | P1 | ECM-068 | BE |
| ECM-071 | Admin xác nhận thanh toán + cập nhật `payment_status` | 2 | P1 | ECM-068 | BE |
| ECM-072 | Liên kết kết quả thanh toán ↔ Order (cập nhật trạng thái, outbox cơ bản) | 3 | P1 | ECM-068, ECM-064 | BE-Sr |

---

## EPIC-10 — Admin API & Dashboard  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-073 | API danh sách đơn (admin) + filter (status, ngày, khách, phone) | 3 | P1 | ECM-060 | BE |
| ECM-074 | API danh sách sản phẩm (admin) + filter (category, brand, status, stock, giá) | 3 | P1 | ECM-031 | BE |
| ECM-075 | API metric dashboard (doanh thu, đơn mới, đơn chờ, sắp hết hàng, bán chạy) | 5 | P1 | ECM-060 | BE-Sr |
| ECM-076 | Phân quyền chi tiết endpoint admin (ADMIN/MANAGER/SUPPORT) | 3 | P1 | ECM-015 | BE-Sr |
| ECM-077 | Audit log hành động admin (cập nhật đơn, sửa giá, điều chỉnh kho) | 3 | P2 | ECM-064 | BE |

---

## EPIC-11 — Search & Filter (DB-based)  _(Phase 1, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-078 | Lọc động theo thuộc tính bằng JPA Specification/Criteria | 5 | P1 | ECM-040 | BE-Sr |
| ECM-079 | Tìm kiếm full-text MVP bằng PostgreSQL tsvector + index GIN | 3 | P1 | ECM-026 | BE |
| ECM-080 | Đếm facet (số lượng theo brand/giá/thuộc tính) | 3 | P2 | ECM-078 | BE |
| ECM-081 | Gợi ý autocomplete cơ bản (tên/brand/category) | 2 | P2 | ECM-079 | BE |

---

## EPIC-12 — Search nâng cao (Elasticsearch)  _(Phase 2, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-082 | Hạ tầng Elasticsearch 8 + định nghĩa index mapping sản phẩm | 5 | P2 | ECM-079 | BE-Sr |
| ECM-083 | Indexer đồng bộ sản phẩm khi thay đổi (qua outbox) | 5 | P2 | ECM-082 | BE-Sr |
| ECM-084 | API search/facet qua ES (full-text + filter thuộc tính) | 5 | P2 | ECM-083 | BE |
| ECM-085 | Job reindex toàn bộ + cơ chế alias zero-downtime | 3 | P2 | ECM-083 | BE |

---

## EPIC-13 — Cổng thanh toán online  _(Phase 2, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-086 | Tích hợp VNPay (tạo URL, return_url) qua Strategy | 5 | P2 | ECM-068 | BE-Sr |
| ECM-087 | Tích hợp MoMo | 5 | P2 | ECM-068 | BE-Sr |
| ECM-088 | Xử lý webhook/IPN + xác thực chữ ký HMAC + idempotent | 5 | P1 | ECM-086 | BE-Sr |
| ECM-089 | Luồng hoàn tiền (refund) + cập nhật trạng thái | 3 | P2 | ECM-088 | BE |

---

## EPIC-14 — Reviews & Ratings  _(Phase 2, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-090 | Entity `Review` + API tạo đánh giá | 3 | P2 | ECM-060 | BE |
| ECM-091 | API danh sách review + tổng hợp rating (trung bình, phân bố sao) | 3 | P2 | ECM-090 | BE |
| ECM-092 | Kiểm duyệt review (admin) | 2 | P2 | ECM-090 | BE |
| ECM-093 | Kiểm tra "đã mua hàng" (verified purchase) | 2 | P3 | ECM-090 | BE |

---

## EPIC-15 — Notification (Async)  _(Phase 2, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-094 | Hạ tầng RabbitMQ + cấu hình consumer/producer | 3 | P2 | ECM-072 | DevOps |
| ECM-095 | Dịch vụ email (SendGrid/SMTP) + hệ thống template | 5 | P2 | ECM-094 | BE |
| ECM-096 | Dịch vụ SMS (Twilio) — tuỳ chọn | 3 | P3 | ECM-094 | BE |
| ECM-097 | Sự kiện hoá thông báo (order placed, status changed) qua **Outbox dispatcher** | 5 | P2 | ECM-094, ECM-072 | BE-Sr |
| ECM-098 | Log gửi thông báo + retry/dead-letter | 3 | P2 | ECM-095 | BE |

---

## EPIC-16 — Shipping Integration  _(Phase 2, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-099 | Engine tính phí ship theo khu vực/khối lượng | 3 | P2 | ECM-060 | BE |
| ECM-100 | Tích hợp hãng vận chuyển (GHN/GHTK): tạo vận đơn | 5 | P2 | ECM-099 | BE-Sr |
| ECM-101 | Đồng bộ trạng thái giao hàng (tracking) | 3 | P2 | ECM-100 | BE |

---

## EPIC-17 — Analytics & Reporting  _(Phase 2, P3)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-102 | Thu thập sự kiện (lượt xem, chuyển đổi) | 3 | P3 | ECM-030 | BE |
| ECM-103 | Job tổng hợp báo cáo (doanh thu, tồn kho, bán chạy) | 5 | P3 | ECM-102 | BE |
| ECM-104 | API báo cáo cho admin + export CSV | 3 | P3 | ECM-103 | BE |

---

## EPIC-18 — Personalization  _(Phase 2, P3)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-105 | Wishlist (thêm/xoá/list) | 3 | P3 | ECM-030 | BE |
| ECM-106 | API so sánh sản phẩm (compare theo thuộc tính comparable) | 3 | P2 | ECM-040 | BE |
| ECM-107 | Sản phẩm đã xem gần đây (recently viewed) | 2 | P3 | ECM-030 | BE |
| ECM-108 | Gợi ý sản phẩm cơ bản (cùng category/brand/tầm giá) | 3 | P3 | ECM-030 | BE |

---

## EPIC-19 — Security Hardening  _(xuyên suốt, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-109 | Rate limiting (login, API nhạy cảm) bằng Bucket4j + Redis | 3 | P1 | ECM-016 | BE-Sr |
| ECM-110 | Hardening validation input + chống OWASP Top 10 (SQLi, XSS, IDOR) | 3 | P1 | ECM-015 | BE-Sr |
| ECM-111 | Quản lý secret + cấu hình HTTPS/TLS + security headers | 2 | P1 | ECM-002 | DevOps |
| ECM-112 | Xử lý PII + audit + chuẩn bị tuân thủ GDPR/PCI (không lưu thẻ thô) | 3 | P2 | ECM-012 | BE-Sr |
| ECM-113 | Quét bảo mật định kỳ (dependency/Trivy) + chuẩn bị pen-test | 2 | P2 | ECM-008 | DevOps |

---

## EPIC-20 — Performance & Observability  _(xuyên suốt, P2)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-114 | Cache Redis cho catalog/category + chiến lược invalidation | 5 | P2 | ECM-030 | BE-Sr |
| ECM-115 | Rà soát & tối ưu index + query (N+1, slow query) | 3 | P2 | ECM-031 | BE-Sr |
| ECM-116 | Metrics Micrometer → Prometheus + dashboard Grafana | 3 | P2 | ECM-011 | DevOps |
| ECM-117 | Distributed tracing (OpenTelemetry) | 3 | P3 | ECM-007 | DevOps |
| ECM-118 | Load test (k6/Gatling) cho luồng mua hàng + báo cáo | 5 | P2 | ECM-061 | QA |

---

## EPIC-21 — Testing & Release  _(xuyên suốt, P1)_

| ID | Task | SP | Pri | Dep | Role |
|---|---|---:|---|---|---|
| ECM-119 | Thiết lập nền test (JUnit5/Mockito) + ngưỡng coverage trong CI | 3 | P1 | ECM-008 | QA |
| ECM-120 | Integration test (Testcontainers) cho luồng đặt hàng + đồng thời tồn kho | 5 | P1 | ECM-061 | QA |
| ECM-121 | Contract test API khớp OpenAPI | 3 | P2 | ECM-006 | QA |
| ECM-122 | Tự động hoá E2E happy path (search→cart→order→payment) | 5 | P2 | ECM-072 | QA |
| ECM-123 | Triển khai Staging + chạy UAT + checklist nghiệm thu | 3 | P1 | ECM-122 | DevOps |
| ECM-124 | Release production + runbook + rollback plan | 3 | P1 | ECM-123 | Lead |

---

## Tổng hợp

- **Tổng số task:** 124 (ECM-001 → ECM-124).
- **Tổng Story Point (ước lượng):** ~405 SP.
- **Phase 1 (MVP):** EPIC-00 → EPIC-11 + phần cross-cutting tối thiểu — mục tiêu M1.
- **Phase 2:** EPIC-12 → EPIC-18 + chiều sâu cross-cutting — mục tiêu M2.

> Để nhập vào Jira/Linear/Azure DevOps: dùng [backlog.csv](backlog.csv) (đầy đủ mô tả, AC, dependency, estimate, phase/sprint).
