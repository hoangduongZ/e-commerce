# Domain Enums & State Model — Nguồn sự thật

> **Tài liệu chuẩn (normative).** Mọi spec màn hình (`docs/theme/*`), backend (`docs/backend-plan/*`) và code phải dùng đúng các enum dưới đây.
> Field JSON dùng **camelCase**; **giá trị** enum dùng **snake_case** (xem `api-conventions.md`).
> Vd: `"orderStatus": "pending_confirmation"`.

---

## 1. Hai trục trạng thái đơn hàng (tách bạch)

Đơn hàng có **2 trục độc lập** — không gộp làm một (đây là nguồn mâu thuẫn cũ giữa storefront và admin):

### 1.1. `orderStatus` — vòng đời nghiệp vụ của đơn

| Giá trị | Ý nghĩa |
|---|---|
| `pending_confirmation` | Chờ xác nhận (vừa đặt) |
| `confirmed` | Đã xác nhận |
| `processing` | Đang xử lý |
| `completed` | Hoàn thành |
| `cancelled` | Đã hủy |
| `returned` | Đã trả hàng / hoàn |

### 1.2. `fulfillmentStatus` — tiến trình thực hiện giao hàng

| Giá trị | Ý nghĩa |
|---|---|
| `unfulfilled` | Chưa xử lý kho |
| `reserved` | Đã giữ tồn kho |
| `packing` | Đang đóng gói |
| `ready_to_ship` | Sẵn sàng giao |
| `shipping` | Đang giao |
| `delivered` | Đã giao |
| `delivery_failed` | Giao thất bại |
| `returned` | Đã trả về |

> **Quy ước từ loại:** dùng **danh động từ** (`packing`, `shipping`) — KHÔNG dùng quá khứ (`packed`, `shipped`). Bỏ các biến thể cũ `created`, `packed`, `shipped`.
> **Hiển thị storefront:** trang khách có thể gộp 2 trục thành 1 timeline thân thiện (vd "Đang giao") nhưng dữ liệu API luôn trả đủ `orderStatus` + `fulfillmentStatus`.

---

## 2. `paymentStatus` — trạng thái thanh toán của đơn

| Giá trị | Ý nghĩa |
|---|---|
| `unpaid` | Chưa thanh toán |
| `pending` | Đang chờ (online, chưa có kết quả) |
| `cod_pending` | COD — chờ thu khi giao |
| `bank_transfer_pending` | Chuyển khoản — chờ xác nhận |
| `payment_verification_required` | Cần admin xác minh thủ công |
| `paid` | Đã thanh toán đủ |
| `failed` | Thất bại |
| `expired` | Hết hạn (link/giữ đơn) |
| `cancelled` | Đã hủy |
| `refunded` | Đã hoàn tiền |
| `partially_refunded` | Hoàn một phần |

## 3. `codState` — dòng tiền mặt COD (chỉ khi `paymentMethod = cod`)

| Giá trị | Ý nghĩa |
|---|---|
| `pending_collection` | Chờ shipper thu |
| `collected` | Đã thu của khách |
| `collection_failed` | Thu thất bại |
| `remitted` | Shipper đã nộp tiền về |
| `reconciled` | Đã đối soát |

> `codState` là chi tiết vận hành tiền mặt, **tách khỏi** `paymentStatus`. Quy tắc map: `collected`+`reconciled` → `paymentStatus = paid`. Bỏ các giá trị cũ `cod_collected`, `cod_reconciled` khỏi `paymentStatus`.

## 4. `paymentMethod`

MVP: `cod`, `bank_transfer`. Phase 2 (online, có `provider`): `vnpay`, `momo`, `zalopay`, `card`.

---

## 5. `stockStatus` — trạng thái tồn kho (1 tên field duy nhất)

> Tên field chuẩn là **`stockStatus`** (camelCase). Bỏ các tên cũ `availability`, `stock_status`.

| Giá trị | Ý nghĩa |
|---|---|
| `in_stock` | Còn hàng |
| `low_stock` | Sắp hết (≤ ngưỡng cảnh báo) |
| `out_of_stock` | Hết hàng |
| `pre_order` | Đặt trước |
| `coming_soon` | Sắp mở bán |
| `discontinued` | Ngừng kinh doanh |
| `not_tracked` | Không theo dõi tồn |
| `oversold` | Bán âm (cảnh báo nội bộ, chủ yếu admin) |

---

## 6. `warranty` — cấu trúc bảo hành (object, không phải chuỗi)

```json
"warranty": {
  "durationMonths": 24,
  "type": "manufacturer",          // manufacturer | store | extended | none
  "startDate": "2026-06-22",        // ISO, set khi giao hàng; null nếu chưa kích hoạt
  "endDate": "2028-06-22"
}
```

- Trang list/card có thể hiển thị chuỗi dẫn xuất ("Bảo hành 24 tháng") nhưng **dữ liệu API là object** như trên.
- Hồ sơ bảo hành/claim (admin `15-...`) tham chiếu cùng `durationMonths`/`type` và bổ sung `claimStatus` riêng.

---

## 7. Bảng quy đổi với tài liệu cũ (để cập nhật theme docs)

| Khái niệm | Giá trị cũ (bỏ) | Giá trị chuẩn |
|---|---|---|
| fulfillment | `packed`, `shipped`, `created` | `packing`, `shipping`, (bỏ `created`) |
| stock field | `availability`, `stock_status` | `stockStatus` |
| COD payment | `cod_collected`, `cod_reconciled` (trong paymentStatus) | chuyển sang `codState` |
| warranty | chuỗi mô tả | object `{ durationMonths, type, startDate, endDate }` |
