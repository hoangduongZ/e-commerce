# 06-storefront-checkout-page.md

> Electronics Storefront Checkout Page Specification  
> Dựa trên: `ecommerce_design_language.md`, `01-electronics-store-theme.md`, `05-storefront-cart-page.md`  
> Mục tiêu: đặc tả đủ chi tiết để coding agent có thể code trang checkout từ đầu đến cuối, không phụ thuộc framework frontend/backend.

---

## 1. Vai trò của trang checkout

Trang checkout là bước cuối trong flow mua hàng:

```text
Product Detail
→ Add to Cart
→ Cart
→ Checkout
→ Order Success / Payment Pending / Payment Failed
```

Mục tiêu chính của trang này:

1. Thu thập thông tin người nhận.
2. Xác nhận địa chỉ giao hàng.
3. Chọn phương thức vận chuyển.
4. Chọn phương thức thanh toán.
5. Kiểm tra tồn kho, giá, coupon trước khi tạo đơn.
6. Hiển thị tóm tắt đơn hàng rõ ràng.
7. Tạo đơn hàng an toàn, tránh double submit.
8. Điều hướng sang trang kết quả phù hợp.

Với web bán đồ điện tử, checkout cần tạo cảm giác **an toàn, rõ ràng, đáng tin** vì giá trị đơn hàng thường cao hơn các ngành hàng phổ thông.

---

## 2. Nguyên tắc thiết kế riêng cho checkout đồ điện tử

Checkout không phải nơi để trang trí quá nhiều. Checkout là nơi giảm ma sát để khách hoàn tất đơn hàng.

### 2.1. Cảm giác giao diện

Checkout phải có cảm giác:

```text
Clear
Secure
Focused
Trustworthy
Low distraction
Mobile-first
```

Không nên có:

```text
Banner sale lớn
Carousel phức tạp
Popup marketing gây nhiễu
Navigation quá nhiều link
Animation thừa
```

### 2.2. Ưu tiên thị giác

Thứ tự ưu tiên trên trang checkout:

```text
1. Thông tin người nhận
2. Địa chỉ giao hàng
3. Phương thức vận chuyển
4. Phương thức thanh toán
5. Tổng tiền cuối cùng
6. Nút đặt hàng
7. Chính sách bảo hành/đổi trả
```

### 2.3. Rule cốt lõi

- Không cho đặt hàng nếu giỏ hàng rỗng.
- Không cho đặt hàng nếu có item hết hàng bắt buộc xử lý.
- Không cho đặt hàng nếu giá/tồn kho thay đổi mà user chưa xác nhận.
- Không submit nhiều lần khi user bấm liên tục.
- Tổng tiền cuối cùng phải rõ ràng trước khi bấm đặt hàng.
- Lỗi validation phải hiển thị ngay gần field bị lỗi.
- Mobile phải có sticky order summary hoặc sticky CTA hợp lý.

---

## 3. Phạm vi trang

### 3.1. Trong scope MVP

MVP checkout cần có:

```text
Guest checkout
Customer checkout nếu đã login
Thông tin người nhận
Địa chỉ giao hàng
Phương thức vận chuyển đơn giản
COD
Chuyển khoản ngân hàng
Coupon đã áp dụng từ cart hoặc nhập ở checkout
Tóm tắt đơn hàng
Đặt hàng thành công
Validation form
Inventory re-check
Price re-check
```

### 3.2. Ngoài scope MVP nhưng cần chừa thiết kế

Có thể mở rộng sau:

```text
VNPay
MoMo
ZaloPay
Stripe
Trả góp
Xuất hóa đơn VAT
Saved cards
Loyalty points
Multiple shipping addresses
Pickup at store
Scheduled delivery
```

---

## 4. Route và điều hướng

### 4.1. Route đề xuất

```text
/checkout
/checkout/success?orderNumber=...
/checkout/payment-pending?orderNumber=...
/checkout/payment-failed?orderNumber=...
```

### 4.2. Điều kiện vào checkout

Khi user vào `/checkout`, hệ thống phải kiểm tra:

```text
Cart tồn tại
Cart không rỗng
Cart có ít nhất 1 item hợp lệ
Item chưa bị xóa khỏi catalog
Item chưa hết hàng
Giá hiện tại có thể tính được
```

Nếu không hợp lệ:

```text
Cart rỗng → redirect /cart hoặc hiển thị Empty Checkout
Item lỗi → hiển thị Checkout Blocking Error
Auth lỗi nếu endpoint yêu cầu → yêu cầu login lại
```

### 4.3. Header checkout

Checkout nên dùng header tối giản.

Desktop:

```text
[Logo]  Checkout Progress                         [Hotline] [Secure Checkout]
```

Mobile:

```text
[Back] [Logo] Checkout
```

Không nên hiển thị mega menu đầy đủ trong checkout vì dễ làm user thoát khỏi flow.

---

## 5. Layout tổng thể

### 5.1. Desktop layout

Breakpoint: `>= 1024px`

```text
┌──────────────────────────────────────────────────────────────┐
│ Checkout Header                                               │
├──────────────────────────────────────────────────────────────┤
│ Checkout Progress                                             │
├───────────────────────────────┬──────────────────────────────┤
│ Left Column                    │ Right Column                 │
│ width: 60-65%                  │ width: 35-40%                │
│                               │                              │
│ 1. Contact / Login             │ Sticky Order Summary         │
│ 2. Shipping Address            │ - Products                   │
│ 3. Delivery Method             │ - Coupon                     │
│ 4. Payment Method              │ - Price breakdown            │
│ 5. Order Note                  │ - Final CTA                  │
│ 6. Terms                       │ - Trust badges               │
│                               │                              │
└───────────────────────────────┴──────────────────────────────┘
```

Right column có thể sticky ở desktop:

```text
top: header height + spacing
max-height: viewport - header
scroll inside if content too long
```

### 5.2. Tablet layout

Breakpoint: `768px - 1023px`

```text
Single column
Order summary ở trên hoặc dưới form tùy A/B test
CTA rõ ở cuối
Không dùng sticky panel quá lớn
```

Khuyến nghị:

```text
1. Checkout Progress
2. Collapsible Order Summary
3. Checkout Form
4. Final CTA
```

### 5.3. Mobile layout

Breakpoint: `< 768px`

```text
┌──────────────────────────────┐
│ Mobile Checkout Header        │
├──────────────────────────────┤
│ Collapsible Order Summary     │
├──────────────────────────────┤
│ Contact Info                  │
│ Shipping Address              │
│ Delivery Method               │
│ Payment Method                │
│ Order Note                    │
│ Terms                         │
├──────────────────────────────┤
│ Sticky Checkout Bar           │
└──────────────────────────────┘
```

Mobile sticky bar:

```text
Left: Final total
Right: Place order button
```

Không để sticky bar che mất field cuối form.

---

## 6. Page sections

## 6.1. Checkout Header

### Mục đích

Giữ user trong flow thanh toán, tạo cảm giác an toàn.

### Thành phần

```text
Logo
Page title: Thanh toán
Back to cart link
Hotline / support link
Secure checkout badge
```

### Rule

- Click logo có thể về home, nhưng nên hỏi xác nhận nếu form đã nhập nhiều thông tin.
- Back to cart luôn rõ ràng.
- Header không hiển thị full category menu.
- Header không hiển thị search bar trên mobile checkout.

---

## 6.2. Checkout Progress

### Mục đích

Cho user biết đang ở bước nào.

### Step đề xuất

```text
Cart → Checkout → Success
```

Hoặc chi tiết hơn:

```text
Cart → Information → Payment → Complete
```

### MVP display

Desktop:

```text
[1 Giỏ hàng] — [2 Thanh toán] — [3 Hoàn tất]
```

Mobile:

```text
Bước 2/3: Thanh toán
```

### Rule

- Current step dùng primary color.
- Completed step dùng success color hoặc check icon.
- Không cho click sang step future.
- Có thể click quay lại Cart.

---

## 6.3. Authentication / Customer Panel

### Mục đích

Hỗ trợ cả khách chưa đăng nhập và khách đã đăng nhập.

### Guest state

Hiển thị:

```text
Bạn đã có tài khoản? Đăng nhập để dùng địa chỉ đã lưu và xem lịch sử đơn hàng.
[Đăng nhập]
Tiếp tục mua hàng không cần tài khoản
```

### Logged-in state

Hiển thị:

```text
Đang thanh toán với tài khoản:
Nguyễn Văn A • nguyenvana@example.com
[Đổi tài khoản]
```

### Rule

- Không bắt buộc đăng nhập ở MVP.
- Nếu user login giữa checkout, merge cart và giữ lại thông tin đã nhập nếu có thể.
- Nếu token hết hạn, cho user đăng nhập lại mà không mất form.

---

## 6.4. Contact Information Form

### Mục đích

Thu thông tin liên hệ để shop xác nhận đơn.

### Fields MVP

| Field | Required | Note |
|---|---:|---|
| full_name | Yes | Tên người nhận |
| phone | Yes | Số điện thoại |
| email | No | Nhận hóa đơn/email |

### UI rule

- Label luôn hiển thị, không chỉ dùng placeholder.
- Lỗi hiển thị dưới input.
- Phone nên dùng keyboard numeric trên mobile.
- Email không bắt buộc trong MVP, nhưng validate format nếu user nhập.

### Validation

```text
full_name:
- required
- min length 2
- max length 80

phone:
- required
- trim space
- chỉ chấp nhận số và ký tự + nếu quốc tế
- độ dài hợp lý theo thị trường

email:
- optional
- nếu có nhập thì phải đúng format email
```

### Error messages tiếng Việt

```text
Vui lòng nhập họ tên người nhận.
Số điện thoại không hợp lệ.
Email không hợp lệ.
```

---

## 6.5. Shipping Address Form

### Mục đích

Thu địa chỉ giao hàng.

### Fields MVP Việt Nam

| Field | Required | Note |
|---|---:|---|
| province_city | Yes | Tỉnh/thành phố |
| district | Yes | Quận/huyện |
| ward | Yes | Phường/xã |
| street_address | Yes | Số nhà, đường |
| address_note | No | Ghi chú |

### UI rule

- Desktop có thể chia 2 cột cho province/district, ward/street.
- Mobile tất cả field full width.
- Nếu dùng dropdown cascading: chọn province mới load district; chọn district mới load ward.
- Khi đổi địa chỉ, phải tính lại shipping fee.
- Nếu địa chỉ nằm ngoài vùng giao, hiển thị lỗi rõ.

### Saved address state

Nếu user đã login và có địa chỉ lưu:

```text
[Radio] Nhà riêng
Nguyễn Văn A • 0900000000
Số 1, Quang Trung, Hà Đông, Hà Nội
[Chỉnh sửa]

[Radio] Công ty
...

[+ Thêm địa chỉ mới]
```

### Address form mode

```text
create_new
edit_existing
use_saved
```

### Error messages

```text
Vui lòng chọn tỉnh/thành phố.
Vui lòng chọn quận/huyện.
Vui lòng chọn phường/xã.
Vui lòng nhập địa chỉ cụ thể.
Địa chỉ này hiện chưa được hỗ trợ giao hàng.
```

---

## 6.6. Delivery Method Selector

### Mục đích

Cho user chọn phương thức giao hàng.

### MVP options

```text
Standard Delivery
Fast Delivery
Store Pickup (optional)
```

### Delivery option card

Mỗi option gồm:

```text
Radio
Tên phương thức
Thời gian dự kiến
Phí giao hàng
Mô tả ngắn
Trạng thái khả dụng
```

Ví dụ:

```text
Giao hàng tiêu chuẩn
2-4 ngày làm việc
30.000đ
Phù hợp với hầu hết đơn hàng
```

### Rule

- Chỉ hiển thị option khả dụng theo địa chỉ và sản phẩm.
- Sản phẩm cồng kềnh có thể giới hạn phương thức giao.
- Nếu đơn có nhiều item từ nhiều kho, có thể cần hiển thị thông báo thời gian giao khác nhau.
- Nếu chưa nhập địa chỉ, delivery section ở trạng thái pending.

### State

```text
pending_address
loading_rates
available
unavailable
error
```

### Messages

```text
Nhập địa chỉ để xem phương thức giao hàng.
Đang tính phí giao hàng...
Không có phương thức giao hàng khả dụng cho địa chỉ này.
Không thể tính phí giao hàng. Vui lòng thử lại.
```

---

## 6.7. Payment Method Selector

### Mục đích

Cho user chọn cách thanh toán.

### MVP payment methods

```text
COD
Bank Transfer
```

### Future payment methods

```text
VNPay
MoMo
ZaloPay
Credit Card
Installment
```

### Payment method card

Mỗi card gồm:

```text
Radio
Icon
Tên phương thức
Mô tả ngắn
Badge nếu có
Nội dung mở rộng nếu selected
```

### COD card

```text
Thanh toán khi nhận hàng (COD)
Bạn thanh toán bằng tiền mặt hoặc chuyển khoản cho nhân viên giao hàng.
```

Rule COD:

- Có thể giới hạn COD theo tổng tiền.
- Có thể không cho COD với sản phẩm đặt cọc/trả góp.
- Nếu vượt ngưỡng COD, hiển thị lý do.

### Bank transfer card

```text
Chuyển khoản ngân hàng
Đơn hàng sẽ được giữ trong thời gian cấu hình sau khi đặt.
```

Khi selected, hiển thị:

```text
Tên ngân hàng
Số tài khoản
Chủ tài khoản
Nội dung chuyển khoản
Lưu ý xác nhận thanh toán
```

Ví dụ nội dung chuyển khoản:

```text
DH-{order_number} {phone}
```

Trong checkout trước khi có order number, có thể hiển thị:

```text
Nội dung chuyển khoản sẽ được hiển thị sau khi đặt hàng.
```

### Online payment placeholder

Nếu chưa tích hợp online payment, không hiển thị option giả hoạt động. Nếu muốn chừa chỗ UI, hiển thị disabled:

```text
VNPay - Sắp ra mắt
MoMo - Sắp ra mắt
```

---

## 6.8. Coupon Section

### Mục đích

Cho user xem hoặc áp dụng mã giảm giá.

### UI

```text
[Mã giảm giá input] [Áp dụng]
Coupon đã áp dụng: SALE10  (-100.000đ) [Gỡ]
```

### Rule

- Coupon có thể đã được áp dụng từ cart page.
- Khi đổi địa chỉ hoặc phương thức giao, coupon có thể cần re-validate.
- Nếu coupon không còn hợp lệ, hiển thị lỗi và remove discount.
- Không để user áp dụng coupon làm tổng tiền âm.

### Error messages

```text
Mã giảm giá không tồn tại.
Mã giảm giá đã hết hạn.
Đơn hàng chưa đạt giá trị tối thiểu.
Mã giảm giá không áp dụng cho sản phẩm trong giỏ.
Mã giảm giá đã được sử dụng hết lượt.
```

---

## 6.9. Order Note

### Mục đích

Cho khách ghi chú giao hàng.

### Field

```text
order_note: textarea, optional, max 500 characters
```

### Placeholder

```text
Ví dụ: Giao giờ hành chính, gọi trước khi giao, kiểm tra hàng trước khi nhận...
```

### Rule

- Không dùng ghi chú để thay đổi giá hoặc yêu cầu ngoài chính sách.
- Nếu có nội dung nhạy cảm hoặc quá dài, validate server-side.

---

## 6.10. Terms and Confirmation

### Mục đích

Xác nhận user đồng ý với chính sách mua hàng.

### UI

```text
[ ] Tôi đồng ý với Điều khoản mua hàng, Chính sách bảo hành và Chính sách đổi trả.
```

### Rule

- Có thể bắt buộc check trước khi đặt hàng.
- Link chính sách mở tab mới hoặc modal.
- Không làm mất dữ liệu form khi mở chính sách.

---

## 6.11. Order Summary

### Mục đích

Hiển thị tổng quan đơn hàng và tổng tiền cuối cùng.

### Desktop

Order summary nằm ở right column, sticky.

### Mobile

Có hai phần:

```text
Collapsible summary ở gần đầu trang
Sticky checkout bar ở cuối màn hình
```

### Thành phần

```text
Product list compact
Subtotal
Discount
Shipping fee
Tax/VAT nếu có
Final total
Payment method note
Place order button
Trust badges
```

### Product row compact

Mỗi sản phẩm hiển thị:

```text
Thumbnail
Tên sản phẩm
Variant/spec ngắn
Số lượng
Giá dòng
Stock warning nếu có
```

### Price breakdown

```text
Tạm tính
Giảm giá
Phí vận chuyển
Thuế/VAT nếu có
Tổng thanh toán
```

### Rule

- Final total phải nổi bật nhất.
- Nếu shipping fee chưa tính được, hiển thị `Chưa tính`.
- Nếu có item giá thay đổi, phải cảnh báo.
- Nếu có item hết hàng, disable CTA.
- Không hiển thị số tiền mập mờ.

---

## 6.12. Place Order Button

### Text theo payment method

| Payment | Button text |
|---|---|
| COD | Đặt hàng |
| Bank transfer | Đặt hàng và xem hướng dẫn chuyển khoản |
| Online payment | Tiếp tục thanh toán |

### State

```text
enabled
disabled
loading
success redirect
error
```

### Disable khi

```text
Cart rỗng
Form invalid
Chưa chọn delivery
Chưa chọn payment
Chưa đồng ý terms nếu required
Đang tính phí ship
Đang validate coupon
Có item hết hàng
Có price change chưa xác nhận
Đang submit
```

### Loading text

```text
Đang tạo đơn hàng...
Đang chuyển đến cổng thanh toán...
```

### Double submit protection

- Disable button ngay sau click đầu tiên.
- Backend cần idempotency key.
- Nếu user reload trong lúc submit, xử lý bằng order state hoặc resume payment.

---

## 7. Component specification

## 7.1. `CheckoutPage`

### Responsibility

Container chính của checkout page.

### Inputs

```ts
cartId?: string;
initialCustomer?: Customer | null;
```

### Responsibilities

```text
Load cart
Load customer profile nếu login
Load saved addresses
Maintain checkout form state
Trigger shipping rate calculation
Trigger coupon validation
Submit order
Handle redirect result
```

### Child components

```text
CheckoutHeader
CheckoutProgress
CheckoutAuthPanel
ContactInfoForm
ShippingAddressSection
DeliveryMethodSelector
PaymentMethodSelector
CouponBox
OrderNoteField
TermsAgreement
OrderSummaryCard
MobileCheckoutBar
CheckoutBlockingAlert
```

---

## 7.2. `ContactInfoForm`

### Props

```ts
type ContactInfo = {
  fullName: string;
  phone: string;
  email?: string;
};
```

### Events

```text
change
validate
blur
```

### UI states

```text
pristine
dirty
valid
invalid
disabled
```

---

## 7.3. `ShippingAddressSection`

### Props

```ts
type ShippingAddress = {
  fullName?: string;
  phone?: string;
  provinceCity: string;
  district: string;
  ward: string;
  streetAddress: string;
  addressNote?: string;
};
```

### Modes

```text
guest_form
saved_address_select
edit_saved_address
new_address_form
```

---

## 7.4. `DeliveryMethodSelector`

### Props

```ts
type DeliveryMethod = {
  id: string;
  name: string;
  description?: string;
  estimatedTime?: string;
  fee: number;
  currency: string;
  isAvailable: boolean;
  unavailableReason?: string;
};
```

### Events

```text
select_delivery_method
shipping_rate_retry
```

---

## 7.5. `PaymentMethodSelector`

### Props

```ts
type PaymentMethod = {
  id: 'cod' | 'bank_transfer' | 'vnpay' | 'momo' | 'card' | 'installment';
  name: string;
  description?: string;
  isAvailable: boolean;
  unavailableReason?: string;
  badge?: string;
};
```

### Rule

- Component chỉ hiển thị option nhận từ config/API.
- Không hard-code logic phức tạp trong UI.
- Payment method disable phải có lý do.

---

## 7.6. `OrderSummaryCard`

### Props

```ts
type OrderSummary = {
  items: CheckoutItem[];
  subtotal: number;
  discountTotal: number;
  shippingFee?: number | null;
  taxTotal?: number;
  grandTotal: number;
  currency: string;
  appliedCoupons: Coupon[];
  warnings: CheckoutWarning[];
};
```

### Display rule

- Nếu `shippingFee = null`, hiển thị `Chưa tính`.
- Nếu `discountTotal > 0`, hiển thị dòng giảm giá.
- Nếu `taxTotal` không dùng, không hiển thị dòng thuế.
- `grandTotal` phải dùng style lớn nhất trong summary.

---

## 7.7. `MobileCheckoutBar`

### Props

```ts
finalTotal: number;
buttonText: string;
disabled: boolean;
loading: boolean;
```

### Rule

- Chỉ hiện trên mobile.
- Luôn có safe-area padding cho iOS.
- Không che input khi keyboard mở.
- Nếu user đang focus input, có thể ẩn sticky bar tùy UX.

---

## 8. Data contract

## 8.1. Checkout item

```ts
type CheckoutItem = {
  cartItemId: string;
  productId: string;
  variantId?: string | null;
  sku: string;
  name: string;
  slug: string;
  imageUrl: string;
  brand?: string;
  shortSpecs: string[];
  quantity: number;
  unitPrice: number;
  originalPrice?: number;
  lineTotal: number;
  currency: string;
  stockStatus: 'in_stock' | 'low_stock' | 'out_of_stock' | 'discontinued';
  maxPurchasableQuantity?: number;
  warranty?: {
    durationMonths?: number;
    type?: string;
  };
};
```

## 8.2. Checkout form state

```ts
type CheckoutFormState = {
  contact: {
    fullName: string;
    phone: string;
    email?: string;
  };
  shippingAddress: {
    savedAddressId?: string;
    provinceCity: string;
    district: string;
    ward: string;
    streetAddress: string;
    addressNote?: string;
  };
  deliveryMethodId?: string;
  paymentMethodId?: string;
  couponCodes: string[];
  orderNote?: string;
  agreeToTerms: boolean;
};
```

## 8.3. Checkout quote

```ts
type CheckoutQuote = {
  quoteId: string;
  cartId: string;
  items: CheckoutItem[];
  subtotal: number;
  discountTotal: number;
  shippingFee: number | null;
  taxTotal: number;
  grandTotal: number;
  currency: string;
  availableDeliveryMethods: DeliveryMethod[];
  availablePaymentMethods: PaymentMethod[];
  appliedCoupons: Coupon[];
  warnings: CheckoutWarning[];
  expiresAt: string;
};
```

## 8.4. Checkout warning

```ts
type CheckoutWarning = {
  code:
    | 'PRICE_CHANGED'
    | 'OUT_OF_STOCK'
    | 'LOW_STOCK'
    | 'COUPON_REMOVED'
    | 'SHIPPING_UNAVAILABLE'
    | 'PAYMENT_UNAVAILABLE';
  severity: 'info' | 'warning' | 'blocking';
  cartItemId?: string;
  message: string;
};
```

---

## 9. API contract

API có thể thay đổi theo backend, nhưng frontend nên cần các capability sau.

### 9.1. Load checkout

```http
GET /api/v1/checkout
```

Response:

```json
{
  "cartId": "cart_123",
  "customer": null,
  "items": [],
  "summary": {},
  "savedAddresses": [],
  "availablePaymentMethods": []
}
```

### 9.2. Create checkout quote

```http
POST /api/v1/checkout/quote
Content-Type: application/json
```

Request:

```json
{
  "cartId": "cart_123",
  "shippingAddress": {
    "provinceCity": "Hà Nội",
    "district": "Hà Đông",
    "ward": "Quang Trung",
    "streetAddress": "Số 1 Quang Trung"
  },
  "couponCodes": ["SALE10"]
}
```

Response:

```json
{
  "quoteId": "quote_123",
  "shippingFee": 30000,
  "grandTotal": 15990000,
  "availableDeliveryMethods": [],
  "availablePaymentMethods": [],
  "warnings": []
}
```

### 9.3. Apply coupon

```http
POST /api/v1/checkout/coupons/apply
```

Request:

```json
{
  "cartId": "cart_123",
  "quoteId": "quote_123",
  "code": "SALE10"
}
```

### 9.4. Remove coupon

```http
DELETE /api/v1/checkout/coupons/{code}
```

### 9.5. Place order

```http
POST /api/v1/orders
Idempotency-Key: <uuid>
Content-Type: application/json
```

Request:

```json
{
  "cartId": "cart_123",
  "quoteId": "quote_123",
  "contact": {
    "fullName": "Nguyễn Văn A",
    "phone": "0900000000",
    "email": "a@example.com"
  },
  "shippingAddress": {
    "provinceCity": "Hà Nội",
    "district": "Hà Đông",
    "ward": "Quang Trung",
    "streetAddress": "Số 1 Quang Trung"
  },
  "deliveryMethodId": "standard",
  "paymentMethodId": "cod",
  "couponCodes": ["SALE10"],
  "orderNote": "Gọi trước khi giao",
  "agreeToTerms": true
}
```

Response COD:

```json
{
  "orderId": "ord_123",
  "orderNumber": "DH202606220001",
  "status": "pending_confirmation",
  "paymentStatus": "cod_pending",
  "redirectUrl": "/checkout/success?orderNumber=DH202606220001"
}
```

Response online payment:

```json
{
  "orderId": "ord_123",
  "orderNumber": "DH202606220002",
  "status": "pending_payment",
  "paymentStatus": "payment_required",
  "redirectUrl": "https://payment-gateway.example/checkout/..."
}
```

---

## 10. State machine

## 10.1. Page loading state

```text
init
→ loading_cart
→ cart_loaded
→ loading_checkout_quote
→ ready
```

Error path:

```text
loading_cart
→ cart_load_error

loading_checkout_quote
→ quote_error
```

## 10.2. Submit order state

```text
ready
→ validating_client
→ validating_server
→ creating_order
→ redirecting
→ completed
```

Error path:

```text
validating_server
→ server_validation_error

creating_order
→ order_create_error

redirecting
→ payment_redirect_error
```

## 10.3. Blocking warnings

Nếu quote response có warning severity = `blocking`:

```text
ready_with_blocking_warning
→ user_fix_required
```

Examples:

```text
OUT_OF_STOCK
SHIPPING_UNAVAILABLE
PAYMENT_UNAVAILABLE
```

CTA phải disabled cho đến khi xử lý xong.

---

## 11. Validation specification

## 11.1. Client-side validation

Client-side dùng để phản hồi nhanh.

| Field | Rule |
|---|---|
| fullName | required |
| phone | required, valid |
| email | optional, valid |
| provinceCity | required |
| district | required |
| ward | required |
| streetAddress | required |
| deliveryMethodId | required |
| paymentMethodId | required |
| agreeToTerms | required nếu bật |

## 11.2. Server-side validation

Server là nguồn sự thật cuối cùng.

Server phải kiểm tra:

```text
Cart ownership/session
Cart không rỗng
Product vẫn active
Variant vẫn active
Tồn kho đủ
Giá hiện tại
Coupon hợp lệ
Shipping method hợp lệ
Payment method hợp lệ
Address hợp lệ
Idempotency key
```

## 11.3. Validation display rule

- Field lỗi có border danger.
- Message lỗi ngay dưới field.
- Summary error ở đầu form nếu submit fail.
- Focus vào field lỗi đầu tiên sau submit.
- Không clear dữ liệu user đã nhập khi API lỗi.

---

## 12. Pricing and totals rule

### 12.1. Price breakdown order

```text
Subtotal
Product discount
Coupon discount
Shipping fee
Tax/VAT nếu có
Grand total
```

### 12.2. Formatting

- Dùng currency format thống nhất với theme.
- VND không cần decimal.
- Grand total dùng font lớn hơn và weight cao hơn.
- Negative discount hiển thị bằng dấu `-`.

Example:

```text
Tạm tính: 16.990.000đ
Giảm giá sản phẩm: -1.000.000đ
Mã giảm giá: -300.000đ
Phí vận chuyển: 30.000đ
Tổng thanh toán: 15.720.000đ
```

### 12.3. Recalculation triggers

Phải tính lại quote khi:

```text
Thay đổi địa chỉ
Thay đổi phương thức giao hàng
Thêm/gỡ coupon
Thay đổi số lượng từ summary nếu cho phép
Login/merge cart
Cart item được cập nhật từ tab khác
```

---

## 13. Electronics-specific checkout rules

## 13.1. Warranty visibility

Với đồ điện tử, thông tin bảo hành phải xuất hiện trong summary hoặc trust box.

Ví dụ:

```text
Bảo hành chính hãng 24 tháng
Đổi mới trong 7 ngày nếu lỗi phần cứng
Hỗ trợ xuất hóa đơn VAT
```

### Product item warranty

Mỗi item có thể hiển thị warranty ngắn:

```text
Bảo hành 24 tháng
```

Không cần hiển thị toàn bộ chính sách trong từng item.

## 13.2. High-value order trust

Nếu đơn hàng có giá trị cao, có thể hiển thị trust box:

```text
Cam kết hàng chính hãng
Cho kiểm tra hàng khi nhận
Hỗ trợ kỹ thuật sau bán
Bảo mật thanh toán
```

## 13.3. Fragile/heavy product handling

Sản phẩm điện tử có thể cần rule giao hàng riêng:

```text
TV lớn
Máy tính để bàn
Màn hình cỡ lớn
Thiết bị dễ vỡ
```

UI cần chừa chỗ cho cảnh báo:

```text
Sản phẩm này cần phương thức giao hàng đặc biệt.
Phí giao hàng có thể thay đổi theo khu vực.
```

## 13.4. Serial/IMEI note

Không cần thu serial/IMEI ở checkout. Serial/IMEI thuộc quy trình fulfillment/admin sau khi đóng gói.

## 13.5. Invoice/VAT optional

Đồ điện tử thường có nhu cầu xuất hóa đơn.

MVP có thể để optional collapsed section:

```text
[ ] Tôi cần xuất hóa đơn công ty
```

Fields future:

```text
Company name
Tax code
Company address
Invoice email
```

---

## 14. Empty, loading, error states

## 14.1. Loading checkout

Hiển thị skeleton:

```text
Header skeleton
Form section skeleton
Order summary skeleton
```

Không hiển thị layout nhảy mạnh khi load xong.

## 14.2. Empty checkout

Khi cart rỗng:

```text
Giỏ hàng của bạn đang trống.
Hãy chọn sản phẩm trước khi thanh toán.
[Quay lại mua hàng]
```

Có thể hiển thị recommended categories.

## 14.3. Cart item unavailable

Khi item không còn bán:

```text
Một số sản phẩm trong giỏ hiện không còn khả dụng.
Vui lòng quay lại giỏ hàng để cập nhật.
[Quay lại giỏ hàng]
```

## 14.4. Price changed

Khi giá thay đổi:

```text
Giá một số sản phẩm đã thay đổi.
Vui lòng kiểm tra lại tổng tiền trước khi đặt hàng.
[Đã hiểu và cập nhật]
```

Nếu chỉ tăng/giảm nhỏ, có thể cho user xác nhận.

## 14.5. Shipping unavailable

```text
Hiện chưa hỗ trợ giao hàng tới địa chỉ này.
Vui lòng chọn địa chỉ khác hoặc liên hệ hỗ trợ.
```

## 14.6. Payment unavailable

```text
Phương thức thanh toán này hiện không khả dụng cho đơn hàng của bạn.
Vui lòng chọn phương thức khác.
```

## 14.7. Submit error

```text
Không thể tạo đơn hàng. Vui lòng kiểm tra lại thông tin và thử lại.
```

Nếu lỗi server có mã cụ thể, map thành message thân thiện.

---

## 15. Success and result routing

## 15.1. COD success

Sau khi đặt COD thành công:

```text
/checkout/success?orderNumber=DH...
```

Success page hiển thị:

```text
Đặt hàng thành công
Mã đơn hàng
Tổng thanh toán
Phương thức thanh toán
Thông tin giao hàng
Trạng thái đơn hàng
CTA xem đơn hàng
CTA tiếp tục mua sắm
```

## 15.2. Bank transfer success / pending

Sau khi chọn chuyển khoản:

```text
/checkout/payment-pending?orderNumber=DH...
```

Hiển thị:

```text
Đơn hàng đã được tạo
Vui lòng chuyển khoản theo thông tin bên dưới
Bank account
Amount
Transfer content
Payment deadline
```

## 15.3. Online payment redirect

Nếu payment method là online:

```text
Order created
→ redirect payment gateway
→ callback
→ success/failed page
```

Không coi order là paid cho đến khi webhook/callback xác nhận.

---

## 16. Responsive rules

## 16.1. Desktop

```text
Two-column layout
Right summary sticky
Form max-width hợp lý
Section card rõ ràng
```

## 16.2. Tablet

```text
Single-column hoặc two-column hẹp
Summary collapsible
Spacing giảm nhẹ
```

## 16.3. Mobile

```text
Single-column
Full-width inputs
Large tap targets
Sticky checkout bar
Collapsible summary
No horizontal overflow
Keyboard-safe layout
```

### Mobile tap target

```text
Minimum 44px height
Radio card clickable toàn bộ
Button full width
```

---

## 17. Accessibility requirements

### 17.1. Form accessibility

- Mỗi input phải có label thật.
- Error message phải liên kết với field bằng `aria-describedby` hoặc tương đương.
- Required field phải được thông báo rõ.
- Keyboard có thể tab qua toàn bộ form.
- Radio card có role đúng.

### 17.2. Focus management

- Sau submit fail, focus vào lỗi đầu tiên.
- Sau modal đóng, focus quay lại trigger.
- Khi section mới load do đổi địa chỉ, không giật focus bất ngờ.

### 17.3. Color contrast

- Text chính đạt contrast tốt.
- Error không chỉ dựa vào màu đỏ; phải có icon/message.
- Disabled state vẫn đọc được.

### 17.4. Screen reader

Order summary update nên có live region nhẹ:

```text
Tổng thanh toán đã được cập nhật.
```

Không spam screen reader khi user đang nhập.

---

## 18. Performance rules

Checkout phải ưu tiên ổn định và nhanh.

### 18.1. Không tải thừa

Không load:

```text
Large banners
Heavy carousel
Unnecessary recommendation list
Large product description
```

### 18.2. Lazy load

Có thể lazy load:

```text
Recommended accessories
Policy modal content
Invoice form if collapsed
```

### 18.3. Debounce

Cần debounce khi:

```text
Tính phí ship sau khi đổi địa chỉ
Validate coupon input
Auto-save checkout draft
```

### 18.4. Keep form stable

Không reset form khi quote recalculation. Chỉ update phần summary và delivery/payment availability.

---

## 19. Security and privacy

### 19.1. Sensitive information

Không log ra console:

```text
Số điện thoại đầy đủ
Email đầy đủ
Địa chỉ đầy đủ
Token
Payment session
```

### 19.2. Payment security

- Không tự xử lý thông tin thẻ ở frontend nếu chưa có chuẩn bảo mật.
- Dùng redirect/hosted checkout từ payment gateway nếu có.
- Verify payment status bằng backend, không tin query param frontend.

### 19.3. Idempotency

Place order phải có idempotency key.

Rule:

```text
Một click đặt hàng = một idempotency key
Retry cùng request = cùng idempotency key
Sau khi thành công = không dùng lại key
```

### 19.4. Bot and abuse

Có thể bổ sung sau:

```text
Rate limit place order
Captcha khi nghi ngờ spam
Fraud check cho đơn COD lớn
```

---

## 20. Analytics events

Checkout cần event để đo funnel.

### 20.1. Events

```text
checkout_viewed
checkout_contact_completed
checkout_address_completed
checkout_delivery_selected
checkout_payment_selected
coupon_applied
coupon_apply_failed
order_submit_clicked
order_create_success
order_create_failed
payment_redirect_started
```

### 20.2. Event properties

Không gửi PII đầy đủ.

Có thể gửi:

```text
cart_id hash
item_count
subtotal
grand_total
payment_method
delivery_method
coupon_code nếu không nhạy cảm
error_code
```

Không gửi:

```text
full_name
phone
email
full_address
```

---

## 21. Admin configuration dependency

Checkout phụ thuộc nhiều cấu hình từ admin.

Admin cần cấu hình được:

```text
Payment methods
Bank transfer information
COD limit
Shipping methods
Shipping fee rule
Free shipping threshold
Supported delivery areas
Terms links
Warranty policy link
Return policy link
Invoice option
Order note enable/disable
```

Frontend không hard-code dữ liệu kinh doanh như số tài khoản, phí ship, ngưỡng COD.

---

## 22. Agent implementation rules

Khi coding agent implement checkout page, phải tuân thủ:

### 22.1. Không được làm

```text
Không hard-code giá tiền cuối cùng ở frontend
Không bỏ qua server-side quote
Không cho submit khi form invalid
Không cho submit khi có blocking warning
Không xóa validation để test pass
Không dùng selector CSS fragile cho E2E nếu có role/label/test-id
Không log PII ra console
Không reset form khi API quote lỗi
Không để mobile overflow ngang
```

### 22.2. Bắt buộc làm

```text
Tách component rõ ràng
Có loading state
Có empty state
Có error state
Có disabled state
Có mobile sticky CTA
Có price breakdown rõ ràng
Có client validation
Có server error mapping
Có double submit protection
Có accessibility label
Có Playwright tests
```

### 22.3. Khi sửa checkout

Agent phải báo cáo:

```text
Files changed
Components affected
APIs touched
Validation rules touched
Tests run
Screenshots/visual result nếu có
Known limitations
```

---

## 23. Playwright test specification

## 23.1. Basic checkout success COD

```text
Given cart has valid product
When user opens checkout
And fills contact info
And fills shipping address
And selects standard delivery
And selects COD
And agrees to terms
And clicks place order
Then success page is shown
And order number is visible
```

### Suggested locator strategy

Use:

```text
getByRole
getByLabel
getByText
getByTestId for repeated product rows
```

Avoid:

```text
:nth-child
.deep .css .selector
class generated by UI framework
```

## 23.2. Required field validation

```text
Open checkout
Click place order without filling form
Expect required errors visible
Expect focus moves to first invalid field
```

## 23.3. Empty cart redirect

```text
Given cart is empty
When user opens checkout
Then user is redirected to cart
Or empty checkout state is visible
```

## 23.4. Out of stock blocking

```text
Given cart has out-of-stock item
When checkout loads
Then blocking warning is visible
And place order button is disabled
```

## 23.5. Price changed warning

```text
Given item price changed after cart
When checkout quote loads
Then price changed warning is visible
And updated total is shown
```

## 23.6. Coupon success

```text
Enter valid coupon
Click apply
Expect discount row visible
Expect grand total updated
```

## 23.7. Coupon fail

```text
Enter invalid coupon
Click apply
Expect coupon error visible
Expect grand total unchanged
```

## 23.8. Shipping unavailable

```text
Enter unsupported address
Expect shipping unavailable message
Expect delivery method disabled
Expect place order disabled
```

## 23.9. Payment method switch

```text
Select COD
Expect button text: Đặt hàng
Select Bank transfer
Expect bank transfer note visible
Expect button text changes
```

## 23.10. Double submit protection

```text
Fill valid checkout form
Double click place order
Expect only one order creation request
Expect button disabled while submitting
```

## 23.11. Mobile layout

```text
Set viewport 375x812
Open checkout
Expect no horizontal overflow
Expect sticky checkout bar visible
Expect order summary collapsible
Fill form successfully
```

## 23.12. Accessibility smoke test

```text
Tab through checkout form
Expect focus visible
Expect radio cards selectable by keyboard
Expect errors announced or associated
```

---

## 24. Visual regression checklist

Capture screenshots for:

```text
Desktop checkout empty form
Desktop checkout filled form
Desktop checkout with coupon
Desktop checkout with error
Mobile checkout empty form
Mobile checkout filled form
Mobile order summary expanded
Mobile validation errors
Bank transfer selected
Out-of-stock blocking warning
```

Viewport list:

```text
1440x900
1024x768
768x1024
375x812
```

Visual must check:

```text
No horizontal overflow
CTA visible
Summary readable
Inputs not clipped
Error messages readable
Sticky bar not covering content
Radio cards aligned
Price breakdown not wrapping badly
```

---

## 25. Definition of Done

Checkout page được coi là hoàn thành khi:

```text
Cart valid thì vào được checkout
Cart empty thì không checkout sai flow
Guest checkout hoạt động
Logged-in checkout hoạt động nếu auth có sẵn
Form validation đầy đủ
Shipping method tính theo địa chỉ
Payment method chọn được
Coupon apply/remove được
Order summary đúng
Place order tạo đơn thành công
COD success route hoạt động
Bank transfer pending route hoạt động nếu bật
Double submit được chặn
Server errors được hiển thị thân thiện
Mobile không overflow
Desktop summary sticky hợp lý
Accessibility cơ bản đạt
Playwright core tests pass
Visual snapshots pass hoặc diff được review
```

---

## 26. MVP implementation checklist

### Must have

```text
Checkout route
Minimal checkout header
Contact form
Address form
Delivery method selector
Payment method selector COD + bank transfer
Order summary
Coupon box
Terms checkbox
Place order button
Order success redirect
Loading/error states
Playwright tests
```

### Should have

```text
Saved address selector
Mobile collapsible summary
Mobile sticky checkout bar
Price changed warning
Out of stock blocking warning
Bank transfer instruction page
```

### Could have

```text
VAT invoice form
Installment placeholder
Store pickup
Delivery time slot
Recently viewed at bottom
```

### Not now

```text
Full online card processing
Multi-package shipment
Split payment
Advanced fraud scoring
Loyalty points
Complex tax engine
```

---

## 27. Suggested file structure

Framework-independent idea:

```text
src/
  pages/
    checkout/
      CheckoutPage
      CheckoutSuccessPage
      PaymentPendingPage
      PaymentFailedPage

  modules/
    checkout/
      components/
        CheckoutHeader
        CheckoutProgress
        ContactInfoForm
        ShippingAddressSection
        DeliveryMethodSelector
        PaymentMethodSelector
        CouponBox
        OrderSummaryCard
        MobileCheckoutBar
        CheckoutBlockingAlert
      api
      types
      validation
      pricing
      state

  modules/
    cart/
    order/
    payment/

tests/
  e2e/
    checkout.spec
  visual/
    checkout.visual.spec
```

---

## 28. Notes for future specs

Trang tiếp theo nên là:

```text
07-storefront-order-success-page.md
```

File đó sẽ đặc tả:

```text
Order success
Payment pending
Payment failed
Bank transfer instruction
Order tracking summary
Continue shopping CTA
Customer account prompt
```
