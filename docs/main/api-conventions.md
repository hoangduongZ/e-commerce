# API Conventions — Nguồn sự thật cho hợp đồng API

> **Tài liệu chuẩn (normative).** Mọi spec màn hình (`docs/theme/*`) và code Backend/Frontend phải tuân theo file này.
> Khi spec màn hình có ví dụ API khác với tài liệu này → **tài liệu này thắng**.
> Đồng bộ với `../backend-plan/01-kien-truc-tech-stack.md` §4 và task `ECM-005`.
> (Đây chính là tài liệu mà `docs/theme/prompt-con-lai.md` tham chiếu dưới tên `22-api-design.md`.)

---

## 1. Nguyên tắc chung

| Hạng mục | Chuẩn | Ghi chú |
|---|---|---|
| **Base path** | `/api/v1` | Mọi endpoint (cả storefront lẫn admin) đều versioned. Tăng version khi breaking change. |
| **Phân nhóm** | `/api/v1/...` cho public/storefront; `/api/v1/admin/...` cho admin | KHÔNG dùng `/api/admin`, `/api/storefront`, `/api/payments` không version. |
| **Casing field JSON** | **camelCase** | Mặc định Spring Boot/Jackson + tự nhiên với TypeScript/Nuxt. |
| **Casing giá trị enum** | **snake_case** (chuỗi hằng) | Vd `"stockStatus": "in_stock"`. Xem `domain-enums.md`. |
| **Định dạng thời gian** | ISO 8601 có offset | Vd `2026-06-22T10:00:00+07:00`. |
| **Tiền tệ** | số nguyên + field `currency` (mặc định `VND`) | Không gửi chuỗi đã format; FE tự format `15.990.000đ`. |
| **Charset/locale** | UTF-8, ngôn ngữ `vi` | |

---

## 2. Envelope phản hồi (BẮT BUỘC)

Mọi response bọc trong envelope thống nhất:

```json
{
  "success": true,
  "data": { },
  "error": null,
  "meta": { "page": 0, "size": 20, "totalElements": 137, "totalPages": 7 }
}
```

Khi lỗi:

```json
{
  "success": false,
  "data": null,
  "error": { "code": "PRODUCT_NOT_FOUND", "message": "Không tìm thấy sản phẩm", "details": [] },
  "meta": null
}
```

- `data`: payload chính (object hoặc array).
- `meta`: chỉ có khi phân trang / cần metadata; ngược lại `null`.
- `error`: `null` khi `success=true`.

> Triển khai BE qua `ApiResponse<T>` + `GlobalExceptionHandler` (`ECM-005`).

---

## 3. Phân trang, sắp xếp, lọc

- Query param: `?page=0&size=20&sort=price,asc` (page 0-based, theo chuẩn Spring Pageable).
- Nhiều sort: lặp param `&sort=field,dir`.
- Lọc: param phẳng (`?categoryId=&brandId=&minPrice=&maxPrice=&stockStatus=`).
- Kết quả phân trang đặt list trong `data` (array), thông tin trang trong `meta`.

```json
{ "success": true, "data": [ /* items */ ], "error": null,
  "meta": { "page": 0, "size": 20, "totalElements": 137, "totalPages": 7, "sort": "price,asc" } }
```

---

## 4. HTTP status & Error code

- Status chuẩn: `200/201/204`, `400/401/403/404/409/422`, `500`.
- `409 CONFLICT` cho hết tồn kho (`INSUFFICIENT_STOCK`), xung đột version.
- `422 UNPROCESSABLE_ENTITY` cho lỗi nghiệp vụ/validation.
- `error.code` là hằng `SCREAMING_SNAKE_CASE` ổn định để FE map thông điệp (vd `PRODUCT_NOT_FOUND`, `COUPON_INVALID`, `INSUFFICIENT_STOCK`, `PAYMENT_FAILED`).

---

## 5. Auth & quy ước khác

- Auth: `Authorization: Bearer <JWT>`. Endpoint admin yêu cầu role (`ADMIN`/`MANAGER`/`SUPPORT`) qua `@PreAuthorize`.
- Idempotency: `POST /api/v1/orders` yêu cầu header `Idempotency-Key`.
- Webhook cổng thanh toán: xác thực chữ ký HMAC, xử lý idempotent.

---

## 6. Ví dụ endpoint (đối chiếu nhanh)

```http
# Storefront
GET    /api/v1/products?search=&page=0&size=20&categoryId=&minPrice=&sort=price,asc
GET    /api/v1/products/{idOrSlug}
GET    /api/v1/cart
POST   /api/v1/cart/items            # { "productId": 1, "variantId": 2, "quantity": 3 }
POST   /api/v1/orders                # header Idempotency-Key
POST   /api/v1/coupons/apply         # { "code": "TECH500" }

# Admin (versioned)
GET    /api/v1/admin/orders?status=&page=0&size=20
PUT    /api/v1/admin/orders/{id}/status
GET    /api/v1/admin/inventory
POST   /api/v1/admin/payments/{orderId}/mark-paid
```

> Tham chiếu enum giá trị (orderStatus, paymentStatus, stockStatus...) tại `domain-enums.md`.
