# 08 — Storefront Customer Account Page Specification

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> File này là đặc tả giao diện và hành vi cho khu vực tài khoản khách hàng của website bán đồ điện tử.  
> Tài liệu này mở rộng từ:
>
> - `../main/ecommerce_design_language.md`
> - `01-electronics-store-theme.md`
> - `02-storefront-home-page.md`
> - `03-storefront-product-list-page.md`
> - `04-storefront-product-detail-page.md`
> - `05-storefront-cart-page.md`
> - `06-storefront-checkout-page.md`
> - `07-storefront-order-success-page.md`

---

## 1. Mục tiêu

Khu vực tài khoản khách hàng giúp người mua quản lý toàn bộ thông tin sau khi đã hoặc sẽ mua hàng:

- Đăng nhập.
- Đăng ký.
- Quên mật khẩu.
- Xem và sửa thông tin cá nhân.
- Quản lý địa chỉ nhận hàng.
- Xem lịch sử đơn hàng.
- Xem chi tiết đơn hàng.
- Theo dõi trạng thái giao hàng.
- Lưu sản phẩm yêu thích.
- Quản lý sản phẩm đã xem.
- Quản lý bảo hành.
- Đổi mật khẩu.
- Đăng xuất.

Với web bán đồ điện tử, khu vực tài khoản không chỉ để “đăng nhập”. Nó còn là nơi khách quay lại để:

- Kiểm tra laptop/điện thoại đã mua.
- Xem thời hạn bảo hành.
- Tải hóa đơn.
- Theo dõi đơn đang giao.
- Lưu sản phẩm đang cân nhắc.
- So sánh và mua lại phụ kiện.

---

## 2. Vai trò trong hệ thống

Customer Account nằm sau các flow:

```text
Home
→ Product List
→ Product Detail
→ Cart
→ Checkout
→ Order Success
→ Customer Account
```

Nó cũng có thể được truy cập trực tiếp từ Header:

```text
Header → Account Icon → Login / My Account
```

Nếu user đã đăng nhập:

```text
Account Icon → Account Dashboard
```

Nếu user chưa đăng nhập:

```text
Account Icon → Login Page
```

---

## 3. Phạm vi trang

File này bao gồm các màn hình sau:

```text
/auth/login
/auth/register
/auth/forgot-password
/auth/reset-password
/account
/account/profile
/account/addresses
/account/orders
/account/orders/{orderId}
/account/wishlist
/account/recently-viewed
/account/warranty
/account/security
```

MVP có thể làm trước:

```text
/auth/login
/auth/register
/account
/account/profile
/account/addresses
/account/orders
/account/orders/{orderId}
/account/security
```

Các phần có thể để phase sau:

```text
/account/wishlist
/account/recently-viewed
/account/warranty
Social login
OTP login
Membership points
Loyalty tier
Invoice download
Return/refund request
```

---

## 4. Nguyên tắc thiết kế

### 4.1. Cảm giác giao diện

Customer Account của web đồ điện tử cần tạo cảm giác:

- Tin cậy.
- Rõ ràng.
- Ít màu nhiễu.
- Dễ tra cứu.
- Có tính “dịch vụ sau bán hàng”.
- Không giống dashboard kỹ thuật quá nặng.

Nên ưu tiên:

- Nền sáng.
- Card rõ ràng.
- Icon đơn giản.
- Status badge dễ hiểu.
- Timeline đơn hàng trực quan.
- Bảng thông tin dễ đọc.

Không nên:

- Dùng quá nhiều gradient.
- Dùng animation gây mất tập trung.
- Ẩn thông tin quan trọng trong nhiều tab khó tìm.
- Bắt khách đăng nhập lại quá thường xuyên.

---

## 5. Layout tổng quan

### 5.1. Desktop layout

Desktop dùng layout 2 cột:

```text
┌─────────────────────────────────────────────────────────────┐
│ Header                                                      │
├─────────────────────────────────────────────────────────────┤
│ Breadcrumb                                                  │
├────────────────┬────────────────────────────────────────────┤
│ Account Sidebar │ Main Content                              │
│ 280px           │ Flexible                                  │
│                 │                                            │
│ - Dashboard     │ Page content                               │
│ - Profile       │                                            │
│ - Addresses     │                                            │
│ - Orders        │                                            │
│ - Wishlist      │                                            │
│ - Warranty      │                                            │
│ - Security      │                                            │
│ - Logout        │                                            │
├────────────────┴────────────────────────────────────────────┤
│ Footer                                                      │
└─────────────────────────────────────────────────────────────┘
```

### 5.2. Tablet layout

Tablet có thể giữ sidebar nhưng thu nhỏ:

```text
Sidebar: 220px
Main: remaining width
```

Nếu dưới breakpoint tablet nhỏ, chuyển sidebar thành horizontal nav hoặc drawer.

### 5.3. Mobile layout

Mobile không dùng sidebar cố định.

```text
┌─────────────────────────────┐
│ Header                      │
├─────────────────────────────┤
│ Account top summary card    │
├─────────────────────────────┤
│ Account menu list           │
├─────────────────────────────┤
│ Page content                │
└─────────────────────────────┘
```

Mobile navigation:

- Dạng menu list trong trang account dashboard.
- Hoặc dropdown “Tài khoản của tôi”.
- Khi vào trang con, có nút back rõ ràng.

---

## 6. Account route guard

### 6.1. Trang cần đăng nhập

Các route sau yêu cầu đăng nhập:

```text
/account
/account/profile
/account/addresses
/account/orders
/account/orders/{orderId}
/account/wishlist
/account/recently-viewed
/account/warranty
/account/security
```

Nếu chưa đăng nhập:

```text
Redirect → /auth/login?redirect={currentUrl}
```

Sau khi login thành công:

```text
Redirect back → redirect URL
```

Ví dụ:

```text
User vào /account/orders/123 khi chưa login
→ redirect /auth/login?redirect=/account/orders/123
→ login thành công
→ quay lại /account/orders/123
```

### 6.2. Trang auth khi đã đăng nhập

Nếu user đã đăng nhập và truy cập:

```text
/auth/login
/auth/register
```

thì redirect về:

```text
/account
```

---

## 7. Auth pages

## 7.1. Login Page

### 7.1.1. Mục tiêu

Cho phép khách đăng nhập bằng email/phone và mật khẩu.

### 7.1.2. Layout desktop

```text
┌──────────────────────────────────────────────┐
│ Header minimal                               │
├──────────────────────────────────────────────┤
│ Left marketing panel │ Login card             │
│ - Trust message      │ - Title                │
│ - Warranty message   │ - Email/phone input    │
│ - Order tracking     │ - Password input       │
│                      │ - Remember me          │
│                      │ - Forgot password      │
│                      │ - Login button         │
│                      │ - Register link        │
└──────────────────────────────────────────────┘
```

### 7.1.3. Layout mobile

```text
Header minimal
Login card full width
Register link
Footer minimal
```

### 7.1.4. Component

```text
AuthLayout
AuthMarketingPanel
LoginForm
PasswordInput
RememberMeCheckbox
AuthActionButton
AuthSwitchLink
AuthErrorAlert
```

### 7.1.5. Field

| Field | Required | Rule |
|---|---:|---|
| email_or_phone | Yes | Email hoặc số điện thoại hợp lệ |
| password | Yes | Không rỗng |
| remember_me | No | Boolean |

### 7.1.6. Validation message

```text
Email hoặc số điện thoại không được để trống.
Email hoặc số điện thoại không hợp lệ.
Mật khẩu không được để trống.
Thông tin đăng nhập không chính xác.
Tài khoản đã bị khóa. Vui lòng liên hệ hỗ trợ.
```

### 7.1.7. State

```text
idle
validating
submitting
success
invalid_credentials
locked_account
network_error
```

### 7.1.8. Login behavior

Khi user submit:

1. Validate client-side.
2. Disable button.
3. Hiển thị loading spinner trong button.
4. Gọi API login.
5. Nếu thành công, lưu session/token theo chiến lược auth của project.
6. Fetch `/users/me`.
7. Redirect về `redirect` nếu có, ngược lại `/account`.
8. Nếu lỗi, hiển thị error alert trên form.

### 7.1.9. UX rule

- Password input có nút show/hide.
- Enter trong input password phải submit form.
- Không clear email/phone nếu login fail.
- Không hiển thị password trong log hoặc error.
- Button login chỉ disabled khi đang submit hoặc form invalid.

---

## 7.2. Register Page

### 7.2.1. Mục tiêu

Cho phép khách tạo tài khoản mới để quản lý đơn hàng và bảo hành.

### 7.2.2. Field

| Field | Required | Rule |
|---|---:|---|
| full_name | Yes | 2-80 ký tự |
| email | Yes | Email hợp lệ |
| phone | Yes | Số điện thoại hợp lệ |
| password | Yes | Tối thiểu 8 ký tự |
| confirm_password | Yes | Trùng password |
| accept_terms | Yes | Bắt buộc tick |
| marketing_opt_in | No | Boolean |

### 7.2.3. Password rule

MVP:

```text
Ít nhất 8 ký tự.
Không được chỉ toàn khoảng trắng.
```

Phase sau:

```text
Có chữ thường.
Có chữ hoa.
Có số.
Có ký tự đặc biệt.
Không nằm trong danh sách password phổ biến.
```

### 7.2.4. Register behavior

1. Validate form.
2. Gọi API register.
3. Nếu thành công:
   - Option A: auto login và redirect `/account`.
   - Option B: yêu cầu xác thực email/phone.
4. Nếu email/phone đã tồn tại, hiển thị lỗi tại field tương ứng.

### 7.2.5. Error message

```text
Họ tên không được để trống.
Email không hợp lệ.
Số điện thoại không hợp lệ.
Mật khẩu phải có ít nhất 8 ký tự.
Mật khẩu xác nhận không khớp.
Bạn cần đồng ý với điều khoản sử dụng.
Email này đã được sử dụng.
Số điện thoại này đã được sử dụng.
```

---

## 7.3. Forgot Password Page

### 7.3.1. Mục tiêu

Cho phép user yêu cầu đặt lại mật khẩu bằng email hoặc số điện thoại.

### 7.3.2. Field

```text
email_or_phone
```

### 7.3.3. Behavior

1. User nhập email/phone.
2. Submit.
3. Hệ thống trả message trung tính:

```text
Nếu thông tin tồn tại, chúng tôi đã gửi hướng dẫn đặt lại mật khẩu.
```

Không nên nói “email không tồn tại” để tránh lộ thông tin tài khoản.

---

## 7.4. Reset Password Page

### 7.4.1. URL

```text
/auth/reset-password?token={token}
```

### 7.4.2. Field

```text
new_password
confirm_new_password
```

### 7.4.3. State

```text
valid_token
invalid_token
expired_token
submitting
success
```

### 7.4.4. Success behavior

Sau khi đổi mật khẩu thành công:

```text
Hiển thị success message
→ nút Đăng nhập
→ redirect /auth/login sau vài giây nếu muốn
```

---

## 8. Account Shell

## 8.1. Account Header Summary

Hiển thị ở đầu khu vực account.

### 8.1.1. Desktop

```text
Xin chào, Nguyễn Văn A
customer@example.com · 0900000000
Member since: 2026
```

### 8.1.2. Mobile

Dạng compact card:

```text
Nguyễn Văn A
0900000000
[Chỉnh sửa hồ sơ]
```

### 8.1.3. Component

```text
AccountSummaryCard
AccountAvatar
AccountName
AccountContact
AccountQuickAction
```

---

## 8.2. Account Sidebar

### 8.2.1. Menu item

```text
Dashboard
Hồ sơ cá nhân
Sổ địa chỉ
Đơn hàng của tôi
Sản phẩm yêu thích
Đã xem gần đây
Bảo hành
Bảo mật
Đăng xuất
```

### 8.2.2. Icon guideline

Icon dùng outline style, stroke consistent.

```text
Dashboard: grid/home icon
Profile: user icon
Addresses: map-pin icon
Orders: package icon
Wishlist: heart icon
Recently viewed: clock icon
Warranty: shield/check icon
Security: lock icon
Logout: log-out icon
```

### 8.2.3. Active state

Menu active:

- Nền primary-subtle.
- Text primary.
- Left border hoặc icon primary.
- `aria-current="page"`.

### 8.2.4. Logout

Logout item nên tách ở cuối sidebar.

Khi click:

1. Hiển thị confirm dialog nếu có dữ liệu chưa lưu.
2. Gọi API logout.
3. Clear session/token.
4. Redirect `/` hoặc `/auth/login`.

---

## 9. Account Dashboard Page

## 9.1. URL

```text
/account
```

## 9.2. Mục tiêu

Cho khách cái nhìn nhanh về tài khoản:

- Đơn hàng gần đây.
- Địa chỉ mặc định.
- Sản phẩm yêu thích.
- Sản phẩm đang bảo hành.
- Gợi ý hành động tiếp theo.

## 9.3. Layout desktop

```text
Page title: Tài khoản của tôi

┌──────────────────┬──────────────────┬──────────────────┐
│ Total Orders     │ Pending Orders   │ Warranty Items   │
└──────────────────┴──────────────────┴──────────────────┘

┌────────────────────────────┬────────────────────────────┐
│ Recent Orders              │ Default Address            │
└────────────────────────────┴────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Recommended Actions                                     │
└─────────────────────────────────────────────────────────┘
```

## 9.4. Dashboard cards

### 9.4.1. Summary Stat Card

```text
Title
Value
Description
Icon
Link
```

Ví dụ:

```text
Đơn hàng
12
Xem tất cả đơn hàng
```

```text
Đang xử lý
2
Theo dõi đơn đang giao
```

```text
Bảo hành
3
Sản phẩm còn bảo hành
```

### 9.4.2. Recent Orders Card

Hiển thị 3-5 đơn gần nhất.

Mỗi item:

```text
Order number
Order date
Status badge
Total amount
View detail link
```

### 9.4.3. Default Address Card

Nếu có địa chỉ mặc định:

```text
Nguyễn Văn A
0900000000
Số 1, Hà Đông, Hà Nội
[Chỉnh sửa]
```

Nếu chưa có:

```text
Bạn chưa có địa chỉ nhận hàng mặc định.
[Thêm địa chỉ]
```

### 9.4.4. Recommended Actions

Gợi ý theo trạng thái:

```text
Bạn có 1 đơn đang chờ thanh toán.
Bạn có 2 sản phẩm yêu thích đang giảm giá.
Bạn chưa thêm địa chỉ mặc định.
```

MVP có thể dùng rule đơn giản, chưa cần AI.

## 9.5. Empty state

Nếu user mới chưa có đơn:

```text
Bạn chưa có đơn hàng nào.
Khám phá sản phẩm công nghệ mới nhất tại cửa hàng.
[Tiếp tục mua sắm]
```

---

## 10. Profile Page

## 10.1. URL

```text
/account/profile
```

## 10.2. Mục tiêu

Cho phép khách xem và chỉnh sửa thông tin cá nhân.

## 10.3. Field

| Field | Editable | Rule |
|---|---:|---|
| avatar_url | Yes | Ảnh hợp lệ |
| full_name | Yes | 2-80 ký tự |
| email | Limited | Có thể cần verify |
| phone | Limited | Có thể cần OTP |
| gender | Yes | optional |
| date_of_birth | Yes | optional |
| marketing_opt_in | Yes | boolean |

## 10.4. Layout

```text
Page title: Hồ sơ cá nhân

Card: Basic Information
- Avatar uploader
- Full name
- Email
- Phone
- Gender
- Date of birth
- Marketing opt-in

Actions:
[Hủy] [Lưu thay đổi]
```

## 10.5. Behavior

- Page fetch `/users/me`.
- Form prefill từ user data.
- Button “Lưu thay đổi” disabled nếu không có thay đổi.
- Khi submit, disable form và button.
- Nếu success, hiển thị toast.
- Nếu lỗi field, hiển thị dưới input.

## 10.6. Email/phone change rule

Nếu thay đổi email hoặc phone:

- MVP: cho phép update trực tiếp nếu backend hỗ trợ.
- Phase sau: yêu cầu verify email/OTP.

UI cần support state:

```text
verified
unverified
verification_pending
```

## 10.7. Avatar rule

- Cho phép jpg/png/webp.
- Giới hạn dung lượng theo backend.
- Có preview trước khi upload.
- Có fallback initials nếu không có avatar.

---

## 11. Address Book Page

## 11.1. URL

```text
/account/addresses
```

## 11.2. Mục tiêu

Cho phép khách quản lý nhiều địa chỉ nhận hàng.

## 11.3. Layout

```text
Page title: Sổ địa chỉ
[Thêm địa chỉ mới]

Address card list
- Address card 1
- Address card 2
- Address card 3
```

## 11.4. Address Card

Mỗi card hiển thị:

```text
Full name
Phone
Full address
Default badge nếu là mặc định
Actions: Sửa | Xóa | Đặt làm mặc định
```

## 11.5. Address Form

Có thể là modal hoặc page riêng.

### Field

| Field | Required | Rule |
|---|---:|---|
| full_name | Yes | 2-80 ký tự |
| phone | Yes | Phone hợp lệ |
| country | Yes | Default Vietnam |
| city | Yes | Tỉnh/thành phố |
| district | Yes | Quận/huyện |
| ward | Yes | Phường/xã |
| street | Yes | Số nhà, tên đường |
| postal_code | No | Optional |
| note | No | Optional |
| is_default | No | Boolean |

## 11.6. Validation message

```text
Họ tên người nhận không được để trống.
Số điện thoại không hợp lệ.
Vui lòng chọn tỉnh/thành phố.
Vui lòng chọn quận/huyện.
Vui lòng chọn phường/xã.
Vui lòng nhập địa chỉ cụ thể.
```

## 11.7. Behavior

### Add address

1. Click `Thêm địa chỉ mới`.
2. Mở modal/page form.
3. Submit.
4. Nếu success, đóng modal và refresh list.
5. Nếu là địa chỉ đầu tiên, tự set default.

### Edit address

1. Click `Sửa`.
2. Form prefill.
3. Submit update.
4. Success toast.

### Delete address

1. Click `Xóa`.
2. Mở confirm dialog.
3. Nếu confirm, delete.
4. Nếu address đang dùng trong order cũ, không ảnh hưởng order snapshot.
5. Nếu xóa default, cần chọn default mới hoặc để trống.

### Set default

1. Click `Đặt làm mặc định`.
2. Update server.
3. Refresh list.
4. Badge default chuyển sang card mới.

## 11.8. Empty state

```text
Bạn chưa có địa chỉ nhận hàng nào.
Thêm địa chỉ để checkout nhanh hơn ở lần mua tiếp theo.
[Thêm địa chỉ]
```

---

## 12. Orders List Page

## 12.1. URL

```text
/account/orders
```

## 12.2. Mục tiêu

Cho khách xem lịch sử đơn hàng và lọc theo trạng thái.

## 12.3. Layout

```text
Page title: Đơn hàng của tôi

Order status tabs:
[Tất cả] [Chờ xác nhận] [Đang xử lý] [Đang giao] [Hoàn thành] [Đã hủy]

Search / filter bar:
- Search order number/product name
- Date range

Order list
```

## 12.4. Order status tab

Status tab cần hiển thị count nếu backend trả về.

```text
Tất cả (12)
Chờ xác nhận (1)
Đang giao (2)
Hoàn thành (8)
Đã hủy (1)
```

Nếu count chưa có, vẫn hiển thị label.

## 12.5. Order List Item

Mỗi order item là một card.

### Header

```text
Order number
Order date
Order status badge
Payment status badge
```

### Body

Hiển thị 1-3 sản phẩm đầu tiên:

```text
Product thumbnail
Product name
Variant/spec summary
Quantity
Price
```

Nếu nhiều hơn:

```text
+3 sản phẩm khác
```

### Footer

```text
Total amount
View detail button
Reorder button
Track shipment button nếu có
Pay now button nếu payment pending
Cancel order button nếu được phép
```

## 12.6. Status mapping

### Order status

```text
pending_confirmation
confirmed
processing
packed
shipping
delivered
completed
cancelled
returned
```

### Display label

| Status | Label |
|---|---|
| pending_confirmation | Chờ xác nhận |
| confirmed | Đã xác nhận |
| processing | Đang xử lý |
| packed | Đã đóng gói |
| shipping | Đang giao |
| delivered | Đã giao |
| completed | Hoàn thành |
| cancelled | Đã hủy |
| returned | Đã trả hàng |

### Badge color intent

```text
pending_confirmation: warning
confirmed: info
processing: info
packed: info
shipping: primary
completed: success
cancelled: danger/neutral
returned: neutral
```

## 12.7. Payment status

```text
unpaid
pending
paid
failed
refunded
partial_refunded
```

Display:

```text
Chưa thanh toán
Đang chờ thanh toán
Đã thanh toán
Thanh toán thất bại
Đã hoàn tiền
Hoàn tiền một phần
```

## 12.8. Action rule

| Action | Condition |
|---|---|
| View detail | Luôn có |
| Pay now | payment_status = unpaid/failed và payment method online |
| Transfer instruction | payment_method = bank_transfer và payment_status = pending/unpaid |
| Cancel order | order_status thuộc pending/confirmed và chưa giao |
| Track shipment | shipping_status có tracking code |
| Reorder | completed/cancelled/delivered |
| Review product | completed/delivered |

## 12.9. Cancel order behavior

Khi user click cancel:

1. Mở confirm dialog.
2. Chọn lý do hủy nếu cần.
3. Gọi API cancel.
4. Nếu success, cập nhật status.
5. Nếu không được hủy nữa, hiển thị message:

```text
Đơn hàng đã được xử lý và không thể hủy trực tuyến. Vui lòng liên hệ hỗ trợ.
```

## 12.10. Empty state

Nếu không có đơn hàng:

```text
Bạn chưa có đơn hàng nào.
Khám phá các sản phẩm công nghệ mới nhất dành cho bạn.
[Tiếp tục mua sắm]
```

Nếu filter không có kết quả:

```text
Không tìm thấy đơn hàng phù hợp.
Hãy thử thay đổi bộ lọc hoặc từ khóa tìm kiếm.
```

---

## 13. Order Detail Page

## 13.1. URL

```text
/account/orders/{orderId}
```

## 13.2. Mục tiêu

Cho khách xem đầy đủ thông tin một đơn hàng:

- Mã đơn.
- Trạng thái.
- Timeline.
- Sản phẩm.
- Địa chỉ.
- Thanh toán.
- Vận chuyển.
- Hóa đơn/bảo hành.
- Hành động tiếp theo.

## 13.3. Layout desktop

```text
Page header
- Back to orders
- Order number
- Status badges

Main column
- Order timeline
- Order items
- Shipping info
- Payment info
- Warranty info

Side column
- Order summary
- Support card
- Actions
```

## 13.4. Layout mobile

Mobile dùng single column:

```text
Back link
Order status summary
Order timeline
Order items
Order summary
Shipping info
Payment info
Support
Actions
```

## 13.5. Order Header

Hiển thị:

```text
Đơn hàng #DH202606220001
Đặt ngày 22/06/2026
Trạng thái: Đang giao
Thanh toán: Đã thanh toán
```

Action:

```text
Copy order number
Contact support
Download invoice nếu có
```

## 13.6. Order Timeline

Timeline gồm các bước:

```text
Đặt hàng
Xác nhận
Đóng gói
Bàn giao vận chuyển
Đang giao
Đã giao
Hoàn thành
```

Mỗi step:

```text
label
status: done/current/pending/failed/cancelled
timestamp
note optional
```

Nếu order bị hủy:

```text
Đặt hàng → Đã hủy
```

Nếu payment failed:

```text
Đặt hàng → Thanh toán thất bại
```

## 13.7. Order Items

Hiển thị danh sách sản phẩm trong đơn.

Mỗi item:

```text
Product image
Product name snapshot
Variant/spec snapshot
SKU
Quantity
Price snapshot
Warranty duration
Review button nếu đủ điều kiện
Buy again button
```

Vì đây là đơn cũ, phải dùng snapshot, không phụ thuộc Product hiện tại.

Ví dụ:

```text
Laptop Dell Inspiron 15
Core i5 / 16GB / SSD 512GB
SKU: DELL-I15-16-512
Số lượng: 1
Giá lúc mua: 15.990.000đ
Bảo hành: 24 tháng
```

## 13.8. Order Summary

```text
Tạm tính
Giảm giá
Phí vận chuyển
Thuế nếu có
Tổng cộng
```

Nếu có coupon:

```text
Coupon: SALE10
Giảm: -500.000đ
```

## 13.9. Shipping Info

```text
Người nhận
Số điện thoại
Địa chỉ
Phương thức giao hàng
Đơn vị vận chuyển
Mã vận đơn
Tracking link nếu có
```

Có nút copy tracking code.

## 13.10. Payment Info

```text
Phương thức thanh toán
Trạng thái thanh toán
Thời gian thanh toán
Mã giao dịch nếu có
```

Nếu bank transfer pending:

```text
Hiển thị lại hướng dẫn chuyển khoản.
```

Nếu online failed:

```text
Hiển thị Retry payment.
```

## 13.11. Warranty Info

Đồ điện tử nên có block bảo hành.

Mỗi item có thể hiển thị:

```text
Warranty status
Warranty start date
Warranty end date
Serial number nếu có
Support contact
```

Nếu chưa có serial:

```text
Thông tin bảo hành sẽ được cập nhật sau khi đơn hàng hoàn tất.
```

## 13.12. Support Card

```text
Cần hỗ trợ đơn hàng này?
Hotline: 1900 xxxx
Email: support@example.com
Chat với tư vấn viên
```

MVP có thể chỉ hiển thị hotline/email.

---

## 14. Wishlist Page

## 14.1. URL

```text
/account/wishlist
```

## 14.2. Mục tiêu

Cho khách lưu sản phẩm đang cân nhắc.

Với đồ điện tử, wishlist thường dùng để:

- Theo dõi giá laptop/điện thoại.
- Lưu model để so sánh.
- Chờ khuyến mãi.

## 14.3. Layout

```text
Page title: Sản phẩm yêu thích
Toolbar: Sort / Filter in stock / Filter price dropped
Product grid/list
```

## 14.4. Wishlist Card

Dựa trên ProductCard của electronics theme nhưng thêm:

```text
Remove from wishlist
Added date
Price drop badge nếu có
Compare checkbox
```

## 14.5. Actions

```text
Remove
Add to cart
Compare
View detail
```

## 14.6. Empty state

```text
Bạn chưa lưu sản phẩm nào.
Lưu laptop, điện thoại hoặc phụ kiện yêu thích để dễ theo dõi giá và khuyến mãi.
[Khám phá sản phẩm]
```

## 14.7. MVP

Wishlist có thể để phase sau. Nếu chưa làm, header/account không nên hiển thị link dead.

---

## 15. Recently Viewed Page

## 15.1. URL

```text
/account/recently-viewed
```

## 15.2. Mục tiêu

Giúp khách quay lại các sản phẩm đã xem.

## 15.3. Data source

Có thể lấy từ:

- Local storage cho guest.
- Server history cho logged-in user.

## 15.4. Rule

- Lưu tối đa 20-50 sản phẩm.
- Không lưu trùng.
- Sản phẩm xem gần nhất lên đầu.
- Nếu sản phẩm đã ngừng bán, hiển thị trạng thái phù hợp.

---

## 16. Warranty Page

## 16.1. URL

```text
/account/warranty
```

## 16.2. Mục tiêu

Cho khách xem danh sách sản phẩm đã mua còn bảo hành.

Đây là điểm quan trọng với web đồ điện tử.

## 16.3. Layout

```text
Page title: Bảo hành sản phẩm
Tabs: Còn bảo hành | Hết bảo hành | Tất cả
Warranty item list
```

## 16.4. Warranty Item Card

```text
Product image
Product name
Order number
Serial number
Warranty start date
Warranty end date
Warranty status badge
Support action
```

## 16.5. Warranty status

```text
active
expiring_soon
expired
pending_activation
unknown
```

Display:

```text
Còn bảo hành
Sắp hết bảo hành
Hết bảo hành
Chờ kích hoạt
Chưa có thông tin
```

## 16.6. Action

```text
View order
Contact support
Request warranty service
Download invoice
```

MVP:

```text
View order
Contact support
```

## 16.7. Empty state

```text
Bạn chưa có sản phẩm bảo hành nào.
Sau khi mua hàng thành công, thông tin bảo hành sẽ hiển thị tại đây.
```

---

## 17. Security Page

## 17.1. URL

```text
/account/security
```

## 17.2. Mục tiêu

Cho khách quản lý bảo mật tài khoản.

## 17.3. Sections

```text
Change Password
Login Sessions
Two-factor Authentication
Account Deletion
```

MVP chỉ cần:

```text
Change Password
Logout
```

## 17.4. Change Password Form

### Field

| Field | Required | Rule |
|---|---:|---|
| current_password | Yes | Không rỗng |
| new_password | Yes | Ít nhất 8 ký tự |
| confirm_new_password | Yes | Trùng new_password |

### Behavior

1. User nhập mật khẩu hiện tại.
2. Nhập mật khẩu mới.
3. Confirm password.
4. Submit.
5. Nếu success, hiển thị toast.
6. Có thể yêu cầu login lại tùy security policy.

### Error message

```text
Mật khẩu hiện tại không đúng.
Mật khẩu mới phải có ít nhất 8 ký tự.
Mật khẩu xác nhận không khớp.
Không thể đổi mật khẩu lúc này. Vui lòng thử lại.
```

## 17.5. Session management phase sau

Nếu có nhiều thiết bị:

```text
Current device
Other sessions
Logout this session
Logout all other sessions
```

## 17.6. Account deletion phase sau

Account deletion là tính năng nhạy cảm.

UI cần:

- Cảnh báo rõ.
- Confirm nhiều bước.
- Yêu cầu password.
- Không xóa đơn hàng lịch sử nếu luật kế toán yêu cầu giữ.

---

## 18. Component Specification

## 18.1. AccountLayout

### Props

```ts
interface AccountLayoutProps {
  user: AccountUser;
  activeMenu: AccountMenuKey;
  children: ReactNode;
}
```

Tên framework chỉ là ví dụ. Có thể chuyển sang Vue/Svelte/Blade.

### Responsibility

- Render header/footer.
- Render account summary.
- Render sidebar/mobile nav.
- Render main content.
- Handle loading auth state.

## 18.2. AccountSidebar

### Props

```ts
interface AccountSidebarProps {
  activeKey: AccountMenuKey;
  items: AccountMenuItem[];
  onLogout: () => void;
}
```

### Rule

- Active item có `aria-current="page"`.
- Keyboard navigable.
- Logout là button, không phải link nếu trigger action.

## 18.3. StatusBadge

Dùng chung cho order, payment, warranty.

### Props

```ts
interface StatusBadgeProps {
  label: string;
  intent: 'neutral' | 'info' | 'success' | 'warning' | 'danger' | 'primary';
}
```

### Rule

- Không chỉ dùng màu để truyền nghĩa.
- Label phải rõ.
- Badge không quá dài.

## 18.4. OrderCard

### Props

```ts
interface OrderCardProps {
  order: OrderSummary;
  onCancel?: () => void;
  onReorder?: () => void;
  onPayNow?: () => void;
}
```

### Rule

- Không lấy giá hiện tại của product để hiển thị order cũ.
- Dùng order snapshot.
- Product name dài tối đa 2 dòng trong list.

## 18.5. AddressCard

### Props

```ts
interface AddressCardProps {
  address: CustomerAddress;
  onEdit: () => void;
  onDelete: () => void;
  onSetDefault: () => void;
}
```

### Rule

- Full address phải wrap tốt trên mobile.
- SĐT có thể mask một phần nếu cần privacy.
- Delete phải có confirm.

## 18.6. ConfirmDialog

Dùng cho:

- Xóa địa chỉ.
- Hủy đơn.
- Đăng xuất khi có form dirty.

### Props

```ts
interface ConfirmDialogProps {
  title: string;
  description: string;
  confirmLabel: string;
  cancelLabel: string;
  intent?: 'danger' | 'neutral';
}
```

---

## 19. Data Contract

## 19.1. AccountUser

```ts
interface AccountUser {
  id: string;
  fullName: string;
  email: string;
  phone: string;
  avatarUrl?: string;
  gender?: 'male' | 'female' | 'other' | 'unknown';
  dateOfBirth?: string;
  emailVerified: boolean;
  phoneVerified: boolean;
  createdAt: string;
  marketingOptIn: boolean;
}
```

## 19.2. CustomerAddress

```ts
interface CustomerAddress {
  id: string;
  fullName: string;
  phone: string;
  country: string;
  city: string;
  district: string;
  ward: string;
  street: string;
  postalCode?: string;
  note?: string;
  isDefault: boolean;
  createdAt: string;
  updatedAt: string;
}
```

## 19.3. OrderSummary

```ts
interface OrderSummary {
  id: string;
  orderNumber: string;
  placedAt: string;
  orderStatus: OrderStatus;
  paymentStatus: PaymentStatus;
  shippingStatus?: ShippingStatus;
  totalAmount: Money;
  items: OrderSummaryItem[];
  availableActions: OrderAction[];
}
```

## 19.4. OrderSummaryItem

```ts
interface OrderSummaryItem {
  id: string;
  productId: string;
  productNameSnapshot: string;
  productImageSnapshot?: string;
  variantSnapshot?: string;
  skuSnapshot?: string;
  quantity: number;
  priceSnapshot: Money;
}
```

## 19.5. OrderDetail

```ts
interface OrderDetail extends OrderSummary {
  timeline: OrderTimelineStep[];
  shippingAddress: OrderAddressSnapshot;
  billingAddress?: OrderAddressSnapshot;
  payment: PaymentInfo;
  shipment?: ShipmentInfo;
  summary: OrderPriceSummary;
  warrantyItems?: WarrantyItem[];
  supportInfo: SupportInfo;
}
```

## 19.6. WarrantyItem

```ts
interface WarrantyItem {
  id: string;
  productName: string;
  productImage?: string;
  orderNumber: string;
  serialNumber?: string;
  warrantyStartDate?: string;
  warrantyEndDate?: string;
  status: 'active' | 'expiring_soon' | 'expired' | 'pending_activation' | 'unknown';
}
```

## 19.7. Money

```ts
interface Money {
  amount: number;
  currency: 'VND' | 'USD' | string;
  formatted: string;
}
```

---

## 20. API Contract

Endpoint chỉ là gợi ý. Có thể thay đổi theo backend.

## 20.1. Auth

```http
POST /api/v1/auth/login
POST /api/v1/auth/register
POST /api/v1/auth/logout
POST /api/v1/auth/forgot-password
POST /api/v1/auth/reset-password
POST /api/v1/auth/change-password
GET  /api/v1/users/me
```

## 20.2. Profile

```http
GET   /api/v1/users/me
PATCH /api/v1/users/me
POST  /api/v1/users/me/avatar
```

## 20.3. Addresses

```http
GET    /api/v1/account/addresses
POST   /api/v1/account/addresses
PATCH  /api/v1/account/addresses/{addressId}
DELETE /api/v1/account/addresses/{addressId}
POST   /api/v1/account/addresses/{addressId}/set-default
```

## 20.4. Orders

```http
GET  /api/v1/account/orders?status=&page=&pageSize=&q=&from=&to=
GET  /api/v1/account/orders/{orderId}
POST /api/v1/account/orders/{orderId}/cancel
POST /api/v1/account/orders/{orderId}/reorder
POST /api/v1/account/orders/{orderId}/pay
```

## 20.5. Wishlist

```http
GET    /api/v1/account/wishlist
POST   /api/v1/account/wishlist/items
DELETE /api/v1/account/wishlist/items/{productId}
```

## 20.6. Warranty

```http
GET /api/v1/account/warranty
GET /api/v1/account/warranty/{warrantyId}
```

---

## 21. State Management

## 21.1. Auth state

```text
unknown
unauthenticated
authenticated
expired
```

### Rule

- Khi app boot, auth state là `unknown`.
- Không render account content khi chưa biết auth state.
- Nếu token hết hạn, redirect login.
- Không render nhấp nháy nội dung private trước redirect.

## 21.2. Page loading state

Mỗi account page cần có:

```text
initial_loading
loaded
empty
error
submitting
```

## 21.3. Form dirty state

Profile, address, security form cần detect dirty state.

Nếu user rời trang khi form dirty:

```text
Bạn có thay đổi chưa lưu. Bạn có chắc muốn rời trang?
```

MVP có thể bỏ qua, nhưng nên có nếu dùng agent code production.

---

## 22. Loading / Empty / Error States

## 22.1. Loading

Dùng skeleton thay vì spinner toàn trang nếu có layout ổn định.

Ví dụ orders list:

```text
Skeleton order card x 3
```

Profile:

```text
Skeleton form fields
```

Address:

```text
Skeleton address card x 2
```

## 22.2. Empty

Empty state phải có:

```text
Icon
Title
Short description
Primary action
```

Không dùng message cụt như:

```text
No data
```

## 22.3. Error

Error state phải có:

```text
Title
Description
Retry button
Support link nếu cần
```

Ví dụ:

```text
Không thể tải đơn hàng.
Vui lòng thử lại hoặc liên hệ hỗ trợ nếu lỗi tiếp tục xảy ra.
[Thử lại]
```

## 22.4. Unauthorized

Nếu session hết hạn:

```text
Phiên đăng nhập đã hết hạn. Vui lòng đăng nhập lại.
```

Sau đó redirect login.

---

## 23. Responsive Rules

## 23.1. Breakpoints

Dùng breakpoint từ design language gốc.

Gợi ý:

```text
Mobile: < 640px
Tablet: 640px - 1023px
Desktop: >= 1024px
Large desktop: >= 1280px
```

## 23.2. Desktop

- Account sidebar cố định bên trái trong content container.
- Main content max width phù hợp.
- Form không quá rộng; form card max width 720px.
- Order detail có thể dùng 2 columns.

## 23.3. Tablet

- Sidebar có thể thu nhỏ.
- Order cards vẫn giữ card layout.
- Form full width trong main column.

## 23.4. Mobile

- Không có sidebar cố định.
- Menu account thành list hoặc dropdown.
- Order cards một cột.
- Table không được overflow ngang; chuyển thành card rows.
- Action buttons stack vertical nếu thiếu width.
- Nút quan trọng phải dễ bấm, chiều cao tối thiểu 44px.

---

## 24. Accessibility Rules

## 24.1. Keyboard

- Tab order hợp lý.
- Modal focus trap.
- Escape đóng modal nếu không nguy hiểm.
- Button/link có focus visible.

## 24.2. Screen reader

- Form field có label thật.
- Error message liên kết với input qua `aria-describedby`.
- Status badge có text rõ ràng.
- Loading region có `aria-busy` nếu cần.

## 24.3. Contrast

- Text chính đạt contrast tốt.
- Badge không chỉ dựa vào màu.
- Disabled state vẫn đọc được.

## 24.4. Form

Không dùng placeholder thay label.

Sai:

```text
Input chỉ có placeholder “Email”
```

Đúng:

```text
Label: Email
Input placeholder: example@email.com
```

---

## 25. Security & Privacy Rules

## 25.1. Không lộ thông tin nhạy cảm

Không hiển thị:

- Password.
- Token.
- Full payment transaction secret.
- Nội dung lỗi backend nhạy cảm.

## 25.2. Masking

Có thể mask một phần:

```text
Email: ng***@example.com
Phone: 090****000
```

Tuy nhiên trong profile edit, có thể hiển thị đầy đủ nếu đã authenticated.

## 25.3. CSRF / XSS

- Form mutation phải dùng CSRF protection nếu dùng cookie session.
- Không render HTML từ user input nếu chưa sanitize.
- Address note không được render HTML.

## 25.4. Authorization

User chỉ được xem order/address của chính mình.

Frontend không được tin route id.

Backend bắt buộc check ownership.

## 25.5. Rate limit UX

Login/forgot password nếu bị rate limited:

```text
Bạn đã thử quá nhiều lần. Vui lòng thử lại sau ít phút.
```

---

## 26. Analytics Events

Không bắt buộc MVP, nhưng nên định nghĩa sẵn.

```text
login_submit
login_success
login_failed
register_submit
register_success
profile_update_success
address_add_success
address_delete_success
order_list_view
order_detail_view
order_cancel_submit
order_cancel_success
wishlist_add
wishlist_remove
warranty_view
password_change_success
```

Event không được chứa password, token, hoặc thông tin cá nhân nhạy cảm.

---

## 27. SEO Rules

Account pages thường không cần index.

### Meta robots

```html
<meta name="robots" content="noindex,nofollow" />
```

Áp dụng cho:

```text
/auth/login
/auth/register
/account/*
```

Public order lookup nếu có cũng nên noindex.

---

## 28. Performance Rules

- Account shell code có thể lazy load sau khi user vào account.
- Order list phân trang, không load toàn bộ lịch sử.
- Address list thường nhỏ, có thể load một lần.
- Order detail fetch riêng theo id.
- Wishlist có pagination nếu nhiều.
- Avatar image cần resize và lazy load.
- Không block page bằng analytics.

---

## 29. Copywriting

## 29.1. Tone

Giọng văn:

- Rõ ràng.
- Thân thiện.
- Không quá kỹ thuật.
- Không đổ lỗi user.

## 29.2. Ví dụ tốt

```text
Không thể tải đơn hàng. Vui lòng thử lại.
```

## 29.3. Ví dụ không tốt

```text
Request failed with status 500.
```

## 29.4. Button labels

Dùng động từ rõ:

```text
Đăng nhập
Tạo tài khoản
Lưu thay đổi
Thêm địa chỉ
Xem chi tiết
Theo dõi đơn hàng
Đổi mật khẩu
Đăng xuất
```

Không dùng label mơ hồ:

```text
OK
Submit
Click here
```

---

## 30. Agent Implementation Rules

Khi agent implement khu vực tài khoản khách hàng, bắt buộc tuân thủ:

### 30.1. Không tự ý thiết kế khác spec

Agent không được tự ý:

- Đổi layout account shell.
- Bỏ sidebar desktop.
- Bỏ mobile account menu.
- Bỏ loading/empty/error state.
- Bỏ validation.
- Bỏ route guard.

Nếu cần thay đổi, phải cập nhật spec trước.

### 30.2. Phải tách component

Không code toàn bộ account vào một file lớn.

Tối thiểu nên tách:

```text
AccountLayout
AccountSidebar
AccountSummaryCard
LoginForm
RegisterForm
ProfileForm
AddressCard
AddressForm
OrderCard
OrderTimeline
OrderSummary
StatusBadge
ConfirmDialog
```

### 30.3. Không hard-code trạng thái phân tán

Status mapping phải gom vào một file/constants.

Ví dụ:

```text
order-status.ts
payment-status.ts
warranty-status.ts
```

### 30.4. Không render dữ liệu private khi chưa auth

Nếu auth state unknown:

```text
Render loading shell
```

Không render account page rồi mới redirect.

### 30.5. Form phải có validation

Form bắt buộc có:

- Required validation.
- Format validation.
- Server error mapping.
- Submit loading state.

### 30.6. Không bỏ qua responsive

Agent phải kiểm tra:

```text
Desktop 1440px
Tablet 768px
Mobile 375px
```

### 30.7. Không kết luận xong nếu chưa test

Sau khi sửa, agent phải chạy test liên quan.

---

## 31. Suggested File Structure

Không phụ thuộc framework, nhưng nên có cấu trúc tương đương:

```text
src/
  modules/
    account/
      components/
        AccountLayout.*
        AccountSidebar.*
        AccountSummaryCard.*
        StatusBadge.*
        ConfirmDialog.*
      auth/
        LoginPage.*
        RegisterPage.*
        ForgotPasswordPage.*
        ResetPasswordPage.*
        LoginForm.*
        RegisterForm.*
      profile/
        ProfilePage.*
        ProfileForm.*
      addresses/
        AddressBookPage.*
        AddressCard.*
        AddressForm.*
      orders/
        OrdersListPage.*
        OrderCard.*
        OrderDetailPage.*
        OrderTimeline.*
        OrderItems.*
        OrderSummary.*
      wishlist/
        WishlistPage.*
      warranty/
        WarrantyPage.*
        WarrantyCard.*
      security/
        SecurityPage.*
        ChangePasswordForm.*
      api.*
      types.*
      constants.*
      mappers.*
      validators.*
```

Tests:

```text
tests/
  e2e/
    auth-login.spec.*
    auth-register.spec.*
    account-profile.spec.*
    account-addresses.spec.*
    account-orders.spec.*
    account-order-detail.spec.*
    account-security.spec.*
  visual/
    account-dashboard.visual.spec.*
    account-orders.visual.spec.*
    account-mobile.visual.spec.*
```

---

## 32. Playwright Test Specification

## 32.1. Auth tests

### Login success

```text
Given user is on login page
When user enters valid email/phone and password
And clicks login
Then user is redirected to account dashboard
And account summary is visible
```

### Login invalid

```text
Given user is on login page
When user enters wrong credentials
Then error message is visible
And password value is cleared or preserved according to security policy
And email/phone remains visible
```

### Login redirect

```text
Given user opens /account/orders while unauthenticated
Then user is redirected to /auth/login?redirect=/account/orders
When login succeeds
Then user returns to /account/orders
```

### Register validation

```text
Given user is on register page
When required fields are empty
And user submits
Then each required field displays validation error
```

### Forgot password neutral response

```text
Given user is on forgot password page
When user submits email
Then page shows neutral instruction message
```

## 32.2. Profile tests

### Update profile success

```text
Given user is logged in
When user updates full name
And clicks save
Then success toast appears
And new full name is shown in account summary
```

### Profile validation

```text
Given user is on profile page
When full name is empty
Then save button is disabled or validation appears
```

## 32.3. Address tests

### Add address

```text
Given user has no address
When user adds a valid address
Then address card appears
And it is marked as default
```

### Edit address

```text
Given user has an address
When user edits phone number
Then updated phone number appears in address card
```

### Delete address confirm

```text
Given user has an address
When user clicks delete
Then confirm dialog appears
When user confirms
Then address is removed
```

### Set default address

```text
Given user has two addresses
When user sets second address as default
Then default badge moves to second address
```

## 32.4. Orders tests

### Orders list visible

```text
Given user has orders
When user opens orders page
Then order cards are visible
And status tabs are visible
```

### Filter by status

```text
Given user has multiple order statuses
When user clicks “Đang giao” tab
Then only shipping orders are visible
```

### Search order

```text
Given user has order #DH001
When user searches DH001
Then matching order is visible
```

### Empty order state

```text
Given user has no orders
When user opens orders page
Then empty state appears
And continue shopping button is visible
```

## 32.5. Order detail tests

### Order detail visible

```text
Given user has an order
When user opens order detail
Then order number is visible
And timeline is visible
And order items are visible
And order summary is visible
```

### Copy order number

```text
Given user is on order detail
When user clicks copy order number
Then success feedback appears
```

### Cancel order

```text
Given order is cancellable
When user clicks cancel order
Then confirm dialog appears
When user confirms
Then order status becomes cancelled
```

### Bank transfer instruction

```text
Given order uses bank transfer and payment is pending
When user opens order detail
Then bank transfer instruction is visible
```

## 32.6. Security tests

### Change password validation

```text
Given user is on security page
When new password and confirm password do not match
Then validation error appears
```

### Change password success

```text
Given user enters correct current password and valid new password
When user submits
Then success toast appears
```

## 32.7. Responsive tests

Run account flow at:

```text
1440x900
768x1024
375x812
```

Check:

```text
No horizontal overflow
Account menu usable on mobile
Order cards readable
Buttons not clipped
Forms usable
Modal fits screen
```

---

## 33. Visual Regression Checklist

Capture screenshots for:

```text
Login page desktop
Login page mobile
Register page desktop
Account dashboard desktop
Account dashboard mobile
Profile page
Address book empty
Address book with cards
Orders list with orders
Orders list empty
Order detail desktop
Order detail mobile
Security page
```

Visual diff should catch:

- Sidebar broken.
- Mobile overflow.
- Form label misalignment.
- Order card layout broken.
- Status badge unreadable.
- Timeline broken.
- Button clipped.

---

## 34. Definition of Done

Một implementation của customer account chỉ được coi là xong khi:

### 34.1. UI

- Desktop account layout đúng 2 cột.
- Mobile không dùng sidebar cố định.
- Header/footer consistent với storefront.
- Status badges hiển thị rõ.
- Loading/empty/error state đầy đủ.
- Không có horizontal overflow ở 375px.

### 34.2. Functional

- Login/register hoạt động.
- Route guard hoạt động.
- Profile update hoạt động.
- Address CRUD hoạt động.
- Orders list hiển thị đúng.
- Order detail hiển thị đúng snapshot.
- Change password hoạt động hoặc có placeholder rõ nếu backend chưa có.

### 34.3. Security

- Không render private data khi chưa auth.
- Không leak token/password.
- User không thể xem order của người khác.
- Error message không lộ thông tin nhạy cảm.

### 34.4. Tests

- Auth E2E pass.
- Profile E2E pass.
- Address E2E pass.
- Orders E2E pass.
- Order detail E2E pass.
- Responsive visual test pass.

### 34.5. Agent report

Agent phải báo cáo:

```text
Files changed
Components created/updated
Routes added/updated
API mocked/integrated
Tests executed
Screenshots/trace if relevant
Known limitations
```

---

## 35. MVP Scope Recommendation

Để đi nhanh, version đầu nên làm:

```text
Login
Register
Forgot password basic
Account dashboard simple
Profile edit
Address book CRUD
Orders list
Order detail
Change password
Logout
```

Tạm hoãn:

```text
Wishlist
Recently viewed server sync
Warranty request workflow
Invoice download
Return/refund workflow
OTP login
Social login
Loyalty points
2FA
Session management
Account deletion
```

---

## 36. Agent Task Prompt Template

Khi giao agent code khu vực account, dùng prompt kiểu này:

```text
Đọc các file:
- ../main/ecommerce_design_language.md
- 01-electronics-store-theme.md
- 08-storefront-customer-account-page.md

Implement customer account module theo spec.

Phạm vi hiện tại:
- Login page
- Register page
- Account dashboard
- Profile page
- Address book page
- Orders list page
- Order detail page
- Security/change password page

Yêu cầu:
1. Không tự ý đổi layout/spec.
2. Tách component rõ ràng.
3. Có route guard.
4. Có loading/empty/error state.
5. Có responsive desktop/tablet/mobile.
6. Có Playwright tests cho auth, profile, addresses, orders.
7. Chụp visual screenshots cho desktop/mobile.
8. Không kết luận xong nếu chưa chạy test.
```

---

## 37. Related Next Specs

Sau file này, các file tiếp theo nên là:

```text
09-admin-dashboard.md
10-admin-product-management.md
11-admin-order-management.md
12-admin-customer-management.md
13-admin-inventory-management.md
14-admin-promotion-management.md
15-admin-warranty-management.md
16-shared-component-spec.md
17-playwright-e2e-test-plan.md
18-agent-coding-rules.md
```

---

## 38. Final Note

Customer Account là khu vực ảnh hưởng lớn đến niềm tin sau mua hàng. Với web bán đồ điện tử, khách không chỉ muốn xem lịch sử đơn, mà còn muốn biết:

- Đơn đang ở đâu.
- Sản phẩm còn bảo hành không.
- Có thể tải hóa đơn không.
- Có thể mua lại phụ kiện không.
- Có thể liên hệ hỗ trợ dễ không.

Vì vậy UI của khu vực này phải rõ ràng, ổn định, an toàn và dễ dùng hơn là quá trang trí.
