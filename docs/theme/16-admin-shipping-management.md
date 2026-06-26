# 16 — Admin Shipping Management Specification

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Module: Shipping / Fulfillment Management  
> Mục tiêu: Đặc tả đủ chi tiết để coding agent/frontend/backend agent có thể xây module quản lý vận chuyển từ đầu đến cuối.  
> Phụ thuộc: `../main/ecommerce_design_language.md`, `01-electronics-store-theme.md`, `09-admin-dashboard.md`, `12-admin-order-management.md`, `13-admin-inventory-management.md`, `../main/system-design.md`.  
> Nguyên tắc: Không phụ thuộc framework. Có thể map sang Nuxt, React, Vue, Laravel, FastAPI, Spring Boot hoặc bất kỳ stack nào.

---

# 0. Prompt cho coding agent

Dùng prompt sau khi giao task cho coding agent:

```text
Bạn là Senior Full-stack Engineer kiêm UX-aware Admin Panel Developer.

Nhiệm vụ của bạn là implement module Admin Shipping Management cho website bán hàng đồ điện tử.

Trước khi code, bắt buộc đọc các tài liệu:

1. ../main/ecommerce_design_language.md
2. 01-electronics-store-theme.md
3. 09-admin-dashboard.md
4. 12-admin-order-management.md
5. 13-admin-inventory-management.md
6. 16-admin-shipping-management.md

Mục tiêu module:

- Quản lý phương thức vận chuyển.
- Quản lý vùng giao hàng.
- Quản lý phí ship theo khu vực, trọng lượng, giá trị đơn, sản phẩm cồng kềnh.
- Quản lý COD nếu có.
- Quản lý hãng vận chuyển.
- Tạo và theo dõi shipment/waybill/tracking code.
- Đồng bộ trạng thái giao hàng với đơn hàng.
- Hỗ trợ đơn giao nội bộ, nhận tại cửa hàng, giao qua đối tác vận chuyển.
- Hỗ trợ hoàn hàng/đổi trả ở mức shipping flow.

Yêu cầu UI:

- Admin UI phải rõ, nhanh, ít màu, dễ thao tác.
- Dùng design token, không hard-code màu bừa bãi.
- Table phải có loading, empty, error state.
- Form phải chia section rõ ràng.
- Action nguy hiểm phải có confirm modal.
- Mobile không được overflow ngang.
- Có responsive strategy cho table.
- Có accessibility cơ bản: label, focus, aria-label cho icon-only button, modal focus trap.

Yêu cầu nghiệp vụ:

- Không tính phí ship ở frontend một cách tin cậy tuyệt đối. Backend phải tính lại.
- Shipping fee ở checkout là estimate cho đến khi đơn được tạo/xác nhận.
- Không cho tạo shipment nếu đơn chưa đủ điều kiện.
- Không cho ship item chưa được reserve/packed nếu hệ thống có inventory flow.
- Mọi thay đổi shipping status phải ghi timeline/audit log.
- Tracking code phải unique theo carrier nếu cần.
- COD amount phải khớp với số tiền cần thu.
- Nếu đơn bị hủy, shipment đang tạo/đang giao phải có flow xử lý rõ.

Sau khi code xong, báo cáo:

- Files changed.
- Components created/updated.
- APIs integrated/mocked.
- State handled.
- Tests added/updated.
- Tests run.
- Responsive checked.
- Known limitations.
```

---

# 1. Vai trò của Shipping Management trong hệ thống

Shipping Management là module quản lý toàn bộ phần **giao hàng sau khi đơn được tạo** và phần **cấu hình phí/phương thức vận chuyển trước khi khách checkout**.

Trong web bán hàng, shipping không chỉ là một dòng `phí ship`. Nó ảnh hưởng trực tiếp tới:

```text
Checkout
Order confirmation
Inventory reserve
Packing
Delivery
COD collection
Return / exchange
Customer support
Order timeline
Admin dashboard
Accounting / reconciliation
```

Với shop đồ điện tử, module shipping càng quan trọng vì:

```text
Sản phẩm có giá trị cao.
Có sản phẩm dễ vỡ.
Có sản phẩm cồng kềnh như màn hình, PC, máy in.
Có đơn hàng giá trị cao cần xác nhận kỹ.
Có thể có COD hoặc chuyển khoản.
Có thể cần đóng gói cẩn thận.
Có thể cần serial/IMEI sau khi giao.
Có thể có bảo hành/đổi trả liên quan đến vận chuyển.
```

Shipping Management phải giúp admin trả lời nhanh:

```text
Shop đang hỗ trợ phương thức giao hàng nào?
Khu vực nào giao được?
Phí ship tính thế nào?
Đơn này đã tạo vận đơn chưa?
Mã vận đơn là gì?
Đơn đang giao tới đâu?
Có đơn nào giao lỗi không?
Có COD nào cần thu/đối soát không?
Có kiện hàng nào cần hoàn về không?
```

---

# 2. Phạm vi module

Module này bao gồm cả **cấu hình vận chuyển** và **vận hành shipment**.

## 2.1. Cấu hình vận chuyển

Bao gồm:

```text
Shipping methods
Shipping zones
Shipping rate rules
Carrier management
Store pickup configuration
COD configuration
Free shipping rules
Bulky/fragile product rules
Delivery time estimate
```

## 2.2. Vận hành shipment

Bao gồm:

```text
Create shipment from order
Assign carrier
Generate tracking code / waybill
Print shipping label
Update shipping status
Track shipment
Handle failed delivery
Handle returned shipment
Sync shipping status to order
```

## 2.3. Không thuộc scope trực tiếp

Các phần sau có liên quan nhưng không phải trung tâm của file này:

```text
Payment gateway integration
Warehouse picking/packing nâng cao
Accounting reconciliation nâng cao
Carrier API production credential security chi tiết
Customer-facing order tracking page chi tiết
Warranty repair logistics nâng cao
```

Những phần này có thể được mở rộng trong các file riêng.

---

# 3. Khái niệm nghiệp vụ chính

## 3.1. Shipping Method

Shipping Method là cách giao hàng mà shop cho phép khách chọn.

Ví dụ:

```text
Giao hàng tiêu chuẩn
Giao hàng nhanh
Giao trong ngày
Nhận tại cửa hàng
Giao nội bộ
Giao qua đối tác vận chuyển
```

Mỗi shipping method có thể có:

```text
Tên hiển thị
Mô tả
Thời gian dự kiến
Điều kiện áp dụng
Phí mặc định
Carrier liên kết
COD enabled / disabled
Trạng thái active / inactive
```

## 3.2. Shipping Zone

Shipping Zone là khu vực địa lý áp dụng một rule vận chuyển.

Ví dụ:

```text
Nội thành Hà Nội
Ngoại thành Hà Nội
Toàn quốc
Miền Bắc
Miền Trung
Miền Nam
Khu vực không hỗ trợ giao hàng
```

Shipping zone có thể dựa trên:

```text
Country
Province / City
District
Ward
Postal code
Custom region
```

MVP có thể chỉ cần `province/city`.

## 3.3. Shipping Rate

Shipping Rate là rule tính phí vận chuyển.

Có thể tính theo:

```text
Phí cố định
Theo tỉnh/thành
Theo trọng lượng
Theo giá trị đơn hàng
Theo số lượng item
Theo loại sản phẩm
Theo carrier
Theo shipping method
Miễn phí nếu đạt điều kiện
```

## 3.4. Carrier

Carrier là đơn vị vận chuyển.

Ví dụ:

```text
Giao hàng nội bộ
GHN
GHTK
Viettel Post
J&T
GrabExpress
Ahamove
Nhận tại cửa hàng
```

MVP không cần tích hợp API thật. Có thể quản lý carrier dạng manual trước.

## 3.5. Shipment

Shipment là kiện/gói giao hàng được tạo từ một đơn hàng.

Một order có thể có:

```text
1 shipment
nhiều shipment
không shipment nếu nhận tại cửa hàng
```

Với MVP, có thể giả định mỗi order chỉ có một shipment.

## 3.6. Waybill / Tracking Code

Tracking code là mã vận đơn dùng để theo dõi shipment.

```text
Tracking code phải hiển thị trong order detail.
Có thể nhập thủ công hoặc lấy từ carrier API.
```

## 3.7. Fulfillment Status và Shipping Status

Cần phân biệt:

```text
Fulfillment status = shop đã xử lý hàng đến đâu
Shipping status    = carrier đang giao đến đâu
```

Ví dụ:

```text
Fulfillment: packing
Shipping: not_shipped

Fulfillment: shipped
Shipping: in_transit

Fulfillment: delivered
Shipping: delivered
```

Không nên dùng một field duy nhất cho mọi thứ nếu hệ thống cần mở rộng.

---

# 4. Route admin đề xuất

```text
/admin/shipping
/admin/shipping/methods
/admin/shipping/methods/new
/admin/shipping/methods/:id/edit
/admin/shipping/zones
/admin/shipping/rates
/admin/shipping/carriers
/admin/shipping/carriers/:id/edit
/admin/shipping/shipments
/admin/shipping/shipments/:id
/admin/shipping/pickups
/admin/shipping/returns
/admin/shipping/settings
```

## 4.1. MVP route

MVP chỉ cần:

```text
/admin/shipping
/admin/shipping/methods
/admin/shipping/zones
/admin/shipping/rates
/admin/shipping/carriers
/admin/shipping/shipments
```

---

# 5. Admin Shipping Overview Page

Route:

```text
/admin/shipping
```

## 5.1. Mục đích

Trang overview giúp admin nhìn nhanh tình trạng vận chuyển.

Cần trả lời:

```text
Có bao nhiêu đơn chưa tạo vận đơn?
Có bao nhiêu đơn đang giao?
Có bao nhiêu đơn giao thất bại?
Có bao nhiêu đơn cần hoàn về?
Carrier nào đang có nhiều lỗi?
Phương thức nào đang active?
```

## 5.2. Layout desktop

```text
Admin Shell
└── Shipping Overview
    ├── Page header
    │   ├── Title: Shipping Management
    │   ├── Subtitle
    │   ├── Date range
    │   └── Quick actions
    ├── KPI cards
    ├── Alerts
    ├── Shipments requiring action
    ├── Carrier performance summary
    └── Recent shipment events
```

## 5.3. KPI cards

Các KPI gợi ý:

```text
Awaiting shipment
In transit
Delivered today
Delivery failed
Return pending
COD to collect
```

MVP chỉ cần:

```text
Awaiting shipment
In transit
Delivery failed
Return pending
```

## 5.4. Alerts

Alert examples:

```text
12 orders are ready but have no shipment.
3 shipments failed delivery today.
5 COD shipments need reconciliation.
Carrier GHN API is currently unavailable.
```

## 5.5. Quick actions

```text
Create shipping method
Create rate rule
View shipments
Import tracking update
```

---

# 6. Shipping Method Management

Route:

```text
/admin/shipping/methods
```

## 6.1. Mục đích

Quản lý các phương thức vận chuyển mà khách có thể chọn ở checkout.

Ví dụ:

```text
Giao hàng tiêu chuẩn
Giao hàng nhanh
Nhận tại cửa hàng
Giao nội bộ Hà Nội
```

## 6.2. Shipping method list

Cột table:

| Column | Nội dung |
|---|---|
| Method | Tên + mô tả |
| Type | standard / express / pickup / internal |
| Carrier | Carrier mặc định |
| Zones | Số vùng áp dụng |
| COD | enabled / disabled |
| Status | active / inactive |
| Updated | thời gian cập nhật |
| Actions | edit / duplicate / disable |

## 6.3. Shipping method form

Fields:

```text
Method name
Method code
Description
Type
Default carrier
Estimated delivery min/max
COD enabled
Active status
Display order
Customer-facing label
Customer-facing description
```

Ví dụ:

```text
Name: Giao hàng tiêu chuẩn
Code: standard_delivery
Description: Giao hàng toàn quốc trong 2-5 ngày làm việc
Estimated: 2-5 days
COD: enabled
Status: active
```

## 6.4. Method types

```text
standard
express
same_day
store_pickup
internal_delivery
carrier_delivery
```

## 6.5. Display rule tại checkout

Shipping method chỉ hiển thị nếu:

```text
Method active.
Customer address thuộc zone được hỗ trợ.
Cart/order thỏa điều kiện weight/value/product.
Payment method tương thích nếu có COD rule.
Carrier khả dụng nếu method phụ thuộc carrier.
```

## 6.6. Validation

```text
Name required.
Code required and unique.
Estimated min <= estimated max.
Pickup method phải có pickup location.
Carrier delivery method nên có carrier nếu không phải manual.
Inactive method không hiển thị ở checkout.
```

---

# 7. Shipping Zone Management

Route:

```text
/admin/shipping/zones
```

## 7.1. Mục đích

Shipping zone giúp cấu hình khu vực giao hàng và phí ship theo vùng.

Ví dụ:

```text
Nội thành Hà Nội
TP Hồ Chí Minh
Tỉnh thành khác
Không hỗ trợ giao hàng
```

## 7.2. Zone list

Cột table:

| Column | Nội dung |
|---|---|
| Zone | tên zone |
| Coverage | tỉnh/quận/phường |
| Methods | phương thức áp dụng |
| Rate rules | số rule phí |
| Status | active / inactive |
| Actions | edit / duplicate / disable |

## 7.3. Zone form

Fields:

```text
Zone name
Zone code
Description
Country
Province / City selection
District selection optional
Ward selection optional
Postal code optional
Status
Priority
```

## 7.4. Zone priority

Một địa chỉ có thể match nhiều zone.

Ví dụ:

```text
Zone A: Toàn quốc
Zone B: Hà Nội
Zone C: Nội thành Hà Nội
```

Rule:

```text
Zone cụ thể hơn được ưu tiên.
Nếu cùng độ cụ thể, priority cao hơn được ưu tiên.
Nếu conflict, admin cần được cảnh báo.
```

## 7.5. Unsupported zone

Có thể cấu hình vùng không giao hàng.

Ví dụ:

```text
Một số khu vực đảo xa
Khu vực carrier không hỗ trợ
```

Checkout phải báo rõ:

```text
Khu vực này hiện chưa hỗ trợ giao hàng. Vui lòng chọn địa chỉ khác hoặc liên hệ shop.
```

---

# 8. Shipping Rate Management

Route:

```text
/admin/shipping/rates
```

## 8.1. Mục đích

Quản lý rule tính phí vận chuyển.

Rate rule cần đủ linh hoạt nhưng không quá phức tạp ở MVP.

## 8.2. Rate rule types

```text
flat_rate
free_shipping
weight_based
order_value_based
quantity_based
carrier_rate
manual_quote
```

## 8.3. MVP rate rule

MVP nên hỗ trợ:

```text
Flat rate theo zone + method.
Free shipping theo order total.
Extra fee theo bulky/fragile product.
```

## 8.4. Rate list table

| Column | Nội dung |
|---|---|
| Rule | tên rule |
| Method | phương thức vận chuyển |
| Zone | vùng áp dụng |
| Type | flat/free/weight/value |
| Fee | phí |
| Conditions | điều kiện |
| Status | active/inactive |
| Actions | edit/duplicate/disable |

## 8.5. Rate rule form

Fields:

```text
Rule name
Shipping method
Shipping zone
Rate type
Base fee
Currency
Min order value
Max order value
Min weight
Max weight
Free shipping threshold
Bulky surcharge
Fragile surcharge
COD surcharge
Active status
Priority
```

## 8.6. Flat rate example

```text
Rule: Standard Hà Nội
Method: Giao hàng tiêu chuẩn
Zone: Hà Nội
Type: flat_rate
Base fee: 30.000đ
```

## 8.7. Free shipping example

```text
Rule: Free shipping over 5M
Method: Giao hàng tiêu chuẩn
Zone: Toàn quốc
Type: free_shipping
Condition: order_total >= 5.000.000đ
Fee: 0đ
```

## 8.8. Weight-based example

```text
0kg - 2kg: 30.000đ
2kg - 5kg: 50.000đ
5kg - 10kg: 90.000đ
```

## 8.9. Electronics-specific surcharge

Với đồ điện tử, cần tính thêm phụ phí nếu:

```text
Product fragile = true
Product bulky = true
Product category = Monitor / Printer / Desktop PC
Order value high and requires insurance
```

Ví dụ:

```text
Màn hình 32 inch: +50.000đ packaging surcharge
Desktop PC: +80.000đ bulky surcharge
Order >= 20.000.000đ: recommend insured shipping
```

MVP có thể chỉ hiển thị warning, chưa cần tính tự động.

## 8.10. Rate calculation order

Thứ tự gợi ý:

```text
1. Determine customer shipping address.
2. Match active shipping zones.
3. Select applicable shipping method.
4. Find active rate rules.
5. Apply most specific rule.
6. Apply surcharge if needed.
7. Apply free shipping if eligible.
8. Return shipping options with fee and ETA.
```

## 8.11. Validation

```text
Rule name required.
Shipping method required.
Zone required.
Fee >= 0.
Min value <= max value.
Min weight <= max weight.
Free shipping threshold >= 0.
Priority must be numeric.
Cannot create exact duplicate active rule.
```

---

# 9. Carrier Management

Route:

```text
/admin/shipping/carriers
```

## 9.1. Mục đích

Quản lý danh sách đối tác vận chuyển.

MVP có thể chưa tích hợp API thật, nhưng vẫn cần cấu hình carrier để admin chọn khi tạo vận đơn.

## 9.2. Carrier list

Cột table:

| Column | Nội dung |
|---|---|
| Carrier | tên + logo optional |
| Code | mã carrier |
| Type | manual / api / internal |
| COD | supported / not supported |
| Tracking | manual / url / api |
| Status | active / inactive |
| Actions | edit / disable / test connection |

## 9.3. Carrier form

Fields:

```text
Carrier name
Carrier code
Carrier type
Support COD
Support pickup
Support return
Tracking URL template
API enabled
API environment
Credential status
Status
Customer support phone
Note
```

## 9.4. Tracking URL template

Ví dụ:

```text
https://carrier.example/track?code={tracking_code}
```

Rule:

```text
Nếu tracking URL template tồn tại, order detail có thể mở link tracking.
Nếu không có, chỉ hiển thị tracking code.
```

## 9.5. API credential rule

Nếu carrier có API:

```text
Không hiển thị secret key đầy đủ trên UI.
Có nút Test connection.
Có trạng thái connected / failed / not configured.
Chỉ role có quyền mới sửa credential.
Audit mọi thay đổi credential.
```

---

# 10. Shipment Management

Route:

```text
/admin/shipping/shipments
```

## 10.1. Mục đích

Quản lý các kiện hàng/vận đơn đã tạo hoặc cần tạo.

## 10.2. Shipment list filters

Filters:

```text
Search by order number
Search by tracking code
Carrier
Shipping method
Shipping status
Payment method
COD status
Created date
Delivery date
Failed delivery
Return status
```

## 10.3. Shipment table

Cột table:

| Column | Nội dung |
|---|---|
| Shipment | shipment number / tracking code |
| Order | order number |
| Customer | tên + phone masked |
| Carrier | carrier |
| Method | shipping method |
| COD | amount / status |
| Shipping status | badge |
| Created | ngày tạo |
| Actions | view / update / print |

## 10.4. Shipment status

Các status đề xuất:

```text
not_created
created
label_printed
ready_for_pickup
picked_up
in_transit
out_for_delivery
delivered
delivery_failed
returning
returned
cancelled
lost
damaged
```

## 10.5. Status tone

| Status | Tone |
|---|---|
| not_created | neutral |
| created | info |
| label_printed | info |
| ready_for_pickup | warning |
| picked_up | info |
| in_transit | primary |
| out_for_delivery | primary |
| delivered | success |
| delivery_failed | danger |
| returning | warning |
| returned | warning |
| cancelled | neutral/danger |
| lost | danger |
| damaged | danger |

## 10.6. Shipment detail

Route:

```text
/admin/shipping/shipments/:id
```

Sections:

```text
Shipment summary
Order summary
Customer / recipient
Shipping address
Carrier information
Package information
COD information
Tracking timeline
Items in shipment
Internal notes
Audit log
Actions
```

---

# 11. Create Shipment Flow

Shipment có thể tạo từ:

```text
Order detail page
Shipment list page
Dashboard orders requiring action
Bulk action từ order list
```

## 11.1. Điều kiện tạo shipment

Chỉ cho tạo shipment nếu:

```text
Order exists.
Order is not cancelled.
Order has shippable items.
Shipping address valid.
Payment condition satisfied if required.
Inventory reserved or packing allowed.
No active shipment already exists nếu MVP one-shipment-per-order.
```

## 11.2. Create shipment form

Fields:

```text
Order number readonly
Recipient name
Recipient phone
Shipping address
Shipping method
Carrier
Package weight
Package dimensions
Fragile flag
Bulky flag
COD enabled
COD amount
Insurance value optional
Pickup date optional
Note for carrier
Internal note
```

## 11.3. Package fields

```text
Weight
Length
Width
Height
Package count
Package type
```

MVP có thể dùng:

```text
Weight
Package count
Note
```

## 11.4. COD fields

Nếu payment method là COD:

```text
COD amount = order total amount to collect
COD status = pending_collection
```

Không cho admin tự sửa COD amount nếu không có quyền.

Nếu payment đã paid:

```text
COD disabled hoặc COD amount = 0
```

## 11.5. Create shipment result

Sau khi tạo thành công:

```text
Shipment status = created
Order fulfillment status = ready_to_ship hoặc shipped tùy flow
Tracking code generated/entered
Timeline event created
Toast success
```

---

# 12. Shipping Label / Waybill

## 12.1. Mục đích

Shipping label dùng để dán lên kiện hàng hoặc in phiếu giao.

## 12.2. Label content

Nội dung tối thiểu:

```text
Shop name
Order number
Shipment number
Tracking code
Carrier
Recipient name
Recipient phone masked/full tùy permission
Shipping address
COD amount nếu có
Package count
Product summary
Barcode/QR optional
```

## 12.3. Label actions

```text
Print label
Download PDF
Regenerate label
Mark label printed
```

## 12.4. Security note

Label có thông tin cá nhân. Chỉ role có quyền mới xem/in.

---

# 13. Pickup Management

Route optional:

```text
/admin/shipping/pickups
```

## 13.1. Mục đích

Quản lý việc carrier tới lấy hàng.

## 13.2. Pickup statuses

```text
scheduled
ready
picked_up
missed
cancelled
```

## 13.3. Pickup fields

```text
Carrier
Pickup date/time
Pickup address
Shipment count
Contact person
Contact phone
Note
Status
```

MVP có thể bỏ pickup management và dùng status `ready_for_pickup` trong shipment.

---

# 14. Delivery Tracking

## 14.1. Tracking source

Tracking có thể cập nhật từ:

```text
Manual admin update
Carrier API polling
Carrier webhook
Import CSV
```

MVP nên hỗ trợ manual update.

## 14.2. Tracking timeline item

Fields:

```text
Status
Message
Location optional
Occurred at
Source
Raw carrier status optional
Created by/system
```

## 14.3. Manual update flow

```text
Open shipment detail
Click Update status
Select new status
Enter note optional/required tùy status
Confirm
Create timeline event
Update order shipping status if needed
Show toast
```

## 14.4. Status mapping từ carrier

Khi tích hợp carrier API, cần mapping:

```text
Carrier raw status -> Internal shipping status
```

Ví dụ:

```text
carrier_status: "delivering" -> internal: out_for_delivery
carrier_status: "success" -> internal: delivered
carrier_status: "fail" -> internal: delivery_failed
```

Không để frontend phụ thuộc raw carrier status.

---

# 15. Failed Delivery Flow

## 15.1. Các lý do giao thất bại

```text
Không liên hệ được khách
Sai địa chỉ
Khách hẹn giao lại
Khách từ chối nhận
Hàng hư hỏng
Carrier delay
COD issue
Khu vực không hỗ trợ
```

## 15.2. Failed delivery form

Fields:

```text
Failure reason
Failure note
Next action
Reschedule date optional
Customer contacted yes/no
Internal owner
```

## 15.3. Next actions

```text
Retry delivery
Hold shipment
Return to sender
Cancel order
Contact customer
```

## 15.4. Rule

Nếu delivery failed:

```text
Order detail phải hiển thị warning.
Dashboard/shipping overview phải có alert.
Support staff cần thấy đơn cần liên hệ.
Không tự hủy order nếu chưa có rule rõ.
```

---

# 16. Return Shipment Flow

Return shipment khác với warranty/service nhưng có liên quan.

## 16.1. Return cases

```text
Khách từ chối nhận hàng
Giao thất bại nhiều lần
Đổi trả sau khi nhận
Bảo hành cần gửi hàng về shop
Hàng hư hỏng cần thu hồi
```

## 16.2. Return shipment statuses

```text
return_requested
return_approved
return_created
return_in_transit
returned_to_store
return_cancelled
```

## 16.3. Return fields

```text
Original shipment
Return tracking code
Return carrier
Return reason
Return fee payer
Return status
Received by
Received at
Condition note
```

## 16.4. Inventory relation

Khi hàng hoàn về:

```text
Không tự cộng tồn kho nếu chưa kiểm tra hàng.
Cần inspection nếu sản phẩm điện tử có rủi ro hư hỏng.
Nếu condition = sellable thì mới restock.
Nếu condition = damaged thì đưa vào service/warranty/defective stock.
```

---

# 17. Store Pickup

## 17.1. Mục đích

Cho khách đặt online và nhận tại cửa hàng.

## 17.2. Pickup method fields

```text
Store location
Pickup available hours
Pickup preparation time
Contact phone
Instruction
```

## 17.3. Pickup order statuses

```text
pickup_pending
pickup_ready
picked_up
pickup_expired
pickup_cancelled
```

## 17.4. Rule

Nếu khách chọn nhận tại cửa hàng:

```text
Shipping fee = 0 hoặc theo cấu hình.
Không cần carrier.
Cần pickup instruction trong order success.
Admin cần action Mark ready for pickup.
Admin cần action Mark picked up.
```

---

# 18. COD Management

## 18.1. COD trong shipping

COD là thu tiền khi giao hàng. Nó liên quan cả payment và shipment.

## 18.2. COD statuses

```text
not_applicable
pending_collection
collected
collection_failed
remitted
reconciled
```

## 18.3. COD display in shipment list

Hiển thị:

```text
COD amount
COD status
Carrier remittance status nếu có
```

## 18.4. COD rules

```text
COD amount phải bằng order amount cần thu.
Nếu order đã paid online, COD không áp dụng.
Nếu payment method = COD, shipment cần hiển thị amount to collect.
Nếu delivery failed, COD vẫn pending hoặc collection_failed.
Nếu delivered, COD có thể collected nhưng chưa reconciled.
```

## 18.5. COD reconciliation future scope

MVP chưa cần đối soát COD chi tiết, nhưng data model nên để mở rộng.

---

# 19. Order Integration

Shipping module phải liên kết chặt với Order Management.

## 19.1. Order detail cần hiển thị

```text
Shipping method
Carrier
Tracking code
Shipping status
Shipment timeline
Shipping address
COD status
Print label action
Create shipment action
```

## 19.2. Order status relation

Ví dụ mapping:

```text
Order status: confirmed
Fulfillment status: packing
Shipping status: not_shipped

Order status: confirmed
Fulfillment status: shipped
Shipping status: in_transit

Order status: completed
Fulfillment status: delivered
Shipping status: delivered
```

## 19.3. Action dependency

Không nên cho:

```text
Mark order delivered khi shipment chưa delivered, trừ role có override quyền.
Cancel order khi shipment đang delivered.
Create shipment khi order cancelled.
```

## 19.4. Timeline

Mỗi shipping event phải xuất hiện trong order timeline.

Ví dụ:

```text
22/06/2026 10:00 — Shipment created by Admin.
22/06/2026 11:30 — Label printed.
22/06/2026 14:00 — Carrier picked up package.
23/06/2026 09:20 — Out for delivery.
23/06/2026 15:40 — Delivered successfully.
```

---

# 20. Checkout Integration

Checkout cần lấy shipping options từ backend.

## 20.1. Checkout shipping options API

Checkout gửi:

```text
Cart items
Shipping address
Payment method optional
Coupon optional
```

Backend trả:

```text
Available shipping methods
Shipping fee
ETA
Unavailable methods with reason optional
```

## 20.2. Không tính phí ship chỉ ở frontend

Frontend có thể preview, nhưng backend phải tính lại khi place order.

Rule:

```text
Checkout summary fee chỉ là kết quả từ backend.
Place order phải revalidate shipping option.
Nếu phí ship thay đổi, báo khách xác nhận lại.
```

## 20.3. Unavailable shipping method

Ví dụ message:

```text
Giao hàng nhanh chưa hỗ trợ khu vực này.
COD không áp dụng cho đơn hàng trên 20.000.000đ.
Sản phẩm cồng kềnh không hỗ trợ giao trong ngày.
```

---

# 21. Inventory Integration

Shipping liên quan tới inventory.

## 21.1. Reserve before shipping

Trước khi tạo shipment:

```text
Order items cần được reserve hoặc confirmed available.
```

## 21.2. Packing relation

Nếu hệ thống có packing status:

```text
confirmed -> packing -> ready_to_ship -> shipped
```

Shipping chỉ bắt đầu khi:

```text
Items packed hoặc admin có quyền override.
```

## 21.3. Return relation

Khi return về kho:

```text
Không auto release/restock nếu cần kiểm tra hàng.
Stock movement phải ghi rõ reason.
```

---

# 22. Data contract

## 22.1. ShippingMethod

```json
{
  "id": "ship_method_standard",
  "code": "standard_delivery",
  "name": "Giao hàng tiêu chuẩn",
  "description": "Giao hàng toàn quốc trong 2-5 ngày làm việc.",
  "type": "standard",
  "defaultCarrierId": "carrier_manual",
  "estimatedMinDays": 2,
  "estimatedMaxDays": 5,
  "codEnabled": true,
  "status": "active",
  "displayOrder": 10,
  "createdAt": "2026-06-26T00:00:00+07:00",
  "updatedAt": "2026-06-26T00:00:00+07:00"
}
```

## 22.2. ShippingZone

```json
{
  "id": "zone_hanoi_inner",
  "code": "hanoi_inner",
  "name": "Nội thành Hà Nội",
  "countries": ["VN"],
  "provinces": ["Hà Nội"],
  "districts": ["Ba Đình", "Hoàn Kiếm", "Đống Đa", "Cầu Giấy"],
  "wards": [],
  "priority": 100,
  "status": "active"
}
```

## 22.3. ShippingRateRule

```json
{
  "id": "rate_standard_hanoi",
  "name": "Standard Hà Nội",
  "shippingMethodId": "ship_method_standard",
  "shippingZoneId": "zone_hanoi_inner",
  "type": "flat_rate",
  "baseFee": 30000,
  "currency": "VND",
  "conditions": {
    "minOrderValue": 0,
    "maxOrderValue": null,
    "minWeightGram": 0,
    "maxWeightGram": 5000
  },
  "surcharges": {
    "fragileFee": 0,
    "bulkyFee": 50000,
    "codFee": 0
  },
  "priority": 100,
  "status": "active"
}
```

## 22.4. Carrier

```json
{
  "id": "carrier_ghtk",
  "code": "ghtk",
  "name": "Giao Hàng Tiết Kiệm",
  "type": "manual",
  "codSupported": true,
  "pickupSupported": true,
  "returnSupported": true,
  "trackingUrlTemplate": "https://carrier.example/track/{trackingCode}",
  "apiEnabled": false,
  "credentialStatus": "not_configured",
  "status": "active"
}
```

## 22.5. Shipment

```json
{
  "id": "shipment_1001",
  "shipmentNumber": "SHP1001",
  "orderId": "order_1001",
  "orderNumber": "DH1001",
  "shippingMethodId": "ship_method_standard",
  "carrierId": "carrier_ghtk",
  "trackingCode": "GHTK123456789",
  "status": "in_transit",
  "recipient": {
    "name": "Nguyễn Văn A",
    "phoneMasked": "090****000",
    "phone": "0900000000"
  },
  "address": {
    "line1": "Số 1, phố ABC",
    "ward": "Dịch Vọng",
    "district": "Cầu Giấy",
    "city": "Hà Nội",
    "country": "VN"
  },
  "package": {
    "weightGram": 2500,
    "lengthCm": 40,
    "widthCm": 30,
    "heightCm": 15,
    "packageCount": 1,
    "fragile": true,
    "bulky": false
  },
  "cod": {
    "enabled": true,
    "amount": 15990000,
    "currency": "VND",
    "status": "pending_collection"
  },
  "fee": {
    "shippingFee": 30000,
    "surcharge": 0,
    "currency": "VND"
  },
  "createdAt": "2026-06-26T10:00:00+07:00",
  "updatedAt": "2026-06-26T10:00:00+07:00"
}
```

## 22.6. ShipmentTrackingEvent

```json
{
  "id": "track_001",
  "shipmentId": "shipment_1001",
  "status": "out_for_delivery",
  "message": "Đơn hàng đang được giao tới khách.",
  "location": "Hà Nội",
  "occurredAt": "2026-06-26T14:30:00+07:00",
  "source": "manual",
  "createdBy": "admin_001"
}
```

---

# 23. API contract

API chỉ là gợi ý, có thể đổi theo backend framework.

## 23.1. Shipping methods

```http
GET    /api/v1/admin/shipping/methods
POST   /api/v1/admin/shipping/methods
GET    /api/v1/admin/shipping/methods/{id}
PATCH  /api/v1/admin/shipping/methods/{id}
POST   /api/v1/admin/shipping/methods/{id}/disable
POST   /api/v1/admin/shipping/methods/{id}/enable
```

## 23.2. Shipping zones

```http
GET    /api/v1/admin/shipping/zones
POST   /api/v1/admin/shipping/zones
GET    /api/v1/admin/shipping/zones/{id}
PATCH  /api/v1/admin/shipping/zones/{id}
POST   /api/v1/admin/shipping/zones/{id}/disable
POST   /api/v1/admin/shipping/zones/{id}/enable
```

## 23.3. Shipping rates

```http
GET    /api/v1/admin/shipping/rates
POST   /api/v1/admin/shipping/rates
GET    /api/v1/admin/shipping/rates/{id}
PATCH  /api/v1/admin/shipping/rates/{id}
POST   /api/v1/admin/shipping/rates/{id}/disable
POST   /api/v1/admin/shipping/rates/{id}/enable
POST   /api/v1/admin/shipping/rates/preview
```

## 23.4. Carriers

```http
GET    /api/v1/admin/shipping/carriers
POST   /api/v1/admin/shipping/carriers
GET    /api/v1/admin/shipping/carriers/{id}
PATCH  /api/v1/admin/shipping/carriers/{id}
POST   /api/v1/admin/shipping/carriers/{id}/test-connection
```

## 23.5. Shipments

```http
GET    /api/v1/admin/shipments
POST   /api/v1/admin/shipments
GET    /api/v1/admin/shipments/{id}
PATCH  /api/v1/admin/shipments/{id}
POST   /api/v1/admin/shipments/{id}/print-label
POST   /api/v1/admin/shipments/{id}/cancel
POST   /api/v1/admin/shipments/{id}/update-status
POST   /api/v1/admin/shipments/{id}/create-return
```

## 23.6. Order shipping actions

```http
POST   /api/v1/admin/orders/{orderId}/shipments
GET    /api/v1/admin/orders/{orderId}/shipments
POST   /api/v1/admin/orders/{orderId}/mark-ready-to-ship
```

## 23.7. Checkout shipping options

```http
POST   /api/v1/checkout/shipping-options
```

Request:

```json
{
  "cartId": "cart_001",
  "address": {
    "city": "Hà Nội",
    "district": "Cầu Giấy",
    "ward": "Dịch Vọng"
  },
  "paymentMethod": "cod"
}
```

Response:

```json
{
  "options": [
    {
      "shippingMethodId": "ship_method_standard",
      "name": "Giao hàng tiêu chuẩn",
      "fee": 30000,
      "currency": "VND",
      "estimatedMinDays": 2,
      "estimatedMaxDays": 5,
      "available": true
    }
  ],
  "unavailableOptions": [
    {
      "shippingMethodId": "ship_method_same_day",
      "name": "Giao trong ngày",
      "available": false,
      "reason": "Sản phẩm cồng kềnh không hỗ trợ giao trong ngày."
    }
  ]
}
```

---

# 24. Validation rules

## 24.1. Shipping method validation

```text
Name required.
Code required and unique.
Estimated min/max valid.
Status must be active/inactive.
Store pickup method requires pickup location.
```

## 24.2. Shipping zone validation

```text
Zone name required.
Zone coverage cannot be empty.
Priority must be numeric.
Warn if zone overlaps another active zone.
```

## 24.3. Rate rule validation

```text
Fee >= 0.
Currency required.
Method required.
Zone required.
Min condition <= max condition.
Cannot duplicate same method + zone + condition active rule.
```

## 24.4. Shipment validation

```text
Order required.
Order must not be cancelled.
Carrier required unless store pickup/internal pickup.
Shipping address required for delivery.
Recipient phone required.
Package weight >= 0.
COD amount >= 0.
Tracking code unique if provided.
Cannot create duplicate active shipment for same order in MVP.
```

## 24.5. Status transition validation

Không phải status nào cũng chuyển được sang mọi status.

Ví dụ allowed transition:

```text
created -> label_printed
created -> cancelled
label_printed -> ready_for_pickup
ready_for_pickup -> picked_up
picked_up -> in_transit
in_transit -> out_for_delivery
out_for_delivery -> delivered
out_for_delivery -> delivery_failed
delivery_failed -> out_for_delivery
delivery_failed -> returning
returning -> returned
```

Không nên cho:

```text
delivered -> created
cancelled -> in_transit
returned -> delivered
```

Trừ khi có role override và audit rõ.

---

# 25. Permission matrix

Roles gợi ý:

```text
Super Admin
Store Manager
Shipping Manager
Warehouse Staff
Sales Staff
Support Staff
Viewer
```

| Permission | Super | Store Manager | Shipping Manager | Warehouse | Sales | Support | Viewer |
|---|---|---|---|---|---|---|---|
| shipping.view | yes | yes | yes | yes | yes | yes | yes |
| shipping.method.manage | yes | yes | no | no | no | no | no |
| shipping.rate.manage | yes | yes | no | no | no | no | no |
| carrier.manage | yes | yes | no | no | no | no | no |
| shipment.create | yes | yes | yes | yes | no | no | no |
| shipment.update | yes | yes | yes | yes | no | yes | no |
| shipment.cancel | yes | yes | yes | no | no | no | no |
| shipment.print_label | yes | yes | yes | yes | no | no | no |
| shipment.view_sensitive | yes | yes | yes | yes | no | limited | no |
| cod.view | yes | yes | yes | no | no | no | no |
| cod.reconcile | yes | yes | no | no | no | no | no |

Rule:

```text
Frontend ẩn action nếu user không có quyền.
Backend vẫn phải validate permission.
Không hiển thị full phone/address nếu không có quyền.
```

---

# 26. Security and privacy

Shipping chứa nhiều dữ liệu cá nhân:

```text
Tên người nhận
Số điện thoại
Địa chỉ
Tracking code
COD amount
```

## 26.1. Privacy rules

```text
Mask phone ở list nếu không cần full.
Không hiển thị full address trên dashboard/widget.
Chỉ detail page mới hiển thị full address cho role có quyền.
Label/waybill có dữ liệu nhạy cảm, cần permission.
Không log full address vào analytics.
```

## 26.2. Credential security

Với carrier API:

```text
Không trả secret key về frontend.
Không hiển thị secret đầy đủ.
Credential update phải audit.
Credential test endpoint chạy backend.
```

## 26.3. Dangerous action

Cần confirm modal:

```text
Cancel shipment
Mark delivered manually
Mark returned
Change COD amount
Disable active shipping method
Disable active rate rule
Delete carrier credential
```

Không dùng `window.confirm` nếu app có design system modal.

---

# 27. Audit log

Mọi thay đổi quan trọng phải ghi audit.

Events:

```text
shipping_method_created
shipping_method_updated
shipping_method_disabled
shipping_zone_created
shipping_zone_updated
shipping_rate_created
shipping_rate_updated
carrier_created
carrier_updated
carrier_credential_updated
shipment_created
shipment_status_updated
shipment_cancelled
shipment_label_printed
shipment_return_created
cod_status_updated
```

Audit fields:

```text
Actor
Action
Entity type
Entity id
Before value
After value
Timestamp
IP/device optional
Reason/note optional
```

---

# 28. Loading / Empty / Error states

## 28.1. Loading

```text
Shipping overview: KPI skeleton + table skeleton.
Method list: table skeleton.
Rate rule list: table skeleton.
Shipment detail: section skeleton.
Create shipment: button loading.
Print label: button loading.
```

## 28.2. Empty states

### No shipping methods

```text
No shipping methods yet.
Create a shipping method so customers can choose delivery at checkout.
[Create shipping method]
```

### No rate rules

```text
No shipping rate rules yet.
Create a rate rule to calculate delivery fees.
[Create rate rule]
```

### No shipments

```text
No shipments found.
Shipments will appear here after orders are ready to ship.
```

### Filtered empty

```text
No shipments match your filters.
Try changing status, carrier, or date range.
[Clear filters]
```

## 28.3. Error states

```text
Cannot load shipping methods.
Cannot save shipping rate.
Cannot create shipment.
Cannot print label.
Cannot update shipping status.
Carrier connection failed.
```

Error phải có:

```text
Thông báo dễ hiểu.
Retry nếu phù hợp.
Không mất dữ liệu form.
```

---

# 29. Responsive rules

## 29.1. Desktop

```text
Sidebar visible.
Tables full width.
Filters inline.
Shipping detail có 2 columns nếu phù hợp.
```

## 29.2. Tablet

```text
Sidebar collapsed.
Tables có horizontal scroll trong container.
Filters wrap hoặc chuyển drawer.
```

## 29.3. Mobile

```text
Sidebar drawer.
Shipment list chuyển thành card.
Filters mở drawer.
Forms single column.
Action buttons sticky bottom nếu form dài.
Không overflow ngang toàn page.
```

## 29.4. Mobile shipment card

Card nên hiển thị:

```text
Tracking code
Order number
Customer masked
Carrier
Status badge
COD amount nếu có
Created date
Primary action
```

---

# 30. Accessibility rules

```text
Tất cả input có label.
Select carrier/method/zone có accessible name.
Table header semantic đúng.
Badge không chỉ dựa vào màu, phải có text.
Icon-only button có aria-label.
Modal confirm trap focus.
Drawer filter trap focus.
Keyboard dùng được cho action menu.
Error message liên kết với field.
Focus visible rõ.
```

Shipping label print/download button phải có label rõ:

```text
Print shipping label for order DH1001
```

---

# 31. Component structure đề xuất

```text
AdminShippingOverviewPage
ShippingKpiCard
ShippingAlertCenter
ShippingMethodListPage
ShippingMethodFormPage
ShippingZoneListPage
ShippingZoneFormPage
ShippingRateListPage
ShippingRateFormPage
CarrierListPage
CarrierFormPage
ShipmentListPage
ShipmentTable
ShipmentMobileCard
ShipmentFilterBar
ShipmentFilterDrawer
ShipmentDetailPage
ShipmentSummaryCard
ShipmentTimeline
ShipmentStatusBadge
CreateShipmentDrawer
UpdateShipmentStatusModal
PrintLabelButton
CarrierConnectionStatus
ShippingRatePreviewPanel
CODStatusBadge
ReturnShipmentPanel
```

Shared components:

```text
AdminLayout
AdminSidebar
AdminTopbar
DataTable
StatusBadge
ConfirmModal
FormSection
EmptyState
ErrorState
Skeleton
Drawer
Toast
```

---

# 32. Page structure recommendation

## 32.1. Shipping Overview

```text
AdminShippingOverviewPage
├── AdminPageHeader
├── ShippingKpiGrid
├── ShippingAlertCenter
├── ShipmentsRequiringActionTable
├── CarrierPerformanceWidget
└── RecentShipmentEvents
```

## 32.2. Shipping Methods

```text
ShippingMethodListPage
├── AdminPageHeader
├── TableToolbar
├── ShippingMethodTable
├── EmptyState
└── Pagination
```

## 32.3. Shipment Detail

```text
ShipmentDetailPage
├── AdminBreadcrumb
├── ShipmentHeader
├── ShipmentSummaryCard
├── OrderSummaryCard
├── RecipientAddressCard
├── PackageInfoCard
├── CODInfoCard
├── ShipmentTimeline
├── ShipmentItemsTable
├── InternalNotes
└── AuditLog
```

---

# 33. Playwright test specification

## 33.1. Shipping methods tests

```text
Admin can view shipping method list.
Admin can create shipping method.
Admin cannot create method without name.
Admin cannot create duplicate method code.
Admin can disable shipping method with confirm modal.
Inactive method shows inactive badge.
```

## 33.2. Shipping zone tests

```text
Admin can view shipping zones.
Admin can create zone with province coverage.
Zone coverage required validation appears.
Overlapping zone warning appears.
Admin can disable zone.
```

## 33.3. Shipping rate tests

```text
Admin can view rate rules.
Admin can create flat rate rule.
Admin can create free shipping rule.
Fee cannot be negative.
Min condition cannot be greater than max condition.
Rate preview returns expected fee.
```

## 33.4. Carrier tests

```text
Admin can view carrier list.
Admin can create manual carrier.
Tracking URL template accepts tracking code placeholder.
Test connection button shows result state.
Secret key is not displayed in full.
```

## 33.5. Shipment list tests

```text
Admin can view shipment list.
Admin can search by order number.
Admin can search by tracking code.
Admin can filter by carrier.
Admin can filter by shipping status.
Mobile shipment list has no horizontal overflow.
```

## 33.6. Create shipment tests

```text
Admin can create shipment from order detail.
Cannot create shipment for cancelled order.
Cannot create shipment without shipping address.
COD amount is shown when order payment method is COD.
Shipment created successfully shows toast.
Tracking code appears in order detail after creation.
```

## 33.7. Shipment status tests

```text
Admin can update shipment status from created to ready_for_pickup.
Admin can update shipment status to in_transit.
Admin can mark delivered with confirm modal.
Invalid transition is blocked.
Delivery failed requires reason.
Return shipment can be created from delivery failed.
```

## 33.8. Permission tests

```text
Viewer can view shipment but cannot update.
Warehouse staff can print label but cannot edit carrier credentials.
Support staff can view delivery failed shipment but cannot change COD amount.
Unauthorized user is redirected to admin login.
```

---

# 34. Visual regression checklist

Capture screenshots:

```text
Shipping overview desktop
Shipping overview mobile
Shipping method list
Shipping method form
Shipping zone form
Shipping rate form
Carrier list
Shipment list desktop
Shipment list mobile card layout
Shipment detail desktop
Create shipment drawer
Update status modal
Delivery failed state
Empty shipment list
Error state
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
No horizontal overflow.
Table scroll contained.
Badge readable.
Form labels visible.
Modal not cut off.
Mobile actions tappable.
Address text wraps safely.
Tracking code does not break layout.
```

---

# 35. Definition of Done

Module Admin Shipping Management được coi là hoàn thành khi:

## 35.1. UI

```text
Shipping overview page hoạt động.
Shipping method list/create/edit hoạt động.
Shipping zone list/create/edit hoạt động.
Shipping rate list/create/edit hoạt động.
Carrier list/create/edit hoạt động ở MVP manual mode.
Shipment list hoạt động.
Shipment detail hoạt động.
Create shipment flow hoạt động.
Update shipment status flow hoạt động.
Loading/empty/error states đầy đủ.
Responsive không vỡ.
```

## 35.2. Business logic

```text
Shipping fee calculation không chỉ dựa frontend.
Rate rules có validation.
Không tạo shipment cho order không hợp lệ.
Status transition được kiểm soát.
COD amount đúng rule.
Order timeline nhận shipping events.
Inventory/order relation không bị phá.
```

## 35.3. Security

```text
Permission hoạt động.
Sensitive data được mask ở list.
Carrier secrets không leak.
Dangerous actions có confirm.
Audit log ghi action quan trọng.
```

## 35.4. Test

```text
Playwright tests chính pass.
Responsive test pass.
Visual screenshots đã kiểm tra.
Không có console error nghiêm trọng.
Không có horizontal overflow.
```

---

# 36. MVP scope

MVP nên làm trước:

```text
Shipping method CRUD.
Shipping zone CRUD theo province/city.
Flat rate rule CRUD.
Free shipping threshold.
Manual carrier CRUD.
Shipment list.
Create shipment manually from order.
Manual tracking code.
Manual status update.
Print/download mock shipping label.
Order detail hiển thị shipment info.
Basic COD display.
```

Chưa cần ngay:

```text
Carrier API thật.
Webhook tracking.
Pickup scheduling nâng cao.
COD reconciliation chi tiết.
Multi-package shipment.
Multi-warehouse shipping optimization.
Insurance fee automation.
Advanced return logistics.
```

---

# 37. Future extension

Sau MVP có thể mở rộng:

```text
Tích hợp API GHN/GHTK/Viettel Post/J&T.
Auto create waybill.
Auto tracking sync.
Shipping fee quote realtime từ carrier.
Bulk print labels.
Pickup batch management.
COD reconciliation.
Multi-package shipment.
Split shipment by warehouse.
Shipping insurance.
Delivery SLA monitoring.
Customer self-service reschedule delivery.
Return label generation.
Advanced shipping analytics.
```

---

# 38. Clone-source notes

Module shipping phải giữ lõi dùng chung cho nhiều ngành:

```text
Shipping method
Zone
Rate rule
Carrier
Shipment
Tracking
Return shipment
COD
```

Phần riêng ngành đồ điện tử nằm ở:

```text
Fragile flag
Bulky flag
High-value insurance warning
Serial/IMEI placeholder
Warranty return relation
Careful packaging note
```

Khi clone sang ngành khác:

```text
Thời trang: chú trọng đổi trả size, phí theo khu vực, nhẹ cân.
Mỹ phẩm: chú trọng đóng gói chống vỡ/chống tràn.
Nội thất: chú trọng bulky item, lịch hẹn giao/lắp đặt.
Thực phẩm: chú trọng giao nhanh, nhiệt độ, khu vực hỗ trợ.
Sản phẩm số: shipping module có thể disabled hoặc chỉ dùng fulfillment digital.
```

Không sửa core shipping flow nếu chỉ đổi ngành hàng. Chỉ đổi rule, copywriting, surcharge và shipping methods.

---

# 39. Tóm tắt cho agent

Nếu chỉ nhớ phần ngắn nhất:

```text
Shipping Management là module nối Checkout, Order, Inventory và Customer Support.
Không tính phí ship chỉ ở frontend.
Shipping method/zone/rate phải cấu hình được.
Shipment phải có tracking/status/timeline.
COD phải rõ amount/status.
Đơn giao lỗi phải nổi bật và có next action.
Return shipment không được tự restock nếu chưa kiểm tra hàng.
Admin UI phải rõ, ít màu, có loading/empty/error state.
Action nguy hiểm phải confirm.
Dữ liệu địa chỉ/số điện thoại phải bảo vệ theo permission.
Mobile không được overflow.
```

---

# 40. Prompt tiếp theo nên tạo

Sau file này, prompt tiếp theo trong roadmap là:

```text
17-payment-design.md
```

Mục tiêu file tiếp theo:

```text
Thiết kế module thanh toán: COD, chuyển khoản, payment status, payment intent, callback/webhook, retry payment, refund, payment reconciliation và liên kết với Order/Checkout/Shipping.
```
