# 14 - Admin Promotion Management Specification

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Module: Promotion / Coupon / Campaign Management  
> Mục tiêu: Đặc tả đủ chi tiết để coding agent/frontend/backend có thể xây module quản lý khuyến mãi từ đầu đến cuối.  
> Phụ thuộc:  
> - `ecommerce_design_language.md`  
> - `01-electronics-store-theme.md`  
> - `09-admin-dashboard.md`  
> - `12-admin-order-management.md`  
> - `13-admin-inventory-management.md`  
> - `system-design.md`

---

# 0. Prompt hoàn chỉnh cho coding agent

Dùng prompt này khi giao task cho AI coding agent:

```text
Bạn là Senior Full-stack Engineer kiêm UI/UX Implementation Agent.

Nhiệm vụ của bạn là implement module Admin Promotion Management cho một website bán hàng đồ điện tử.

Trước khi code, bắt buộc đọc và tuân thủ các tài liệu sau:

1. ecommerce_design_language.md
2. 01-electronics-store-theme.md
3. 09-admin-dashboard.md
4. 12-admin-order-management.md
5. 13-admin-inventory-management.md
6. 14-admin-promotion-management.md
7. system-design.md

Mục tiêu module:
- Admin có thể tạo, sửa, xem, lọc, kích hoạt, tạm dừng, kết thúc, archive promotion/coupon/campaign.
- Hỗ trợ coupon giảm theo số tiền, giảm theo phần trăm, miễn phí vận chuyển, flash sale, product campaign, category campaign, combo/bundle ở mức mở rộng.
- Có rule điều kiện áp dụng rõ ràng: thời gian, min order, sản phẩm/danh mục áp dụng, khách hàng áp dụng, số lượt dùng, số lượt dùng mỗi khách, giới hạn giảm tối đa.
- Có preview tính giá để admin kiểm tra trước khi publish.
- Có conflict detection để tránh nhiều promotion chồng nhau gây sai giá hoặc lỗ.
- Có audit log cho mọi thay đổi nhạy cảm.
- Có permission rõ: ai được xem, ai được tạo, ai được publish, ai được sửa campaign đang chạy.
- Có loading/empty/error/permission states.
- Responsive không overflow mobile.
- Có test Playwright cho các flow chính.

Yêu cầu UI:
- Dùng admin design language: rõ, nhanh, ít màu, ưu tiên dữ liệu và thao tác chính xác.
- Không dùng màu hard-code ngoài design token.
- Status badge phải nhất quán.
- Action nguy hiểm như end campaign, archive, delete draft, force stop phải có confirm modal.
- Form dài phải chia section, không dồn một khối.
- Table phải có search/filter/pagination/bulk action hợp lý.
- Mobile dùng card list hoặc drawer, không để table làm overflow toàn page.

Yêu cầu nghiệp vụ:
- Không tin frontend khi tính discount. Backend phải tính lại toàn bộ.
- Không cho publish promotion thiếu điều kiện quan trọng.
- Không cho percentage discount không có max discount nếu policy yêu cầu chống lỗ.
- Không cho thời gian kết thúc nhỏ hơn thời gian bắt đầu.
- Không cho coupon active nếu code trùng với coupon active khác.
- Không cho sửa các trường nhạy cảm của promotion đang chạy nếu không có quyền cao.
- Mọi discount áp dụng vào order phải lưu snapshot tại thời điểm đặt hàng.

Sau khi code xong, báo cáo:
- Files changed
- Components created/updated
- APIs integrated/mocked
- Data assumptions
- Permission assumptions
- Tests added/updated
- Tests run
- Viewports checked
- Known limitations
```

---

# 1. Vai trò của module Promotion Management

Promotion Management là module giúp shop tạo các chương trình thúc đẩy bán hàng.

Trong website bán đồ điện tử, promotion thường ảnh hưởng trực tiếp tới:

```text
Giá hiển thị ngoài storefront
Product card
Product detail
Cart
Checkout
Order total
Payment amount
Admin order detail
Analytics
Inventory planning
Marketing campaign
```

Nếu promotion sai, hậu quả có thể rất lớn:

```text
Khách nhìn một giá nhưng checkout ra giá khác
Coupon áp dụng sai sản phẩm
Discount vượt mức gây lỗ
Flash sale vẫn chạy sau khi hết thời gian
Coupon bị dùng nhiều hơn giới hạn
Đơn hàng lưu sai discount snapshot
Admin không biết campaign nào đang ảnh hưởng doanh thu
```

Vì vậy module này cần được thiết kế theo hướng:

```text
Rõ điều kiện
Rõ phạm vi
Rõ thời gian
Rõ giới hạn
Rõ conflict
Rõ trạng thái
Có preview trước khi publish
Có audit log
```

---

# 2. Các loại promotion cần hỗ trợ

## 2.1. Coupon code

Coupon code là mã khách nhập ở cart/checkout.

Ví dụ:

```text
SALE10
LAPTOP500K
FREESHIP
BACKTOSCHOOL
```

Coupon thường có rule:

```text
Giảm theo phần trăm
Giảm số tiền cố định
Miễn phí vận chuyển
Điều kiện đơn tối thiểu
Giảm tối đa
Giới hạn lượt dùng
Giới hạn mỗi khách
Thời gian hiệu lực
Áp dụng cho sản phẩm/danh mục/thương hiệu cụ thể
```

## 2.2. Automatic promotion

Automatic promotion tự áp dụng khi đơn hàng thỏa điều kiện.

Ví dụ:

```text
Giảm 500.000đ cho laptop Dell từ 15 triệu
Giảm 5% cho màn hình gaming
Miễn phí ship đơn từ 3 triệu
Tặng balo khi mua laptop văn phòng
```

Khách không cần nhập mã.

## 2.3. Flash sale

Flash sale là campaign có thời gian ngắn, thường áp dụng vào sản phẩm cụ thể.

Ví dụ:

```text
Flash sale laptop gaming 20:00 - 23:59
MacBook giảm 2 triệu cuối tuần
Tai nghe sale sốc trong 2 giờ
```

Flash sale cần kiểm soát:

```text
Thời gian bắt đầu/kết thúc
Số lượng bán trong campaign
Giá sale theo từng SKU/variant
Tồn kho có thể bán
Ưu tiên hiển thị ngoài storefront
```

## 2.4. Product campaign

Promotion áp dụng cho nhóm sản phẩm.

Ví dụ:

```text
Laptop văn phòng giảm đến 15%
Màn hình 144Hz giảm 700.000đ
Phụ kiện Logitech giảm 10%
```

## 2.5. Category campaign

Promotion áp dụng cho danh mục.

Ví dụ:

```text
Toàn bộ phụ kiện giảm 8%
Danh mục màn hình freeship
Laptop gaming giảm thêm 1 triệu khi thanh toán chuyển khoản
```

## 2.6. Brand campaign

Promotion áp dụng theo thương hiệu.

Ví dụ:

```text
Dell Back To School
ASUS Gaming Week
Apple Student Offer
```

## 2.7. Bundle / combo promotion

Mở rộng sau MVP.

Ví dụ:

```text
Mua laptop + chuột giảm 300.000đ
Mua màn hình + arm giảm 10%
Mua điện thoại + ốp + cường lực giảm 15%
```

## 2.8. Gift promotion

Mở rộng sau MVP.

Ví dụ:

```text
Mua laptop tặng balo
Mua điện thoại tặng củ sạc
Mua màn hình tặng cáp HDMI
```

---

# 3. Trạng thái promotion

## 3.1. Promotion status

Các trạng thái chuẩn:

| Status | Ý nghĩa | Có hiển thị ngoài storefront? | Có áp dụng discount? |
|---|---|---:|---:|
| Draft | Nháp | No | No |
| Scheduled | Đã lên lịch | Có thể hiển thị teaser | No trước start time |
| Active | Đang chạy | Yes | Yes |
| Paused | Tạm dừng | No hoặc badge paused trong admin | No |
| Expired | Hết hạn theo thời gian | No | No |
| Ended | Admin kết thúc thủ công | No | No |
| Archived | Lưu trữ | No | No |

## 3.2. Status transition

Luồng trạng thái hợp lệ:

```text
Draft -> Scheduled
Draft -> Active
Scheduled -> Active
Scheduled -> Paused
Active -> Paused
Paused -> Active
Active -> Ended
Scheduled -> Archived
Draft -> Archived
Expired -> Archived
Ended -> Archived
```

Không nên cho:

```text
Archived -> Active
Expired -> Active nếu không chỉnh lại thời gian
Ended -> Active nếu campaign đã có báo cáo khóa sổ
```

## 3.3. Status badge visual rule

| Status | Tone |
|---|---|
| Draft | Neutral |
| Scheduled | Info |
| Active | Success |
| Paused | Warning |
| Expired | Neutral |
| Ended | Neutral / Danger nhẹ |
| Archived | Neutral |

Rule:

```text
Status badge không chỉ dựa vào màu. Phải có text.
Active promotion phải nổi bật nhưng không dùng màu quá gắt trong admin.
Paused/Expired/Ended phải phân biệt rõ.
```

---

# 4. Admin routes

Route gợi ý:

```text
/admin/promotions
/admin/promotions/new
/admin/promotions/:id
/admin/promotions/:id/edit
/admin/promotions/:id/preview
/admin/promotions/:id/analytics
/admin/coupons
/admin/coupons/new
/admin/flash-sales
/admin/flash-sales/new
```

MVP có thể gom vào:

```text
/admin/promotions
/admin/promotions/new
/admin/promotions/:id/edit
```

Trong form dùng field `promotion_type` để phân biệt:

```text
coupon
automatic_discount
flash_sale
free_shipping
bundle
free_gift
```

---

# 5. Promotion list page

## 5.1. Mục đích

Trang danh sách promotion giúp admin:

```text
Xem tất cả chương trình khuyến mãi
Tìm promotion theo tên/code
Lọc theo loại, trạng thái, thời gian
Biết promotion nào đang chạy
Biết promotion nào sắp hết hạn
Biết promotion nào có conflict
Tạo promotion mới
Pause/end/archive promotion
Đi tới detail/edit/analytics
```

## 5.2. Desktop layout

```text
┌─────────────────────────────────────────────────────────────────────┐
│ Admin Topbar                                                        │
├───────────────┬─────────────────────────────────────────────────────┤
│ Sidebar       │ Promotions                                          │
│               │ Manage coupons, flash sales and automatic discounts │
│               │                                                     │
│               │ [Search promotion/code] [Type] [Status] [Date]      │
│               │ [Conflict] [Create promotion]                       │
│               │                                                     │
│               │ Quick tabs: All / Active / Scheduled / Draft / Issue │
│               │                                                     │
│               │ Promotion table                                     │
│               │                                                     │
│               │ Pagination                                          │
└───────────────┴─────────────────────────────────────────────────────┘
```

## 5.3. Mobile layout

Mobile admin không phải chính, nhưng phải usable.

```text
Topbar compact
Page title
Search
Filter button
Create button
Promotion cards
Pagination / Load more
```

Rule:

```text
Không render bảng đầy đủ trên mobile nếu gây overflow.
Promotion card mobile phải hiển thị: name, code/type, status, time, usage, action chính.
Action phụ đưa vào menu.
```

---

# 6. Promotion list toolbar

Toolbar gồm:

```text
Search input
Type filter
Status filter
Date range filter
Conflict filter
Channel filter optional
Create promotion button
Export button optional
```

## 6.1. Search

Search theo:

```text
Promotion name
Coupon code
Campaign ID
Product name trong campaign
Brand name
Category name
```

Placeholder:

```text
Search by promotion name, coupon code, product, brand...
```

Behavior:

```text
Debounce 300ms
Enter để search ngay
Có clear button
Query giữ trên URL
```

Ví dụ URL:

```text
/admin/promotions?search=backtoschool&type=coupon&status=active&page=1
```

## 6.2. Type filter

Options:

```text
All types
Coupon code
Automatic discount
Flash sale
Free shipping
Bundle
Free gift
```

## 6.3. Status filter

Options:

```text
All statuses
Draft
Scheduled
Active
Paused
Expired
Ended
Archived
```

## 6.4. Date filter

Filter theo:

```text
Start date
End date
Created date
Updated date
```

Preset:

```text
Today
This week
This month
Scheduled upcoming
Ending soon
Expired recently
Custom range
```

## 6.5. Conflict filter

Options:

```text
All
Has conflict
No conflict
Needs review
```

Conflict giúp admin phát hiện campaign chồng điều kiện.

---

# 7. Promotion table

## 7.1. Cột mặc định

| Column | Nội dung |
|---|---|
| Checkbox | Chọn nhiều |
| Promotion | Tên + type + code nếu có |
| Discount | Giá trị giảm |
| Scope | Áp dụng cho gì |
| Schedule | Thời gian bắt đầu/kết thúc |
| Usage | Đã dùng / giới hạn |
| Status | Trạng thái |
| Issues | Conflict/warning |
| Updated | Cập nhật lần cuối |
| Actions | Hành động |

## 7.2. Promotion cell

Hiển thị:

```text
Tên promotion
Loại promotion
Coupon code nếu có
Campaign ID optional
```

Ví dụ:

```text
Back To School Laptop Deal
Coupon: LAPTOP500K
Type: Coupon code
```

## 7.3. Discount cell

Hiển thị theo loại:

```text
10% off, max 1.000.000đ
500.000đ off
Free shipping, max 50.000đ
Flash price: 15.990.000đ
Gift: Laptop backpack
```

Rule:

```text
Percentage discount luôn hiển thị max discount nếu có.
Nếu không có max discount, hiển thị warning nếu policy yêu cầu.
```

## 7.4. Scope cell

Scope có thể là:

```text
All products
Selected products: 12
Category: Laptop > Gaming
Brand: Dell
Customer segment: Returning customers
Order subtotal >= 10.000.000đ
```

Không đưa danh sách sản phẩm dài vào table. Dùng summary + link/detail.

## 7.5. Schedule cell

Hiển thị:

```text
22/06/2026 00:00 -> 30/06/2026 23:59
Starts in 2 days
Ends in 4 hours
Expired 3 days ago
```

Rule:

```text
Campaign sắp hết hạn trong 24h có warning nhẹ.
Scheduled campaign hiển thị thời gian bắt đầu rõ.
```

## 7.6. Usage cell

Hiển thị:

```text
128 / 500 used
23 customers
No limit
Per customer: 1
```

Có progress bar nhẹ nếu usage limit tồn tại.

## 7.7. Issues cell

Hiển thị badge:

```text
No issue
Conflict
Missing max discount
No eligible product
Expired but active
Low stock campaign item
```

Rule:

```text
Issue nghiêm trọng phải có link tới phần cần sửa.
Không chỉ hiển thị icon đỏ không có giải thích.
```

## 7.8. Actions

Actions thường:

```text
View
Edit
Duplicate
Pause
Resume
End
Archive
Analytics
```

Rule:

```text
Promotion active không nên cho sửa mọi trường.
Pause/End/Archive cần confirm.
Duplicate tạo draft mới.
Analytics mở trang báo cáo campaign.
```

---

# 8. Quick tabs

Quick tabs giúp admin xử lý nhanh.

Gợi ý:

```text
All
Active
Scheduled
Draft
Ending soon
Needs review
Expired
Archived
```

Tab `Needs review` gồm:

```text
Has conflict
Missing max discount
No eligible item
Invalid schedule
Usage near limit
Low stock product in flash sale
```

---

# 9. Bulk actions

Bulk actions dùng cho danh sách promotion.

Actions:

```text
Pause selected
Resume selected
Archive selected
Export selected
End selected
```

Rule:

```text
Bulk end/pause/archive phải confirm.
Hiển thị số promotion bị ảnh hưởng.
Nếu một phần fail, hiển thị report.
Không cho bulk resume promotion expired nếu chưa sửa thời gian.
```

Ví dụ confirm:

```text
Pause 5 active promotions?
These promotions will stop applying discounts immediately.
Existing orders will not be changed.
```

---

# 10. Create/Edit promotion page

## 10.1. Mục đích

Form tạo/sửa promotion phải giúp admin cấu hình đúng mà không nhập sai.

Nên chia thành section:

```text
1. Basic information
2. Promotion type
3. Discount rule
4. Eligibility conditions
5. Applicable scope
6. Usage limits
7. Schedule
8. Display & storefront placement
9. Stacking & conflict rules
10. Preview calculation
11. Status & publish checklist
```

## 10.2. Desktop layout

```text
┌──────────────────────────────────────────────────────────────────┐
│ Topbar                                                           │
├──────────────┬─────────────────────────────────────┬─────────────┤
│ Sidebar      │ Main form                           │ Side panel  │
│              │ Basic information                   │ Status      │
│              │ Type                                │ Preview     │
│              │ Discount                            │ Checklist   │
│              │ Conditions                          │ Conflicts   │
│              │ Scope                               │ Actions     │
│              │ Usage limit                         │             │
│              │ Schedule                            │             │
│              │ Display                             │             │
└──────────────┴─────────────────────────────────────┴─────────────┘
```

Side panel sticky gồm:

```text
Save draft
Publish / Schedule / Update
Preview discount
Validation checklist
Conflict summary
Last updated
Created by
```

## 10.3. Mobile layout

```text
Single column form
Section accordion optional
Bottom action bar: Save draft / Preview / Publish
```

Rule:

```text
Không để side panel làm ngang page overflow.
Error summary phải xuất hiện trên đầu form sau submit fail.
```

---

# 11. Basic information section

Fields:

```text
Promotion name
Internal description
Promotion type
Coupon code nếu type = coupon
Priority
Tags optional
Owner/team optional
```

## 11.1. Promotion name

Rule:

```text
Required
Max 120 ký tự
Rõ campaign và mục tiêu
Không cần unique tuyệt đối nhưng nên dễ phân biệt
```

Ví dụ tốt:

```text
Back To School - Laptop Dell giảm 500K
```

Ví dụ kém:

```text
Sale tháng này
```

## 11.2. Internal description

Dùng cho admin hiểu lý do tạo campaign.

Ví dụ:

```text
Campaign xả tồn laptop văn phòng Dell, áp dụng trong tuần tựu trường.
```

Không hiển thị ngoài storefront mặc định.

## 11.3. Coupon code

Chỉ xuất hiện khi `promotion_type = coupon`.

Rule:

```text
Required với coupon
Uppercase recommended
Không chứa khoảng trắng
Unique trong các coupon active/scheduled overlapping time
Max 32 ký tự
Cho phép chữ, số, dấu gạch ngang, gạch dưới
```

Ví dụ:

```text
LAPTOP500K
DELL-SCHOOL-2026
FREESHIP
```

Validation error:

```text
Coupon code already exists for another active promotion.
```

## 11.4. Priority

Priority dùng khi nhiều promotion có thể áp dụng.

Rule:

```text
Số càng cao ưu tiên càng lớn hoặc ngược lại tùy convention, nhưng phải ghi rõ.
Default priority = 100.
Không dùng priority để che conflict mà không báo admin.
```

---

# 12. Promotion type section

Promotion types:

```text
coupon_fixed_amount
coupon_percentage
coupon_free_shipping
automatic_fixed_amount
automatic_percentage
automatic_free_shipping
flash_sale_price
bundle_discount
free_gift
```

MVP cần:

```text
coupon_fixed_amount
coupon_percentage
coupon_free_shipping
automatic_fixed_amount
automatic_percentage
flash_sale_price
```

Rule:

```text
Khi đổi promotion type, form discount rule thay đổi theo.
Nếu type đổi làm mất dữ liệu cũ, phải confirm.
```

---

# 13. Discount rule section

## 13.1. Fixed amount discount

Fields:

```text
Discount amount
Currency
Apply level: order / item / shipping
```

Rule:

```text
Amount > 0
Amount không được lớn hơn min order nếu policy yêu cầu
Currency phải khớp store currency hoặc có conversion strategy
```

Ví dụ:

```text
Giảm 500.000đ cho đơn laptop từ 15.000.000đ
```

## 13.2. Percentage discount

Fields:

```text
Discount percent
Maximum discount amount
Apply level
```

Rule:

```text
Percent > 0 và <= 100
Max discount strongly recommended
Với sản phẩm giá trị cao như đồ điện tử, nên bắt buộc max discount để chống lỗ
```

Ví dụ:

```text
Giảm 10%, tối đa 1.000.000đ
```

Không tốt:

```text
Giảm 50% không giới hạn cho toàn bộ laptop
```

## 13.3. Free shipping

Fields:

```text
Free shipping type: full / capped
Maximum shipping discount
Shipping methods included
Regions included
```

Rule:

```text
Nếu shipping fee cao theo vùng xa, nên có max shipping discount.
Không áp dụng free shipping cho item cồng kềnh nếu policy không cho.
```

## 13.4. Flash sale price

Fields:

```text
Product/variant
Flash sale price
Original price snapshot
Campaign stock quantity optional
Per customer limit optional
```

Rule:

```text
Flash sale price phải nhỏ hơn current selling price.
Phải hiển thị cảnh báo nếu sản phẩm tồn kho thấp.
Không làm thay đổi base price vĩnh viễn.
Giá flash sale chỉ có hiệu lực trong campaign time.
```

## 13.5. Bundle discount

Mở rộng sau MVP.

Fields:

```text
Bundle condition
Required products/categories
Discount type
Discount value
Bundle display title
```

Ví dụ:

```text
Mua laptop + chuột giảm 300.000đ
```

## 13.6. Free gift

Mở rộng sau MVP.

Fields:

```text
Gift product/variant
Gift quantity
Gift stock source
Gift replacement policy
```

Rule:

```text
Gift cũng cần kiểm tra tồn kho.
Order item gift cần đánh dấu price = 0 và có reason snapshot.
```

---

# 14. Eligibility conditions

Eligibility là điều kiện để promotion được áp dụng.

## 14.1. Order conditions

Fields:

```text
Minimum order subtotal
Maximum order subtotal optional
Minimum quantity
Payment method condition
Shipping method condition
```

Ví dụ:

```text
Đơn từ 10.000.000đ
Chỉ áp dụng thanh toán chuyển khoản
Chỉ áp dụng COD optional
```

## 14.2. Customer conditions

Fields:

```text
All customers
Logged-in customers only
New customers
Returning customers
Customer segment
Specific customer list
Customer rank optional
```

Rule privacy:

```text
Không hiển thị danh sách khách hàng quá nhiều thông tin nhạy cảm.
Nếu import customer list, validate quyền.
```

## 14.3. Product conditions

Fields:

```text
All products
Selected products
Selected variants
Selected categories
Selected brands
Excluded products
Excluded categories
Excluded brands
```

Rule:

```text
Phải có summary rõ số sản phẩm eligible.
Nếu eligible product count = 0 thì không cho publish.
Nếu selected category có subcategory, cần option include subcategories.
```

## 14.4. Time conditions

Fields:

```text
Start datetime
End datetime
Timezone
Recurring schedule optional
```

Rule:

```text
Start < End
Timezone rõ ràng
Campaign active theo server time, không tin client time
```

## 14.5. Channel conditions

Mở rộng:

```text
Storefront web
Mobile app
Admin manual order
Marketplace sync
POS
```

MVP có thể default:

```text
Storefront web only
```

---

# 15. Applicable scope builder

## 15.1. Scope selector UI

Scope selector nên có tabs:

```text
Products
Categories
Brands
Customer segments
Payment/Shipping
Exclusions
```

## 15.2. Product picker

Product picker cần:

```text
Search name/SKU
Filter category
Filter brand
Filter stock status
Select product/variant
Show current price
Show stock
Show existing active promotion warning
```

Đồ điện tử cần chọn theo variant nếu giá/tồn kho khác nhau.

Ví dụ:

```text
Laptop Dell Inspiron 15 / 16GB / 512GB
Laptop Dell Inspiron 15 / 32GB / 1TB
```

Không chỉ chọn product cha nếu flash sale áp dụng theo variant.

## 15.3. Category picker

Category picker dạng tree.

Fields:

```text
Category
Include subcategories
Excluded subcategories optional
```

## 15.4. Brand picker

Brand picker searchable.

Fields:

```text
Brand name
Status
Product count
```

## 15.5. Exclusion rules

Exclusion dùng để loại sản phẩm/danh mục không áp dụng.

Ví dụ:

```text
Áp dụng toàn bộ Laptop nhưng loại trừ MacBook
Áp dụng Dell nhưng loại trừ dòng đang flash sale riêng
```

Rule:

```text
Exclusion ưu tiên hơn inclusion.
Phải hiển thị final eligible count.
```

---

# 16. Usage limits

## 16.1. Global usage limit

Fields:

```text
Total usage limit
Used count
Remaining count
```

Rule:

```text
Nếu limit = 0 hoặc null phải định nghĩa rõ là no limit hay disabled.
Không dùng ambiguous value.
```

## 16.2. Per customer limit

Fields:

```text
Usage per customer
Per day/week/month optional
```

Ví dụ:

```text
Mỗi khách dùng 1 lần
Mỗi khách dùng tối đa 3 lần trong campaign
```

## 16.3. Per order limit

Thông thường mỗi order chỉ áp dụng 1 coupon code.

Rule:

```text
Một order không được dùng cùng một coupon nhiều lần.
```

## 16.4. Flash sale quantity limit

Fields:

```text
Campaign stock quantity
Sold in campaign
Remaining campaign stock
```

Rule:

```text
Campaign stock không được vượt available stock nếu inventory strict.
Cần xử lý race condition ở backend.
```

---

# 17. Schedule section

Fields:

```text
Start datetime
End datetime
Timezone
Auto start
Auto end
Early end allowed
```

## 17.1. Date/time display

Admin Việt Nam format:

```text
22/06/2026 00:00
30/06/2026 23:59
```

Có helper:

```text
Starts in 2 days
Ends in 4 hours
Expired 3 days ago
```

## 17.2. Schedule validation

Rules:

```text
Start time required khi publish/schedule
End time required nếu policy không cho evergreen promotion
Start time < end time
Cannot schedule in the past unless publish immediately
Cannot extend expired promotion without permission
```

## 17.3. Evergreen promotion

Một số promotion có thể không có end date.

Ví dụ:

```text
Freeship cho đơn từ 5 triệu
Coupon welcome cho khách mới
```

Rule:

```text
Nếu no end date, phải có warning để admin hiểu promotion chạy vô thời hạn.
Có thể yêu cầu quyền cao.
```

---

# 18. Display & storefront placement

Không phải promotion nào cũng cần hiển thị ngoài storefront.

Fields:

```text
Show on product card
Show on product detail
Show on cart coupon list
Show on homepage campaign banner
Show in promotion page
Badge text
Short description for customer
Terms link / terms content
```

## 18.1. Badge text

Rule:

```text
Ngắn, rõ, không quá 20 ký tự nếu hiển thị trên card.
Không dùng câu dài trong badge.
```

Ví dụ tốt:

```text
-10%
Giảm 500K
Freeship
Trả góp 0%
```

Ví dụ kém:

```text
Chương trình khuyến mãi siêu khủng chỉ hôm nay
```

## 18.2. Customer-facing terms

Nội dung điều kiện hiển thị cho khách:

```text
Áp dụng cho laptop Dell từ 15.000.000đ.
Mỗi khách dùng 1 lần.
Không áp dụng cùng mã giảm giá khác.
Hiệu lực đến 30/06/2026.
```

Rule:

```text
Không giấu điều kiện quan trọng.
Điều kiện phải đồng bộ với rule thực tế backend.
```

---

# 19. Stacking & conflict rules

Promotion stacking là việc nhiều promotion cùng áp dụng vào một order.

## 19.1. Stacking modes

Options:

```text
Cannot combine with other promotions
Can combine with automatic promotions
Can combine with free shipping only
Can combine with product-level discounts
Can combine with all eligible promotions
```

MVP khuyến nghị:

```text
Một order chỉ dùng 1 coupon code.
Automatic product discount có thể áp dụng trước coupon nếu policy cho.
Free shipping có thể combine nếu cấu hình.
```

## 19.2. Discount calculation order

Cần định nghĩa rõ thứ tự tính:

```text
1. Product base price
2. Product-level sale/flash sale
3. Cart subtotal
4. Order-level coupon/automatic discount
5. Shipping discount
6. Tax/fee nếu có
7. Final payable amount
```

Hoặc nếu tax tính trước discount, phải ghi rõ theo luật/logic business.

## 19.3. Conflict detection

Conflict examples:

```text
Hai flash sale active cùng một variant cùng thời gian
Coupon percentage toàn bộ laptop chồng campaign laptop category quá cao
Promotion active nhưng không có eligible product
Coupon code trùng
Promotion scheduled overlap với campaign đã có priority cao
Free shipping không có max cap trong vùng phí cao
Percentage discount thiếu max discount
Flash sale price cao hơn current price
```

## 19.4. Conflict severity

| Severity | Ý nghĩa | Có cho publish? |
|---|---|---:|
| Info | Cần biết | Yes |
| Warning | Cần xem xét | Có thể có confirm |
| Blocking | Không hợp lệ | No |

Ví dụ blocking:

```text
Start date sau end date
Coupon code trùng active
Eligible product count = 0
Flash sale price >= current price
```

Ví dụ warning:

```text
Percentage discount không có max cap
Campaign overlap category khác
No end date
Low stock in campaign item
```

---

# 20. Preview calculation

Preview là phần cực kỳ quan trọng.

Admin cần biết promotion sẽ giảm như thế nào trước khi publish.

## 20.1. Preview input

Cho phép admin nhập/chọn:

```text
Sample product
Sample variant
Quantity
Cart subtotal
Shipping fee
Customer type
Payment method
Existing promotions optional
```

## 20.2. Preview output

Hiển thị:

```text
Original subtotal
Product discount
Coupon discount
Shipping discount
Final total
Reason applied
Reason not applied nếu không đủ điều kiện
```

Ví dụ:

```text
Subtotal: 18.990.000đ
Product flash sale: -2.000.000đ
Coupon LAPTOP500K: -500.000đ
Shipping: 0đ
Final total: 16.490.000đ
```

## 20.3. Not eligible preview

Nếu không đủ điều kiện:

```text
Promotion not applied.
Reason: Order subtotal must be at least 20.000.000đ.
Current subtotal: 18.990.000đ.
```

Rule:

```text
Preview chỉ là mô phỏng. Backend checkout vẫn phải tính lại thật.
```

---

# 21. Promotion detail page

Detail page dùng để xem campaign sau khi tạo.

Sections:

```text
Header summary
Status
Discount rule
Eligibility
Scope
Usage stats
Schedule
Conflict/warning
Order usage list
Audit log
Actions
```

## 21.1. Header summary

Hiển thị:

```text
Promotion name
Type
Status badge
Coupon code nếu có
Created by
Created at
Last updated
```

## 21.2. Usage summary

Metrics:

```text
Total uses
Total discount amount
Revenue influenced
Orders using promotion
Average order value
Remaining usage
```

MVP có thể chỉ cần:

```text
Used count
Usage limit
Total discount amount
```

## 21.3. Related orders

Table đơn đã dùng promotion:

```text
Order number
Customer masked
Order total
Discount amount
Used at
Payment status
Order status
```

Privacy:

```text
Mask phone/email nếu không có quyền.
Không hiển thị full address ở promotion detail.
```

---

# 22. Coupon code behavior at checkout

Dù file này là admin, vẫn cần định nghĩa behavior ở cart/checkout.

## 22.1. Apply coupon flow

```text
Customer enters coupon code
Frontend calls validate/apply coupon API
Backend checks eligibility
Backend returns discount preview
Cart/checkout displays applied coupon
Place order backend recalculates discount
Order stores promotion snapshot
```

## 22.2. Coupon invalid messages

Customer-facing messages phải dễ hiểu:

```text
Mã giảm giá không hợp lệ hoặc đã hết hạn.
Đơn hàng chưa đủ điều kiện áp dụng mã này.
Mã này đã hết lượt sử dụng.
Mã này không áp dụng cho sản phẩm trong giỏ hàng.
Bạn đã sử dụng mã này trước đó.
```

Không hiển thị lỗi kỹ thuật:

```text
PromotionRuleException: usage_limit_exceeded
```

## 22.3. Order snapshot

Khi order được tạo, lưu snapshot:

```text
Promotion ID
Promotion name
Coupon code
Discount type
Discount value
Discount amount actually applied
Eligibility reason
Applied items
Applied at
```

Rule:

```text
Nếu promotion bị sửa sau khi order tạo, order cũ không đổi.
```

---

# 23. Data model

## 23.1. Promotion object

```json
{
  "id": "promo_001",
  "name": "Back To School - Laptop Dell giảm 500K",
  "type": "coupon_fixed_amount",
  "status": "scheduled",
  "code": "LAPTOP500K",
  "description_internal": "Campaign tựu trường cho laptop Dell.",
  "customer_description": "Giảm 500.000đ cho laptop Dell từ 15.000.000đ.",
  "discount": {
    "type": "fixed_amount",
    "amount": 500000,
    "percent": null,
    "max_discount_amount": null,
    "currency": "VND",
    "apply_level": "order"
  },
  "conditions": {
    "min_order_subtotal": 15000000,
    "max_order_subtotal": null,
    "customer_segment": "all",
    "payment_methods": [],
    "shipping_methods": []
  },
  "scope": {
    "include_products": [],
    "include_variants": [],
    "include_categories": ["cat_laptop"],
    "include_brands": ["brand_dell"],
    "exclude_products": [],
    "exclude_categories": [],
    "include_subcategories": true
  },
  "usage_limit": {
    "total": 500,
    "per_customer": 1,
    "used": 128
  },
  "schedule": {
    "start_at": "2026-06-22T00:00:00+07:00",
    "end_at": "2026-06-30T23:59:59+07:00",
    "timezone": "Asia/Ho_Chi_Minh"
  },
  "stacking": {
    "mode": "coupon_only",
    "priority": 100,
    "can_combine_with_free_shipping": true
  },
  "display": {
    "show_on_product_card": true,
    "show_on_product_detail": true,
    "show_on_cart": true,
    "badge_text": "Giảm 500K"
  },
  "created_by": "admin_001",
  "created_at": "2026-06-20T10:00:00+07:00",
  "updated_at": "2026-06-21T09:00:00+07:00"
}
```

## 23.2. Promotion usage object

```json
{
  "id": "usage_001",
  "promotion_id": "promo_001",
  "order_id": "order_001",
  "order_number": "DH1024",
  "customer_id": "cus_001",
  "coupon_code": "LAPTOP500K",
  "discount_amount": 500000,
  "currency": "VND",
  "applied_items": [
    {
      "order_item_id": "item_001",
      "product_id": "prod_001",
      "variant_id": "var_001",
      "discount_amount": 500000
    }
  ],
  "used_at": "2026-06-22T14:35:00+07:00"
}
```

## 23.3. Promotion conflict object

```json
{
  "id": "conflict_001",
  "promotion_id": "promo_001",
  "severity": "warning",
  "type": "overlapping_scope",
  "title": "Overlaps with active laptop campaign",
  "message": "This promotion overlaps with 'Laptop Gaming Week' for 24 eligible products.",
  "blocking": false,
  "related_promotion_ids": ["promo_002"],
  "created_at": "2026-06-21T09:10:00+07:00"
}
```

---

# 24. API contract

API chỉ là gợi ý, có thể đổi theo framework.

## 24.1. Promotion list

```http
GET /api/admin/promotions
```

Query params:

```text
search
type
status
start_from
start_to
end_from
end_to
has_conflict
page
page_size
sort
```

## 24.2. Promotion detail

```http
GET /api/admin/promotions/{id}
```

## 24.3. Create promotion

```http
POST /api/admin/promotions
```

## 24.4. Update promotion

```http
PATCH /api/admin/promotions/{id}
```

## 24.5. Save draft

```http
POST /api/admin/promotions/{id}/save-draft
```

## 24.6. Publish / schedule

```http
POST /api/admin/promotions/{id}/publish
POST /api/admin/promotions/{id}/schedule
```

## 24.7. Pause / resume / end / archive

```http
POST /api/admin/promotions/{id}/pause
POST /api/admin/promotions/{id}/resume
POST /api/admin/promotions/{id}/end
POST /api/admin/promotions/{id}/archive
```

## 24.8. Duplicate

```http
POST /api/admin/promotions/{id}/duplicate
```

Response tạo draft mới.

## 24.9. Validate before publish

```http
POST /api/admin/promotions/{id}/validate-publish
```

Response:

```json
{
  "valid": false,
  "blockingIssues": [],
  "warnings": [],
  "eligibleProductCount": 24,
  "conflicts": []
}
```

## 24.10. Preview discount

```http
POST /api/admin/promotions/preview
```

Body:

```json
{
  "promotionDraft": {},
  "sampleCart": {
    "items": [
      {
        "product_id": "prod_001",
        "variant_id": "var_001",
        "quantity": 1
      }
    ],
    "shipping_fee": 30000,
    "payment_method": "bank_transfer"
  },
  "customer_context": {
    "customer_id": null,
    "segment": "guest"
  }
}
```

## 24.11. Storefront apply coupon

```http
POST /api/v1/cart/coupons/apply
DELETE /api/v1/cart/coupons/{code}
```

## 24.12. Promotion usage

```http
GET /api/admin/promotions/{id}/usages
GET /api/admin/promotions/{id}/analytics
```

---

# 25. Validation rules

## 25.1. Draft validation

Draft cho phép thiếu nhiều field nhưng vẫn validate dữ liệu nguy hiểm:

```text
Coupon code format nếu có
Discount value không âm
Percent không vượt 100
Start/end nếu nhập phải hợp lệ
Rich text terms phải sanitize
```

## 25.2. Publish validation

Publish yêu cầu:

```text
Name required
Type required
Discount rule valid
Schedule valid
Eligible scope not empty nếu không phải all products
Coupon code required nếu type coupon
Coupon code unique trong thời gian overlap
Usage limit valid nếu có
Customer-facing terms required nếu promotion public
No blocking conflict
Preview calculation successful
```

## 25.3. Electronics-specific validation

Với đồ điện tử giá trị cao:

```text
Percentage discount nên có max discount.
Flash sale product phải có stock.
Flash sale nên apply theo variant nếu variant có giá khác nhau.
Không cho promotion làm final price âm.
Không cho discount lớn hơn item subtotal.
Không cho coupon áp dụng vào sản phẩm discontinued/archived.
```

---

# 26. Permission matrix

Roles tham khảo:

```text
Super Admin
Store Manager
Marketing Manager
Marketing Staff
Product Manager
Order Manager
Viewer
```

| Permission | Super | Store Manager | Marketing Manager | Marketing Staff | Product Manager | Order Manager | Viewer |
|---|---:|---:|---:|---:|---:|---:|---:|
| promotion.view | yes | yes | yes | yes | yes | yes | yes |
| promotion.create | yes | yes | yes | yes | no | no | no |
| promotion.update_draft | yes | yes | yes | yes | no | no | no |
| promotion.publish | yes | yes | yes | no | no | no | no |
| promotion.pause | yes | yes | yes | no | no | no | no |
| promotion.end | yes | yes | yes | no | no | no | no |
| promotion.archive | yes | yes | yes | no | no | no | no |
| promotion.view_analytics | yes | yes | yes | yes | yes | no | yes |
| promotion.force_override_conflict | yes | yes | no | no | no | no | no |

Rule:

```text
UI ẩn action nếu user không có quyền.
Backend vẫn phải kiểm tra permission.
Không cho role thấp sửa promotion active nếu ảnh hưởng giá thật.
```

---

# 27. Security rules

```text
Backend phải tính discount, frontend chỉ hiển thị kết quả.
Không trust coupon amount từ client.
Không trust final total từ client.
Không lộ rule nội bộ nhạy cảm cho customer.
Rate limit apply coupon để tránh brute force code.
Coupon code không nên dễ đoán nếu dành riêng cho nhóm nhỏ.
Sanitize terms/description nếu dùng rich text.
Audit mọi thay đổi liên quan discount.
```

## 27.1. Coupon brute force protection

Cần rate limit theo:

```text
IP
Session
Customer ID
Coupon code attempts
```

Customer-facing error không nên nói quá chi tiết với coupon private:

```text
Mã giảm giá không hợp lệ hoặc không thể áp dụng.
```

## 27.2. Sensitive internal fields

Không expose ra storefront:

```text
Internal description
Cost impact
Profit margin
Created by
Conflict notes nội bộ
Priority nếu không cần
```

---

# 28. Audit log

Promotion audit events:

```text
Promotion created
Promotion updated
Promotion published
Promotion scheduled
Promotion paused
Promotion resumed
Promotion ended
Promotion archived
Coupon code changed
Discount value changed
Scope changed
Schedule changed
Usage limit changed
Stacking rule changed
Conflict overridden
```

Audit fields:

```text
Actor
Action
Entity
Before
After
Reason optional
Timestamp
IP/device optional
```

Rule:

```text
Thay đổi discount value, schedule, scope của promotion active phải ghi audit rõ.
Nếu action yêu cầu reason, reason phải lưu.
```

---

# 29. Loading, empty, error states

## 29.1. List loading

```text
Toolbar visible
Table skeleton rows
Không hiển thị fake data
```

## 29.2. Empty promotions

Message:

```text
No promotions yet.
Create your first coupon or campaign to start boosting sales.
```

Actions:

```text
Create coupon
Create flash sale
```

## 29.3. Filtered empty

Message:

```text
No promotions match your filters.
Try changing keywords or clearing filters.
```

Action:

```text
Clear filters
```

## 29.4. Form loading

```text
Skeleton từng section
Side panel skeleton
Không cho submit khi chưa load xong
```

## 29.5. Error states

Lỗi thường gặp:

```text
Cannot load promotions
Cannot save promotion
Coupon code already exists
Invalid discount value
Schedule is invalid
Promotion has blocking conflicts
Permission denied
Preview calculation failed
```

Rule:

```text
Form save fail không được mất dữ liệu admin đã nhập.
Validation error phải link tới field.
Network error có retry.
```

---

# 30. Confirmation dialogs

Các action cần confirm:

```text
Pause active promotion
Resume promotion with warning
End active promotion
Archive promotion
Delete draft
Override conflict warning
Change discount value of active promotion
Change scope of active promotion
```

Dialog phải có:

```text
Title rõ
Mô tả hậu quả
Promotion name/code
Số order/customer có thể ảnh hưởng nếu biết
Cancel button
Confirm button
Reason textarea nếu action nhạy cảm
```

Ví dụ:

```text
End this promotion now?
Customers will no longer be able to use coupon LAPTOP500K.
Existing orders will not be changed.
```

---

# 31. Accessibility rules

```text
Tất cả input có label.
Coupon code input có helper text format.
Error message liên kết field.
Modal confirm trap focus.
Icon-only button có aria-label.
Badge không chỉ dùng màu.
Table header semantic đúng.
Preview result có text rõ, không chỉ biểu đồ/màu.
Keyboard thao tác được filter, action menu, modal.
```

---

# 32. Responsive rules

## 32.1. Desktop

```text
Sidebar visible
Promotion table đầy đủ
Form dùng main + side panel
Preview panel sticky
```

## 32.2. Tablet

```text
Table có thể ẩn cột phụ
Filter row scroll/wrap hợp lý
Side panel chuyển xuống dưới hoặc sticky hẹp
```

## 32.3. Mobile

```text
Sidebar drawer
Promotion list chuyển thành card
Filter drawer
Form single column
Bottom action bar
Không overflow ngang
```

Mobile card hiển thị:

```text
Promotion name
Type/code
Status
Discount summary
Schedule
Usage
Primary action
More menu
```

---

# 33. Performance rules

```text
Promotion list phân trang.
Search debounce 300ms.
Product picker trong scope phải search async.
Không load toàn bộ product list vào form.
Eligible product count có thể tính async.
Conflict detection có thể gọi API riêng.
Analytics campaign load lazy.
Không render chart nặng nếu chưa mở analytics.
```

---

# 34. Analytics events

Admin analytics optional:

```text
admin_promotion_list_viewed
admin_promotion_search_used
admin_promotion_filter_applied
admin_promotion_created
admin_promotion_saved_draft
admin_promotion_published
admin_promotion_paused
admin_promotion_ended
admin_promotion_archived
admin_promotion_preview_used
admin_promotion_conflict_clicked
admin_promotion_duplicate_clicked
```

Storefront/customer events:

```text
coupon_apply_attempted
coupon_applied
coupon_rejected
promotion_viewed
flash_sale_product_clicked
order_created_with_promotion
```

Không gửi dữ liệu nhạy cảm không cần thiết.

---

# 35. Suggested component structure

```text
AdminPromotionListPage
AdminPromotionToolbar
AdminPromotionFilterDrawer
AdminPromotionTable
AdminPromotionRow
AdminPromotionMobileCard
PromotionBulkActionBar
AdminPromotionFormPage
PromotionBasicInfoSection
PromotionTypeSection
PromotionDiscountRuleSection
PromotionEligibilitySection
PromotionScopeBuilder
PromotionProductPicker
PromotionCategoryPicker
PromotionBrandPicker
PromotionUsageLimitSection
PromotionScheduleSection
PromotionDisplaySection
PromotionStackingSection
PromotionPreviewPanel
PromotionConflictPanel
PromotionPublishChecklist
PromotionErrorSummary
PromotionAuditLog
PromotionUsageTable
PromotionAnalyticsPanel
PromotionConfirmDialog
```

Shared components có thể tái dùng:

```text
DataTable
StatusBadge
FormSection
ConfirmDialog
Drawer
Modal
DateRangePicker
MoneyInput
PercentageInput
AsyncProductSelect
```

---

# 36. Playwright test specification

## 36.1. Promotion list tests

Test cases:

```text
Admin can view promotion list
Admin can search by promotion name
Admin can search by coupon code
Admin can filter by type
Admin can filter by status
Admin can open promotion detail
Admin can open create promotion page
Promotion list empty state appears
Filtered empty state appears
Mobile promotion list has no horizontal overflow
```

## 36.2. Coupon create tests

```text
Admin can create fixed amount coupon draft
Admin can create percentage coupon draft
Admin cannot publish coupon without code
Admin cannot publish duplicate coupon code
Admin cannot publish invalid schedule
Admin cannot publish percentage discount above 100
Admin sees warning when percentage discount has no max cap
Admin can preview coupon calculation
Admin can publish valid coupon
```

## 36.3. Automatic promotion tests

```text
Admin can create automatic product discount
Admin can select category scope
Admin can select brand scope
Admin can exclude product from scope
Eligible product count updates
Promotion cannot publish with zero eligible products
```

## 36.4. Flash sale tests

```text
Admin can create flash sale for selected variant
Flash sale price must be lower than current price
Low stock warning appears for flash sale item
Admin can set campaign stock limit
Admin can schedule flash sale
```

## 36.5. Status action tests

```text
Admin can pause active promotion with confirm modal
Admin can resume paused promotion
Admin can end active promotion with confirm modal
Admin can archive draft promotion
User without permission cannot see publish action
```

## 36.6. Conflict tests

```text
Blocking conflict prevents publish
Warning conflict allows publish only with confirm if policy allows
Conflict panel shows related promotion
Duplicate active coupon code shows validation error
Overlapping flash sale variant shows blocking conflict
```

## 36.7. Checkout coupon integration tests

```text
Customer can apply valid coupon in cart
Invalid coupon shows friendly message
Expired coupon is rejected
Coupon not eligible for product is rejected
Usage limit exceeded coupon is rejected
Order stores promotion snapshot after checkout
```

---

# 37. Visual regression checklist

Capture screenshots:

```text
Promotion list desktop
Promotion list mobile
Promotion empty state
Promotion filtered empty state
Promotion form desktop
Promotion form mobile
Coupon code section
Discount rule section
Scope builder
Product picker drawer
Preview calculation panel
Conflict panel
Publish validation error
Confirm end promotion modal
Promotion detail page
```

Viewports:

```text
1440px desktop
1024px laptop
768px tablet
375px mobile
320px small mobile
```

Must verify:

```text
No horizontal overflow
Form sections readable
Table/card status clear
Action buttons not hidden
Modal fits viewport
Preview numbers not cut
Long promotion names do not break layout
Coupon code long enough does not overflow
```

---

# 38. Definition of Done

Module Admin Promotion Management được coi là xong khi:

```text
Promotion list hoạt động
Search/filter/pagination hoạt động
Create/edit coupon hoạt động
Create/edit automatic promotion hoạt động ở MVP
Create/edit flash sale hoạt động ở MVP nếu nằm trong scope
Save draft hoạt động
Publish validation hoạt động
Conflict detection hiển thị rõ
Preview calculation hoạt động
Pause/resume/end/archive có confirm
Usage limit hiển thị và validate
Scope selector không load toàn bộ sản phẩm một lần
Permission UI hoạt động
Backend vẫn kiểm tra permission
Order/checkout discount tính lại ở backend
Order lưu promotion snapshot
Loading/empty/error states đầy đủ
Responsive không overflow
Accessibility cơ bản đạt
Playwright tests chính pass
Không có console error nghiêm trọng
```

---

# 39. MVP scope

Để làm nhanh, MVP chỉ cần:

```text
Promotion list
Search by name/code
Filter type/status
Create coupon fixed amount
Create coupon percentage with max discount
Create coupon free shipping
Create automatic product/category discount
Create flash sale simple by product/variant
Schedule start/end
Usage limit total/per customer
Scope by product/category/brand
Save draft
Publish
Pause/end/archive
Preview calculation basic
Conflict check cơ bản
Apply coupon at cart/checkout
Order promotion snapshot
```

Chưa cần ngay:

```text
Bundle complex
Free gift stock handling nâng cao
Customer segmentation nâng cao
Recurring promotion
Multi-channel promotion
Advanced campaign analytics
AI campaign suggestion
Dynamic pricing
A/B testing
Marketplace promotion sync
```

---

# 40. Future extensions

Sau MVP có thể mở rộng:

```text
Campaign calendar
Promotion analytics dashboard
Gross margin protection
AI promotion suggestion
Coupon batch generation
Private coupon per customer group
Affiliate/referral coupon
Bundle builder nâng cao
Gift promotion with inventory reservation
Campaign A/B testing
Promotion approval workflow
Promotion budget cap
Marketplace sync
POS promotion sync
```

---

# 41. Ghi chú cho source clone nhiều ngành

Promotion core phải dùng chung cho nhiều ngành.

Core không nên hard-code:

```text
Laptop
RAM
SSD
GPU
Warranty months
```

Core nên dựa vào:

```text
Product
Variant
Category
Brand
Attribute filter optional
Customer segment
Order subtotal
Shipping/payment method
```

Phần riêng ngành đồ điện tử nằm ở:

```text
Flash sale theo variant
Max discount cho sản phẩm giá trị cao
Low stock warning khi chạy campaign
Warranty/service copywriting
High value order caution
```

Khi clone sang ngành khác như thời trang/mỹ phẩm/nội thất, vẫn giữ promotion engine, chỉ đổi:

```text
Scope picker default
Badge wording
Campaign templates
Validation policy riêng ngành
```

---

# 42. Tóm tắt cho agent

Nếu agent chỉ nhớ một phần, hãy nhớ:

```text
Promotion là module ảnh hưởng trực tiếp đến tiền.
Frontend chỉ hiển thị, backend phải tính lại discount.
Coupon code, discount value, schedule, scope, usage limit đều phải validate chặt.
Percentage discount cho đồ điện tử nên có max discount.
Flash sale nên xét theo variant/SKU.
Promotion active không được sửa bừa nếu không có quyền.
Mọi thay đổi nhạy cảm phải audit.
Order phải lưu promotion snapshot.
UI admin phải rõ, ít màu, có preview, có conflict warning, có confirm cho action nguy hiểm.
Mobile không overflow.
Test phải bao phủ create/publish/apply coupon/conflict/permission.
```
