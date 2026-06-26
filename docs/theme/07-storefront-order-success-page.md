# 07 - Storefront Order Success / Payment Result / Order Tracking Page

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> File này đặc tả nhóm màn hình sau khi khách hoàn tất checkout trong website bán đồ điện tử.
>
> File này kế thừa:
>
> - `../main/ecommerce_design_language.md` hoặc `../main/ecommerce_design_language.md`
> - `01-electronics-store-theme.md`
> - `06-storefront-checkout-page.md`
>
> Mục tiêu: agent có thể dựa vào file này để code đầy đủ luồng sau đặt hàng mà không phải đoán UI/UX.

---

## 1. Mục tiêu của nhóm trang

Sau khi khách bấm `Đặt hàng` ở checkout, hệ thống cần điều hướng khách đến một trong các trạng thái kết quả.

Các trang cần xử lý:

```text
1. Order Success Page
2. Bank Transfer Pending Page
3. Payment Pending Page
4. Payment Failed Page
5. Payment Cancelled Page
6. Order Detail / Order Tracking Page
7. Guest Order Lookup Page
```

Mục tiêu chính:

```text
Xác nhận đơn đã được ghi nhận
Hiển thị mã đơn hàng rõ ràng
Hướng dẫn khách bước tiếp theo
Cho khách biết trạng thái thanh toán
Cho khách biết trạng thái xử lý/giao hàng
Tạo cảm giác tin cậy sau khi đặt hàng
Giảm việc khách phải gọi shop hỏi lại
Tăng khả năng mua tiếp / xem sản phẩm liên quan
```

---

## 2. Nguyên tắc thiết kế riêng cho màn sau đặt hàng

Trang sau đặt hàng không nên quá rối. Người dùng vừa hoàn thành một hành động quan trọng, UI cần trả lời ngay các câu hỏi:

```text
Đơn của tôi đã được tạo chưa?
Mã đơn là gì?
Tôi phải thanh toán chưa?
Tôi chuyển khoản vào đâu?
Khi nào shop xác nhận?
Tôi có thể theo dõi đơn ở đâu?
Tôi có thể liên hệ ai nếu có vấn đề?
```

Nguyên tắc:

```text
Thông tin quan trọng nhất nằm trên cùng
Mã đơn hàng phải dễ copy
Tổng tiền phải rõ
Trạng thái thanh toán phải rõ
Trạng thái đơn hàng phải rõ
CTA tiếp theo phải rõ
Không ép khách đăng nhập sau khi đặt hàng
Không để khách bối rối nếu thanh toán online fail
```

---

## 3. Route đề xuất

```text
/orders/success/{orderNumber}
/orders/payment-pending/{orderNumber}
/orders/payment-failed/{orderNumber}
/orders/payment-cancelled/{orderNumber}
/orders/{orderNumber}
/order-lookup
```

Hoặc nếu muốn gom chung:

```text
/orders/result?order_number=DH000123&status=success
/orders/result?order_number=DH000123&status=bank_transfer_pending
/orders/result?order_number=DH000123&status=payment_failed
```

Khuyến nghị:

```text
Dùng route riêng cho từng trạng thái nếu muốn SEO/analytics rõ.
Dùng route chung nếu muốn code gọn.
```

---

## 4. Page: Order Success Page

### 4.1. Mục đích

Hiển thị khi đơn hàng đã được tạo thành công và không cần thanh toán online ngay, hoặc thanh toán đã thành công.

Áp dụng cho:

```text
COD order created
Online payment successful
Bank transfer order created but chưa cần hiển thị payment pending riêng
```

### 4.2. Desktop layout

```text
┌──────────────────────────────────────────────┐
│ Header                                       │
├──────────────────────────────────────────────┤
│ Success Hero                                 │
│ - Icon success                               │
│ - Title: Đặt hàng thành công                 │
│ - Subtitle                                   │
│ - Order number + copy button                 │
├──────────────────────────────────────────────┤
│ Main content                                 │
│ ┌──────────────────────┐ ┌─────────────────┐ │
│ │ Order information    │ │ Order summary   │ │
│ │ Customer info        │ │ Payment summary │ │
│ │ Shipping address     │ │ CTA box         │ │
│ │ Delivery method      │ │ Support box     │ │
│ └──────────────────────┘ └─────────────────┘ │
├──────────────────────────────────────────────┤
│ Next step timeline                           │
├──────────────────────────────────────────────┤
│ Recommended products / accessories           │
├──────────────────────────────────────────────┤
│ Footer                                       │
└──────────────────────────────────────────────┘
```

### 4.3. Mobile layout

```text
Header compact
Success hero
Order number card
Order summary
Payment status
Shipping address
Next step timeline
Support box
CTA buttons
Recommended products
Footer
```

Mobile không dùng layout 2 cột.

---

## 5. Component: Success Hero

### 5.1. Thành phần

```text
Success icon
Title
Subtitle
Order number
Copy order number button
Short status badge
```

### 5.2. Nội dung đề xuất

Title:

```text
Đặt hàng thành công
```

Subtitle cho COD:

```text
Cảm ơn bạn đã đặt hàng. Chúng tôi sẽ liên hệ xác nhận đơn trong thời gian sớm nhất.
```

Subtitle cho online payment success:

```text
Thanh toán của bạn đã được ghi nhận. Đơn hàng đang chờ xử lý.
```

Order number format:

```text
Mã đơn hàng: DH202606220001
```

### 5.3. Rule

```text
Mã đơn hàng luôn hiển thị trên màn đầu tiên
Có nút copy cạnh mã đơn hàng
Sau khi copy, hiển thị toast: Đã sao chép mã đơn hàng
Không hiển thị thông tin nội bộ như database id
Không hiển thị payment transaction id nếu không cần thiết
```

### 5.4. Visual

```text
Icon success dùng màu success token
Card nền nhẹ
Title dùng heading lớn
Order number dùng monospace hoặc font số dễ đọc
```

---

## 6. Component: Order Summary Card

### 6.1. Thành phần

```text
Danh sách sản phẩm
Số lượng
Giá từng sản phẩm
Tạm tính
Giảm giá
Phí vận chuyển
Tổng thanh toán
```

### 6.2. Rule hiển thị

```text
Tên sản phẩm hiển thị tối đa 2 dòng
Nếu sản phẩm có biến thể, hiển thị biến thể dưới tên
Ảnh sản phẩm nhỏ nhưng rõ
Tổng thanh toán phải nổi bật nhất trong card
Không cho phép sửa số lượng ở trang success
Không cho phép xóa sản phẩm ở trang success
```

### 6.3. Ví dụ item

```text
Laptop Dell Inspiron 15
Core i5 / 16GB / SSD 512GB
Số lượng: 1
15.990.000đ
```

### 6.4. Empty fallback

Trường hợp API không trả được item list nhưng order tồn tại:

```text
Hiển thị mã đơn + tổng tiền nếu có
Hiển thị thông báo: Không thể tải danh sách sản phẩm. Vui lòng thử lại hoặc liên hệ hỗ trợ.
Không làm mất thông tin mã đơn hàng
```

---

## 7. Component: Customer Information Card

### 7.1. Thành phần

```text
Tên người nhận
Số điện thoại
Email nếu có
Địa chỉ giao hàng
Ghi chú đơn hàng nếu có
```

### 7.2. Rule bảo mật

```text
Không hiển thị quá nhiều thông tin nhạy cảm nếu trang có thể truy cập qua link public
Số điện thoại có thể mask nếu không có token đăng nhập
Ví dụ: 090****000
Email có thể mask nếu cần
```

### 7.3. Guest access rule

Nếu khách không đăng nhập nhưng có order token trong URL:

```text
Cho xem thông tin cơ bản của đơn
Không cho sửa thông tin
Không cho xem thông tin tài khoản khác
Token phải có expiry
```

---

## 8. Component: Payment Status Card

### 8.1. Các trạng thái thanh toán

```text
unpaid
pending
paid
failed
cancelled
refunded
partially_refunded
cod_pending
bank_transfer_pending
```

### 8.2. Mapping label

```text
unpaid = Chưa thanh toán
pending = Đang chờ thanh toán
paid = Đã thanh toán
failed = Thanh toán thất bại
cancelled = Thanh toán đã hủy
refunded = Đã hoàn tiền
partially_refunded = Hoàn tiền một phần
cod_pending = Thanh toán khi nhận hàng
bank_transfer_pending = Chờ chuyển khoản
```

### 8.3. Visual token

```text
paid dùng success
pending dùng warning
failed dùng danger
cancelled dùng neutral
cod_pending dùng info
bank_transfer_pending dùng warning/info
```

### 8.4. Rule

```text
Trạng thái thanh toán phải hiển thị rõ gần tổng tiền
Nếu đã thanh toán, hiển thị thời gian thanh toán nếu có
Nếu chờ chuyển khoản, hiển thị hướng dẫn chuyển khoản
Nếu thất bại, hiển thị nút thử lại thanh toán
Nếu COD, hiển thị thông báo chuẩn bị tiền khi nhận hàng
```

---

## 9. Page: Bank Transfer Pending Page

### 9.1. Mục đích

Dành cho đơn hàng chọn phương thức chuyển khoản ngân hàng.

Trang này phải hướng dẫn khách chuyển khoản chính xác.

### 9.2. Layout

```text
Pending hero
Bank transfer instruction card
QR code card
Order summary
Important note
Support box
CTA buttons
```

### 9.3. Bank Transfer Instruction Card

Thành phần:

```text
Tên ngân hàng
Tên chủ tài khoản
Số tài khoản
Số tiền cần chuyển
Nội dung chuyển khoản
QR code nếu có
Nút copy từng trường
```

Ví dụ:

```text
Ngân hàng: Vietcombank
Chủ tài khoản: CONG TY ABC
Số tài khoản: 0123456789
Số tiền: 15.990.000đ
Nội dung: DH202606220001
```

### 9.4. Rule bắt buộc

```text
Nội dung chuyển khoản phải chứa mã đơn hàng
Số tiền phải đúng tổng cần thanh toán
Có nút copy số tài khoản
Có nút copy nội dung chuyển khoản
Có cảnh báo không chuyển thiếu/sai nội dung
Có note thời gian xác nhận sau khi chuyển khoản
```

### 9.5. Cảnh báo đề xuất

```text
Vui lòng chuyển đúng số tiền và ghi đúng nội dung chuyển khoản để đơn hàng được xác nhận nhanh hơn.
```

### 9.6. QR code rule

```text
Nếu có QR code, hiển thị rõ và đủ lớn trên mobile
Có nút tải QR nếu cần
Nếu QR load lỗi, vẫn hiển thị thông tin bank transfer text
Không phụ thuộc hoàn toàn vào QR
```

---

## 10. Page: Payment Pending Page

### 10.1. Mục đích

Dành cho thanh toán online đang chờ callback từ cổng thanh toán.

Ví dụ:

```text
Người dùng đã được redirect về site nhưng webhook chưa cập nhật
Gateway đang xử lý
Thanh toán qua ví có delay
```

### 10.2. UI

```text
Icon pending/loading
Title: Đang xác nhận thanh toán
Subtitle
Order number
Payment status card
Refresh status button
Back to home button
Support box
```

### 10.3. Rule

```text
Không tự động tạo đơn mới
Không cho khách thanh toán lại ngay nếu payment vẫn pending
Có thể auto refresh trạng thái mỗi 5-10 giây trong giới hạn
Có nút kiểm tra lại trạng thái
Nếu quá thời gian timeout, chuyển sang trạng thái cần liên hệ hỗ trợ hoặc cho thử lại
```

### 10.4. Timeout rule

```text
Nếu pending dưới 2 phút: tiếp tục chờ
Nếu pending quá 2 phút: hiển thị nút Kiểm tra lại
Nếu pending quá 15 phút: hướng dẫn liên hệ hỗ trợ hoặc thử thanh toán lại nếu backend cho phép
```

---

## 11. Page: Payment Failed Page

### 11.1. Mục đích

Hiển thị khi thanh toán online thất bại nhưng đơn có thể vẫn được giữ tạm.

### 11.2. Layout

```text
Failed hero
Reason card nếu có
Order summary
Retry payment CTA
Change payment method CTA
Contact support
Back to cart / home
```

### 11.3. Message đề xuất

Title:

```text
Thanh toán chưa thành công
```

Subtitle:

```text
Đơn hàng của bạn chưa được thanh toán. Bạn có thể thử lại hoặc chọn phương thức thanh toán khác.
```

### 11.4. Rule

```text
Không nói đơn hàng đã hủy nếu backend chưa hủy
Hiển thị trạng thái giữ hàng nếu có
Nếu đơn còn hiệu lực, cho phép retry payment
Nếu đơn hết hạn, hướng khách tạo đơn mới
Nếu lỗi đến từ gateway, hiển thị message thân thiện, không show raw error
```

### 11.5. CTA

```text
Thử thanh toán lại
Chọn phương thức khác
Về giỏ hàng
Liên hệ hỗ trợ
```

---

## 12. Page: Payment Cancelled Page

### 12.1. Mục đích

Hiển thị khi khách tự hủy thanh toán ở gateway.

### 12.2. Message đề xuất

```text
Bạn đã hủy thanh toán
Đơn hàng chưa được thanh toán. Bạn có thể tiếp tục thanh toán hoặc chọn phương thức khác.
```

### 12.3. Rule

```text
Phân biệt cancelled với failed
Cancelled không phải lỗi hệ thống
Không hiển thị màu danger quá mạnh
Cho phép resume payment nếu order còn hiệu lực
```

---

## 13. Page: Order Detail / Order Tracking Page

### 13.1. Mục đích

Cho khách xem trạng thái đơn hàng sau khi đã đặt.

Áp dụng cho:

```text
Khách đăng nhập xem lịch sử đơn
Khách guest truy cập bằng order token
Admin có thể có màn riêng, không dùng page này
```

### 13.2. Layout desktop

```text
Header
Order status hero
Order tracking timeline
Order information grid
Shipment card
Payment card
Order items
Support card
Footer
```

### 13.3. Order status

Các trạng thái đơn hàng chuẩn:

```text
created
pending_confirmation
confirmed
processing
packed
shipped
delivered
completed
cancelled
returned
refund_pending
refunded
```

Mapping label:

```text
created = Đã tạo đơn
pending_confirmation = Chờ xác nhận
confirmed = Đã xác nhận
processing = Đang xử lý
packed = Đã đóng gói
shipped = Đang giao hàng
delivered = Đã giao hàng
completed = Hoàn thành
cancelled = Đã hủy
returned = Đã hoàn hàng
refund_pending = Chờ hoàn tiền
refunded = Đã hoàn tiền
```

### 13.4. Tracking timeline

Timeline nên hiển thị:

```text
Thời gian
Trạng thái
Mô tả
Ghi chú nếu có
```

Ví dụ:

```text
22/06/2026 16:30 - Đã tạo đơn
22/06/2026 16:45 - Đã xác nhận
23/06/2026 09:15 - Đang giao hàng
```

### 13.5. Rule timeline

```text
Trạng thái mới nhất nằm trên cùng hoặc highlight rõ
Không hiển thị step tương lai như đã hoàn thành
Step tương lai có thể hiển thị dạng disabled nếu dùng stepper
Nếu không có tracking từ đơn vị vận chuyển, hiển thị trạng thái nội bộ của shop
```

---

## 14. Component: Shipment Card

### 14.1. Thành phần

```text
Shipping method
Shipping provider
Tracking code
Tracking URL nếu có
Estimated delivery date
Shipping status
Receiver address
```

### 14.2. Rule

```text
Nếu có tracking code, có nút copy
Nếu có tracking URL, mở ở tab mới
Nếu chưa có tracking code, hiển thị: Mã vận đơn sẽ được cập nhật sau khi đơn được bàn giao cho đơn vị vận chuyển
Không để field rỗng gây khó hiểu
```

---

## 15. Component: Support Box

### 15.1. Mục đích

Sau khi đặt hàng, khách có thể cần hỗ trợ. Support box nên dễ thấy nhưng không lấn át thông tin đơn.

### 15.2. Nội dung

```text
Hotline
Email
Chat link
Giờ làm việc
Mã đơn hàng được nhắc lại
```

### 15.3. CTA

```text
Gọi hỗ trợ
Chat với shop
Gửi email
```

### 15.4. Rule

```text
Khi khách bấm hỗ trợ, nếu có thể, truyền kèm mã đơn hàng
Không yêu cầu khách phải copy mã đơn thủ công nếu có deep link hỗ trợ
```

---

## 16. Page: Guest Order Lookup Page

### 16.1. Mục đích

Cho khách không đăng nhập kiểm tra đơn hàng.

### 16.2. Form fields

```text
Mã đơn hàng
Số điện thoại hoặc email đặt hàng
```

### 16.3. Validation

```text
Mã đơn hàng không được trống
Số điện thoại/email không được trống
Nếu dùng số điện thoại, validate format cơ bản
Nếu dùng email, validate email format
```

### 16.4. Error messages

```text
Không tìm thấy đơn hàng phù hợp
Thông tin xác thực không đúng
Vui lòng kiểm tra mã đơn hàng hoặc số điện thoại/email
```

### 16.5. Security rule

```text
Không chỉ dùng mã đơn hàng để xem chi tiết đầy đủ
Phải xác minh thêm số điện thoại/email hoặc token
Rate limit lookup API
Không trả về thông tin nhạy cảm nếu thông tin xác thực sai
```

---

## 17. Data contract

### 17.1. OrderResult model

```ts
export type OrderResultStatus =
  | 'success'
  | 'bank_transfer_pending'
  | 'payment_pending'
  | 'payment_failed'
  | 'payment_cancelled';

export interface OrderResult {
  orderNumber: string;
  resultStatus: OrderResultStatus;
  orderStatus: OrderStatus;
  paymentStatus: PaymentStatus;
  paymentMethod: PaymentMethod;
  customer: OrderCustomer;
  shippingAddress: OrderAddress;
  billingAddress?: OrderAddress;
  items: OrderItemSnapshot[];
  pricing: OrderPricing;
  paymentInstruction?: BankTransferInstruction;
  shipment?: ShipmentInfo;
  createdAt: string;
  updatedAt: string;
  expiresAt?: string;
}
```

### 17.2. OrderStatus

```ts
export type OrderStatus =
  | 'created'
  | 'pending_confirmation'
  | 'confirmed'
  | 'processing'
  | 'packed'
  | 'shipped'
  | 'delivered'
  | 'completed'
  | 'cancelled'
  | 'returned'
  | 'refund_pending'
  | 'refunded';
```

### 17.3. PaymentStatus

```ts
export type PaymentStatus =
  | 'unpaid'
  | 'pending'
  | 'paid'
  | 'failed'
  | 'cancelled'
  | 'refunded'
  | 'partially_refunded'
  | 'cod_pending'
  | 'bank_transfer_pending';
```

### 17.4. PaymentMethod

```ts
export type PaymentMethod =
  | 'cod'
  | 'bank_transfer'
  | 'vnpay'
  | 'momo'
  | 'zalopay'
  | 'stripe'
  | 'paypal';
```

### 17.5. OrderCustomer

```ts
export interface OrderCustomer {
  id?: string;
  fullName: string;
  phone: string;
  email?: string;
  isGuest: boolean;
}
```

### 17.6. OrderAddress

```ts
export interface OrderAddress {
  fullName: string;
  phone: string;
  street: string;
  ward?: string;
  district?: string;
  city: string;
  country: string;
  postalCode?: string;
}
```

### 17.7. OrderItemSnapshot

```ts
export interface OrderItemSnapshot {
  id: string;
  productId: string;
  variantId?: string;
  productName: string;
  productSlug?: string;
  imageUrl: string;
  selectedSpecs?: string[];
  sku?: string;
  warrantyText?: string;
  unitPrice: number;
  quantity: number;
  lineTotal: number;
}
```

### 17.8. OrderPricing

```ts
export interface OrderPricing {
  subtotal: number;
  discountTotal: number;
  shippingFee: number;
  taxTotal?: number;
  grandTotal: number;
  currency: string;
}
```

### 17.9. BankTransferInstruction

```ts
export interface BankTransferInstruction {
  bankName: string;
  accountName: string;
  accountNumber: string;
  amount: number;
  transferContent: string;
  qrCodeUrl?: string;
  expireAt?: string;
}
```

### 17.10. ShipmentInfo

```ts
export interface ShipmentInfo {
  providerName?: string;
  trackingCode?: string;
  trackingUrl?: string;
  methodName: string;
  estimatedDeliveryDate?: string;
  shippingStatus: string;
}
```

---

## 18. API contract

### 18.1. Get order result

```http
GET /api/v1/orders/{orderNumber}/result
```

Response:

```json
{
  "orderNumber": "DH202606220001",
  "resultStatus": "bank_transfer_pending",
  "orderStatus": "pending_confirmation",
  "paymentStatus": "bank_transfer_pending",
  "paymentMethod": "bank_transfer",
  "pricing": {
    "subtotal": 15990000,
    "discountTotal": 0,
    "shippingFee": 30000,
    "grandTotal": 16020000,
    "currency": "VND"
  }
}
```

### 18.2. Get order detail

```http
GET /api/v1/orders/{orderNumber}
```

Auth rule:

```text
Logged-in customer can access own order
Guest can access with secure token
Admin endpoint should be separate
```

### 18.3. Retry payment

```http
POST /api/v1/orders/{orderNumber}/payments/retry
```

Request:

```json
{
  "paymentMethod": "vnpay"
}
```

Response:

```json
{
  "redirectUrl": "https://payment-gateway.example/checkout/..."
}
```

### 18.4. Change payment method

```http
POST /api/v1/orders/{orderNumber}/payments/change-method
```

Request:

```json
{
  "paymentMethod": "bank_transfer"
}
```

### 18.5. Guest lookup

```http
POST /api/v1/orders/lookup
```

Request:

```json
{
  "orderNumber": "DH202606220001",
  "identity": "0900000000"
}
```

Response:

```json
{
  "accessToken": "short_lived_order_access_token",
  "orderNumber": "DH202606220001"
}
```

---

## 19. State handling

### 19.1. Loading state

```text
Show skeleton cho hero, summary, order items
Không show layout nhảy mạnh
Không show empty state khi đang loading
```

### 19.2. Not found state

Khi order không tồn tại:

```text
Title: Không tìm thấy đơn hàng
Message: Vui lòng kiểm tra lại mã đơn hoặc liên hệ hỗ trợ.
CTA: Tra cứu đơn hàng
CTA: Về trang chủ
```

### 19.3. Unauthorized state

Khi khách không có quyền xem đơn:

```text
Title: Không thể xem đơn hàng
Message: Bạn cần đăng nhập hoặc xác minh thông tin đơn hàng để tiếp tục.
CTA: Đăng nhập
CTA: Tra cứu đơn hàng
```

### 19.4. API error state

```text
Title: Không thể tải thông tin đơn hàng
Message: Vui lòng thử lại sau.
CTA: Thử lại
```

### 19.5. Partial data state

Nếu chỉ có order summary nhưng thiếu tracking:

```text
Vẫn hiển thị order summary
Shipment card hiển thị fallback
Không làm fail toàn trang
```

---

## 20. Responsive rules

### 20.1. Desktop

```text
Max content width: theo design token container-lg hoặc container-xl
Hero full width trong container
Main content có thể chia 2 cột
Cột trái chiếm khoảng 65%
Cột phải chiếm khoảng 35%
Order summary card có thể sticky trong viewport nếu nội dung dài
```

### 20.2. Tablet

```text
Giảm khoảng cách ngang
Có thể giữ 2 cột nếu đủ rộng
Nếu dưới breakpoint tablet, chuyển về 1 cột
```

### 20.3. Mobile

```text
Tất cả card xếp 1 cột
Order summary không sticky side
CTA chính nằm dưới hero và cuối trang
Bank transfer QR chiếm đủ rộng nhưng không overflow
Không có horizontal scroll
```

Mobile viewport cần test:

```text
375px
390px
414px
```

---

## 21. Accessibility rules

```text
Success/failed/pending icon phải có text label tương ứng
Không dùng màu là tín hiệu duy nhất
Mã đơn có label rõ ràng
Nút copy có accessible name
Toast copy phải dùng aria-live polite
Timeline dùng semantic list hoặc ordered list
Form tra cứu đơn dùng label thật
Error message liên kết với input bằng aria-describedby
Focus state phải rõ
Sau khi load result, heading chính có thể nhận focus nếu cần
```

Ví dụ accessible name:

```text
Copy mã đơn hàng DH202606220001
Copy số tài khoản ngân hàng
Copy nội dung chuyển khoản
```

---

## 22. SEO rules

Các trang order cá nhân thường không nên index.

```text
Use noindex, nofollow cho order success/detail nếu chứa thông tin cá nhân
Không đưa mã đơn hàng vào meta description public
Không render dữ liệu cá nhân cho crawler
```

Meta đề xuất:

```html
<meta name="robots" content="noindex,nofollow" />
```

---

## 23. Security & privacy rules

```text
Không hiển thị database id
Không hiển thị raw payment gateway error
Không hiển thị full token/callback params
Không cho xem order chỉ bằng orderNumber nếu không đăng nhập hoặc không có token xác thực
Mask thông tin nhạy cảm trong một số context
Rate limit API tra cứu đơn hàng
Order access token phải ngắn hạn
Không lưu thông tin bank/payment nhạy cảm ở client storage
```

---

## 24. Analytics events

Nên track:

```text
order_success_viewed
bank_transfer_instruction_viewed
bank_transfer_copy_account_clicked
bank_transfer_copy_content_clicked
payment_failed_viewed
payment_retry_clicked
payment_method_change_clicked
order_tracking_viewed
guest_order_lookup_submitted
support_contact_clicked
continue_shopping_clicked
```

Event payload không chứa dữ liệu nhạy cảm.

Payload gợi ý:

```json
{
  "order_number_hash": "hash_value",
  "payment_method": "bank_transfer",
  "payment_status": "bank_transfer_pending",
  "grand_total_bucket": "10m_20m"
}
```

Không gửi:

```text
Số điện thoại
Email
Địa chỉ
Mã đơn raw nếu không cần
```

---

## 25. Component list

```text
OrderResultHero
OrderNumberCopy
OrderSummaryCard
OrderItemSnapshotCard
PaymentStatusCard
BankTransferInstructionCard
QRCodeBox
CustomerInfoCard
ShippingAddressCard
ShipmentCard
OrderTimeline
SupportBox
OrderResultCTAGroup
GuestOrderLookupForm
RecommendedProductsSection
```

---

## 26. Agent implementation rules

Khi agent code nhóm trang này, bắt buộc tuân thủ:

```text
Không tự ý đổi trạng thái order/payment đã định nghĩa
Không hard-code bank info trong component nếu dữ liệu đến từ API/admin config
Không hiển thị raw payment error
Không bỏ qua trạng thái payment failed/cancelled/pending
Không chỉ làm success page mà bỏ các trạng thái lỗi
Không cho guest xem order detail chỉ bằng order number
Không làm mất mã đơn hàng khi API trả partial data
Không để QR code gây overflow mobile
Không dùng màu làm tín hiệu duy nhất cho success/fail
```

Agent phải báo cáo sau khi làm:

```text
File đã sửa
Route đã thêm
Component đã thêm
API/mock đã thêm
Test đã chạy
Screenshot desktop/mobile nếu có
Các trạng thái đã cover
```

---

## 27. Playwright test specification

### 27.1. Order success test

```text
Given order result status success
When user opens /orders/success/DH202606220001
Then success title is visible
And order number is visible
And order summary is visible
And payment status is visible
And CTA continue shopping is visible
```

### 27.2. Copy order number test

```text
Given order success page
When user clicks copy order number
Then copy toast is visible
And copied text equals order number
```

### 27.3. Bank transfer pending test

```text
Given order payment method bank_transfer
When user opens bank transfer pending page
Then bank name is visible
And account number is visible
And transfer content is visible
And grand total is visible
And copy account button works
And copy transfer content button works
```

### 27.4. QR fallback test

```text
Given QR image fails to load
When user opens bank transfer pending page
Then text transfer instruction is still visible
And user can copy account number
```

### 27.5. Payment failed test

```text
Given payment status failed
When user opens payment failed page
Then failed message is visible
And retry payment button is visible
And change payment method button is visible
```

### 27.6. Payment cancelled test

```text
Given payment status cancelled
When user opens payment cancelled page
Then cancelled message is visible
And resume payment button is visible if order is still valid
```

### 27.7. Payment pending test

```text
Given payment status pending
When user opens payment pending page
Then pending message is visible
And check status button is visible
And page does not create duplicate order
```

### 27.8. Order tracking test

```text
Given order has status history
When user opens order detail page
Then timeline is visible
And latest status is highlighted
And order items are visible
And shipment information is visible or fallback is visible
```

### 27.9. Guest lookup success test

```text
Given valid order number and phone
When user submits guest lookup form
Then user is redirected to order detail with short-lived token
```

### 27.10. Guest lookup failure test

```text
Given invalid order number or phone
When user submits guest lookup form
Then error message is visible
And no order detail is displayed
```

### 27.11. Responsive test

```text
Given viewport 375px
When user opens each order result page
Then there is no horizontal overflow
And primary information is visible without layout breaking
```

---

## 28. Visual regression checklist

Chụp screenshot cho các trạng thái:

```text
Order success desktop
Order success mobile
Bank transfer pending desktop
Bank transfer pending mobile
Payment pending desktop
Payment failed desktop
Payment cancelled desktop
Order tracking desktop
Order tracking mobile
Guest lookup form mobile
```

Các lỗi visual cần bắt:

```text
Order number bị cắt
QR code overflow
Timeline lệch
Summary card quá rộng
Mobile có horizontal scroll
CTA bị che
Text trạng thái quá nhỏ
Card spacing không đồng nhất
```

---

## 29. Definition of Done

Một implementation được coi là hoàn thành khi:

```text
Có route cho success/pending/failed/cancelled/order tracking/guest lookup
Hiển thị được mã đơn hàng
Hiển thị được tổng tiền
Hiển thị được trạng thái thanh toán
Hiển thị được trạng thái đơn hàng
Bank transfer có đủ hướng dẫn chuyển khoản
Payment failed có retry/change method CTA
Payment pending không tạo đơn trùng
Order tracking có timeline
Guest lookup có xác thực order number + phone/email
Không có horizontal overflow mobile
Có loading/empty/error/unauthorized state
Có accessibility label cho nút copy và trạng thái
Có noindex cho trang chứa thông tin đơn hàng
Có Playwright test cho các trạng thái chính
Có visual screenshot desktop/mobile cho trạng thái chính
```

---

## 30. MVP scope

Trong MVP, bắt buộc có:

```text
Order Success Page
Bank Transfer Pending Page
Payment Failed Page đơn giản
Order Detail Page cơ bản
Guest Order Lookup Page cơ bản
Copy order number
Copy bank account
Copy transfer content
Mobile responsive
```

Có thể để sau:

```text
Auto refresh payment pending nâng cao
Deep integration shipping provider
Full refund tracking
Advanced recommendation sau khi mua
Live chat integration
Invoice download
```

---

## 31. Ghi chú cho source clone nhiều ngành hàng

Nhóm trang này gần như tái sử dụng được cho mọi loại web bán hàng.

Các phần cần biến thành config:

```text
Bank transfer information
Support hotline/email
Order status labels
Payment method labels
Shipment provider labels
Recommended products strategy
Brand tone copywriting
```

Các phần không nên hard-code theo đồ điện tử:

```text
Tên sản phẩm laptop/phone
Warranty text cụ thể
Brand cụ thể
Shipping provider cụ thể
Bank account cụ thể
```

Phần riêng cho đồ điện tử có thể bổ sung:

```text
Warranty text trong order item
Serial/IMEI note nếu có
Installation/support service note nếu là thiết bị cần lắp đặt
Accessory recommendation sau khi mua laptop/phone
```
