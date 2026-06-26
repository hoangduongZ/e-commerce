# Báo Cáo Kiểm Tra Đồng Bộ Tài Liệu (Doc Sync Audit)

> Ngày: 2026-06-26
> Phạm vi: toàn bộ `docs/` (~40 file Markdown/HTML, ≈46k dòng).
> Mục tiêu: xác định các tài liệu **chưa đồng bộ với nhau** và đề xuất kế hoạch cập nhật.
> Trạng thái: **chỉ là báo cáo audit** — chưa sửa các tài liệu khác (theo yêu cầu). Kế hoạch sửa ở §5.

---

## 1. Tổng quan

Repo hiện **chỉ có tài liệu** (chưa có code). Tài liệu chia thành 5 cụm:

| Cụm | File | Vai trò | Đồng bộ nội bộ |
|---|---|---|---|
| `docs/main/` | `system-design.md`, `ecommerce_design_language.md` | Thiết kế hệ thống tổng quát (generic) + Design System gốc | ⚠️ 2 file lệch palette |
| `docs/backend-plan/` | `README`, `00`–`05`, `backlog.csv`, `jira_csv_overview_explained.md` | Kế hoạch phát triển Backend (Spring Boot) | ✅ Rất chuẩn |
| `docs/theme/` | `01` theme + `02`–`08` storefront + `09`–`17` admin + `prompt-con-lai.md` | Spec UI từng màn hình | ❌ Nhiều mâu thuẫn |
| `docs/theme-html/` | 11 file `.html` + `CLAUDE.md` | Mockup HTML + luật tạo mockup | ✅ Tốt (nhưng thiếu trang) |
| `docs/agent-prompt/`, `my-mind.md`, `idea/jira.md`, `info/jira.md` | Prompt khởi tạo dự án + ghi chú | ⚠️ Trùng lặp/mồ côi |

**Kết luận tổng:** Mỗi cụm tự nó tương đối ổn, nhưng **giữa các cụm CHƯA đồng bộ**. Ba nhóm vấn đề nghiêm trọng nhất:
1. **Hợp đồng API** (envelope, base path, casing) lệch giữa `backend-plan` và `theme`.
2. **Mô hình trạng thái/enum** (order, payment, fulfillment, stock) mỗi tài liệu một kiểu.
3. **Design token** (họ màu neutral, màu success) lệch giữa `ecommerce_design_language.md` gốc và lớp theme.

Đây đều là những điểm sẽ gây xung đột thật khi Frontend (Nuxt 3) ráp với Backend (Spring Boot).

---

## 2. Điểm ĐÃ đồng bộ tốt (giữ nguyên)

- **`backend-plan/` nội bộ chuẩn:** `05-task-breakdown.md` có đúng 124 task `ECM-001 → ECM-124`; `backlog.csv` có 147 dòng = 1 header + 124 task + 22 epic (`EPIC-00 → EPIC-21`). Số lượng và ID khớp tuyệt đối. Critical path, đồ thị phụ thuộc, quy ước ID/trạng thái/priority (`README.md`) nhất quán xuyên suốt `00`–`05`.
- **Tech stack Backend nhất quán:** Java 21 / Spring Boot 3.3+ / PostgreSQL 16 / Redis 7 / Flyway / Maven / JWT-RBAC giống nhau giữa `backend-plan/00`, `backend-plan/01` và `agent-prompt/backend/khoi-tao-base-du-an.md`.
- **Tech stack Frontend (nơi có nêu) nhất quán:** Nuxt 3 + TypeScript + Pinia + Tailwind + pnpm trong `agent-prompt/frontend/*` khớp với ghi chú `my-mind.md`, `idea/jira.md` và `backend-plan/00` ("Frontend (NuxtJS)").
- **Liên kết chéo trong `backend-plan/` đúng:** dùng relative path chuẩn (`../main/system-design.md`, `../theme/01-electronics-store-theme.md`).
- **Mockup HTML khớp `CLAUDE.md`:** các file `theme-html/*.html` bám đúng palette/typography/spacing do `theme-html/CLAUDE.md` quy định.

---

## 3. Vấn đề KHÔNG đồng bộ (xếp theo mức độ)

### 🔴 P0-1 — Hợp đồng API lệch giữa Backend-plan và Theme

| Khía cạnh | Backend-plan (chuẩn dự định) | Theme docs (thực tế) |
|---|---|---|
| **Envelope** | `{ success, data, error, meta }` — `backend-plan/01-kien-truc-tech-stack.md` §4 và task `ECM-005` (`05-task-breakdown.md:17`) | JSON **phẳng**, không envelope. Quét toàn `docs/theme`: chỉ `16-admin-shipping-management.md` có chuỗi `"success"` (và là cờ webhook, không phải envelope). |
| **Base path** | `/api/v1` (BE + agent-prompt + storefront theme) | Admin theme dùng `/api/admin/...`, `/api/storefront/...`, `/api/payments/...`. Đếm trong `docs/theme`: `/api/admin` **155** lần · `/api/v1` **45** · `/api/storefront` **5** → đa số không versioned. |
| **Casing field** | camelCase (mặc định Jackson) | **Loạn**: snake_case ở `03,05,10,11,13,14,15,16,17`; camelCase ở `04,07,08,09,12`. Mâu thuẫn ngay trong chính cụm theme. |
| **Pagination** | `?page=0&size=20&sort=price,asc` (`01` §4) | Object `pagination`/`summary` lồng trong response (vd `12`, `13`). |

**Tác động:** FE phải tự đoán hình dạng response; khi ráp BE thật sẽ sai field, sai bao bọc, sai phân trang.

**Đề xuất canonical:** `/api/v1` + envelope `{success,data,error,meta}` + **camelCase** + pagination `?page=&size=&sort=` (xem quyết định §4).

---

### 🔴 P0-2 — Mô hình trạng thái (state model) mỗi tài liệu một kiểu

**Order / Fulfillment status** (đã kiểm chứng bằng grep):

| Tài liệu | Tập trạng thái |
|---|---|
| `07-storefront-order-success-page.md` | `created, pending_confirmation, confirmed, processing, packed, shipped, delivered, completed, cancelled, returned, refund_pending, refunded` — **lẫn cả `shipped` và `shipping`** trong cùng file |
| `08-storefront-customer-account-page.md` | `pending_confirmation, confirmed, processing, packed, shipping, delivered, completed, cancelled, returned` — dùng `shipping` (không `shipped`) |
| `12-admin-order-management.md` | **Tách 2 trục**: `order_status` = `{pending_confirmation, confirmed, processing, completed, cancelled, returned}`; `fulfillment_status` = `{unfulfilled, reserved, packing, ready_to_ship, shipping, delivered, delivery_failed, returned}` — dùng `packing` (không `packed`) |
| `backend-plan/02-database-design.md` | Có cột `order_status` + `shipping_status` nhưng **không liệt kê giá trị**. `ECM-064` nói "state machine" nhưng chưa định nghĩa enum. |

→ Storefront gộp đóng gói/giao vào `order_status`, Admin tách `order_status` + `fulfillment_status`; thì quá khứ vs danh động từ (`packed/shipped` vs `packing/shipping`) không thống nhất.

**Payment status** (đã kiểm chứng):

| Tài liệu | Tập trạng thái |
|---|---|
| `07` | `unpaid, pending, paid, failed, cancelled, refunded, partially_refunded, cod_pending, bank_transfer_pending` |
| `12` | thêm `expired`, `payment_verification_required` |
| `17-payment-design.md` | thêm `cod_collected`, `cod_reconciled` (không có `bank_transfer_pending`) |

**COD sub-state:** `16-admin-shipping-management.md` `{pending_collection, collected, collection_failed, remitted, reconciled}` vs `17` `{cod_pending, cod_collected, cod_reconciled}` — cùng khái niệm, khác tên.

**Đề xuất canonical:** một bảng enum duy nhất phủ `order_status` + `fulfillment_status` + `payment_status` + `cod_state`, dùng chung cho cả storefront, admin và backend.

---

### 🟠 P1-3 — `stock_status`: khác cả TÊN field lẫn TẬP giá trị

| Tài liệu | Tên field | Giá trị |
|---|---|---|
| `03-storefront-product-list-page.md` | `availability` (7 lần) | `…, pre_order, coming_soon` |
| `04-storefront-product-detail-page.md` | `stockStatus` | `…, pre_order, discontinued` |
| `05-storefront-cart-page.md` | `stock_status` | chỉ 3 giá trị `in_stock/low_stock/out_of_stock` |
| `13-admin-inventory-management.md` | `stock_status` | `…, oversold, not_tracked, discontinued` |

**Đề xuất:** chốt 1 tên field + 1 tập giá trị đầy đủ (gộp `pre_order/coming_soon/discontinued/oversold/not_tracked`), đặt trong tài liệu enum chung.

---

### 🟠 P1-4 — Design token: palette gray vs slate, success emerald vs green

- `docs/main/ecommerce_design_language.md` đặt tên token `neutral-*` nhưng **hex thực tế thuộc họ GRAY**: `#111827, #1f2937, #374151, #6b7280, #9ca3af, #e5e7eb, #f3f4f6, #f9fafb`.
- `docs/theme/01-electronics-store-theme.md` + `docs/theme-html/CLAUDE.md` map `neutral` sang **SLATE**: `#0f172a, #334155, #64748b, #e2e8f0, #f1f5f9, #f8fafc`. `CLAUDE.md` §2 thậm chí **cấm dùng gray/zinc/neutral** (bắt buộc slate).
- **Success:** gốc `#059669` (emerald) vs theme `#22c55e` / `#16a34a` (green).
- Tài liệu gốc còn lẫn `#ff0000` (đỏ thuần, lệch khỏi token `#dc2626`).
- Brand thì khớp: cả hai dùng `#2563eb` / `#1d4ed8`.

→ Agent đọc `ecommerce_design_language.md` sẽ tạo UI tông **gray**; đọc theme/mockup sẽ ra **slate**. Mâu thuẫn trực tiếp giữa doc gốc và lớp theme.

**Đề xuất canonical:** **slate** (+ success **green**), sửa hex trong `ecommerce_design_language.md` cho khớp theme + mockup (xem §4).

---

### 🟠 P1-5 — Liên kết chéo trong theme bị hỏng / loạn tên

File thật: `docs/main/ecommerce_design_language.md` (tiêu đề "00 - Ngôn ngữ thiết kế chung"). Nhưng các theme docs gọi nó bằng nhiều tên & path khác nhau, **không file nào dùng relative path đúng** (`../main/...`):

| Cách gọi sai/không nhất quán | Xuất hiện |
|---|---|
| `ecommerce_design_language.md` (bare, thiếu path) | `03:4, 04:4, 05:6, 08:6, 09:6, 13:9, 14:8/28` |
| `00-ecommerce-design-language.md` (sai tên: thêm `00-`, đổi `_`→`-`) | `01:16, 12:8/26, 15:8/29` |
| `00-design-system.md` | `07:7` |
| path sai `docs/design/00-ecommerce-design-language.md` / `docs/design/...` (thư mục `docs/design/` **không tồn tại**) | `12:26/27, 08:2723/2724` |
| `docs/system-design.md` (thật ra ở `docs/main/`) | `12:30` |

**Đề xuất:** thống nhất **một** tên file + sửa mọi link sang relative path đúng từ `docs/theme/` (`../main/ecommerce_design_language.md`, `../main/system-design.md`).

---

### 🟡 P2 — Các vấn đề nhỏ hơn

- **Warranty shape không nhất quán:** object `{durationMonths, type}` (`04`) vs chuỗi hiển thị (`05`) vs `{warrantyStartDate, warrantyEndDate, status}` (`08`, `15`).
- **FE stack drift:** dự án đã chốt Nuxt 3, nhưng các theme docs tự khai "không phụ thuộc framework" và liệt kê cả **Spring Boot / FastAPI / Laravel** như lựa chọn "frontend" (gây nhiễu, mâu thuẫn quyết định stack).
- **Breakpoint:** `768px` (`02, 04, 05`) vs `767px` (`03`, nghi là typo) vs `640px` (`08`); `theme-html/CLAUDE.md` dùng chuẩn Tailwind (sm 640 / md 768 / lg 1024).
- **Trùng lặp / mồ côi / thiếu:**
  - `my-mind.md` ≈ `idea/jira.md` (gần như trùng, `idea/jira.md` có typo "thsistem"); `info/jira.md` lại là Jira-MCP rules → 3 file rải rác cùng chủ đề.
  - `backend-plan/jira_csv_overview_explained.md` **không có** trong bảng mục lục của `backend-plan/README.md`.
  - `theme/prompt-con-lai.md` trỏ tới `22-api-design.md, 23-frontend-architecture.md, 25-agent-coding-rules.md, 26-playwright-test-strategy.md` — **chưa tồn tại**; lại lệch số với roadmap trong `theme/01` (đánh `18-agent-coding-rules.md`).
  - `theme-html/` mới có ~11/17 trang mockup (thiếu admin 10,11,13,14,15,16 và payment 17…).
- **Audit log / Role model trong admin docs không thống nhất:** `14/15/16/17` mỗi file định nghĩa bộ role và cấu trúc audit log riêng (Actor vs actor, có/không `role`).
- **`system-design.md` đã cũ so với mô hình điện tử:** thiếu `Brand`, `AttributeTemplate/Group/Definition`, `Warehouse`, `Warranty`, `payment_status`, `outbox`, `coupon_redemptions` mà `backend-plan/02` đã bổ sung (đây là "layer trên", nhưng nên ghi chú rõ doc nào là nguồn sự thật cho ERD).

---

## 4. Quyết định canonical đã chốt

Người dùng uỷ quyền chốt theo "góc nhìn senior 10 năm Microsoft". Hai quyết định nền tảng:

1. **Casing field API = `camelCase`.**
   Lý do: là mặc định của Spring Boot/Jackson (BE không cần cấu hình thêm `PropertyNamingStrategy`), và là quy ước tự nhiên của TypeScript/Nuxt phía FE → ít ma sát nhất, ít rủi ro mapping nhất. Các theme docs đang dùng snake_case sẽ được đưa về camelCase.

2. **Họ màu neutral = `slate`; màu success = `green`.**
   Lý do: đã có **mockup HTML thật** dựng theo slate, và `theme-html/CLAUDE.md` ràng buộc cứng slate (cấm gray). Sửa 1 file gốc (`ecommerce_design_language.md`) rẻ hơn nhiều so với sửa theme + toàn bộ mockup. Brand giữ `#2563eb`.

> Hai quyết định này áp dụng khi thực thi Phase A–C bên dưới (chưa thực hiện trong lần này).

---

## 5. Kế hoạch cập nhật (Phase A → C) — *chưa thực hiện*

### Phase A — Chốt "nguồn sự thật" (ưu tiên cao nhất)
- **A1.** Tạo `docs/main/api-conventions.md` (chính là `22-api-design.md` mà `prompt-con-lai.md` đang trỏ): chốt base `/api/v1`, envelope `{success,data,error,meta}`, **camelCase**, pagination `?page=&size=&sort=`, bộ HTTP status + error code. Đồng bộ với `backend-plan/01` §4 và `ECM-005`.
- **A2.** Tạo `docs/main/domain-enums.md`: bảng canonical cho `order_status`, `fulfillment_status`, `payment_status`, `cod_state`, `stock_status`, warranty shape. Mọi doc khác tham chiếu về đây.
- **A3.** Sửa `docs/main/ecommerce_design_language.md`: đổi hex neutral gray → **slate**, success emerald → **green**, bỏ `#ff0000`; ghi rõ brand `#2563eb`. Đảm bảo khớp `theme/01` + `CLAUDE.md`.

### Phase B — Đồng bộ theme docs về chuẩn
- **B1.** Sửa toàn bộ API trong `theme/02`–`17`: thống nhất base path `/api/v1`, bọc envelope, đổi sang camelCase, pagination chuẩn.
- **B2.** Thay enum order/fulfillment/payment/stock/warranty bằng bộ canonical từ A2.
- **B3.** Sửa breakpoint về chuẩn Tailwind; gỡ phần "framework-agnostic + liệt kê BE làm FE", nêu rõ Nuxt 3.
- **B4.** Sửa toàn bộ link chéo sang relative path đúng + thống nhất 1 tên file (P1-5).
- **B5.** Thống nhất role model + cấu trúc audit log giữa các admin docs.

### Phase C — Dọn dẹp & index
- **C1.** Gộp `my-mind.md` / `idea/jira.md` / `info/jira.md` về 1 chỗ (vd `docs/notes/`), xoá bản trùng + sửa typo.
- **C2.** Thêm `jira_csv_overview_explained.md` vào mục lục `backend-plan/README.md`.
- **C3.** Thống nhất đánh số doc tương lai; tạo stub hoặc gỡ tham chiếu tới doc chưa tồn tại trong `prompt-con-lai.md` & `theme/01`.
- **C4.** (Tuỳ chọn) Bổ sung mockup HTML còn thiếu cho admin/payment.
- **C5.** Cập nhật `system-design.md` ghi chú rõ `backend-plan/02` là nguồn sự thật cho ERD chi tiết ngành điện tử.

---

## 6. Phụ lục — Ma trận đồng bộ nhanh

| Hạng mục | system-design | design-language | backend-plan | theme storefront | theme admin | theme-html |
|---|---|---|---|---|---|---|
| Base path API | `/api/v1` (phẳng) | — | `/api/v1` + envelope | `/api/v1` phẳng | `/api/admin`… phẳng | — |
| Casing | snake (ERD) | — | camel (ngụ ý) | trộn | trộn | — |
| Order status | mô tả chung | — | cột, chưa enum | `packed/shipped` | tách `packing/shipping` | — |
| Palette neutral | — | **gray** | — | **slate** | (trỏ design-lang) | **slate** |
| Success color | — | emerald | — | green | (trỏ) | green |
| FE framework | generic | generic | Nuxt | "agnostic" | "agnostic" | Tailwind CDN |

> ✅ = đã thống nhất · ⚠️ = lệch nhẹ · ❌ = mâu thuẫn. Chi tiết xem §3.
