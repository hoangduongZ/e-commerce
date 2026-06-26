# 00 — Tổng Quan Giai Đoạn Phát Triển Backend

> Người soạn: Tech Lead (định hướng theo kinh nghiệm vận hành hệ thống quy mô doanh nghiệp).
> Cơ sở: [system-design.md](../main/system-design.md) + [01-electronics-store-theme.md](../theme/01-electronics-store-theme.md).

---

## 1. Bối cảnh

Hệ thống là website thương mại điện tử **chuyên ngành đồ điện tử** (laptop, điện thoại, màn hình, phụ kiện, linh kiện...). Đặc thù ngành hàng đặt ra yêu cầu backend riêng so với e-commerce phổ thông:

- **Thuộc tính sản phẩm động & phong phú** (CPU, RAM, SSD, tần số quét...) → cần hệ thống *Attribute Template* thay vì hard-code cột.
- **Biến thể (variant)** theo RAM/Storage/Color, mỗi biến thể có SKU, giá, tồn kho riêng.
- **So sánh & lọc theo thông số** → dữ liệu spec phải có cấu trúc (filterable/comparable/quick-spec).
- **Giá trị đơn hàng cao** → yêu cầu cao về tính nhất quán tồn kho, chống bán âm, bảo mật thanh toán, bảo hành.

Giai đoạn này tập trung **xây dựng Backend** (API, nghiệp vụ, dữ liệu). Frontend (NuxtJS) đã có spec riêng và sẽ tiêu thụ API theo hợp đồng (contract) do backend cung cấp.

---

## 2. Mục tiêu

### 2.1. Mục tiêu sản phẩm
- Cung cấp đầy đủ API cho luồng mua hàng MVP: duyệt → lọc → xem chi tiết → chọn biến thể → giỏ hàng → đặt hàng → thanh toán (COD/chuyển khoản) → theo dõi đơn.
- Cung cấp API quản trị (Admin) cho sản phẩm, attribute template, tồn kho, đơn hàng, khuyến mãi.

### 2.2. Mục tiêu kỹ thuật (Non-functional)
| Tiêu chí | Mục tiêu MVP |
|---|---|
| Thời gian phản hồi API (p95) | < 300ms cho đọc, < 800ms cho ghi |
| Tính nhất quán tồn kho | Không oversell (kiểm soát qua reserve + khoá lạc quan/bi quan) |
| Bảo mật | JWT + RBAC, BCrypt/Argon2, rate-limit login, HTTPS, không lưu thẻ thô |
| Test coverage | ≥ 70% domain/service, integration test cho luồng đặt hàng |
| Khả năng mở rộng | Modular monolith, tách module sạch để dễ tách microservice sau |
| Observability | Health check, metrics, structured logging, trace id |

---

## 3. Phạm vi (Scope)

### 3.1. Trong phạm vi giai đoạn này
- Toàn bộ REST API backend (versioned `/api/v1`).
- Mô hình dữ liệu + migration.
- Cơ chế Auth/RBAC.
- Pricing & promotion engine cơ bản.
- Tích hợp Redis (cache + guest cart), PostgreSQL.
- CI/CD pipeline, Docker hoá.

### 3.2. Ngoài phạm vi (giai đoạn sau / team khác)
- Giao diện Frontend (NuxtJS) — team FE.
- Cổng thanh toán online thật (VNPay/MoMo) — Phase 2.
- Elasticsearch — Phase 2 (MVP dùng PostgreSQL full-text + filter).
- Mobile app, BI dashboard nâng cao.

### 3.3. MVP ngành hàng
Theo khuyến nghị của theme: **bắt đầu với "Laptop Store MVP"** vì laptop có đủ độ phức tạp (danh mục, thương hiệu, thông số, biến thể, bảo hành, tồn kho, giá sale, so sánh). Khi ổn định sẽ clone template sang điện thoại/màn hình/phụ kiện chỉ bằng việc đổi *Attribute Template*.

---

## 4. Phương pháp luận & nhịp làm việc

- **Mô hình:** Scrum, sprint 2 tuần.
- **Nghi thức:** Sprint Planning, Daily Standup, Sprint Review, Retrospective, Backlog Refinement giữa sprint.
- **Đơn vị backlog:** Epic → Story/Task (mỗi task có ID `ECM-NNN`).
- **Velocity giả định:** ~25–30 SP/sprint cho team 3 backend + 1 lead (điều chỉnh sau Sprint 1–2).
- **Mọi task** phải đạt **Definition of Ready** trước khi vào sprint và **Definition of Done** trước khi đóng (xem [03-quy-trinh-phat-trien.md](03-quy-trinh-phat-trien.md)).

---

## 5. Cơ cấu nhân sự đề xuất (RACI rút gọn)

| Vai trò | Số lượng | Trách nhiệm chính |
|---|---|---|
| Tech Lead | 1 | Kiến trúc, review, chuẩn code, gỡ blocker, quyết định ADR |
| Backend Engineer | 3 | Hiện thực Epic/Task, viết test, sửa bug |
| DevOps (part-time) | 1 | CI/CD, Docker, môi trường, monitoring |
| QA Engineer | 1 | Test plan, integration/e2e, regression |
| Product Owner | 1 | Ưu tiên backlog, làm rõ yêu cầu, nghiệm thu |

---

## 6. Rủi ro chính & biện pháp

| Rủi ro | Ảnh hưởng | Biện pháp |
|---|---|---|
| Oversell tồn kho khi nhiều người mua cùng lúc | Cao | Reserve atomic + optimistic lock + test đồng thời ([ECM-044]) |
| Mô hình attribute động phức tạp, làm chậm dev | Trung bình | Làm sớm ở Sprint 2, seed template laptop, có spike ([ECM-034..041]) |
| Tích hợp cổng thanh toán trễ | Trung bình | MVP chỉ COD + chuyển khoản; thiết kế Payment theo Strategy để cắm gateway sau |
| Tính nhất quán giữa Order/Payment/Inventory | Cao | Transaction trong monolith + Outbox pattern chuẩn bị cho async |
| Hiệu năng tìm kiếm/lọc theo thuộc tính | Trung bình | MVP dùng JPA Specification + index; Phase 2 chuyển Elasticsearch |

---

## 7. Tiêu chí thành công của giai đoạn

- [ ] Toàn bộ API MVP chạy được end-to-end (đặt hàng COD thành công, tồn kho trừ đúng).
- [ ] Có Swagger/OpenAPI đầy đủ cho team FE.
- [ ] CI xanh, coverage đạt ngưỡng, không lỗ hổng bảo mật nghiêm trọng (dependency scan).
- [ ] Triển khai được lên môi trường Staging và qua UAT.
