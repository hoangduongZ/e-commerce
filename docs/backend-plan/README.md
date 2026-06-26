# Kế Hoạch Triển Khai Backend — E-Commerce Đồ Điện Tử

> Tài liệu lập kế hoạch giai đoạn **phát triển Backend** cho hệ thống thương mại điện tử ngành đồ điện tử.
> Công nghệ chủ đạo: **Spring Boot 3 / Java 21**.
> Cách tổ chức và chia task tuân theo **flow chuẩn doanh nghiệp** (Epic → Story/Task có ID riêng, có Definition of Ready / Definition of Done, code review, CI/CD).

---

## 1. Bộ tài liệu này gồm những gì

| File | Nội dung | Dành cho |
|---|---|---|
| [README.md](README.md) | Mục lục, quy ước ID, quy ước trạng thái | Tất cả |
| [00-tong-quan.md](00-tong-quan.md) | Bối cảnh, mục tiêu, phạm vi, team, phương pháp luận | PM, Lead, stakeholder |
| [01-kien-truc-tech-stack.md](01-kien-truc-tech-stack.md) | Kiến trúc hệ thống, tech stack, các quyết định kiến trúc (ADR), cấu trúc package | Lead, Backend |
| [02-database-design.md](02-database-design.md) | Mô hình dữ liệu, chiến lược migration (Flyway), index, xử lý đồng thời (tồn kho) | Backend, DBA |
| [03-quy-trinh-phat-trien.md](03-quy-trinh-phat-trien.md) | Git flow, vòng đời ticket, DoR/DoD, code review, CI/CD, môi trường | Tất cả engineer |
| [04-lo-trinh-sprint.md](04-lo-trinh-sprint.md) | Lộ trình theo phase/sprint, milestone, bản đồ Epic, đồ thị phụ thuộc | PM, Lead |
| [05-task-breakdown.md](05-task-breakdown.md) | Chia task chi tiết theo Epic, mỗi task có ID, mô tả, AC, phụ thuộc, estimate | Backend, QA |
| [backlog.csv](backlog.csv) | Toàn bộ backlog dạng bảng, import thẳng vào Jira/Linear/Azure DevOps | PM, Lead |

**Thứ tự đọc khuyến nghị:** `00` → `01` → `02` → `03` → `04` → `05` → import `backlog.csv`.

---

## 2. Quy ước ID (ID convention)

Theo chuẩn ticketing doanh nghiệp (kiểu Jira), project key là `ECM` (E-CoMmerce).

| Loại | Định dạng | Ví dụ |
|---|---|---|
| Epic | `EPIC-NN` | `EPIC-03` |
| Task / Story | `ECM-NNN` (số chạy toàn cục, không reset theo epic) | `ECM-061` |
| Quyết định kiến trúc | `ADR-NN` | `ADR-01` |
| Migration DB | `Vyyyymmddhhmm__mo_ta` (Flyway) | `V202606231000__create_product` |

> Mỗi task có **một ID duy nhất, không tái sử dụng** kể cả khi bị huỷ. ID gắn theo task suốt vòng đời (branch, commit, PR đều tham chiếu ID).

---

## 3. Quy ước trạng thái (Status legend)

Vòng đời ticket chuẩn:

```
Backlog → To Do → In Progress → Code Review → QA/Test → Done
                        │
                        └── Blocked (tạm dừng, ghi rõ lý do + task chặn)
```

| Trạng thái | Ý nghĩa |
|---|---|
| `Backlog` | Đã ghi nhận, chưa lên kế hoạch sprint |
| `To Do` | Đã đưa vào sprint, đạt Definition of Ready |
| `In Progress` | Đang code |
| `Code Review` | Đã mở PR, chờ review |
| `QA` | Đã merge, đang kiểm thử |
| `Done` | Đạt Definition of Done |
| `Blocked` | Bị chặn bởi task/điều kiện khác |

---

## 4. Quy ước độ ưu tiên & ước lượng

- **Priority:** `P0` (chặn release) > `P1` (must-have MVP) > `P2` (nên có) > `P3` (mở rộng).
  - Trong [backlog.csv](backlog.csv) (để import Jira) các mức này được map sang priority chuẩn của Jira: `P0→Highest`, `P1→High`, `P2→Medium`, `P3→Low`. Các file `.md` vẫn dùng ký hiệu `P0–P3` cho dễ đọc.
- **Estimate:** Story Point theo dãy Fibonacci (`1, 2, 3, 5, 8, 13`). 13 là tín hiệu cần tách nhỏ task.
- **Role gợi ý:** `Lead` (tech lead), `BE` (backend engineer), `BE-Sr` (senior), `DevOps`, `QA`, `DBA`.

---

## 5. Tóm tắt phạm vi

- **Phase 0 — Foundation:** dựng nền tảng dự án, DevOps, chuẩn code.
- **Phase 1 — MVP Backend (Laptop Store):** Auth, Catalog, Attribute/Spec, Inventory, Cart, Promotion, Order, Payment (COD + chuyển khoản), Admin API, Search cơ bản.
- **Phase 2 — Mở rộng:** Elasticsearch, cổng thanh toán online, Review, Notification, Shipping, Analytics, Personalization.
- **Cross-cutting (xuyên suốt):** Security, Performance/Observability, Testing/Release.

Chi tiết xem [04-lo-trinh-sprint.md](04-lo-trinh-sprint.md) và [05-task-breakdown.md](05-task-breakdown.md).
