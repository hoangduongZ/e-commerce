# 04 — Lộ Trình, Phase & Sprint

> Giả định: team 3 BE + 1 Lead + 1 QA (part-time DevOps). Sprint 2 tuần. Velocity khởi điểm ~26 SP/sprint.
> Tổng quan: **Phase 0 (Foundation)** → **Phase 1 (MVP, 6 sprint)** → **Phase 2 (Mở rộng, 6 sprint)**. Các Epic cross-cutting (Security, Performance, Testing) chạy xuyên suốt.

---

## 1. Bản đồ Epic

| Epic | Tên | Phase | Priority | Phụ thuộc |
|---|---|---|---|---|
| **EPIC-00** | Foundation & DevOps | 0 | P0 | — |
| **EPIC-01** | Auth & User Management | 1 | P1 | EPIC-00 |
| **EPIC-02** | Catalog: Category & Brand | 1 | P1 | EPIC-00 |
| **EPIC-03** | Catalog: Product / Image / Variant | 1 | P1 | EPIC-02 |
| **EPIC-04** | Attribute Template & Specs | 1 | P1 | EPIC-03 |
| **EPIC-05** | Inventory Management | 1 | P1 | EPIC-03 |
| **EPIC-06** | Cart | 1 | P1 | EPIC-03, EPIC-01 |
| **EPIC-07** | Promotion & Pricing | 1 | P1 | EPIC-03 |
| **EPIC-08** | Order Management | 1 | P1 | EPIC-05, EPIC-06, EPIC-07 |
| **EPIC-09** | Payment (COD + chuyển khoản) | 1 | P1 | EPIC-08 |
| **EPIC-10** | Admin API & Dashboard | 1 | P1 | EPIC-03, EPIC-08 |
| **EPIC-11** | Search & Filter (DB-based) | 1 | P1 | EPIC-04 |
| **EPIC-12** | Search nâng cao (Elasticsearch) | 2 | P2 | EPIC-11 |
| **EPIC-13** | Cổng thanh toán online | 2 | P2 | EPIC-09 |
| **EPIC-14** | Reviews & Ratings | 2 | P2 | EPIC-08 |
| **EPIC-15** | Notification (email/SMS, async) | 2 | P2 | EPIC-08 |
| **EPIC-16** | Shipping integration | 2 | P2 | EPIC-08 |
| **EPIC-17** | Analytics & Reporting | 2 | P3 | EPIC-08 |
| **EPIC-18** | Personalization (wishlist/compare/reco) | 2 | P3 | EPIC-03 |
| **EPIC-19** | Security hardening | xuyên suốt | P1 | EPIC-01 |
| **EPIC-20** | Performance & Observability | xuyên suốt | P2 | EPIC-00 |
| **EPIC-21** | Testing & Release | xuyên suốt | P1 | tất cả |

---

## 2. Phân bổ Sprint (gợi ý)

### Phase 0
| Sprint | Trọng tâm | Epic | Milestone |
|---|---|---|---|
| **Sprint 0** (2 tuần) | Dựng nền tảng: project, Docker, CI, exception/response envelope, OpenAPI, Flyway baseline | EPIC-00 | **M0: Skeleton chạy được, CI xanh, Swagger sống** |

### Phase 1 — MVP
| Sprint | Trọng tâm | Epic chính |
|---|---|---|
| **Sprint 1** | Auth/JWT/RBAC + User/Address; Category & Brand | EPIC-01, EPIC-02 |
| **Sprint 2** | Product/Image/Variant; bắt đầu Attribute Template | EPIC-03, EPIC-04 |
| **Sprint 3** | Hoàn thiện Attribute/Spec + seed Laptop template; Inventory | EPIC-04, EPIC-05 |
| **Sprint 4** | Cart (guest+user); Promotion & Pricing engine | EPIC-06, EPIC-07 |
| **Sprint 5** | Order (tạo đơn, reserve tồn kho, state machine); Payment COD/chuyển khoản | EPIC-08, EPIC-09 |
| **Sprint 6** | Admin API + Dashboard; Search & Filter DB; ổn định + hardening MVP | EPIC-10, EPIC-11, EPIC-19/20/21 |

> **M1 (cuối Sprint 6): MVP Backend hoàn chỉnh** — đặt hàng COD end-to-end, tồn kho đúng, Admin quản lý được, Swagger đầy đủ, qua UAT staging.

### Phase 2 — Mở rộng
| Sprint | Trọng tâm | Epic chính |
|---|---|---|
| **Sprint 7** | Elasticsearch index + search/facet | EPIC-12 |
| **Sprint 8** | Cổng thanh toán VNPay/MoMo + webhook | EPIC-13 |
| **Sprint 9** | Notification async (RabbitMQ + email) + Outbox | EPIC-15 |
| **Sprint 10** | Reviews & Ratings; Shipping integration | EPIC-14, EPIC-16 |
| **Sprint 11** | Analytics & Reporting; Personalization | EPIC-17, EPIC-18 |
| **Sprint 12** | Performance/Observability sâu, load test, release prod | EPIC-20, EPIC-21 |

> **M2 (cuối Sprint 12): Hệ thống đầy đủ tính năng mở rộng**, thanh toán online, tìm kiếm ES, thông báo async, sẵn sàng scale.

---

## 3. Đồ thị phụ thuộc (rút gọn)

```
EPIC-00 ──┬─▶ EPIC-01 ─▶ EPIC-06 ─┐
          ├─▶ EPIC-02 ─▶ EPIC-03 ─┼─▶ EPIC-05 ─┐
          │                       ├─▶ EPIC-04 ─▶ EPIC-11 ─▶ EPIC-12
          │                       └─▶ EPIC-07 ─┤
          │                                     ▼
          │                                EPIC-08 ─▶ EPIC-09 ─▶ EPIC-13
          │                                  │  └─▶ EPIC-10
          │                                  ├─▶ EPIC-14
          │                                  ├─▶ EPIC-15
          │                                  ├─▶ EPIC-16
          │                                  └─▶ EPIC-17
          └─▶ EPIC-19 / EPIC-20 / EPIC-21 (xuyên suốt)
```

---

## 4. Đường găng (Critical Path)

`EPIC-00 → EPIC-02 → EPIC-03 → EPIC-05 → EPIC-08 → EPIC-09`

Đây là chuỗi quyết định mốc MVP. Ưu tiên nguồn lực và review nhanh cho các task trên đường găng (đặc biệt reserve tồn kho [ECM-044] và tạo đơn giao dịch [ECM-061]).

---

## 5. Milestone tổng kết

| Mốc | Thời điểm | Tiêu chí nghiệm thu |
|---|---|---|
| **M0** | Cuối Sprint 0 | Skeleton + CI/CD + Swagger + Flyway baseline |
| **M1** | Cuối Sprint 6 | MVP backend: luồng mua COD end-to-end, admin, search cơ bản, UAT pass |
| **M2** | Cuối Sprint 12 | Thanh toán online, ES search, notification async, analytics, sẵn sàng prod scale |
