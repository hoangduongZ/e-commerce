# 15 - Admin Warranty & Service Management Specification

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Module: Warranty & Service Management  
> Mục tiêu: đặc tả đủ chi tiết để coding agent/frontend/backend agent có thể xây module quản lý bảo hành, đổi trả, sửa chữa và dịch vụ sau bán hàng cho website bán đồ điện tử.  
> Phụ thuộc trực tiếp:  
> - `../main/ecommerce_design_language.md`  
> - `01-electronics-store-theme.md`  
> - `09-admin-dashboard.md`  
> - `12-admin-order-management.md`  
> - `13-admin-inventory-management.md`  
> - `../main/system-design.md`  
>
> File này vừa là **prompt triển khai cho agent**, vừa là **spec nghiệp vụ/UI/API/test** cho module warranty/service.

---

# 0. Prompt dùng cho coding agent

Dùng prompt này khi yêu cầu agent code module Warranty & Service Management:

```text
Bạn là Senior Full-stack E-commerce Agent.

Hãy implement module Admin Warranty & Service Management cho website bán đồ điện tử.

Trước khi code, bắt buộc đọc các tài liệu:
1. ../main/ecommerce_design_language.md
2. 01-electronics-store-theme.md
3. 09-admin-dashboard.md
4. 12-admin-order-management.md
5. 13-admin-inventory-management.md
6. 15-admin-warranty-service-management.md

Mục tiêu:
- Xây màn danh sách yêu cầu bảo hành/dịch vụ.
- Xây màn chi tiết yêu cầu bảo hành/dịch vụ.
- Xây màn quản lý chính sách bảo hành.
- Hỗ trợ tạo warranty claim từ order item.
- Hỗ trợ serial/IMEI/service tag ở mức có thể mở rộng.
- Hỗ trợ trạng thái tiếp nhận, kiểm tra, sửa chữa, đổi mới, hoàn trả, từ chối, hoàn tất.
- Hỗ trợ ghi chú nội bộ, timeline, file đính kèm, ảnh/video lỗi.
- Hỗ trợ liên kết order, customer, product, variant, inventory và shipment nếu cần.
- Tạo đủ loading/empty/error state.
- Tạo đủ permission guard.
- Không hard-code riêng laptop; module phải dùng được cho điện thoại, tablet, màn hình, phụ kiện, linh kiện.

Nguyên tắc:
- Warranty/service là nghiệp vụ sau bán hàng, không được làm sai lịch sử đơn hàng.
- Không sửa trực tiếp order item snapshot nếu tạo yêu cầu bảo hành.
- Không tự ý hoàn tiền/đổi hàng nếu chưa qua flow xác nhận.
- Mọi action quan trọng phải ghi audit log.
- Mọi action nguy hiểm phải có confirm modal.
- Không hiển thị đầy đủ thông tin cá nhân nếu role không có quyền.
- Mobile không được overflow ngang.
- Table desktop phải có strategy responsive.

Sau khi code xong, báo cáo:
- Files changed
- Components created/updated
- APIs integrated/mocked
- Validation implemented
- Permissions implemented
- Tests added/updated
- Tests run
- Screenshots checked
- Known limitations
```

---

# 1. Vai trò của module Warranty & Service

## 1.1. Vì sao module này quan trọng

Với website bán đồ điện tử, bán xong chưa phải là kết thúc. Khách thường quan tâm rất mạnh tới:

```text
Bảo hành bao lâu?
Bảo hành chính hãng hay cửa hàng?
Nếu lỗi thì gửi về đâu?
Đổi mới trong bao nhiêu ngày?
Có cần serial/IMEI không?
Đang sửa đến bước nào?
Khi nào nhận lại hàng?
Có bị từ chối bảo hành không? Vì sao?
```

Module Warranty & Service giúp shop quản lý toàn bộ phần sau bán hàng:

```text
Chính sách bảo hành
Tra cứu bảo hành
Yêu cầu bảo hành
Yêu cầu đổi trả
Yêu cầu sửa chữa
Kiểm tra serial/IMEI
Tiếp nhận sản phẩm lỗi
Ghi nhận tình trạng lỗi
Theo dõi tiến độ xử lý
Trả hàng cho khách
Ghi nhận chi phí sửa chữa nếu có
Liên kết order/product/customer/inventory
```

Nếu module này làm tốt, shop tăng niềm tin và giảm tranh chấp.

Nếu module này làm kém, khách sẽ mất niềm tin dù sản phẩm ban đầu tốt.

---

# 2. Phạm vi nghiệp vụ

## 2.1. Bao gồm trong module

Module này bao gồm:

```text
Warranty policy management
Warranty lookup
Warranty claim list
Warranty claim detail
Create warranty claim
Service ticket management
Return / replacement request
Repair workflow
Serial / IMEI / service tag record
Claim status timeline
Customer communication log
Internal notes
Attachment management
Inspection result
Resolution decision
Replacement flow
Return-to-customer flow
Warranty analytics basic
Audit log
Permission
```

## 2.2. Không bao gồm đầy đủ trong MVP

Các phần sau có thể để mở rộng:

```text
Tích hợp API hãng bảo hành chính hãng
Tự động tạo vận đơn hai chiều
OCR phiếu bảo hành
AI phân loại lỗi qua ảnh/video
Tính phí sửa chữa nâng cao
Quản lý linh kiện sửa chữa chi tiết
Bảo hành mở rộng trả phí
Multi-service-center routing nâng cao
```

---

# 3. Đối tượng sử dụng

## 3.1. Admin roles

| Role | Mục đích |
|---|---|
| Super Admin | Toàn quyền |
| Store Manager | Quản lý toàn bộ bảo hành/dịch vụ |
| Support Staff | Tiếp nhận yêu cầu, trao đổi khách |
| Warranty Staff | Kiểm tra, xử lý, cập nhật trạng thái bảo hành |
| Warehouse Staff | Nhận/trả hàng bảo hành, cập nhật tồn kho đổi mới |
| Finance Staff | Xử lý hoàn tiền/chi phí nếu có |
| Viewer | Chỉ xem |

## 3.2. Customer-facing liên quan

Dù file này tập trung admin, module cần liên kết với customer area:

```text
Khách xem sản phẩm còn bảo hành không
Khách gửi yêu cầu bảo hành
Khách upload ảnh/video lỗi
Khách theo dõi trạng thái claim
Khách nhận hướng dẫn gửi hàng
Khách nhận kết quả xử lý
```

Admin module phải thiết kế data đủ để storefront/customer account có thể dùng lại.

---

# 4. Entity khái niệm

## 4.1. Warranty Policy

Chính sách bảo hành áp dụng cho sản phẩm/danh mục/thương hiệu.

Ví dụ:

```text
Laptop Dell: bảo hành chính hãng 12 tháng
MacBook: bảo hành Apple 12 tháng
Laptop gaming: đổi mới 7 ngày nếu lỗi phần cứng
Tai nghe: bảo hành cửa hàng 6 tháng
Phụ kiện cáp/sạc: bảo hành 3 tháng
```

## 4.2. Warranty Registration

Bản ghi bảo hành phát sinh từ order item hoặc serial.

Một order item sau khi giao thành công có thể sinh ra warranty registration.

## 4.3. Warranty Claim

Yêu cầu bảo hành/đổi trả/sửa chữa do khách hoặc admin tạo.

Ví dụ:

```text
Khách mua laptop Dell, sau 2 tháng bị lỗi màn hình.
Support tạo claim liên kết order item.
Warranty staff tiếp nhận, kiểm tra, gửi hãng hoặc đổi mới.
```

## 4.4. Service Ticket

Một service ticket có thể là bản xử lý kỹ thuật bên trong warranty claim.

Có thể hiểu:

```text
Warranty Claim = yêu cầu từ khách
Service Ticket = quá trình xử lý kỹ thuật/vận hành bên trong
```

MVP có thể gộp claim và ticket nếu muốn đơn giản.

## 4.5. Serial / IMEI / Service Tag

Đồ điện tử thường cần định danh thiết bị:

```text
Laptop: serial number / service tag
Điện thoại: IMEI
Màn hình: serial number
Tai nghe: serial number nếu có
Linh kiện: serial/batch number nếu cần
```

Module phải hỗ trợ optional serial-level tracking.

Không phải sản phẩm nào cũng có serial.

---

# 5. Route admin đề xuất

```text
/admin/warranty
/admin/warranty/claims
/admin/warranty/claims/new
/admin/warranty/claims/:id
/admin/warranty/claims/:id/edit
/admin/warranty/policies
/admin/warranty/policies/new
/admin/warranty/policies/:id/edit
/admin/warranty/lookup
/admin/warranty/service-tickets
/admin/warranty/service-tickets/:id
/admin/warranty/settings
```

## 5.1. Route chính

`/admin/warranty/claims`

Dùng để xem danh sách yêu cầu bảo hành/dịch vụ.

## 5.2. Route chi tiết

`/admin/warranty/claims/:id`

Dùng để xử lý một claim cụ thể.

## 5.3. Route policy

`/admin/warranty/policies`

Dùng để quản lý chính sách bảo hành.

## 5.4. Route lookup

`/admin/warranty/lookup`

Dùng để tra cứu nhanh theo:

```text
Order number
Customer phone
Product SKU
Serial number
IMEI
Warranty claim number
```

---

# 6. Tổng quan màn hình module

## 6.1. Các màn chính

```text
Warranty dashboard mini overview
Warranty claim list
Warranty claim detail
Create warranty claim
Warranty policy list
Warranty policy create/edit
Warranty lookup
Service ticket list
Service ticket detail
Warranty settings
```

## 6.2. MVP tối thiểu

Nếu làm nhanh MVP, cần ít nhất:

```text
Claim list
Claim detail
Create claim from order item
Update claim status
Warranty policy list
Warranty policy form
Warranty timeline
Internal notes
Attachment upload mock
Permission basic
```

---

# 7. Warranty claim lifecycle

## 7.1. Claim status

Trạng thái claim đề xuất:

```text
new
received
under_review
waiting_for_customer
waiting_for_product
inspecting
repairing
sent_to_manufacturer
replacement_approved
refund_approved
rejected
ready_to_return
returned_to_customer
completed
cancelled
```

## 7.2. Ý nghĩa trạng thái

| Status | Ý nghĩa |
|---|---|
| new | Yêu cầu mới được tạo |
| received | Admin đã tiếp nhận |
| under_review | Đang xem xét thông tin |
| waiting_for_customer | Cần khách bổ sung thông tin |
| waiting_for_product | Chờ khách gửi sản phẩm lỗi |
| inspecting | Đang kiểm tra sản phẩm |
| repairing | Đang sửa chữa |
| sent_to_manufacturer | Đã gửi hãng/đối tác bảo hành |
| replacement_approved | Được duyệt đổi mới |
| refund_approved | Được duyệt hoàn tiền |
| rejected | Từ chối bảo hành |
| ready_to_return | Sẵn sàng trả hàng cho khách |
| returned_to_customer | Đã trả hàng cho khách |
| completed | Hoàn tất |
| cancelled | Đã hủy yêu cầu |

## 7.3. Status transition

Luồng phổ biến:

```text
new
→ received
→ under_review
→ waiting_for_product
→ inspecting
→ repairing
→ ready_to_return
→ returned_to_customer
→ completed
```

Luồng đổi mới:

```text
new
→ received
→ inspecting
→ replacement_approved
→ ready_to_return
→ returned_to_customer
→ completed
```

Luồng gửi hãng:

```text
new
→ received
→ inspecting
→ sent_to_manufacturer
→ repairing
→ ready_to_return
→ returned_to_customer
→ completed
```

Luồng từ chối:

```text
new
→ received
→ inspecting
→ rejected
→ ready_to_return
→ returned_to_customer
→ completed
```

Luồng cần bổ sung thông tin:

```text
new
→ under_review
→ waiting_for_customer
→ under_review
```

## 7.4. Rule chuyển trạng thái

Không phải trạng thái nào cũng chuyển được sang mọi trạng thái.

Ví dụ không hợp lệ:

```text
completed → repairing
cancelled → replacement_approved
rejected → repairing nếu chưa reopen
new → completed trực tiếp
```

Cần có action `reopen` với quyền cao nếu muốn mở lại claim đã đóng.

---

# 8. Warranty claim list page

## 8.1. Mục đích

Trang danh sách claim giúp admin:

```text
Xem toàn bộ yêu cầu bảo hành/dịch vụ
Tìm kiếm theo mã claim, mã đơn, khách, SĐT, serial
Lọc claim theo trạng thái, loại yêu cầu, priority, ngày tạo
Xem nhanh claim nào quá SLA
Gán người xử lý
Đi vào chi tiết claim
Xuất danh sách nếu cần
```

## 8.2. Layout desktop

```text
┌────────────────────────────────────────────────────────────────────┐
│ Admin Topbar                                                       │
├───────────────┬────────────────────────────────────────────────────┤
│ Sidebar       │ Warranty & Service                                 │
│               │                                                    │
│               │ [Claim KPI cards]                                  │
│               │                                                    │
│               │ [Search claim/order/serial/customer]               │
│               │ [Status] [Type] [Priority] [Date] [Assignee]       │
│               │ [Create claim] [Export]                            │
│               │                                                    │
│               │ Claims table                                       │
│               │                                                    │
│               │ Pagination                                         │
└───────────────┴────────────────────────────────────────────────────┘
```

## 8.3. Layout mobile

```text
Topbar compact
Page title
KPI cards 1 column
Search
Filter button
Claim cards
Pagination / load more
```

Mobile không dùng table đầy đủ. Dùng card list.

---

# 9. Warranty claim KPI cards

## 9.1. KPI gợi ý

```text
New claims
Claims over SLA
Waiting for product
In repair
Ready to return
Rejected this month
```

## 9.2. Card structure

Mỗi KPI card gồm:

```text
Label
Value
Trend/helper text
Severity nếu cần
Click link tới filtered list
```

Ví dụ:

```text
Claims over SLA
8
Need attention
/admin/warranty/claims?sla=overdue
```

## 9.3. Rule

```text
Không dùng quá nhiều KPI.
KPI ưu tiên action cần xử lý.
Số quá hạn SLA dùng warning/danger.
Click card phải áp dụng filter tương ứng.
```

---

# 10. Search và filter

## 10.1. Search input

Placeholder:

```text
Search by claim ID, order number, phone, serial, IMEI...
```

Search theo:

```text
Claim number
Order number
Customer name
Customer phone
Customer email
Product name
Product SKU
Variant SKU
Serial number
IMEI
Service tag
```

Behavior:

```text
Debounce 300ms
Enter search ngay
Có clear button
Giữ query trên URL
```

## 10.2. Quick tabs

Gợi ý quick tabs:

```text
All
New
Overdue
Waiting for product
Inspecting
Repairing
Ready to return
Completed
Rejected
```

## 10.3. Filter nâng cao

Các filter:

```text
Claim status
Claim type
Resolution type
Priority
Warranty eligibility
Product category
Brand
Assignee
Created date range
Updated date range
SLA status
Service center
```

## 10.4. URL rule

Filter phải phản ánh trên URL:

```text
/admin/warranty/claims?status=repairing&priority=high&brand=dell&page=1
```

---

# 11. Warranty claim table

## 11.1. Cột mặc định desktop

| Column | Nội dung |
|---|---|
| Checkbox | Chọn claim |
| Claim | Mã claim + loại |
| Customer | Tên + SĐT đã mask |
| Product | Sản phẩm + SKU/variant |
| Serial/IMEI | Mã thiết bị nếu có |
| Status | Trạng thái claim |
| Priority | Độ ưu tiên |
| SLA | Còn hạn/quá hạn |
| Assignee | Người xử lý |
| Updated | Cập nhật gần nhất |
| Actions | View / Update / More |

## 11.2. Claim cell

Hiển thị:

```text
#WR-2026-000128
Warranty claim
Created 22/06/2026
```

Nếu claim là return/replacement:

```text
#WR-2026-000129
Replacement request
```

## 11.3. Product cell

Hiển thị:

```text
Laptop Dell Inspiron 15 3520
SKU: DELL-INS-3520-I5-16-512
Variant: Silver / 16GB / 512GB
```

Tên sản phẩm tối đa 2 dòng.

## 11.4. Status badge

Status badge phải dùng semantic tone:

| Status | Tone |
|---|---|
| new | info |
| received | info |
| under_review | warning |
| waiting_for_customer | warning |
| waiting_for_product | warning |
| inspecting | info |
| repairing | primary/info |
| sent_to_manufacturer | primary |
| replacement_approved | success |
| refund_approved | success |
| rejected | danger |
| ready_to_return | success |
| completed | neutral/success |
| cancelled | neutral |

## 11.5. SLA cell

Hiển thị:

```text
On time
Due in 2 days
Overdue 1 day
```

Overdue phải có icon + text, không chỉ dùng màu.

---

# 12. Claim mobile card

Mobile card cần có:

```text
Claim number
Status badge
Customer masked
Product name
Serial/IMEI nếu có
Priority
SLA
Updated time
Primary action: View
```

Ví dụ:

```text
#WR-2026-000128        Repairing
Nguyễn V. A · 090****000
Dell Inspiron 15 3520 · 16GB / 512GB
Serial: SVT-89X2-2026
SLA: Due in 2 days
[View detail]
```

Rule:

```text
Không overflow ngang.
Action dễ bấm.
Không nhồi quá nhiều thông tin.
```

---

# 13. Bulk actions

Bulk action dùng cho danh sách claim.

Actions:

```text
Assign to staff
Change priority
Export selected
Mark as received
Close selected nếu đủ điều kiện
```

Không nên bulk các action nguy hiểm như:

```text
Reject claims
Approve refunds
Approve replacements
Delete claims
```

Nếu vẫn cần, phải có quyền cao + confirm kỹ.

---

# 14. Create warranty claim flow

## 14.1. Entry points

Có thể tạo claim từ:

```text
Admin Warranty module
Order detail page
Customer detail page
Customer account request
Support conversation
```

## 14.2. Create claim steps

Flow đề xuất:

```text
Step 1: Find order/customer/product
Step 2: Select order item / product variant
Step 3: Check warranty eligibility
Step 4: Enter issue information
Step 5: Upload evidence
Step 6: Choose initial action
Step 7: Create claim
```

## 14.3. Step 1 - Find order/customer/product

Search theo:

```text
Order number
Phone
Email
Customer name
Serial/IMEI
SKU
```

Nếu tìm theo order:

```text
Hiển thị danh sách order items có thể claim.
```

## 14.4. Step 2 - Select order item

Hiển thị:

```text
Product image
Product name
Variant
SKU
Quantity purchased
Delivered date
Warranty policy snapshot
Warranty end date
Existing claims
```

Rule:

```text
Một order item có thể có nhiều claim, nhưng phải cảnh báo nếu claim đang mở.
Không cho tạo claim trùng nếu cùng item đang có claim active, trừ khi role có quyền override.
```

## 14.5. Step 3 - Warranty eligibility

Hệ thống kiểm tra:

```text
Order đã delivered chưa
Ngày mua/ngày giao
Warranty months
Return period days
Serial/IMEI có khớp không
Sản phẩm có chính sách bảo hành không
Claim trước đó có đang mở không
```

Kết quả:

```text
Eligible
Expired
Not eligible
Need manual review
```

## 14.6. Step 4 - Issue information

Fields:

```text
Claim type
Issue category
Issue description
Customer reported date
Product condition
Serial/IMEI
Accessories included
Preferred resolution
Customer note
```

Claim type:

```text
Warranty repair
Replacement request
Return request
Refund request
Technical support
```

Issue category ví dụ:

```text
No power
Screen issue
Battery issue
Keyboard issue
Speaker issue
Connectivity issue
Physical damage
Software issue
Accessory missing
Other
```

## 14.7. Step 5 - Evidence upload

Cho phép upload:

```text
Ảnh lỗi
Video lỗi
Hóa đơn/phiếu mua hàng nếu cần
Ảnh serial/IMEI
Biên bản nhận hàng
```

Rule:

```text
Validate file type
Validate file size
Có preview
Có delete
Không bắt upload nếu claim tạo bởi admin và có lý do bỏ qua
```

## 14.8. Step 6 - Initial action

Admin chọn:

```text
Receive claim only
Ask customer to send product
Schedule pickup
Send troubleshooting instruction
Reject immediately nếu rõ không hợp lệ
```

Reject ngay phải cần reason.

---

# 15. Warranty claim detail page

## 15.1. Mục đích

Trang chi tiết claim là nơi xử lý toàn bộ case.

Admin cần thấy:

```text
Claim là gì
Khách là ai
Sản phẩm nào
Có còn bảo hành không
Lỗi là gì
Đang ở trạng thái nào
Ai đang xử lý
Đã làm những gì
Cần làm gì tiếp theo
```

## 15.2. Layout desktop

```text
┌────────────────────────────────────────────────────────────────────┐
│ Header: Claim #WR-2026-000128 + Status + Actions                   │
├────────────────────────────────────────────┬───────────────────────┤
│ Main content                               │ Side panel            │
│                                            │                       │
│ Claim summary                              │ Customer card         │
│ Product/order card                         │ Eligibility card      │
│ Issue details                              │ SLA card              │
│ Inspection result                          │ Assignee card         │
│ Resolution                                 │ Related links         │
│ Timeline                                   │                       │
│ Notes                                      │                       │
│ Attachments                                │                       │
└────────────────────────────────────────────┴───────────────────────┘
```

## 15.3. Header actions

Actions theo trạng thái:

```text
Receive
Request product
Start inspection
Send to manufacturer
Start repair
Approve replacement
Approve refund
Reject claim
Ready to return
Mark returned
Complete
Cancel
Reopen
Print service form
```

Không hiển thị action không hợp lệ theo status hiện tại.

## 15.4. Claim summary section

Fields:

```text
Claim number
Claim type
Status
Priority
Created at
Updated at
Created by
Source
Assignee
Service center
```

Source có thể là:

```text
Customer account
Admin manual
Order detail
Support ticket
```

## 15.5. Customer card

Hiển thị:

```text
Customer name
Masked phone
Masked email
Customer segment nếu có
Link customer detail
```

Không hiển thị full phone/email nếu role không có quyền.

## 15.6. Product/order card

Hiển thị:

```text
Order number
Order date
Delivered date
Product image
Product name
Variant
SKU
Serial/IMEI
Warranty policy snapshot
Warranty start/end
```

CTA:

```text
View order
View product
View inventory item
```

## 15.7. Eligibility card

Hiển thị:

```text
Warranty status: Active / Expired / Not eligible / Manual review
Warranty start date
Warranty end date
Days remaining
Policy name
Return period status
```

Nếu expired:

```text
Warranty expired 18 days ago
```

Nếu manual review:

```text
Needs manual review: serial number not matched
```

## 15.8. Issue details

Fields:

```text
Issue category
Issue description
Customer preferred resolution
Reported condition
Accessories included
Customer note
Evidence attachments
```

## 15.9. Inspection result

Warranty staff cập nhật:

```text
Inspection date
Inspector
Actual condition
Root cause
Warranty coverage result
Damage type
Repair estimate
Manufacturer case number
Inspection attachments
```

Damage type:

```text
Manufacturing defect
User damage
Shipping damage
Software issue
No fault found
Unknown
```

Coverage result:

```text
Covered
Not covered
Partially covered
Need manufacturer decision
```

## 15.10. Resolution section

Resolution options:

```text
Repair
Replace with same product
Replace with equivalent product
Refund
Reject
Return without repair
Provide technical instruction only
```

Resolution fields:

```text
Resolution type
Approved by
Approval date
Reason
Replacement product/variant nếu có
Refund amount nếu có
Repair cost nếu có
Customer charge nếu có
Expected completion date
```

---

# 16. Service ticket detail

Nếu tách Service Ticket, mỗi claim có thể có 1 hoặc nhiều ticket.

## 16.1. Service ticket fields

```text
Ticket number
Claim id
Technician
Service center
Issue diagnosis
Repair action
Parts used
Labor cost
Parts cost
Status
Started at
Completed at
```

## 16.2. Service ticket status

```text
open
assigned
diagnosing
waiting_for_parts
repairing
testing
completed
cancelled
```

## 16.3. Parts used optional

Nếu sau này quản lý linh kiện sửa chữa:

```text
Part SKU
Part name
Quantity
Cost
Source warehouse
```

MVP chưa cần parts inventory chi tiết.

---

# 17. Warranty policy management

## 17.1. Warranty policy list

Route:

```text
/admin/warranty/policies
```

Cột table:

| Column | Nội dung |
|---|---|
| Policy name | Tên chính sách |
| Applies to | Category/brand/product |
| Warranty months | Số tháng bảo hành |
| Return days | Số ngày đổi trả |
| Provider | Manufacturer/store |
| Status | Active/inactive |
| Updated | Cập nhật |
| Actions | Edit/duplicate/archive |

## 17.2. Warranty policy form

Fields:

```text
Policy name
Policy code
Provider type
Warranty months
Warranty start rule
Return period days
Replacement period days
Coverage description
Exclusion description
Applicable scope
Requires serial
Requires proof of purchase
Status
```

Provider type:

```text
Manufacturer warranty
Store warranty
Extended warranty
Supplier warranty
No warranty
```

Warranty start rule:

```text
From order delivered date
From order placed date
From invoice date
From manual activation date
```

Applicable scope:

```text
All products
Category
Brand
Product
Variant
Tag
```

## 17.3. Policy rule

```text
Một sản phẩm active nên có warranty policy nếu thuộc ngành điện tử.
Order item phải lưu snapshot warranty policy tại thời điểm mua.
Thay đổi policy sau này không được làm thay đổi đơn cũ, trừ khi có migration chủ động.
```

## 17.4. Exclusions

Chính sách nên có vùng loại trừ:

```text
Rơi vỡ, móp méo
Vào nước
Cháy nổ do nguồn điện không ổn định
Tự ý tháo máy
Mất tem bảo hành
Không có serial/IMEI hợp lệ
Hết hạn bảo hành
```

Không cần hard-code exclusions. Cho admin nhập text hoặc template.

---

# 18. Warranty lookup

## 18.1. Mục đích

Tra cứu nhanh bảo hành tại quầy/support.

Search theo:

```text
Serial number
IMEI
Order number
Customer phone
Product SKU
Claim number
```

## 18.2. Lookup result

Hiển thị:

```text
Product
Variant
Customer
Order
Delivered date
Warranty policy
Warranty status
Existing claims
CTA create claim
```

## 18.3. No result state

Message:

```text
No warranty record found.
Try searching by order number, phone, serial, or IMEI.
```

CTA:

```text
Create manual claim
```

Manual claim cần quyền riêng.

---

# 19. Replacement flow

## 19.1. Khi nào dùng replacement

Dùng khi:

```text
Sản phẩm lỗi trong thời gian đổi mới
Sản phẩm không sửa được
Nhà cung cấp/hãng duyệt đổi
Shop chủ động đổi để giữ khách
```

## 19.2. Replacement data

Fields:

```text
Original product/variant
Replacement product/variant
Replacement SKU
Replacement serial/IMEI
Price difference
Approved by
Reason
Inventory source
```

## 19.3. Inventory interaction

Nếu đổi hàng:

```text
Trừ tồn replacement item
Ghi stock movement type = warranty_replacement_out
Có thể nhận lại hàng lỗi vào kho lỗi/RMA stock
Ghi stock movement type = warranty_return_in
```

MVP có thể chỉ ghi log, chưa cần kho lỗi chi tiết.

## 19.4. Rule

```text
Không tự động đổi hàng nếu replacement product hết tồn.
Nếu đổi sang sản phẩm khác giá, cần hiển thị difference.
Action approve replacement cần quyền cao.
```

---

# 20. Refund flow

## 20.1. Khi nào refund

Dùng khi:

```text
Không còn hàng thay thế
Khách được duyệt hoàn tiền
Sản phẩm lỗi nghiêm trọng
Chính sách cho phép hoàn tiền
```

## 20.2. Refund data

Fields:

```text
Refund amount
Original paid amount
Refund method
Refund reason
Approved by
Payment reference
Finance note
```

## 20.3. Rule

```text
Không refund vượt số tiền đã thanh toán.
Refund phải liên kết payment/order.
Refund cần quyền riêng.
Refund phải có audit log.
MVP có thể chỉ tạo refund request, chưa thực hiện hoàn tiền tự động.
```

---

# 21. Rejection flow

## 21.1. Khi nào reject

Claim có thể bị từ chối khi:

```text
Hết hạn bảo hành
Lỗi do người dùng
Không có serial/IMEI hợp lệ
Sản phẩm không thuộc đơn hàng
Tự ý sửa chữa trước đó
Không cung cấp được bằng chứng cần thiết
```

## 21.2. Reject modal

Bắt buộc có:

```text
Reject reason
Detailed note
Customer-facing message
Internal note optional
Attachment optional
```

## 21.3. Rule

```text
Không cho reject không có lý do.
Customer-facing message phải rõ, không quá kỹ thuật.
Có thể chuyển claim sang ready_to_return sau reject nếu đã nhận sản phẩm.
```

---

# 22. Return-to-customer flow

## 22.1. Khi nào trả hàng

Sau khi:

```text
Sửa xong
Đổi mới xong
Từ chối bảo hành và trả lại hàng
Không phát hiện lỗi
Khách hủy yêu cầu
```

## 22.2. Return fields

```text
Return method
Customer pickup / shipping
Shipping provider
Tracking code
Return address
Returned at
Receiver name
Proof of delivery
```

## 22.3. Rule

```text
Nếu gửi ship, cần tracking code nếu có.
Nếu khách nhận tại cửa hàng, cần người xác nhận.
Sau khi returned_to_customer có thể complete claim.
```

---

# 23. Attachments

## 23.1. Attachment types

```text
Customer issue image
Customer issue video
Serial/IMEI image
Invoice/proof of purchase
Inspection image
Repair report
Manufacturer report
Shipping document
Return proof
```

## 23.2. Attachment fields

```json
{
  "id": "att_001",
  "claim_id": "wclaim_001",
  "type": "inspection_image",
  "filename": "screen-issue.jpg",
  "url": "/uploads/warranty/screen-issue.jpg",
  "mime_type": "image/jpeg",
  "size_bytes": 520000,
  "uploaded_by": "admin_1",
  "created_at": "2026-06-22T10:30:00+07:00"
}
```

## 23.3. Security rule

```text
Validate file type.
Validate file size.
Không cho upload executable/script.
Không expose URL private nếu không có permission.
Có thể cần signed URL nếu dùng object storage.
```

---

# 24. Notes và communication

## 24.1. Internal notes

Internal note chỉ admin thấy.

Fields:

```text
Author
Role
Content
Created at
Edited flag nếu có
```

Rule:

```text
Không gửi internal note cho khách.
Có audit nếu sửa/xóa note.
```

## 24.2. Customer message

Customer message là nội dung có thể gửi cho khách.

Ví dụ:

```text
Shop đã tiếp nhận yêu cầu bảo hành của anh/chị.
Vui lòng gửi sản phẩm về địa chỉ...
Sản phẩm đã được sửa xong và đang chờ gửi trả.
```

MVP có thể chỉ lưu message log, chưa cần gửi email/SMS thật.

---

# 25. Timeline

## 25.1. Timeline events

Các event cần ghi:

```text
Claim created
Claim received
Status changed
Assignee changed
Customer message sent
Attachment uploaded
Inspection completed
Resolution approved
Product received
Sent to manufacturer
Repair started
Repair completed
Replacement approved
Refund requested
Refund approved
Rejected
Returned to customer
Completed
Reopened
Cancelled
```

## 25.2. Timeline item

```json
{
  "id": "evt_001",
  "claim_id": "wclaim_001",
  "type": "status_changed",
  "title": "Status changed to Inspecting",
  "description": "Warranty staff started inspecting the product.",
  "actor_id": "admin_1",
  "actor_name": "Minh",
  "created_at": "2026-06-22T11:00:00+07:00",
  "metadata": {
    "from": "received",
    "to": "inspecting"
  }
}
```

## 25.3. Rule

```text
Timeline append-only.
Không xóa timeline event bằng UI thường.
Timeline dùng để audit nhẹ và hỗ trợ vận hành.
Audit log chính thức vẫn nên tách nếu cần.
```

---

# 26. Data contract

## 26.1. WarrantyPolicy

```json
{
  "id": "wpol_001",
  "name": "Laptop manufacturer warranty 12 months",
  "code": "LAPTOP-MFG-12M",
  "provider_type": "manufacturer",
  "warranty_months": 12,
  "return_period_days": 7,
  "replacement_period_days": 7,
  "warranty_start_rule": "delivered_date",
  "requires_serial": true,
  "requires_proof_of_purchase": true,
  "coverage_description": "Manufacturing defects are covered.",
  "exclusion_description": "Physical damage, water damage, and unauthorized repair are excluded.",
  "applicable_scope": {
    "type": "category",
    "category_ids": ["cat_laptop"]
  },
  "status": "active",
  "created_at": "2026-06-22T00:00:00+07:00",
  "updated_at": "2026-06-22T00:00:00+07:00"
}
```

## 26.2. WarrantyRegistration

```json
{
  "id": "wreg_001",
  "order_id": "ord_001",
  "order_item_id": "ord_item_001",
  "customer_id": "cus_001",
  "product_id": "prod_001",
  "variant_id": "var_001",
  "sku": "DELL-INS-3520-I5-16-512",
  "serial_number": "SVT-89X2-2026",
  "imei": null,
  "policy_snapshot": {
    "policy_id": "wpol_001",
    "name": "Laptop manufacturer warranty 12 months",
    "warranty_months": 12,
    "return_period_days": 7
  },
  "warranty_start_at": "2026-06-22T00:00:00+07:00",
  "warranty_end_at": "2027-06-22T23:59:59+07:00",
  "status": "active"
}
```

## 26.3. WarrantyClaim

```json
{
  "id": "wclaim_001",
  "claim_number": "WR-2026-000128",
  "type": "warranty_repair",
  "status": "inspecting",
  "priority": "high",
  "source": "admin_manual",
  "customer_id": "cus_001",
  "customer_name": "Nguyễn Văn A",
  "customer_phone_masked": "090****000",
  "order_id": "ord_001",
  "order_number": "DH1024",
  "order_item_id": "ord_item_001",
  "product_id": "prod_001",
  "variant_id": "var_001",
  "product_name_snapshot": "Laptop Dell Inspiron 15 3520 i5 16GB 512GB",
  "variant_label_snapshot": "Silver / 16GB / 512GB",
  "sku_snapshot": "DELL-INS-3520-I5-16-512",
  "serial_number": "SVT-89X2-2026",
  "imei": null,
  "issue_category": "screen_issue",
  "issue_description": "Screen flickers after 10 minutes of use.",
  "preferred_resolution": "repair",
  "eligibility_status": "eligible",
  "assigned_to": "admin_2",
  "service_center_id": "svc_001",
  "sla_due_at": "2026-06-25T17:00:00+07:00",
  "created_at": "2026-06-22T10:00:00+07:00",
  "updated_at": "2026-06-22T11:00:00+07:00"
}
```

## 26.4. InspectionResult

```json
{
  "id": "insp_001",
  "claim_id": "wclaim_001",
  "inspector_id": "admin_3",
  "actual_condition": "Device powers on, screen flickers under brightness above 70%.",
  "root_cause": "Display panel defect",
  "damage_type": "manufacturing_defect",
  "coverage_result": "covered",
  "repair_estimate_amount": 0,
  "customer_charge_amount": 0,
  "manufacturer_case_number": "DELLCASE-2026-8841",
  "created_at": "2026-06-22T13:00:00+07:00"
}
```

---

# 27. API contract

API chỉ là gợi ý, có thể đổi theo framework.

## 27.1. Claim list

```http
GET /api/v1/admin/warranty/claims
```

Query params:

```text
search
status
type
priority
eligibility_status
product_id
category_id
brand_id
assignee_id
service_center_id
sla_status
created_from
created_to
updated_from
updated_to
page
page_size
sort
```

## 27.2. Claim detail

```http
GET /api/v1/admin/warranty/claims/{id}
```

## 27.3. Create claim

```http
POST /api/v1/admin/warranty/claims
```

Request:

```json
{
  "order_id": "ord_001",
  "order_item_id": "ord_item_001",
  "type": "warranty_repair",
  "issue_category": "screen_issue",
  "issue_description": "Screen flickers after 10 minutes of use.",
  "serial_number": "SVT-89X2-2026",
  "preferred_resolution": "repair"
}
```

## 27.4. Update claim status

```http
POST /api/v1/admin/warranty/claims/{id}/status
```

Request:

```json
{
  "status": "inspecting",
  "note": "Product received and inspection started."
}
```

## 27.5. Assign claim

```http
POST /api/v1/admin/warranty/claims/{id}/assign
```

Request:

```json
{
  "assignee_id": "admin_2"
}
```

## 27.6. Add inspection result

```http
POST /api/v1/admin/warranty/claims/{id}/inspection
```

## 27.7. Approve resolution

```http
POST /api/v1/admin/warranty/claims/{id}/resolution
```

Request:

```json
{
  "resolution_type": "repair",
  "reason": "Covered manufacturing defect.",
  "expected_completion_date": "2026-06-25"
}
```

## 27.8. Reject claim

```http
POST /api/v1/admin/warranty/claims/{id}/reject
```

Request:

```json
{
  "reason_code": "user_damage",
  "customer_message": "The warranty claim cannot be approved because the product has physical damage not covered by the policy.",
  "internal_note": "Visible liquid damage near keyboard."
}
```

## 27.9. Add note

```http
POST /api/v1/admin/warranty/claims/{id}/notes
```

## 27.10. Upload attachment

```http
POST /api/v1/admin/warranty/claims/{id}/attachments
```

## 27.11. Lookup warranty

```http
GET /api/v1/admin/warranty/lookup?query=SVT-89X2-2026
```

## 27.12. Warranty policy APIs

```http
GET    /api/v1/admin/warranty/policies
GET    /api/v1/admin/warranty/policies/{id}
POST   /api/v1/admin/warranty/policies
PATCH  /api/v1/admin/warranty/policies/{id}
POST   /api/v1/admin/warranty/policies/{id}/archive
```

---

# 28. Validation rules

## 28.1. Create claim validation

Bắt buộc:

```text
Claim type required
Issue category required
Issue description required
Customer/order/product reference required nếu không phải manual claim
Serial/IMEI required nếu policy requires_serial
```

## 28.2. Status update validation

```text
Status transition phải hợp lệ.
Reject required reason.
Approve refund required refund amount.
Approve replacement required replacement item.
Ready to return required return method nếu sản phẩm đã nhận.
Complete chỉ được nếu claim đã returned hoặc không cần return.
```

## 28.3. Policy validation

```text
Policy name required
Warranty months >= 0
Return period days >= 0
Replacement period days >= 0
Applicable scope required
Policy code unique
Active policy không được thiếu provider type
```

## 28.4. Attachment validation

```text
Allowed file types: jpg, jpeg, png, webp, pdf, mp4 optional
Max file size theo config
Không allow executable/script
```

---

# 29. Permission matrix

| Permission | Super Admin | Store Manager | Support | Warranty Staff | Warehouse | Finance | Viewer |
|---|---|---|---|---|---|---|---|
| warranty.view | yes | yes | yes | yes | yes | yes | yes |
| warranty.create | yes | yes | yes | yes | no | no | no |
| warranty.update | yes | yes | yes | yes | limited | no | no |
| warranty.assign | yes | yes | no | yes | no | no | no |
| warranty.inspect | yes | yes | no | yes | no | no | no |
| warranty.approve_replacement | yes | yes | no | limited | no | no | no |
| warranty.approve_refund | yes | yes | no | no | no | yes | no |
| warranty.reject | yes | yes | no | yes | no | no | no |
| warranty.policy.manage | yes | yes | no | no | no | no | no |
| warranty.attachment.upload | yes | yes | yes | yes | yes | no | no |
| warranty.note.internal | yes | yes | yes | yes | yes | yes | no |

Rule:

```text
Frontend ẩn action nếu thiếu quyền.
Backend vẫn phải kiểm tra permission.
Viewer không được thấy internal notes nhạy cảm nếu policy yêu cầu.
Finance chỉ thấy phần refund liên quan.
```

---

# 30. Security và privacy

## 30.1. PII masking

Thông tin nhạy cảm cần mask mặc định:

```text
Phone: 090****000
Email: n***@example.com
Address: chỉ hiển thị đầy đủ nếu role có quyền
```

## 30.2. Attachment security

```text
File private cần signed URL.
Không expose path nội bộ.
Scan file nếu hệ thống production.
Giới hạn quyền xem file.
```

## 30.3. Audit

Các action phải audit:

```text
Create claim
Update status
Reject claim
Approve replacement
Approve refund
Change assignee
Change policy
Upload/delete attachment
Add/edit internal note
Reopen claim
Cancel claim
```

Audit fields:

```text
actor
role
action
target
before
after
reason
timestamp
ip/device optional
```

---

# 31. Loading, empty, error states

## 31.1. Claim list loading

```text
KPI skeleton
Toolbar visible
Table skeleton rows
Không hiển thị fake data
```

## 31.2. Claim detail loading

```text
Header skeleton
Summary skeleton
Product card skeleton
Timeline skeleton
Side panel skeleton
```

## 31.3. Empty claim list

Message:

```text
No warranty claims yet.
Claims created by customers or support staff will appear here.
```

CTA:

```text
Create claim
Lookup warranty
```

## 31.4. Filtered empty

Message:

```text
No claims match your filters.
Try changing the search keyword or clearing filters.
```

CTA:

```text
Clear filters
```

## 31.5. Error state

Lỗi thường gặp:

```text
Cannot load claims
Cannot load claim detail
Cannot update status
Invalid status transition
Permission denied
Attachment upload failed
Warranty policy conflict
```

Rule:

```text
Không mất dữ liệu form khi save fail.
Có retry nếu lỗi network.
Validation error phải gắn với field cụ thể.
```

---

# 32. Responsive rules

## 32.1. Desktop

```text
Sidebar visible
Claim list dùng table
Claim detail dùng 2 column: main + side panel
Timeline full width trong main
```

## 32.2. Tablet

```text
Sidebar collapsed
Table có thể horizontal scroll
Claim detail side panel chuyển xuống dưới nếu thiếu rộng
```

## 32.3. Mobile

```text
Sidebar drawer
Claim list chuyển thành card
Toolbar wrap thành search + filter button
Claim detail single column
Action bar sticky bottom nếu nhiều action
Không overflow ngang
```

---

# 33. Accessibility rules

```text
Mọi input có label.
Status badge có text, không chỉ màu.
Action icon-only có aria-label.
Modal confirm trap focus.
Error summary focus được.
Timeline có cấu trúc semantic.
Attachment preview có alt text nếu là ảnh.
Table header đúng semantic.
Keyboard dùng được cho filter, modal, action menu.
```

---

# 34. Component structure đề xuất

```text
AdminWarrantyClaimListPage
WarrantyClaimToolbar
WarrantyClaimKpiCards
WarrantyClaimTable
WarrantyClaimMobileCard
WarrantyClaimStatusBadge
WarrantySlaBadge
WarrantyClaimDetailPage
WarrantyClaimHeader
WarrantyClaimSummaryCard
WarrantyCustomerCard
WarrantyProductOrderCard
WarrantyEligibilityCard
WarrantyIssueDetails
WarrantyInspectionPanel
WarrantyResolutionPanel
WarrantyTimeline
WarrantyInternalNotes
WarrantyAttachmentList
WarrantyStatusActionMenu
CreateWarrantyClaimPage
CreateWarrantyClaimStepper
WarrantyLookupPage
WarrantyPolicyListPage
WarrantyPolicyFormPage
WarrantyRejectModal
WarrantyResolutionModal
WarrantyReturnModal
WarrantyAttachmentUploader
```

---

# 35. Interaction rules

## 35.1. Click claim row

Click claim number hoặc row chính:

```text
/admin/warranty/claims/{id}
```

Row action không được bị trigger nếu click vào checkbox/action menu.

## 35.2. Update status

Flow:

```text
Click action
Open modal nếu cần thông tin
Validate
Call API
Show loading on action
Success toast
Refresh claim detail/timeline
Failure show error and keep modal data
```

## 35.3. Reject claim

Flow:

```text
Click Reject
Open reject modal
Input reason + customer message
Confirm
Call API
Update status rejected
Append timeline
```

## 35.4. Approve replacement

Flow:

```text
Click Approve replacement
Open replacement modal
Select replacement variant
Check stock
Confirm
Call API
Create inventory movement/reservation if integrated
Update claim status replacement_approved
```

## 35.5. Upload attachment

Flow:

```text
Choose file
Validate client-side
Show upload progress
Success append attachment
Failure show retry
```

---

# 36. Analytics events

Events optional:

```text
admin_warranty_claim_list_viewed
admin_warranty_claim_created
admin_warranty_claim_status_changed
admin_warranty_claim_rejected
admin_warranty_replacement_approved
admin_warranty_refund_approved
admin_warranty_policy_created
admin_warranty_lookup_used
admin_warranty_attachment_uploaded
```

Không gửi dữ liệu cá nhân không cần thiết.

---

# 37. Playwright test specification

## 37.1. Claim list tests

```text
Admin can view warranty claim list
Admin can search by claim number
Admin can search by order number
Admin can search by serial/IMEI
Admin can filter by status
Admin can filter by priority
Admin can open claim detail
Empty state appears when no claims exist
Filtered empty state appears when no claims match filters
Mobile claim list has no horizontal overflow
```

## 37.2. Create claim tests

```text
Admin can open create claim page
Admin can find order by order number
Admin can select order item
Eligibility status is displayed
Admin cannot create claim without issue description
Admin can create valid warranty claim
Duplicate active claim warning appears
```

## 37.3. Claim detail tests

```text
Admin can view claim summary
Admin can view customer masked info
Admin can view product/order info
Admin can view eligibility card
Admin can view timeline
Admin can add internal note
Admin can upload attachment mock
```

## 37.4. Status action tests

```text
Admin can receive claim
Admin can start inspection
Admin cannot perform invalid transition
Reject claim requires reason
Approve replacement requires replacement item
Ready to return requires return method
Completed claim cannot be edited by normal role
```

## 37.5. Policy tests

```text
Admin can view policy list
Admin can create warranty policy
Policy code must be unique
Warranty months cannot be negative
Applicable scope is required
Admin can archive inactive policy
```

## 37.6. Permission tests

```text
Viewer can view claim but cannot update
Support can create claim but cannot approve refund
Warranty staff can inspect and reject
Finance can approve refund but cannot inspect
Unauthorized user is redirected or blocked
```

---

# 38. Visual regression checklist

Capture screenshots:

```text
Warranty claim list desktop
Warranty claim list mobile
Filtered empty state
Claim detail desktop
Claim detail mobile
Create claim stepper
Reject modal
Resolution modal
Policy list
Policy form
Attachment upload state
Timeline state
Permission-limited state
```

Viewports:

```text
1440px desktop
1024px laptop
768px tablet
375px mobile
```

Must verify:

```text
No horizontal overflow
Status badge readable
Timeline not broken
Modal fits mobile
Action bar not covering content
Long product name does not break layout
Masked PII displayed correctly
```

---

# 39. Definition of Done

Module Warranty & Service được coi là hoàn thành khi:

```text
Claim list page hoạt động
Search/filter/pagination hoạt động
Claim detail page hoạt động
Create claim flow hoạt động ở mức MVP
Warranty eligibility hiển thị rõ
Status transition được validate
Timeline được ghi khi update trạng thái
Internal notes hoạt động
Attachment upload mock hoặc thật hoạt động
Warranty policy list/form hoạt động
Permission UI hoạt động
Security/PII masking được xử lý
Loading/empty/error state đầy đủ
Responsive không overflow
Playwright tests chính pass
Visual snapshots không có diff bất thường
Không có console error nghiêm trọng
```

---

# 40. MVP scope

Nếu làm MVP trước, chỉ cần:

```text
Warranty claim list
Search claim/order/phone/serial
Filter status/type
Claim detail
Create claim from order item
Eligibility display basic
Update status basic
Reject claim with reason
Internal notes
Timeline
Policy list
Policy create/edit
Permission basic
Mobile responsive
```

Chưa cần ngay:

```text
Manufacturer API integration
Advanced service center routing
Parts inventory
Auto shipping label
Refund automation
AI diagnosis
Advanced SLA configuration
Customer-facing full portal
```

---

# 41. Future extension

Sau MVP có thể mở rộng:

```text
Customer warranty portal
Serial/IMEI batch import
Warranty activation QR code
Manufacturer warranty API sync
Auto-create shipping label
Repair parts inventory
Service center capacity planning
AI issue classification
Warranty cost analytics
Extended warranty upsell
Warranty fraud detection
```

---

# 42. Clone-source notes

Module này phải clone được sang nhiều ngành.

Core giữ nguyên:

```text
Policy
Registration
Claim
Ticket
Timeline
Attachment
Note
Resolution
Audit
```

Phần riêng ngành nằm ở:

```text
Issue categories
Serial/IMEI requirement
Warranty duration
Return rules
Resolution options
Service center workflow
```

Khi clone sang ngành khác:

```text
Điện tử: serial/IMEI rất quan trọng
Thời trang: đổi size/đổi màu quan trọng hơn sửa chữa
Mỹ phẩm: đổi trả do lỗi/dị ứng có rule riêng
Nội thất: bảo hành lắp đặt/vận chuyển quan trọng
Sản phẩm số: license/support ticket quan trọng
```

Không hard-code laptop/phone vào core warranty model.

