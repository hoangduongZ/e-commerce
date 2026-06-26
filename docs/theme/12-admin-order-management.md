# 12 - Admin Order Management Specification

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Màn hình/module: Quản lý đơn hàng  
> Mục tiêu: Đặc tả đủ chi tiết để coding agent/frontend agent/backend agent có thể code module quản lý đơn hàng admin từ đầu đến cuối.  
> Phụ thuộc:  
> - `00-ecommerce-design-language.md`  
> - `01-electronics-store-theme.md`  
> - `09-admin-dashboard.md`  
> - `system-design.md`  
>
> Tài liệu này không phụ thuộc công nghệ frontend/backend cụ thể. Có thể triển khai bằng Nuxt, Next.js, Vue, React, Angular, Laravel, Spring Boot, FastAPI hoặc bất kỳ stack nào.

---

# 0. Prompt sử dụng cho coding agent

Khi giao việc cho coding agent, có thể dùng prompt sau:

```text
Bạn là Senior Frontend/Fullstack Agent.

Hãy đọc và tuân thủ các tài liệu sau trước khi code:

1. docs/design/00-ecommerce-design-language.md
2. docs/design/01-electronics-store-theme.md
3. docs/design/09-admin-dashboard.md
4. docs/design/12-admin-order-management.md
5. docs/system-design.md nếu cần hiểu backend/domain.

Nhiệm vụ:
Implement module Admin Order Management cho web bán đồ điện tử.

Phạm vi bắt buộc:
- Admin order list page.
- Admin order detail page.
- Search/filter/sort/pagination.
- Order status badge.
- Payment status badge.
- Shipping/fulfillment status badge.
- Order timeline.
- Order action panel.
- Confirm/update/cancel/refund action flow.
- Internal notes.
- Customer/shipping/payment/order item sections.
- Loading/empty/error/permission states.
- Responsive desktop/tablet/mobile.
- Không horizontal overflow ở mobile.
- Không lộ dữ liệu nhạy cảm quá mức.
- Action nguy hiểm phải có confirm modal.
- Không hard-code transition tuỳ tiện; phải theo state machine trong spec.
- Nếu có dữ liệu tồn kho, phải hiển thị cảnh báo item hết hàng hoặc thiếu hàng.

Sau khi code xong, phải báo cáo:
- Files changed.
- Components created/updated.
- APIs integrated/mocked.
- Tests added/updated.
- Tests run.
- Screenshot/viewport checked.
- Known limitations.
```

---

# 1. Vai trò của Admin Order Management

Admin Order Management là module trung tâm vận hành sau khi khách đặt hàng.

Module này giúp admin:

- Xem toàn bộ đơn hàng.
- Tìm kiếm đơn theo mã đơn, khách hàng, số điện thoại, email, mã vận đơn.
- Lọc đơn theo trạng thái đơn, thanh toán, vận chuyển, ngày đặt, giá trị đơn.
- Xem chi tiết từng đơn.
- Xác nhận đơn mới.
- Cập nhật trạng thái đóng gói, giao hàng, hoàn tất.
- Hủy đơn theo quyền và điều kiện.
- Ghi chú nội bộ.
- Kiểm tra lịch sử trạng thái.
- Theo dõi thanh toán COD/chuyển khoản/online.
- Theo dõi vận chuyển.
- In hóa đơn, phiếu đóng gói hoặc xuất dữ liệu.
- Xử lý đơn có rủi ro: giá trị cao, thanh toán lỗi, thiếu tồn kho, khách yêu cầu hủy, giao hàng lỗi.

Với shop đồ điện tử, order management đặc biệt quan trọng vì:

```text
Sản phẩm có giá trị cao.
Nhiều đơn cần kiểm tra thanh toán kỹ.
Nhiều sản phẩm có biến thể RAM/SSD/màu/dung lượng.
Có bảo hành theo sản phẩm hoặc theo đơn.
Có thể cần serial/IMEI ở giai đoạn mở rộng.
Có tồn kho theo variant.
Có đơn chuyển khoản cần xác minh.
Có đơn giá trị cao cần cảnh báo.
```

---

# 2. Nguyên tắc thiết kế module

## 2.1. Admin order là màn xử lý nghiệp vụ, không phải màn trang trí

Giao diện cần ưu tiên:

```text
Rõ
Nhanh
Chính xác
Ít màu
Dễ lọc
Dễ thao tác nhiều đơn
Giảm nhầm lẫn trạng thái
Có xác nhận khi action nguy hiểm
```

Không ưu tiên animation, hiệu ứng hoặc layout quá marketing.

## 2.2. Mọi trạng thái phải có ý nghĩa nghiệp vụ

Không chỉ dùng một status chung `processing`.

Order nên tách ít nhất 3 nhóm trạng thái:

```text
Order status        = trạng thái vòng đời đơn hàng
Payment status      = trạng thái thanh toán
Fulfillment status  = trạng thái xử lý/giao hàng
```

Ví dụ:

```text
Order status: pending, confirmed, completed, cancelled
Payment status: unpaid, paid, failed, refunded, cod_pending
Fulfillment status: unfulfilled, packing, shipping, delivered, returned
```

Tách như vậy giúp admin hiểu đơn đang vướng ở đâu.

## 2.3. Không phải trạng thái nào cũng chuyển được sang mọi trạng thái

Module phải có state machine.

Không cho phép chuyển bừa:

```text
completed -> pending
cancelled -> shipping
refunded -> paid nếu không có nghiệp vụ rõ
```

Mỗi action cần kiểm tra:

```text
Trạng thái hiện tại
Quyền của admin
Trạng thái thanh toán
Trạng thái tồn kho
Điều kiện vận chuyển
Điều kiện hoàn tiền
```

## 2.4. Action nguy hiểm phải có confirm modal

Các action cần confirm:

```text
Cancel order
Refund payment
Mark as paid
Mark as delivered
Change shipping address after confirmation
Remove order item
Edit order total
Force release inventory
```

Không dùng `window.confirm` trong app thật. Dùng modal theo design system.

## 2.5. Không làm mất dữ liệu khi cập nhật lỗi

Nếu admin nhập ghi chú hoặc chỉnh thông tin rồi API lỗi:

```text
Không reset form.
Không đóng modal đột ngột.
Hiển thị lỗi rõ.
Cho phép retry.
```

---

# 3. Phạm vi module

## 3.1. Trang chính

```text
/admin/orders
/admin/orders/:id
/admin/orders/:id/edit
/admin/orders/:id/timeline
/admin/orders/export
/admin/orders/returns
/admin/orders/refunds
```

MVP bắt buộc:

```text
/admin/orders
/admin/orders/:id
```

## 3.2. Màn hình trong module

1. Admin Order List Page.
2. Admin Order Detail Page.
3. Order Status Update Modal.
4. Cancel Order Modal.
5. Mark Paid Modal.
6. Refund Modal hoặc placeholder nếu chưa làm payment online.
7. Internal Note Panel.
8. Print/Export action.
9. Mobile Order Card List.
10. Empty/Error/Permission states.

---

# 4. User roles và permission

## 4.1. Role tham khảo

| Role | Mục đích |
|---|---|
| Super Admin | Toàn quyền |
| Store Manager | Quản lý vận hành đơn hàng |
| Sales Staff | Xác nhận đơn, gọi khách |
| Warehouse Staff | Đóng gói, cập nhật kho/giao hàng |
| Support Staff | Xem đơn, ghi chú, hỗ trợ khách |
| Accountant | Xem/cập nhật thanh toán, hoàn tiền |
| Viewer | Chỉ xem |

## 4.2. Permission gợi ý

| Permission | Mục đích |
|---|---|
| order.view | Xem danh sách/chi tiết đơn |
| order.update | Cập nhật thông tin đơn |
| order.confirm | Xác nhận đơn mới |
| order.cancel | Hủy đơn |
| order.refund | Hoàn tiền |
| order.mark_paid | Đánh dấu đã thanh toán |
| order.fulfillment.update | Cập nhật đóng gói/giao hàng |
| order.note.create | Thêm ghi chú nội bộ |
| order.export | Xuất dữ liệu đơn |
| order.print | In phiếu/hóa đơn |
| customer.private.view | Xem đầy đủ thông tin khách |
| payment.private.view | Xem thông tin thanh toán nhạy cảm |

## 4.3. Rule hiển thị theo quyền

- Không hiển thị action nếu admin không có quyền.
- Nếu cần giải thích, có thể disable action và hiển thị tooltip `You do not have permission`.
- Backend vẫn phải kiểm tra quyền.
- UI permission không thay thế bảo mật backend.

Ví dụ:

```text
Viewer: xem đơn, không thấy nút Confirm/Cancel/Refund.
Sales Staff: confirm/cancel đơn pending, không được refund.
Warehouse Staff: cập nhật packing/shipping, không được mark paid.
Accountant: mark paid/refund, không được sửa sản phẩm trong đơn.
```

---

# 5. Order domain model

## 5.1. Order object

```json
{
  "id": "order_1024",
  "orderNumber": "DH1024",
  "customerId": "cus_001",
  "customerSnapshot": {
    "fullName": "Nguyễn Văn A",
    "phoneMasked": "090****000",
    "phone": "0900000000",
    "emailMasked": "n***@gmail.com",
    "email": "nguyenvana@gmail.com"
  },
  "orderStatus": "pending_confirmation",
  "paymentStatus": "bank_transfer_pending",
  "fulfillmentStatus": "unfulfilled",
  "currency": "VND",
  "subtotal": 31980000,
  "discountAmount": 1000000,
  "shippingFee": 0,
  "taxAmount": 0,
  "totalAmount": 30980000,
  "paymentMethod": "bank_transfer",
  "shippingMethod": "standard_delivery",
  "shippingAddress": {},
  "billingAddress": {},
  "items": [],
  "timeline": [],
  "internalNotes": [],
  "riskFlags": ["high_value_order"],
  "createdAt": "2026-06-22T09:30:00+07:00",
  "updatedAt": "2026-06-22T09:45:00+07:00"
}
```

## 5.2. Order item object

```json
{
  "id": "order_item_1",
  "productId": "prod_001",
  "variantId": "var_001",
  "sku": "DELL-INS-3520-I5-16-512",
  "productNameSnapshot": "Laptop Dell Inspiron 15 3520 i5 16GB 512GB",
  "variantLabel": "Silver / 16GB RAM / SSD 512GB",
  "imageUrl": "/images/products/dell-inspiron.jpg",
  "unitPrice": 15990000,
  "quantity": 2,
  "lineTotal": 31980000,
  "warrantySnapshot": {
    "warrantyMonths": 24,
    "provider": "Manufacturer",
    "type": "official"
  },
  "inventoryStatus": "reserved",
  "serialNumbers": []
}
```

## 5.3. Shipping address object

```json
{
  "fullName": "Nguyễn Văn A",
  "phone": "0900000000",
  "addressLine": "Số 1 Nguyễn Trãi",
  "ward": "Văn Quán",
  "district": "Hà Đông",
  "city": "Hà Nội",
  "country": "Việt Nam",
  "postalCode": null,
  "note": "Gọi trước khi giao"
}
```

## 5.4. Order timeline event

```json
{
  "id": "event_001",
  "type": "status_changed",
  "title": "Order confirmed",
  "description": "Order was confirmed by Sales Admin.",
  "fromStatus": "pending_confirmation",
  "toStatus": "confirmed",
  "actor": {
    "id": "admin_1",
    "name": "Minh Anh",
    "role": "Sales Staff"
  },
  "createdAt": "2026-06-22T09:45:00+07:00",
  "metadata": {
    "note": "Đã gọi xác nhận khách."
  }
}
```

---

# 6. Order status model

## 6.1. Order status

| Status | Label | Ý nghĩa |
|---|---|---|
| pending_confirmation | Chờ xác nhận | Đơn mới tạo, admin chưa xác nhận |
| confirmed | Đã xác nhận | Đã xác nhận thông tin đơn |
| processing | Đang xử lý | Đơn đang được xử lý nội bộ |
| completed | Hoàn thành | Đơn đã hoàn tất nghiệp vụ |
| cancelled | Đã hủy | Đơn bị hủy |
| returned | Hoàn hàng | Đơn đã trả hàng |

## 6.2. Payment status

| Status | Label | Ý nghĩa |
|---|---|---|
| unpaid | Chưa thanh toán | Chưa ghi nhận thanh toán |
| cod_pending | COD chờ thu | Thu tiền khi giao hàng |
| bank_transfer_pending | Chờ chuyển khoản | Khách cần chuyển khoản hoặc chờ xác minh |
| payment_verification_required | Cần xác minh | Có giao dịch cần kiểm tra |
| paid | Đã thanh toán | Đã ghi nhận thanh toán |
| failed | Thanh toán lỗi | Cổng thanh toán/thanh toán thất bại |
| refunded | Đã hoàn tiền | Đã hoàn tiền toàn bộ |
| partially_refunded | Hoàn tiền một phần | Đã hoàn một phần |

## 6.3. Fulfillment status

| Status | Label | Ý nghĩa |
|---|---|---|
| unfulfilled | Chưa xử lý hàng | Chưa đóng gói |
| reserved | Đã giữ hàng | Tồn kho đã reserve |
| packing | Đang đóng gói | Kho đang xử lý |
| ready_to_ship | Sẵn sàng giao | Đã đóng gói xong |
| shipping | Đang giao | Đã bàn giao vận chuyển |
| delivered | Đã giao | Khách đã nhận |
| delivery_failed | Giao thất bại | Đơn vị vận chuyển báo lỗi |
| returned | Đã hoàn hàng | Hàng quay về kho |

## 6.4. Badge color rule

| Nhóm | Tone |
|---|---|
| Pending / waiting | warning |
| Confirmed / processing | info/brand |
| Paid / delivered / completed | success |
| Failed / cancelled / refund issue | danger |
| Archived / neutral state | neutral |

Không chỉ dùng màu. Badge phải có text rõ.

---

# 7. Order state machine

## 7.1. Luồng đơn hàng MVP

```text
pending_confirmation
  -> confirmed
  -> packing
  -> shipping
  -> delivered
  -> completed
```

Luồng hủy:

```text
pending_confirmation -> cancelled
confirmed            -> cancelled
packing              -> cancelled nếu chưa bàn giao vận chuyển
shipping             -> delivery_failed hoặc returned tùy nghiệp vụ
```

## 7.2. Luồng thanh toán

COD:

```text
cod_pending -> paid khi giao thành công hoặc admin xác nhận thu tiền
cod_pending -> cancelled nếu đơn hủy
```

Chuyển khoản:

```text
bank_transfer_pending -> payment_verification_required -> paid
bank_transfer_pending -> cancelled nếu quá hạn hoặc khách hủy
```

Online payment:

```text
unpaid -> paid
unpaid -> failed
failed -> unpaid nếu retry
paid -> refunded hoặc partially_refunded
```

## 7.3. Transition rules

| From | Allowed actions |
|---|---|
| pending_confirmation | confirm, cancel, edit address, add note |
| confirmed | start packing, cancel, add note, print invoice |
| packing | mark ready to ship, cancel if not shipped, add tracking |
| ready_to_ship | mark shipping, assign carrier |
| shipping | mark delivered, mark delivery failed |
| delivered | complete, return request |
| completed | refund if allowed, create warranty case |
| cancelled | view only, restore only if policy allows |

## 7.4. Action guard

Mỗi action phải kiểm tra:

```text
Current status
Permission
Inventory state
Payment state
Shipping state
Refund state
Business policy
```

Ví dụ:

```text
Không được mark shipping nếu chưa packing.
Không được complete nếu chưa delivered.
Không được refund nếu payment chưa paid.
Không được cancel completed order.
Không được mark paid nếu thiếu quyền order.mark_paid.
```

---

# 8. Admin order list page

## 8.1. Mục đích

Trang danh sách đơn hàng giúp admin:

- Nhìn nhanh toàn bộ đơn.
- Ưu tiên đơn cần xử lý.
- Tìm kiếm và lọc theo nhiều tiêu chí.
- Thực hiện action nhanh.
- Điều hướng vào chi tiết đơn.

## 8.2. Desktop layout

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Admin Topbar                                                        │
├───────────────┬─────────────────────────────────────────────────────┤
│ Sidebar       │ Orders                                              │
│               │ Tổng quan và xử lý đơn hàng                         │
│               │                                                     │
│               │ [Search order/customer/phone] [Status] [Payment]    │
│               │ [Fulfillment] [Date range] [More filters] [Export]  │
│               │                                                     │
│               │ Quick tabs: All | New | Pending payment | Shipping  │
│               │                                                     │
│               │ Order Table                                         │
│               │                                                     │
│               │ Pagination                                          │
└───────────────┴─────────────────────────────────────────────────────┘
```

## 8.3. Tablet layout

```text
Topbar
Page header
Search full width
Filter row horizontal scroll
Order table compact
```

Cột mặc định tablet:

```text
Order
Customer
Total
Payment
Status
Action
```

## 8.4. Mobile layout

Mobile admin không phải trải nghiệm chính, nhưng vẫn phải usable.

```text
Topbar compact
Page title
Search input
[Filter] [Status tabs]
Order cards
Pagination / Load more
```

Không dùng table đầy đủ trên mobile. Chuyển sang order card.

---

# 9. Order list toolbar

## 9.1. Search input

Placeholder:

```text
Search by order number, customer, phone, email, tracking code...
```

Search hỗ trợ:

```text
Order number
Customer name
Phone
Email
Tracking code
Payment reference
Product SKU optional
```

Behavior:

```text
Debounce 300ms.
Enter search ngay.
Có clear button.
Giữ query trên URL.
Không reset filter khác khi search.
```

Ví dụ URL:

```text
/admin/orders?search=DH1024&status=pending_confirmation&page=1
```

## 9.2. Quick tabs

Quick tabs giúp xử lý nhanh:

```text
All
New orders
Pending payment
Need confirmation
Packing
Shipping
Delivery issues
Cancelled
Returned
```

Mỗi tab có count nếu API hỗ trợ.

## 9.3. Filters

Filter chính:

```text
Order status
Payment status
Fulfillment status
Date range
Payment method
Shipping method
Order value range
High value orders
Has risk flag
Assigned staff
```

Filter nâng cao:

```text
Coupon used
Product SKU
Category
Brand
City/province
Carrier
Tracking code
Refund status
Return status
Warranty attached
```

## 9.4. Date range presets

```text
Today
Yesterday
Last 7 days
Last 30 days
This month
Last month
Custom
```

## 9.5. Applied filter chips

Khi filter đang áp dụng, hiển thị chip:

```text
Status: Pending confirmation
Payment: Bank transfer pending
Date: Last 7 days
High value
```

Có nút:

```text
Clear all
```

---

# 10. Order table

## 10.1. Cột mặc định desktop

| Column | Nội dung |
|---|---|
| Checkbox | chọn nhiều đơn |
| Order | mã đơn + thời gian |
| Customer | tên + phone masked |
| Items | số item / preview sản phẩm |
| Total | tổng tiền |
| Payment | trạng thái thanh toán |
| Fulfillment | trạng thái xử lý/giao |
| Order status | trạng thái đơn |
| Risk | cảnh báo |
| Updated | cập nhật gần nhất |
| Actions | thao tác |

Không đưa câu dài vào bảng.

## 10.2. Order cell

Hiển thị:

```text
#DH1024
22/06/2026 14:35
```

Nếu đơn mới chưa xem:

```text
New badge
```

Nếu đơn quá SLA:

```text
SLA badge
```

## 10.3. Customer cell

Hiển thị mặc định:

```text
Nguyễn Văn A
090****000
```

Nếu admin có quyền `customer.private.view`, có thể hiển thị đầy đủ trong detail hoặc khi bấm reveal.

Không hiển thị đầy đủ email/phone trên list nếu không cần.

## 10.4. Items cell

Hiển thị:

```text
2 items
Dell Inspiron 15 + 1 more
```

Không nhồi toàn bộ item vào table.

## 10.5. Total cell

Rule:

```text
Căn phải nếu dùng table.
Format tiền nhất quán: 30.980.000đ.
Nếu high value, thêm badge ngắn.
```

Ví dụ:

```text
30.980.000đ
High value
```

## 10.6. Payment cell

Hiển thị badge:

```text
Paid
COD pending
Bank transfer pending
Failed
Refunded
```

Nếu chuyển khoản chờ quá lâu, thêm warning.

## 10.7. Fulfillment cell

Hiển thị badge:

```text
Unfulfilled
Packing
Shipping
Delivered
Delivery failed
Returned
```

## 10.8. Risk cell

Risk flags:

```text
High value
Payment failed
SLA exceeded
Stock issue
Address issue
Customer cancel request
Delivery issue
```

Không hiển thị quá 2 risk badge trên table. Nếu nhiều hơn, dùng `+N`.

## 10.9. Actions cell

Action chính theo trạng thái:

```text
View
Confirm
Mark paid
Pack
Ship
Complete
Cancel
More
```

Rule:

```text
Không hiển thị quá 2 action trực tiếp.
Action phụ đưa vào More menu.
Action nguy hiểm dùng danger tone và confirm.
```

---

# 11. Mobile order card

Mobile card hiển thị:

```text
Order number + status badge
Customer masked
Total
Payment status
Fulfillment status
Age / created time
Risk badge nếu có
Primary action
Secondary action menu
```

Ví dụ:

```text
#DH1024       Chờ xác nhận
Nguyễn Văn A · 090****000
30.980.000đ
Chờ chuyển khoản · Chưa xử lý hàng
42 phút trước · High value
[View] [Confirm]
```

Rule:

```text
Không overflow ngang.
Button đủ lớn để bấm.
Text dài clamp hợp lý.
Status luôn nhìn thấy.
```

---

# 12. Bulk actions

Bulk action bar hiển thị khi chọn nhiều đơn.

Actions gợi ý:

```text
Export selected
Assign staff
Print invoices
Print packing slips
Mark as confirmed
Mark as packing
Cancel selected
```

Rule:

```text
Bulk cancel cần confirm.
Bulk status update cần kiểm tra từng đơn.
Nếu một số đơn không hợp lệ, hiển thị partial result.
Không được silent fail.
```

Ví dụ partial result:

```text
12 orders selected
8 orders updated successfully
4 orders skipped because their status is not eligible
[Download report]
```

---

# 13. Empty, loading, error states cho order list

## 13.1. Loading

Dùng skeleton:

```text
Toolbar skeleton nhẹ nếu chưa có filter config.
Table row skeleton.
Mobile card skeleton.
```

Không hiển thị số 0 giả khi đang loading.

## 13.2. Empty no orders

Message:

```text
No orders yet.
When customers place orders, they will appear here.
```

CTA:

```text
View storefront
Create test order optional
```

## 13.3. Empty filtered

Message:

```text
No orders match your filters.
Try changing filters or clearing all filters.
```

CTA:

```text
Clear filters
```

## 13.4. Error

Message:

```text
Could not load orders.
Please try again.
```

CTA:

```text
Retry
```

Không crash toàn admin shell.

---

# 14. Admin order detail page

## 14.1. Mục đích

Order detail là nơi admin xem toàn bộ thông tin và xử lý đơn.

Trang này cần trả lời:

```text
Đơn này của ai?
Khách mua gì?
Tổng tiền bao nhiêu?
Đã thanh toán chưa?
Hàng đã giữ/chưa?
Đơn đang ở trạng thái nào?
Có vấn đề gì không?
Admin có thể làm action gì tiếp theo?
Lịch sử thay đổi là gì?
```

## 14.2. Desktop layout

```text
┌───────────────────────────────────────────────────────────────────┐
│ Topbar                                                            │
├──────────────┬──────────────────────────────────────┬─────────────┤
│ Sidebar      │ Main content                         │ Side panel  │
│              │ Order header                         │ Actions     │
│              │ Alert/risk banner                    │ Status      │
│              │ Items                                │ Payment     │
│              │ Customer & shipping                  │ Fulfillment │
│              │ Timeline                             │ Notes       │
│              │ Audit log                            │             │
└──────────────┴──────────────────────────────────────┴─────────────┘
```

## 14.3. Mobile layout

```text
Order header
Status summary
Action buttons sticky or top section
Items
Payment
Customer/shipping
Timeline accordion
Notes
```

Side panel chuyển thành section trong page hoặc bottom action bar.

## 14.4. Header

Header hiển thị:

```text
Order number
Created date
Order status badge
Payment status badge
Fulfillment status badge
Back to orders
Print/export actions
```

Ví dụ:

```text
Order #DH1024
Created at 22/06/2026 14:35
[Pending confirmation] [Bank transfer pending] [Unfulfilled]
```

---

# 15. Risk/alert banner trong order detail

Hiển thị ở đầu detail nếu có vấn đề:

```text
High value order: 30.980.000đ. Please verify payment before shipping.
Payment pending for more than 24 hours.
Some items are no longer available.
Shipping address may be incomplete.
Customer requested cancellation.
```

Severity:

```text
Info
Warning
Danger
```

Rule:

```text
Alert phải có action nếu có thể xử lý.
Alert nghiêm trọng không được dismiss nếu chưa xử lý.
Không dùng quá nhiều alert tách rời; có thể gom lại.
```

---

# 16. Order items section

## 16.1. Mục đích

Hiển thị chính xác sản phẩm trong đơn.

Với đồ điện tử, item phải có:

```text
Ảnh
Tên snapshot
SKU
Variant label
Thông số ngắn nếu cần
Số lượng
Giá tại thời điểm mua
Tổng dòng
Bảo hành snapshot
Inventory status
Serial/IMEI placeholder nếu mở rộng
```

## 16.2. Desktop table columns

| Column | Nội dung |
|---|---|
| Product | ảnh + tên + SKU |
| Variant | RAM/SSD/màu/dung lượng |
| Warranty | bảo hành |
| Unit price | giá |
| Qty | số lượng |
| Line total | tổng dòng |
| Inventory | reserve/issue |

## 16.3. Product item display

Ví dụ:

```text
Laptop Dell Inspiron 15 3520 i5 16GB 512GB
SKU: DELL-INS-3520-I5-16-512
Variant: Silver / 16GB / 512GB
Warranty: 24 months official
```

## 16.4. Inventory warning

Nếu tồn kho có vấn đề:

```text
Stock not reserved
Only 1 item available
Variant is discontinued
```

Action:

```text
View inventory
Release item
Replace item optional
```

MVP chỉ cần warning + link inventory.

## 16.5. Snapshot rule

Order item phải dùng snapshot:

```text
Product name snapshot
Price snapshot
Warranty snapshot
Variant snapshot
```

Không phụ thuộc hoàn toàn vào Product hiện tại vì sau này sản phẩm có thể đổi tên/giá.

---

# 17. Order summary section

Hiển thị:

```text
Subtotal
Discount
Coupon
Shipping fee
Tax nếu có
Total
Paid amount
Refunded amount nếu có
Outstanding amount nếu có
```

Rule:

```text
Tổng tiền phải rõ.
Căn phải số tiền trong table.
Nếu có discount, hiển thị mã coupon.
Nếu có refund, hiển thị rõ amount đã hoàn.
Không để admin nhầm total và paid.
```

Ví dụ:

```text
Subtotal          31.980.000đ
Discount          -1.000.000đ
Shipping fee      0đ
Total             30.980.000đ
Paid              0đ
Outstanding       30.980.000đ
```

---

# 18. Customer section

Hiển thị:

```text
Customer name
Phone
Email
Customer type: guest/registered
Total previous orders optional
Customer note optional
```

Privacy rule:

```text
List page mask phone/email.
Detail page có thể hiển thị đầy đủ nếu role có quyền.
Không hiển thị dữ liệu nhạy cảm nếu không cần.
```

Actions:

```text
Call customer
Copy phone
View customer profile
Send message optional
```

---

# 19. Shipping address section

Hiển thị:

```text
Receiver full name
Phone
Full address
Delivery note
Shipping method
Carrier
Tracking code
Estimated delivery
```

Actions:

```text
Copy address
Edit address nếu trạng thái cho phép
Create shipment
Update tracking
```

Rule edit address:

```text
Cho phép sửa trước khi shipping.
Sau khi shipping, sửa địa chỉ cần quyền cao và confirm.
Mọi thay đổi địa chỉ phải ghi timeline/audit.
```

---

# 20. Payment section

Hiển thị:

```text
Payment method
Payment status
Payment amount
Payment reference
Payment date
Transaction logs optional
```

## 20.1. COD

```text
Method: COD
Status: COD pending
Amount to collect: 30.980.000đ
```

Action:

```text
Mark COD collected
```

## 20.2. Bank transfer

```text
Method: Bank transfer
Status: Pending verification
Expected amount
Transfer content
Bank account
```

Actions:

```text
Mark as paid
Request payment proof optional
Cancel if expired
```

Mark paid cần modal xác nhận.

## 20.3. Online payment

```text
Method: VNPay/MoMo/ZaloPay/Card
Status: Paid/Failed/Refunded
Transaction ID
Gateway response optional
```

Actions:

```text
Retry payment optional
Refund
View gateway log optional
```

---

# 21. Fulfillment section

Hiển thị:

```text
Fulfillment status
Warehouse
Reserved items
Packing status
Carrier
Tracking code
Shipping status
Delivered date
```

Actions:

```text
Reserve stock
Start packing
Mark ready to ship
Assign carrier
Mark shipping
Mark delivered
Report delivery failed
```

Rule:

```text
Không mark shipping nếu chưa có carrier/tracking khi policy bắt buộc.
Không mark delivered nếu chưa shipping, trừ quyền override.
Mọi action fulfillment phải ghi timeline.
```

---

# 22. Warranty/service section

Với đồ điện tử, order detail nên có section bảo hành.

Hiển thị:

```text
Warranty snapshot per item
Warranty start date
Warranty end date
Warranty provider
Warranty type
Service request link nếu có
Serial/IMEI placeholder
```

MVP:

```text
Hiển thị warranty months từ order item snapshot.
Chưa cần quản lý serial.
```

Mở rộng:

```text
Gán serial/IMEI khi packing hoặc shipping.
Tra cứu bảo hành theo serial.
Tạo warranty case từ order detail.
```

---

# 23. Internal notes

## 23.1. Mục đích

Ghi chú nội bộ giúp nhân sự vận hành phối hợp.

Ví dụ:

```text
Đã gọi khách xác nhận, khách yêu cầu giao buổi chiều.
Khách đã chuyển khoản nhưng thiếu nội dung.
Kho báo thiếu màu bạc, đề xuất đổi màu đen.
```

## 23.2. Note fields

```json
{
  "id": "note_001",
  "orderId": "order_1024",
  "content": "Đã gọi khách xác nhận.",
  "visibility": "internal",
  "createdBy": "admin_1",
  "createdAt": "2026-06-22T10:00:00+07:00"
}
```

## 23.3. Rule

```text
Note nội bộ không gửi cho khách.
Note không được chứa thông tin thẻ/thanh toán nhạy cảm.
Có thể edit/delete note trong thời gian ngắn nếu policy cho phép.
Ghi chú quan trọng có thể pin.
```

---

# 24. Order timeline

## 24.1. Mục đích

Timeline cho admin biết lịch sử đơn.

Timeline events:

```text
Order created
Payment status changed
Order confirmed
Inventory reserved
Packing started
Carrier assigned
Tracking code added
Order shipped
Order delivered
Order cancelled
Refund created
Internal note added
Address changed
```

## 24.2. Timeline UI

Mỗi event hiển thị:

```text
Icon
Title
Description
Actor
Timestamp
Metadata optional
```

Ví dụ:

```text
Order confirmed
Minh Anh confirmed this order.
22/06/2026 14:48
```

## 24.3. Rule

```text
Timeline chỉ append, không sửa lịch sử.
Nếu cần sửa, tạo event mới.
Không dùng timeline thay thế audit log bảo mật.
```

---

# 25. Main actions

## 25.1. Confirm order

Điều kiện:

```text
orderStatus = pending_confirmation
permission = order.confirm
```

Flow:

```text
Click Confirm
Open confirm modal
Optional note
Submit
API update
Show success toast
Update status + timeline
```

## 25.2. Mark as paid

Điều kiện:

```text
paymentStatus in unpaid/bank_transfer_pending/payment_verification_required/cod_pending
permission = order.mark_paid
```

Modal fields:

```text
Payment method
Paid amount
Payment reference optional
Payment date
Internal note
```

Rule:

```text
Paid amount không vượt total nếu không có policy.
Mark paid là action nhạy cảm.
Phải confirm rõ.
Ghi audit log.
```

## 25.3. Start packing

Điều kiện:

```text
orderStatus = confirmed
fulfillmentStatus in unfulfilled/reserved
permission = order.fulfillment.update
```

Flow:

```text
Check inventory reserved
Start packing
Timeline event
```

## 25.4. Mark shipping

Điều kiện:

```text
fulfillmentStatus in packing/ready_to_ship
carrier/tracking required nếu policy bật
```

Fields:

```text
Carrier
Tracking code
Shipping date
Note
```

## 25.5. Mark delivered

Điều kiện:

```text
fulfillmentStatus = shipping
```

Nếu COD:

```text
Có thể đồng thời mark COD collected nếu policy cho phép.
```

## 25.6. Cancel order

Điều kiện:

```text
orderStatus chưa completed/cancelled
permission = order.cancel
```

Modal fields:

```text
Cancel reason
Restock items checkbox
Notify customer checkbox
Internal note
```

Cancel reasons:

```text
Customer requested
Payment timeout
Out of stock
Invalid address
Duplicate order
Fraud suspicion
Other
```

Rule:

```text
Cancel phải ghi reason.
Nếu đã paid, cần refund flow hoặc warning.
Nếu đã reserved stock, cần release stock.
Nếu đã shipping, cancel trực tiếp có thể không được phép.
```

## 25.7. Refund

Điều kiện:

```text
paymentStatus in paid/partially_refunded
permission = order.refund
```

Modal fields:

```text
Refund amount
Refund reason
Refund method
Items to refund optional
Restock items optional
Internal note
```

Rule:

```text
Refund amount <= paid amount - refunded amount.
Refund cần confirm nguy hiểm.
Ghi audit log.
MVP có thể chỉ tạo refund request, chưa gọi cổng thanh toán.
```

---

# 26. Print and export

## 26.1. Print actions

```text
Print invoice
Print packing slip
Print shipping label optional
```

## 26.2. Invoice content

```text
Shop info
Order number
Customer info
Order items
Total
Payment method
Created date
```

## 26.3. Packing slip content

```text
Order number
Items
SKU
Variant
Quantity
Warehouse note
No sensitive payment info
```

## 26.4. Export order list

Export fields:

```text
Order number
Created date
Customer name
Phone masked/full by permission
Total
Payment status
Fulfillment status
Order status
Shipping city
Carrier
Tracking code
```

Rule:

```text
Export cần permission order.export.
Export dữ liệu cá nhân cần kiểm soát quyền.
Ghi audit log nếu export chứa dữ liệu nhạy cảm.
```

---

# 27. API contract tham khảo

## 27.1. Order list

```http
GET /api/admin/orders
```

Query params:

```text
search
order_status
payment_status
fulfillment_status
payment_method
shipping_method
date_from
date_to
amount_min
amount_max
risk_flag
assigned_staff_id
page
page_size
sort
```

Response:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 156,
    "totalPages": 8
  },
  "summary": {
    "all": 156,
    "pendingConfirmation": 18,
    "pendingPayment": 9,
    "shipping": 42,
    "deliveryIssues": 2
  }
}
```

## 27.2. Order detail

```http
GET /api/admin/orders/{orderId}
```

## 27.3. Confirm order

```http
POST /api/admin/orders/{orderId}/confirm
```

Body:

```json
{
  "note": "Đã gọi khách xác nhận."
}
```

## 27.4. Update status

```http
POST /api/admin/orders/{orderId}/status
```

Body:

```json
{
  "orderStatus": "confirmed",
  "fulfillmentStatus": "packing",
  "note": "Kho bắt đầu đóng gói."
}
```

## 27.5. Mark paid

```http
POST /api/admin/orders/{orderId}/mark-paid
```

Body:

```json
{
  "amount": 30980000,
  "paymentMethod": "bank_transfer",
  "paymentReference": "FT202606220001",
  "paidAt": "2026-06-22T10:00:00+07:00",
  "note": "Đã đối soát chuyển khoản."
}
```

## 27.6. Cancel order

```http
POST /api/admin/orders/{orderId}/cancel
```

Body:

```json
{
  "reason": "customer_requested",
  "restockItems": true,
  "notifyCustomer": true,
  "note": "Khách đổi ý."
}
```

## 27.7. Refund

```http
POST /api/admin/orders/{orderId}/refunds
```

Body:

```json
{
  "amount": 1000000,
  "reason": "partial_return",
  "restockItems": false,
  "note": "Hoàn tiền phụ kiện."
}
```

## 27.8. Add internal note

```http
POST /api/admin/orders/{orderId}/notes
```

Body:

```json
{
  "content": "Khách yêu cầu giao giờ hành chính."
}
```

## 27.9. Update tracking

```http
POST /api/admin/orders/{orderId}/tracking
```

Body:

```json
{
  "carrier": "GHTK",
  "trackingCode": "GHTK123456",
  "note": "Đã bàn giao vận chuyển."
}
```

---

# 28. Validation rules

## 28.1. Order update validation

```text
Không cho update status không hợp lệ.
Không cho mark paid amount âm.
Không cho refund vượt paid amount.
Không cho cancel completed order.
Không cho ship nếu thiếu item hoặc tồn kho chưa reserve.
Không cho delivered nếu chưa shipping.
Không cho edit address sau shipping nếu thiếu quyền.
```

## 28.2. Form validation

Cancel order:

```text
Reason required.
Note required nếu reason = Other.
```

Mark paid:

```text
Amount required.
Amount > 0.
Payment date required.
Payment reference required nếu policy bật.
```

Refund:

```text
Amount required.
Amount <= refundable amount.
Reason required.
```

Internal note:

```text
Content required.
Max length tuỳ config, ví dụ 1000 ký tự.
Không cho script/html nguy hiểm.
```

---

# 29. Security and privacy rules

## 29.1. Auth guard

Admin order pages chỉ truy cập được khi đã đăng nhập admin.

Nếu token hết hạn:

```text
Redirect to /admin/login
Show message: Session expired. Please log in again.
```

## 29.2. Permission guard

Backend phải check permission cho mọi action.

Frontend chỉ hỗ trợ UX.

## 29.3. Sensitive data

Dữ liệu nhạy cảm:

```text
Full phone
Full email
Full address
Payment reference
Refund details
Internal notes
```

Rule:

```text
Không hiển thị full phone/email trên list mặc định.
Không export dữ liệu nhạy cảm nếu thiếu quyền.
Không log dữ liệu thanh toán nhạy cảm ở console.
Không hiển thị raw gateway error cho user không có quyền.
```

## 29.4. Audit log

Mọi action quan trọng phải ghi audit:

```text
confirm order
cancel order
mark paid
refund
change address
update tracking
update status
export orders
print invoice optional
```

Audit fields:

```text
actor
role
action
target order
before value
after value
reason/note
timestamp
ip/device optional
```

---

# 30. Performance rules

Order list có thể rất lớn.

Rule:

```text
Phải phân trang.
Không load toàn bộ đơn.
Search debounce.
Filter query trên URL.
Table row không render quá nặng.
Ảnh item chỉ load thumbnail nếu cần.
Order detail lazy-load timeline dài nếu cần.
Export chạy async nếu dữ liệu lớn.
```

Cache:

```text
Order list: cache ngắn hoặc không cache tùy realtime.
Order detail: có thể cache ngắn nhưng phải refresh sau action.
Status summary: cache 30-60s nếu chấp nhận.
```

---

# 31. Accessibility rules

- Mọi button có accessible name.
- Icon-only action phải có `aria-label`.
- Badge không chỉ dựa vào màu.
- Table có header semantic.
- Modal trap focus.
- ESC đóng modal nếu không có dữ liệu chưa lưu.
- Error message liên kết với field.
- Timeline có cấu trúc đọc được.
- Toast không phải nơi duy nhất chứa lỗi quan trọng.
- Keyboard thao tác được filter, action menu, modal.

---

# 32. Responsive rules

## 32.1. Desktop

```text
Sidebar visible.
Order table đầy đủ.
Order detail có main content + side action panel.
Side action panel sticky.
```

## 32.2. Tablet

```text
Sidebar collapsed.
Order table compact hoặc horizontal scroll trong container.
Side panel chuyển dưới header hoặc vẫn sticky nếu đủ rộng.
```

## 32.3. Mobile

```text
Sidebar drawer.
Order list dùng card.
Filter dùng drawer/bottom sheet.
Order detail single column.
Action bar có thể sticky bottom.
Không overflow ngang toàn page.
```

Viewport bắt buộc kiểm tra:

```text
320px
375px
390px
768px
1024px
1440px
```

---

# 33. UI component structure

Tên component không bắt buộc, nhưng nên tách như sau:

```text
AdminOrderListPage
AdminOrderToolbar
AdminOrderQuickTabs
AdminOrderFilterDrawer
AdminOrderTable
AdminOrderRow
AdminOrderMobileCard
AdminOrderBulkActionBar
AdminOrderDetailPage
OrderDetailHeader
OrderRiskBanner
OrderStatusBadges
OrderActionPanel
OrderItemsSection
OrderSummarySection
OrderCustomerSection
OrderShippingSection
OrderPaymentSection
OrderFulfillmentSection
OrderWarrantySection
OrderTimeline
OrderInternalNotes
OrderAuditLog
ConfirmOrderModal
CancelOrderModal
MarkPaidModal
RefundModal
UpdateTrackingModal
PrintOrderActions
```

Shared components dùng lại:

```text
AdminLayout
AdminSidebar
AdminTopbar
DataTable
StatusBadge
ConfirmDialog
Modal
Drawer
Toast
Skeleton
EmptyState
ErrorState
Pagination
```

---

# 34. Page structure recommendation

## 34.1. Order list

```text
AdminOrderListPage
├── AdminPageHeader
├── OrderSummaryTabs
├── AdminOrderToolbar
│   ├── SearchInput
│   ├── StatusFilter
│   ├── PaymentFilter
│   ├── FulfillmentFilter
│   ├── DateRangeFilter
│   └── ExportButton
├── AppliedFilterChips
├── AdminOrderBulkActionBar
├── AdminOrderTable desktop/tablet
├── AdminOrderMobileList mobile
└── Pagination
```

## 34.2. Order detail

```text
AdminOrderDetailPage
├── OrderDetailHeader
├── OrderRiskBanner
├── OrderDetailLayout
│   ├── MainColumn
│   │   ├── OrderItemsSection
│   │   ├── OrderSummarySection
│   │   ├── CustomerSection
│   │   ├── ShippingSection
│   │   ├── PaymentSection
│   │   ├── FulfillmentSection
│   │   ├── WarrantySection
│   │   ├── TimelineSection
│   │   └── InternalNotesSection
│   └── SidePanel
│       ├── OrderActionPanel
│       ├── StatusSummaryCard
│       ├── StaffAssignmentCard optional
│       └── AuditSummaryCard
└── Modals
```

---

# 35. State management rules

## 35.1. Order list state

```text
searchQuery
filters
sort
pagination
selectedRows
loading
error
summaryCounts
```

## 35.2. Order detail state

```text
order
loading
error
actionLoading
activeModal
notesSubmitting
timelineLoading
```

## 35.3. After action success

Sau action:

```text
Refresh order detail.
Refresh order list summary nếu đang ở list.
Append/update timeline.
Show success toast.
Close modal.
```

## 35.4. After action failure

```text
Keep modal open.
Show error near form.
Do not lose input.
Allow retry.
Do not fake status update.
```

---

# 36. Analytics events

Admin analytics optional nhưng nên có:

```text
admin_order_list_viewed
admin_order_search_used
admin_order_filter_applied
admin_order_detail_viewed
admin_order_confirm_clicked
admin_order_confirmed
admin_order_cancel_clicked
admin_order_cancelled
admin_order_mark_paid_clicked
admin_order_marked_paid
admin_order_refund_created
admin_order_tracking_updated
admin_order_note_created
admin_order_export_clicked
```

Payload không chứa dữ liệu nhạy cảm nếu không cần.

Ví dụ:

```json
{
  "adminId": "admin_1",
  "role": "store_manager",
  "orderId": "order_1024",
  "action": "confirm",
  "currentStatus": "pending_confirmation"
}
```

---

# 37. Loading/empty/error state checklist

## 37.1. Order list

```text
Loading table skeleton.
Empty no orders.
Empty filtered.
API error with retry.
Permission denied.
```

## 37.2. Order detail

```text
Loading detail skeleton.
Order not found.
API error with retry.
Permission denied.
Partial section error, ví dụ timeline lỗi nhưng items vẫn hiển thị.
```

## 37.3. Modals

```text
Submit loading.
Validation error.
API error.
Success close + toast.
```

---

# 38. Playwright test specification

## 38.1. Order list tests

Test cases:

```text
Admin can view order list.
Admin can search by order number.
Admin can search by phone.
Admin can filter by order status.
Admin can filter by payment status.
Admin can filter by fulfillment status.
Admin can filter by date range.
Admin can clear filters.
Admin can open order detail.
Order list empty state appears.
Order list API error state appears.
Mobile order list has no horizontal overflow.
```

## 38.2. Order detail tests

Test cases:

```text
Admin can view order detail.
Order items are visible.
Customer section is visible.
Shipping address is visible.
Payment section is visible.
Timeline is visible.
Internal notes are visible.
Risk banner appears for high value order.
Order not found state appears.
```

## 38.3. Order action tests

Test cases:

```text
Admin can confirm pending order.
Confirm order opens modal.
Confirm success updates status.
Admin cannot confirm order if status is not pending.
Admin can add internal note.
Admin can mark bank transfer order as paid.
Mark paid requires amount.
Admin can cancel pending order.
Cancel order requires reason.
Refund action requires permission.
Danger actions show confirm modal.
```

## 38.4. Permission tests

Test cases:

```text
Viewer can view order but cannot update.
Sales Staff can confirm order but cannot refund.
Warehouse Staff can update fulfillment but cannot mark paid.
Accountant can mark paid/refund but cannot cancel without permission.
Unauthorized user redirected to admin login.
```

## 38.5. Responsive tests

Viewport:

```text
1440x900
1024x768
768x1024
390x844
375x812
320x568
```

Assertions:

```text
No page-level horizontal overflow.
Mobile list uses cards.
Filter drawer opens and closes.
Action modal fits viewport.
Sticky action bar does not cover content.
```

---

# 39. Visual regression checklist

Capture screenshots for:

```text
Order list desktop.
Order list mobile.
Order list with filters.
Order list empty state.
Order list error state.
Order detail desktop.
Order detail mobile.
Order detail high value warning.
Confirm order modal.
Cancel order modal.
Mark paid modal.
Timeline section.
Internal notes section.
```

Không approve snapshot nếu:

```text
Table overflow toàn page.
Mobile card bị cắt nội dung quan trọng.
Badge màu sai semantic.
Action nguy hiểm không nổi bật.
Modal bị vượt màn hình.
Customer phone/email hiển thị full ở list mặc định.
```

---

# 40. Definition of Done

Module Admin Order Management được coi là hoàn thành khi:

## 40.1. UI

```text
Order list đúng layout desktop/mobile.
Order detail đúng layout desktop/mobile.
Search/filter/sort/pagination hiển thị đúng.
Status badge đúng semantic.
Action panel rõ.
Timeline đọc được.
Internal notes dùng được.
Loading/empty/error states đầy đủ.
Không overflow ngang mobile.
```

## 40.2. Function

```text
Search hoạt động.
Filter hoạt động.
Pagination hoạt động.
Open detail hoạt động.
Confirm order hoạt động hoặc mock rõ.
Cancel order hoạt động hoặc mock rõ.
Mark paid hoạt động hoặc mock rõ.
Add note hoạt động hoặc mock rõ.
Print/export có action hoặc placeholder rõ.
```

## 40.3. Business rules

```text
Không cho transition sai.
Action nguy hiểm có confirm.
Cancel cần reason.
Mark paid cần amount.
Refund không vượt paid amount.
Status update ghi timeline.
Dữ liệu khách được mask theo quyền.
```

## 40.4. Security

```text
Auth guard.
Permission guard.
Không lộ sensitive data trên list.
Không log sensitive data ra console.
Audit log cho action quan trọng.
```

## 40.5. Test

```text
Playwright tests chính pass.
Visual desktop/mobile đã kiểm tra.
Không có console error nghiêm trọng.
Không xóa test để pass.
```

---

# 41. MVP scope

Nếu làm MVP trước, chỉ cần:

```text
Order list.
Search by order number/customer/phone.
Filter by order status/payment status/date.
Order table desktop.
Order card mobile.
Order detail page.
Customer/shipping/payment/items sections.
Order timeline basic.
Internal notes basic.
Confirm order.
Cancel order.
Mark paid for bank transfer/COD.
Update fulfillment: packing/shipping/delivered.
Loading/empty/error states.
Permission UI cơ bản.
```

Chưa cần ngay:

```text
Advanced refund automation.
Return management full flow.
Carrier API integration.
Multi-package shipment.
Serial/IMEI assignment.
Fraud scoring nâng cao.
Bulk status update phức tạp.
Advanced invoice template.
Partial shipment.
Partial refund by item.
```

---

# 42. Future extension

Sau MVP có thể mở rộng:

```text
Return/refund module riêng.
Shipping carrier integration.
Auto payment reconciliation.
Serial/IMEI assignment during packing.
Warranty case creation from order.
Order fraud/risk scoring.
Split shipment.
Backorder management.
Customer communication log.
Admin assignment/work queue.
SLA dashboard for order operation.
```

---

# 43. Ghi chú cho source clone nhiều ngành hàng

Phần lõi dùng chung cho mọi ngành:

```text
Order
OrderItem
Payment status
Fulfillment status
Order status
Customer snapshot
Shipping address
Timeline
Internal notes
Order actions
Permissions
Audit log
```

Phần riêng ngành đồ điện tử:

```text
Variant/spec snapshot hiển thị rõ.
Warranty snapshot trên từng item.
High value order warning.
Bank transfer verification rõ.
Inventory theo variant.
Serial/IMEI placeholder.
```

Khi clone sang ngành khác như thời trang, mỹ phẩm, thực phẩm:

```text
Không sửa core order flow.
Chỉ chỉnh item display, warranty section, risk flags, fulfillment policy, return policy.
```

---

# 44. Tóm tắt cho agent

Nếu agent chỉ đọc phần này, hãy nhớ:

```text
Admin Order Management là module vận hành đơn hàng.
Nó phải rõ, nhanh, chính xác và an toàn.
Tách order status, payment status, fulfillment status.
Không cho chuyển trạng thái bừa bãi.
Action nguy hiểm phải có confirm modal.
Dữ liệu khách phải được mask theo quyền.
Order item phải dùng snapshot.
Đồ điện tử cần warranty, high value warning, tồn kho theo variant.
Mobile không được overflow.
Mỗi màn phải có loading/empty/error state.
Phải có test cho list, detail, action, permission, responsive.
```
