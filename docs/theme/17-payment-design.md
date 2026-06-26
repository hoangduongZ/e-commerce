# 17 — Payment Design Specification

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Dự án: Electronics Store Theme  
> Khu vực: Storefront + Admin + Backend Domain  
> Module: Payment Design  
> Mục tiêu: Đặc tả đầy đủ thiết kế thanh toán cho website bán hàng đồ điện tử, bao gồm COD, chuyển khoản ngân hàng, thanh toán online, payment status, callback/webhook, retry, refund, reconciliation và các rule bảo mật.  
> Vai trò file: Vừa là **prompt cho coding agent**, vừa là **spec nghiệp vụ/kỹ thuật** để frontend/backend/admin agent có thể code đúng từ đầu đến cuối.  
> Không phụ thuộc framework cụ thể.

---

# 0. Prompt dùng cho coding agent

Khi giao file này cho coding agent, dùng prompt sau:

```text
Bạn là Senior Full-stack Engineer kiêm Payment Domain Designer.

Hãy đọc kỹ file 17-payment-design.md và implement module Payment cho website bán hàng đồ điện tử.

Yêu cầu bắt buộc:

1. Không hard-code một cổng thanh toán cụ thể vào core payment flow.
2. Core payment phải hỗ trợ tối thiểu COD và Bank Transfer.
3. Payment online phải thiết kế theo dạng provider adapter để sau này thêm VNPay, MoMo, ZaloPay, Stripe, PayPal hoặc cổng khác.
4. Payment status phải tách khỏi order status.
5. Không coi order paid chỉ vì frontend redirect success.
6. Webhook/callback từ payment provider phải verify chữ ký hoặc secret.
7. Payment event phải idempotent, nhận trùng webhook không được cộng tiền hoặc đổi trạng thái sai.
8. Không lưu dữ liệu thẻ nhạy cảm nếu không đạt chuẩn bảo mật tương ứng.
9. Mọi thay đổi payment/refund phải có audit log.
10. Checkout, Order Success, Admin Order Detail và Admin Payment Detail phải dùng chung payment state model.
11. Cần có loading, empty, error, pending, failed, expired, refunded states.
12. Cần có Playwright tests cho COD, bank transfer, online success, online failed, retry payment và admin mark paid.
13. Nếu API chưa có, tạo mock contract rõ ràng, không nhét fake data production.
14. Sau khi code, báo cáo files changed, APIs mocked/integrated, tests run và known limitations.
```

---

# 1. Vai trò của Payment trong hệ thống e-commerce

Payment là module xử lý việc **thu tiền, xác nhận tiền, hoàn tiền và đối soát tiền** cho đơn hàng.

Trong website bán đồ điện tử, payment đặc biệt quan trọng vì:

```text
Giá trị đơn hàng thường cao.
Khách có thể dùng chuyển khoản, COD hoặc online payment.
Thanh toán lỗi cần xử lý rõ để không mất đơn.
Chuyển khoản cần hướng dẫn chính xác nội dung thanh toán.
Admin cần xác nhận tiền an toàn.
Hoàn tiền/đổi trả có thể xảy ra khi bảo hành, hủy đơn hoặc giao hàng lỗi.
```

Payment không chỉ là một nút trong checkout. Nó ảnh hưởng tới:

```text
Checkout
Order creation
Order success page
Order tracking
Admin order management
Shipping / fulfillment
Inventory reservation
Refund / return
Accounting / reconciliation
Notification
Audit log
Fraud/risk check
```

---

# 2. Nguyên tắc thiết kế Payment

## 2.1. Payment status tách khỏi Order status

Không được gộp trạng thái đơn hàng và trạng thái thanh toán vào một field duy nhất.

Ví dụ sai:

```text
order.status = paid
order.status = shipping
order.status = cancelled
```

Ví dụ đúng:

```text
order_status = pending_confirmation | confirmed | packing | shipping | delivered | cancelled
payment_status = unpaid | pending | paid | failed | expired | refunded | partially_refunded
fulfillment_status = unfulfilled | packing | shipped | delivered | returned
```

Lý do:

```text
Một đơn có thể đã xác nhận nhưng chưa thanh toán.
Một đơn có thể đã thanh toán nhưng chưa giao.
Một đơn có thể bị hủy và cần hoàn tiền.
Một đơn COD có thể đã giao nhưng tiền chưa đối soát.
```

## 2.2. Không tin redirect từ frontend

Với online payment, không được coi thanh toán thành công chỉ vì user quay về trang success.

Flow đúng:

```text
User thanh toán ở provider
Provider gửi webhook/callback về backend
Backend verify chữ ký
Backend cập nhật PaymentTransaction
Backend cập nhật payment_status
Frontend chỉ đọc lại trạng thái từ backend
```

Redirect success chỉ là UX, không phải nguồn sự thật.

## 2.3. Payment phải idempotent

Webhook/callback có thể gửi nhiều lần.

Backend phải đảm bảo:

```text
Cùng provider_transaction_id không xử lý trùng.
Cùng event_id không tạo nhiều payment events.
Nếu payment đã paid, webhook paid trùng không đổi dữ liệu sai.
Nếu refund đã completed, webhook refund trùng không cộng thêm refund.
```

## 2.4. Chuyển khoản cần rõ nội dung

Với Bank Transfer, phải hiển thị rõ:

```text
Tên ngân hàng
Số tài khoản
Tên chủ tài khoản
Số tiền
Nội dung chuyển khoản
Thời hạn giữ đơn
Trạng thái xác nhận thanh toán
```

Nội dung chuyển khoản nên có mã đơn:

```text
DH1024
PAY DH1024
```

Không nên yêu cầu khách tự viết nội dung dài, dễ sai.

## 2.5. COD không đồng nghĩa đã thanh toán

COD là trả tiền khi nhận hàng.

Khi khách đặt COD:

```text
payment_method = cod
payment_status = cod_pending hoặc unpaid
order_status = pending_confirmation
```

Chỉ khi đơn giao thành công và đối soát COD xong mới có thể coi là paid.

---

# 3. Payment methods MVP và mở rộng

## 3.1. MVP payment methods

MVP bắt buộc hỗ trợ:

```text
COD
Bank Transfer
```

Lý do:

```text
Dễ triển khai.
Phù hợp thị trường Việt Nam.
Không cần tích hợp cổng thanh toán ngay.
Đủ để chạy thử e-commerce thật.
```

## 3.2. Payment methods mở rộng

Sau MVP có thể thêm:

```text
VNPay
MoMo
ZaloPay
Stripe
PayPal
Credit/Debit Card
Installment Payment
Buy Now Pay Later
Bank QR
Store Credit
Wallet Balance
```

## 3.3. Core design không phụ thuộc provider

Không viết logic kiểu:

```text
if provider == "vnpay" ở khắp nơi
```

Nên dùng adapter pattern:

```text
PaymentProviderAdapter
- createPaymentIntent(order)
- verifyCallback(payload, headers)
- parseProviderEvent(payload)
- queryTransaction(transactionId)
- requestRefund(payment, amount, reason)
```

Mỗi provider là một adapter riêng.

---

# 4. Payment domain concepts

## 4.1. Payment Method

Payment Method là cách khách chọn để thanh toán.

Ví dụ:

```text
cod
bank_transfer
vnpay
momo
credit_card
installment
```

## 4.2. Payment Intent

Payment Intent là ý định thanh toán được tạo sau khi order được tạo hoặc trước khi redirect provider.

Nó đại diện cho:

```text
Order nào cần thanh toán
Số tiền cần thu
Phương thức thanh toán
Trạng thái thanh toán hiện tại
Hạn thanh toán
Provider transaction reference nếu có
```

## 4.3. Payment Transaction

Payment Transaction là một lần giao dịch cụ thể với provider hoặc admin confirmation.

Một Payment Intent có thể có nhiều transaction nếu:

```text
Thanh toán failed rồi retry.
Khách đổi phương thức thanh toán.
Provider gửi nhiều attempt.
Admin ghi nhận thanh toán thủ công.
```

## 4.4. Payment Event

Payment Event là log sự kiện liên quan đến thanh toán:

```text
payment_intent_created
payment_redirect_created
provider_callback_received
payment_paid
payment_failed
payment_expired
manual_mark_paid
refund_requested
refund_completed
refund_failed
```

Payment Event phục vụ audit và debug.

## 4.5. Refund

Refund là nghiệp vụ hoàn tiền một phần hoặc toàn bộ.

Refund có thể phát sinh từ:

```text
Hủy đơn đã thanh toán
Trả hàng
Bảo hành đổi/trả
Giao hàng thất bại nhưng đã thu tiền
Admin điều chỉnh sai đơn
```

---

# 5. Payment status model

## 5.1. Payment status bắt buộc

```text
unpaid
pending
paid
failed
expired
cancelled
refunded
partially_refunded
cod_pending
cod_collected
cod_reconciled
```

## 5.2. Ý nghĩa từng status

| Status | Ý nghĩa |
|---|---|
| unpaid | Chưa thanh toán |
| pending | Đang chờ thanh toán/xác nhận |
| paid | Đã thanh toán thành công |
| failed | Thanh toán thất bại |
| expired | Hết hạn thanh toán |
| cancelled | Payment bị hủy |
| refunded | Đã hoàn tiền toàn bộ |
| partially_refunded | Đã hoàn tiền một phần |
| cod_pending | COD chờ thu khi giao |
| cod_collected | Đơn vị giao đã thu COD |
| cod_reconciled | COD đã đối soát về shop |

## 5.3. Payment status transition

```text
unpaid -> pending
pending -> paid
pending -> failed
pending -> expired
pending -> cancelled
failed -> pending        # retry
expired -> pending       # recreate payment if allowed
paid -> partially_refunded
paid -> refunded
partially_refunded -> refunded
cod_pending -> cod_collected
cod_collected -> cod_reconciled
```

## 5.4. Không cho transition nguy hiểm

Không nên cho:

```text
paid -> unpaid
refunded -> paid
cod_reconciled -> cod_pending
failed -> paid nếu không có provider callback hoặc manual permission
```

Admin mark paid phải có quyền riêng và audit.

---

# 6. Payment methods detail

---

# 7. COD payment design

## 7.1. COD là gì trong hệ thống

COD là phương thức khách trả tiền khi nhận hàng.

Khi khách chọn COD:

```text
payment_method = cod
payment_status = cod_pending
order_status = pending_confirmation
```

## 7.2. COD checkout UI

Checkout cần hiển thị:

```text
Thanh toán khi nhận hàng
Bạn sẽ thanh toán cho nhân viên giao hàng khi nhận sản phẩm.
Có thể kiểm tra hàng theo chính sách của shop.
```

Nếu có phí COD:

```text
Phí COD: 20.000đ
```

Nếu đơn giá trị cao cần hạn chế COD:

```text
Đơn từ 20.000.000đ cần thanh toán chuyển khoản trước hoặc xác nhận qua nhân viên.
```

## 7.3. COD business rules

Có thể cấu hình:

```text
Cho phép COD theo tỉnh/thành
Không cho COD với đơn quá giá trị X
Không cho COD với sản phẩm cồng kềnh
Không cho COD với khách có lịch sử bom hàng
COD fee
COD requires phone verification
```

## 7.4. COD admin flow

Admin order detail cần thấy:

```text
Payment method: COD
Payment status: COD pending
COD amount: total order amount
COD collection status
Carrier COD status nếu có
```

Sau khi giao thành công:

```text
Shipping delivered
COD collected
COD reconciled
```

COD reconciliation có thể làm ở module accounting sau.

---

# 8. Bank Transfer payment design

## 8.1. Bank Transfer flow

```text
Customer chọn Bank Transfer
System tạo order
System tạo payment intent pending
Order success page hiển thị hướng dẫn chuyển khoản
Customer chuyển khoản
Admin hoặc bank integration xác nhận tiền
Payment status -> paid
Order tiếp tục xử lý
```

## 8.2. Bank Transfer UI tại checkout

Hiển thị method card:

```text
Chuyển khoản ngân hàng
Sau khi đặt hàng, bạn sẽ nhận thông tin tài khoản và nội dung chuyển khoản.
Đơn hàng sẽ được xử lý sau khi shop xác nhận thanh toán.
```

## 8.3. Order success page cho Bank Transfer

Bắt buộc hiển thị:

```text
Mã đơn hàng
Số tiền cần chuyển
Ngân hàng
Số tài khoản
Tên chủ tài khoản
Nội dung chuyển khoản
QR code nếu có
Thời hạn thanh toán
Trạng thái: Chờ thanh toán
```

Ví dụ:

```text
Số tiền: 18.500.000đ
Nội dung chuyển khoản: DH1024
Thời hạn giữ đơn: 24 giờ
```

## 8.4. Bank Transfer admin confirmation

Admin có thể mark paid nếu có quyền:

Form cần:

```text
Payment received amount
Received at
Bank account
Transaction reference
Attachment optional
Internal note
```

Rule:

```text
Không cho mark paid nếu amount nhận < amount cần thanh toán, trừ khi cho phép partial payment.
Mark paid phải có confirm modal.
Mark paid phải tạo PaymentTransaction manual.
Mark paid phải ghi audit log.
```

## 8.5. Bank transfer expiration

Nếu quá hạn:

```text
payment_status = expired
order_status có thể giữ pending hoặc auto_cancel tùy settings
reserved inventory có thể release
```

Settings:

```text
Bank transfer payment expires after: 24 hours
Auto cancel unpaid order: yes/no
Notify customer before expiry: yes/no
```

---

# 9. Online payment design

## 9.1. Online payment generic flow

```text
Customer chọn online payment
Backend tạo order hoặc payment intent
Backend gọi provider create payment
Provider trả payment_url hoặc client_secret
Frontend redirect / render provider flow
Customer thanh toán
Provider redirect customer về return_url
Provider gửi webhook/callback về backend
Backend verify callback
Backend cập nhật transaction/payment/order
Frontend đọc lại payment status từ backend
```

## 9.2. Tạo payment online

API gợi ý:

```http
POST /api/v1/payments/intents
```

Request:

```json
{
  "order_id": "ord_1024",
  "method": "vnpay",
  "return_url": "https://shop.vn/payment/return",
  "cancel_url": "https://shop.vn/payment/cancel"
}
```

Response:

```json
{
  "payment_intent_id": "pi_001",
  "status": "pending",
  "provider": "vnpay",
  "redirect_url": "https://provider.vn/pay/...",
  "expires_at": "2026-06-26T23:59:59+07:00"
}
```

## 9.3. Payment return page

Return page không tự kết luận thành công.

Nó cần:

```text
Đọc query provider trả về.
Gọi backend verify hoặc fetch payment status.
Hiển thị loading: Đang xác nhận thanh toán.
Nếu paid: chuyển order success.
Nếu failed: hiển thị payment failed + retry.
Nếu pending: hiển thị payment pending.
```

## 9.4. Webhook/callback rule

Backend phải:

```text
Verify signature/HMAC.
Check provider transaction id.
Check amount.
Check currency.
Check order/payment intent mapping.
Check event id duplicate.
Check transaction status.
Update payment trong transaction.
Emit domain event.
```

Không được:

```text
Trust mọi callback không verify.
Update paid nếu amount không khớp.
Update paid nếu currency sai.
Xử lý webhook trùng gây double update.
```

---

# 10. Installment payment design

## 10.1. Vai trò

Đồ điện tử thường có trả góp.

Installment có thể là:

```text
Trả góp qua thẻ tín dụng
Trả góp qua công ty tài chính
Buy Now Pay Later
Shop tự chia kỳ thanh toán
```

## 10.2. MVP scope

MVP chưa cần tích hợp trả góp thật.

Có thể chỉ hiển thị placeholder:

```text
Trả góp 0% — Liên hệ tư vấn
```

Hoặc method disabled:

```text
Trả góp — Sắp ra mắt
```

## 10.3. Future installment data

```json
{
  "provider": "installment_provider",
  "term_months": 6,
  "monthly_amount": 3150000,
  "interest_rate": 0,
  "down_payment": 0,
  "approval_status": "pending"
}
```

## 10.4. Rule

Không hiển thị trả góp 0% nếu chưa có điều kiện rõ.

Phải hiển thị:

```text
Số tiền trả trước
Số tháng
Số tiền mỗi tháng
Phí chuyển đổi nếu có
Điều kiện áp dụng
```

---

# 11. Checkout payment UX

## 11.1. Payment method selector

Checkout cần section:

```text
Phương thức thanh toán
```

Mỗi method card gồm:

```text
Radio
Icon
Tên phương thức
Mô tả ngắn
Fee nếu có
Availability state
```

Ví dụ:

```text
[ ] Thanh toán khi nhận hàng
    Trả tiền cho nhân viên giao hàng.

[ ] Chuyển khoản ngân hàng
    Shop xử lý đơn sau khi nhận thanh toán.

[ ] VNPay
    Thanh toán qua thẻ/QR/VNPay.
```

## 11.2. Payment method availability

Một method có thể disabled nếu:

```text
Không hỗ trợ khu vực giao hàng.
Order amount vượt giới hạn.
Sản phẩm không hỗ trợ COD.
Provider đang bảo trì.
Customer bị hạn chế COD.
```

Disabled method phải có lý do.

Ví dụ:

```text
COD không khả dụng cho đơn trên 20.000.000đ.
```

## 11.3. Payment fee display

Nếu payment có fee:

```text
Phí COD
Phí thanh toán thẻ
Phí chuyển đổi trả góp
```

Order summary phải hiển thị riêng:

```text
Tạm tính
Giảm giá
Phí vận chuyển
Phí thanh toán
Tổng cộng
```

## 11.4. Place order behavior

Khi click đặt hàng:

```text
Disable button.
Validate cart.
Validate shipping.
Validate payment method.
Create order.
Create payment intent if needed.
Redirect hoặc show success.
```

Không cho double submit.

---

# 12. Order success / payment result UX

## 12.1. COD success

Hiển thị:

```text
Đặt hàng thành công
Mã đơn hàng
Tổng tiền COD
Bạn sẽ thanh toán khi nhận hàng
Trạng thái: Chờ xác nhận
CTA: Theo dõi đơn hàng
```

## 12.2. Bank transfer pending

Hiển thị:

```text
Đơn hàng đã được tạo
Đang chờ chuyển khoản
Thông tin ngân hàng
Nội dung chuyển khoản
Countdown hết hạn thanh toán
CTA: Tôi đã chuyển khoản / Theo dõi đơn
```

Nút `Tôi đã chuyển khoản` không tự mark paid. Nó chỉ:

```text
Gửi thông báo cho admin.
Hoặc chuyển trạng thái customer_reported_paid.
```

## 12.3. Online payment success

Hiển thị:

```text
Thanh toán thành công
Mã đơn hàng
Số tiền đã thanh toán
Payment reference nếu có
Trạng thái đơn hàng
```

## 12.4. Online payment failed

Hiển thị:

```text
Thanh toán không thành công
Đơn hàng của bạn đã được tạo nhưng chưa thanh toán.
Bạn có thể thử lại hoặc chọn phương thức khác.
[Thử thanh toán lại]
[Đổi phương thức thanh toán]
[Liên hệ hỗ trợ]
```

## 12.5. Payment pending

Hiển thị:

```text
Đang xác nhận thanh toán
Vui lòng chờ trong giây lát.
Bạn có thể quay lại trang theo dõi đơn hàng để kiểm tra trạng thái.
```

---

# 13. Admin payment management

## 13.1. Payment information trong Order Detail

Admin Order Detail cần card Payment gồm:

```text
Payment method
Payment status
Amount due
Amount paid
Amount refunded
Payment fee
Provider
Transaction reference
Paid at
Expires at
Actions
```

Actions tùy trạng thái:

```text
Mark as paid
Send payment instruction
Retry payment link
Change payment method
Request refund
View transactions
View payment events
```

## 13.2. Payment transaction list

Table:

| Column | Meaning |
|---|---|
| Transaction ID | mã giao dịch nội bộ |
| Provider | provider/method |
| Type | payment/refund/adjustment |
| Amount | số tiền |
| Status | trạng thái |
| Reference | mã provider |
| Created at | thời gian |
| Actions | xem chi tiết |

## 13.3. Manual mark paid

Chỉ role có quyền mới được làm.

Modal cần:

```text
Confirm title
Amount received
Payment method
Received date/time
Reference code
Attachment
Note
```

Warning:

```text
This action will mark the order as paid and may trigger fulfillment.
```

Không dùng `window.confirm`.

## 13.4. Change payment method

Cho phép đổi method nếu:

```text
payment_status chưa paid
order_status chưa delivered/completed
method mới khả dụng
```

Ví dụ:

```text
Bank transfer -> COD
Online failed -> Bank transfer
Online failed -> COD
```

Không nên đổi payment method sau khi paid nếu không có refund/adjustment flow rõ.

---

# 14. Refund design

## 14.1. Refund use cases

Refund phát sinh khi:

```text
Admin hủy đơn đã thanh toán.
Khách trả hàng.
Đơn giao thất bại nhưng đã thanh toán.
Bảo hành đổi/trả.
Thanh toán thừa.
Admin xử lý sai cần hoàn tiền.
```

## 14.2. Refund types

```text
Full refund
Partial refund
Manual refund
Provider refund
Store credit refund
```

## 14.3. Refund status

```text
requested
approved
processing
completed
failed
cancelled
rejected
```

## 14.4. Refund request form

Fields:

```text
Refund amount
Refund reason
Refund method
Related order items
Customer bank info if manual
Attachment optional
Internal note
```

## 14.5. Refund validation

Rule:

```text
Refund amount > 0.
Refund amount <= refundable amount.
Cannot refund unpaid order.
Cannot refund more than paid amount minus already refunded amount.
Dangerous action requires confirm.
Refund must create audit log.
```

## 14.6. Refund state update

Nếu refund toàn bộ:

```text
payment_status = refunded
```

Nếu refund một phần:

```text
payment_status = partially_refunded
```

Order status không nhất thiết đổi theo payment status. Ví dụ partial refund do bồi thường nhưng đơn vẫn delivered.

---

# 15. Payment reconciliation

## 15.1. Mục đích

Reconciliation là đối soát thanh toán giữa hệ thống shop và nguồn tiền thực tế.

Nguồn đối soát:

```text
Bank statement
Payment provider settlement report
COD carrier report
Manual accounting import
```

## 15.2. MVP scope

MVP có thể chưa cần reconciliation module đầy đủ.

Nhưng data model phải để chỗ cho:

```text
settlement_id
settled_at
reconciled_status
reconciled_by
provider_fee
net_amount
```

## 15.3. Reconciliation status

```text
not_reconciled
matched
mismatched
manual_review
reconciled
```

## 15.4. COD reconciliation

COD flow:

```text
Order delivered
Carrier collected COD
Carrier transfers money to shop
Admin imports settlement or marks reconciled
payment_status -> cod_reconciled
```

---

# 16. Payment data model

## 16.1. PaymentMethodConfig

```json
{
  "id": "pm_bank_transfer",
  "code": "bank_transfer",
  "name": "Chuyển khoản ngân hàng",
  "status": "active",
  "description": "Chuyển khoản sau khi đặt hàng.",
  "sort_order": 2,
  "min_order_amount": 0,
  "max_order_amount": null,
  "enabled_regions": ["VN"],
  "fee_type": "fixed",
  "fee_value": 0,
  "settings": {
    "expires_after_minutes": 1440,
    "bank_accounts": ["bank_acc_001"]
  }
}
```

## 16.2. BankAccount

```json
{
  "id": "bank_acc_001",
  "bank_name": "Vietcombank",
  "account_number": "0123456789",
  "account_holder": "CONG TY ABC",
  "branch": "Ha Noi",
  "qr_image_url": "/images/payment/vcb-qr.png",
  "status": "active",
  "is_default": true
}
```

## 16.3. PaymentIntent

```json
{
  "id": "pi_001",
  "order_id": "ord_1024",
  "order_number": "DH1024",
  "method": "bank_transfer",
  "provider": null,
  "amount": 18500000,
  "currency": "VND",
  "status": "pending",
  "payment_content": "DH1024",
  "expires_at": "2026-06-27T12:00:00+07:00",
  "created_at": "2026-06-26T12:00:00+07:00",
  "updated_at": "2026-06-26T12:00:00+07:00"
}
```

## 16.4. PaymentTransaction

```json
{
  "id": "pay_txn_001",
  "payment_intent_id": "pi_001",
  "order_id": "ord_1024",
  "type": "payment",
  "method": "bank_transfer",
  "provider": null,
  "provider_transaction_id": "BANK-REF-123",
  "amount": 18500000,
  "currency": "VND",
  "status": "paid",
  "paid_at": "2026-06-26T13:15:00+07:00",
  "created_by": "admin_1",
  "note": "Confirmed from bank statement"
}
```

## 16.5. PaymentEvent

```json
{
  "id": "pay_evt_001",
  "payment_intent_id": "pi_001",
  "event_type": "manual_mark_paid",
  "source": "admin",
  "payload": {},
  "created_at": "2026-06-26T13:15:00+07:00",
  "created_by": "admin_1"
}
```

## 16.6. Refund

```json
{
  "id": "refund_001",
  "order_id": "ord_1024",
  "payment_transaction_id": "pay_txn_001",
  "amount": 500000,
  "currency": "VND",
  "reason": "Customer returned accessory item",
  "status": "completed",
  "refund_method": "bank_transfer",
  "created_by": "admin_1",
  "created_at": "2026-06-27T10:00:00+07:00",
  "completed_at": "2026-06-27T15:00:00+07:00"
}
```

---

# 17. API contract

## 17.1. Public/checkout payment methods

```http
GET /api/payment-methods?cart_id={cart_id}&shipping_address_id={id}
```

Response:

```json
{
  "methods": [
    {
      "code": "cod",
      "name": "Thanh toán khi nhận hàng",
      "description": "Trả tiền cho nhân viên giao hàng.",
      "available": true,
      "fee": 0
    },
    {
      "code": "bank_transfer",
      "name": "Chuyển khoản ngân hàng",
      "description": "Chuyển khoản sau khi đặt hàng.",
      "available": true,
      "fee": 0
    }
  ]
}
```

## 17.2. Create payment intent

```http
POST /api/v1/payments/intents
```

## 17.3. Get payment intent

```http
GET /api/v1/payments/intents/{payment_intent_id}
```

## 17.4. Retry payment

```http
POST /api/orders/{order_id}/payments/retry
```

## 17.5. Change payment method

```http
POST /api/orders/{order_id}/payments/change-method
```

Request:

```json
{
  "method": "bank_transfer"
}
```

## 17.6. Provider callback/webhook

```http
POST /api/v1/payments/providers/{provider}/webhook
```

Backend only.

## 17.7. Admin mark paid

```http
POST /api/v1/admin/orders/{order_id}/payments/mark-paid
```

Request:

```json
{
  "amount": 18500000,
  "received_at": "2026-06-26T13:15:00+07:00",
  "method": "bank_transfer",
  "reference": "BANK-REF-123",
  "note": "Confirmed from bank statement"
}
```

## 17.8. Admin request refund

```http
POST /api/v1/admin/orders/{order_id}/refunds
```

## 17.9. Admin payment config

```http
GET /api/v1/admin/payment-methods
POST /api/v1/admin/payment-methods
PATCH /api/v1/admin/payment-methods/{id}
POST /api/v1/admin/payment-methods/{id}/enable
POST /api/v1/admin/payment-methods/{id}/disable
```

## 17.10. Admin bank accounts

```http
GET /api/v1/admin/payment/bank-accounts
POST /api/v1/admin/payment/bank-accounts
PATCH /api/v1/admin/payment/bank-accounts/{id}
DELETE /api/v1/admin/payment/bank-accounts/{id}
```

---

# 18. Admin payment settings

## 18.1. Payment methods list

Admin settings page cần hiển thị:

```text
Method name
Code
Status
Fee
Availability
Provider
Sort order
Actions
```

Actions:

```text
Configure
Enable/disable
Set default
Test connection nếu provider online
```

## 18.2. COD settings

Fields:

```text
Enable COD
COD fee
Min order amount
Max order amount
Allowed shipping zones
Excluded products/categories
Require phone verification
High value order confirmation required
```

## 18.3. Bank transfer settings

Fields:

```text
Enable bank transfer
Bank accounts
Default bank account
Payment expiry minutes
Instruction template
Auto cancel expired unpaid order
Notify admin when customer reports paid
```

## 18.4. Online provider settings

Fields generic:

```text
Provider code
Environment sandbox/production
Merchant id
Public key
Secret key
Webhook secret
Return URL
Cancel URL
Webhook URL
Enabled currencies
Test connection
```

Sensitive fields:

```text
Secret key chỉ nhập, không hiển thị lại đầy đủ.
Mask secret sau khi lưu.
Chỉ role có quyền cao mới chỉnh sửa.
```

---

# 19. Validation rules

## 19.1. Checkout validation

```text
Payment method required.
Payment method must be available for current order.
Payment amount must equal order payable amount.
Cannot create payment for empty cart.
Cannot create payment if order already paid.
Cannot create payment if order cancelled.
```

## 19.2. Payment callback validation

```text
Provider signature valid.
Provider event not processed before.
Amount matches expected amount.
Currency matches.
Order exists.
Payment intent exists.
Status transition allowed.
Provider transaction id unique.
```

## 19.3. Admin mark paid validation

```text
User has payment.mark_paid permission.
Order not cancelled/completed in incompatible state.
Payment not already paid.
Amount > 0.
Amount equals outstanding amount unless partial payment supported.
Reference required for bank transfer if configured.
Note required if manual override.
```

## 19.4. Refund validation

```text
User has refund.create permission.
Order has paid amount.
Refund amount <= refundable amount.
Refund reason required.
Refund method valid.
Cannot refund cancelled payment that was never paid.
```

---

# 20. Permission matrix

| Permission | Super Admin | Store Manager | Accountant | Sales Staff | Support | Viewer |
|---|---|---|---|---|---|---|
| payment.view | yes | yes | yes | yes | yes | yes |
| payment.config.view | yes | yes | yes | no | no | no |
| payment.config.update | yes | no | no | no | no | no |
| payment.mark_paid | yes | yes | yes | no | no | no |
| payment.change_method | yes | yes | yes | yes | no | no |
| refund.view | yes | yes | yes | yes | yes | yes |
| refund.create | yes | yes | yes | no | no | no |
| refund.approve | yes | yes | yes | no | no | no |
| refund.cancel | yes | yes | yes | no | no | no |
| payment.audit.view | yes | yes | yes | no | no | no |

Rule:

```text
Frontend ẩn action nếu không có quyền.
Backend vẫn phải enforce permission.
Payment config secret chỉ role cao mới sửa.
Refund và mark paid phải audit.
```

---

# 21. Security rules

## 21.1. Không lưu thẻ nhạy cảm

Không lưu:

```text
Full card number
CVV
Raw card data
Provider secret in plain text
```

Nếu tích hợp card payment, nên để provider xử lý card input.

## 21.2. Webhook security

```text
Verify signature.
Use HTTPS.
Allow timestamp tolerance nếu provider hỗ trợ.
Protect against replay attack.
Log invalid callback.
Do not expose raw secret.
```

## 21.3. Admin security

```text
Mark paid requires permission.
Refund requires permission.
Changing payment config requires high permission.
Sensitive settings masked.
Audit log required.
```

## 21.4. Amount integrity

Không để frontend quyết định số tiền cuối.

Backend phải tự tính:

```text
subtotal
discount
shipping fee
payment fee
tax if any
total payable
```

Payment amount phải khớp với backend-calculated payable amount.

---

# 22. Audit log

Audit events:

```text
payment_intent_created
payment_method_selected
payment_provider_redirect_created
payment_callback_received
payment_callback_verified
payment_status_changed
manual_mark_paid
payment_method_changed
refund_requested
refund_approved
refund_completed
refund_failed
payment_config_updated
bank_account_updated
```

Audit fields:

```text
actor
role
action
entity_type
entity_id
before
after
ip
device
timestamp
```

---

# 23. Notification rules

Payment events cần gửi notification:

```text
Order created COD -> email/SMS order confirmation
Bank transfer pending -> send payment instruction
Bank transfer near expiry -> remind customer
Payment paid -> notify customer/admin
Payment failed -> notify customer with retry link
Refund completed -> notify customer
```

Không gửi thông tin nhạy cảm trong email nếu không cần.

---

# 24. Loading / empty / error states

## 24.1. Checkout payment loading

```text
Loading available payment methods...
```

Không show method giả nếu chưa load xong.

## 24.2. Payment processing state

```text
Đang tạo yêu cầu thanh toán...
Đang chuyển sang cổng thanh toán...
Đang xác nhận thanh toán...
```

## 24.3. Payment error state

Ví dụ:

```text
Không thể tạo thanh toán. Vui lòng thử lại.
Cổng thanh toán đang tạm thời không khả dụng.
Phương thức thanh toán này không áp dụng cho đơn hàng hiện tại.
```

## 24.4. Admin empty state

Nếu order chưa có transaction:

```text
Chưa có giao dịch thanh toán nào.
```

## 24.5. Admin error state

```text
Không thể tải lịch sử thanh toán.
Không thể đánh dấu đã thanh toán.
Không thể tạo yêu cầu hoàn tiền.
```

---

# 25. Accessibility rules

```text
Payment method radio phải dùng được bằng keyboard.
Method card phải có label rõ.
Disabled method phải có lý do text, không chỉ màu xám.
Payment status không chỉ dựa vào màu.
Modal mark paid/refund phải trap focus.
Error message phải liên kết với field.
Icon-only button phải có aria-label.
```

---

# 26. Responsive rules

## 26.1. Checkout mobile

```text
Payment methods stack 1 cột.
Order summary collapsible.
CTA đặt hàng full width.
Bank transfer instruction dễ copy.
QR code không vượt màn hình.
```

## 26.2. Admin mobile

```text
Payment transaction table chuyển card list hoặc horizontal scroll trong container.
Modals vừa màn hình.
Long transaction reference wrap được.
Action nguy hiểm vẫn dễ bấm.
```

---

# 27. Playwright test specification

## 27.1. Checkout payment tests

```text
User can select COD.
User can select Bank Transfer.
Disabled payment method shows reason.
Payment fee updates order summary.
Place order disables button while submitting.
Cannot place order without payment method.
```

## 27.2. COD tests

```text
User places COD order successfully.
Order success page shows COD pending.
Admin order detail shows COD pending.
```

## 27.3. Bank transfer tests

```text
User places Bank Transfer order successfully.
Order success page shows bank account, amount and payment content.
Payment content copy button works.
Expired bank transfer shows expired state.
Admin can mark bank transfer as paid with valid permission.
Admin cannot mark paid without required fields.
```

## 27.4. Online payment tests

```text
User selects online payment.
System creates payment intent.
User is redirected to provider mock.
Provider success callback marks payment paid.
Provider failed callback shows payment failed.
Payment return page fetches backend status before showing success.
Retry payment creates new payment attempt.
```

## 27.5. Refund tests

```text
Admin can create full refund for paid order.
Admin can create partial refund.
Refund amount cannot exceed paid amount.
Refund requires reason.
Refund completion updates payment status.
```

## 27.6. Security / permission tests

```text
Viewer cannot mark paid.
Sales staff cannot refund.
Invalid webhook signature is rejected.
Duplicate webhook does not create duplicate transaction.
Amount mismatch webhook does not mark paid.
```

## 27.7. Responsive tests

```text
Checkout payment method selector has no horizontal overflow at 375px.
Bank transfer QR/instruction fits mobile.
Admin payment card/table usable at 375px.
```

---

# 28. Visual regression checklist

Capture:

```text
Checkout payment section desktop
Checkout payment section mobile
COD selected
Bank transfer selected
Online payment selected
Order success COD
Order success bank transfer pending
Payment failed page
Payment pending page
Admin order detail payment card
Admin mark paid modal
Admin refund modal
Admin payment settings page
```

Reject screenshot if:

```text
Payment method disabled reason bị cắt.
QR code vượt màn hình.
Order summary total không rõ.
Payment status badge dùng sai màu.
Admin action danger không nổi rõ.
Mobile overflow ngang.
```

---

# 29. Definition of Done

Module Payment được coi là hoàn thành khi:

```text
COD hoạt động end-to-end.
Bank transfer hoạt động end-to-end.
Payment status tách khỏi order status.
Checkout hiển thị payment methods đúng availability.
Order success hiển thị đúng theo method.
Admin order detail hiển thị payment card.
Admin mark paid có permission, confirm modal và audit.
Refund MVP có validation.
Payment callback/webhook có verify/idempotency nếu provider online mock.
Payment amount do backend tính, frontend không quyết định.
Loading/empty/error states đầy đủ.
Responsive không overflow.
Playwright tests chính pass.
Không có console error nghiêm trọng.
Không lộ secret/payment sensitive data.
```

---

# 30. MVP scope

MVP Payment chỉ cần:

```text
COD
Bank Transfer
Payment method selector in checkout
Order success by method
Bank transfer instruction
Admin payment card in order detail
Admin mark paid
Payment transaction history basic
Payment status model
Refund manual basic
Permission basic
Audit log basic
Playwright tests for COD + Bank Transfer
```

Chưa cần ngay:

```text
VNPay/MoMo/ZaloPay integration thật
Stripe/PayPal thật
Automated bank reconciliation
Installment approval flow
Advanced fraud detection
Multi-currency
Partial capture
Split payment
Store credit
Advanced accounting report
```

---

# 31. Future extensions

```text
Payment provider plugin system
Bank statement auto matching
QR payment dynamic generation
COD carrier reconciliation import
Fraud/risk scoring
Partial payment/deposit
Installment workflow
Payment link gửi qua SMS/email
Multi-currency
Multi-store settlement
Accounting export
Webhook monitoring dashboard
Payment failure analytics
```

---

# 32. Clone-source notes

Khi clone source sang ngành khác, giữ nguyên:

```text
Payment method abstraction
Payment status model
Payment intent/transaction/event
Webhook idempotency
Refund model
Audit/security rules
Checkout payment selector
Admin payment card
```

Có thể thay đổi theo ngành:

```text
COD availability rule
Payment method priority
Installment visibility
Bank transfer instruction copywriting
High value order threshold
Refund policy text
```

Với đồ điện tử, nên ưu tiên:

```text
Bank transfer rõ ràng.
COD có giới hạn giá trị.
Trả góp để chỗ mở rộng.
Refund/warranty liên kết tốt với service module.
Admin confirmation an toàn vì giá trị đơn cao.
```

---

# 33. Tóm tắt cho agent

Nếu agent chỉ đọc phần này, hãy nhớ:

```text
Payment là module thu tiền, xác nhận tiền, hoàn tiền và đối soát.
Không gộp payment status với order status.
COD không phải paid ngay.
Bank transfer phải có hướng dẫn rõ và admin mark paid an toàn.
Online payment không được tin redirect frontend.
Webhook phải verify và idempotent.
Refund không được vượt số tiền đã thanh toán.
Payment config có secret phải bảo mật.
Mọi thao tác payment/refund phải audit.
Mobile checkout không được overflow.
Admin phải có payment card trong order detail.
```
