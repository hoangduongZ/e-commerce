# 13-admin-inventory-management.md

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

# Admin Inventory Management Specification

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Màn hình/module: Quản lý tồn kho  
> Mục tiêu: Đặc tả đủ chi tiết để coding agent/frontend/backend có thể code module quản lý tồn kho từ đầu đến cuối.  
> Phụ thuộc: `../main/ecommerce_design_language.md`, `01-electronics-store-theme.md`, `09-admin-dashboard.md`, `10-admin-product-management.md`, `12-admin-order-management.md`, `../main/system-design.md`  
> Không phụ thuộc công nghệ frontend/backend cụ thể.

---

## 0. Prompt dùng cho coding agent

Sử dụng prompt này khi giao task cho frontend/backend/fullstack agent:

```text
Bạn là Senior Fullstack Engineer kiêm UI Implementation Agent.

Hãy đọc và tuân thủ các tài liệu sau trước khi code:

1. ../main/ecommerce_design_language.md
2. 01-electronics-store-theme.md
3. 09-admin-dashboard.md
4. 10-admin-product-management.md
5. 12-admin-order-management.md
6. 13-admin-inventory-management.md
7. ../main/system-design.md

Nhiệm vụ của bạn là implement module Admin Inventory Management cho website bán hàng đồ điện tử.

Phạm vi bắt buộc:

- Admin inventory overview.
- Inventory list theo product / variant / SKU / warehouse.
- Search / filter / sort / pagination.
- Low stock, out of stock, oversold, reserved stock.
- Stock movement history.
- Stock adjustment modal.
- Import stock.
- Export stock.
- Reserve / release stock liên quan đến order.
- Inventory detail page.
- Permission theo role.
- Loading / empty / error state.
- Responsive desktop/tablet/mobile.
- Accessibility cơ bản.
- Playwright test cho các flow chính.

Nguyên tắc bắt buộc:

- Không hard-code laptop-only inventory logic.
- Tồn kho phải ưu tiên theo variant/SKU, không chỉ product cha.
- Không sửa tồn kho trực tiếp mà không tạo StockMovement.
- Không cho quantity âm trừ khi nghiệp vụ cho phép oversold rõ ràng.
- Không để một action nguy hiểm chạy mà không có confirm modal.
- Không để frontend là nguồn sự thật về tồn kho.
- Backend phải validate permission và tính toán tồn kho lại.
- Không dùng màu hard-code ngoài design token.
- Không để mobile 375px overflow ngang.
- Không xóa test để pass.

Sau khi code xong, báo cáo:

- Files changed.
- Components created/updated.
- APIs integrated/mocked.
- Tests added/updated.
- Tests run.
- Screenshots checked.
- Known limitations.
```

---

# 1. Vai trò của module Inventory Management

Module Inventory Management là trung tâm kiểm soát tồn kho của shop.

Với website bán đồ điện tử, tồn kho không chỉ là một con số đơn giản. Một sản phẩm có thể có nhiều biến thể:

```text
Laptop A / RAM 16GB / SSD 512GB / Silver
Laptop A / RAM 32GB / SSD 1TB / Black
iPhone 15 Pro / 256GB / Natural Titanium
Màn hình 27 inch / 2K / 165Hz
```

Mỗi biến thể có thể có:

```text
SKU riêng
Giá riêng
Tồn kho riêng
Kho riêng
Ngưỡng cảnh báo riêng
Serial/IMEI riêng trong tương lai
Bảo hành riêng
Trạng thái bán riêng
```

Vì vậy module inventory phải quản lý theo **SKU/variant-level** trước, product-level chỉ là tổng hợp.

---

# 2. Mục tiêu của module

Admin dùng module này để:

- Xem tồn kho toàn bộ sản phẩm.
- Xem tồn kho từng biến thể.
- Tìm kiếm SKU nhanh.
- Lọc sản phẩm tồn kho thấp.
- Lọc sản phẩm hết hàng.
- Lọc sản phẩm bị oversold.
- Điều chỉnh tồn kho có lý do.
- Nhập kho.
- Xuất kho.
- Reserve hàng khi có đơn.
- Release hàng khi đơn hủy/thanh toán lỗi.
- Xem lịch sử biến động kho.
- Biết sản phẩm nào cần nhập thêm.
- Xuất báo cáo tồn kho.
- Import tồn kho từ file.
- Kiểm soát quyền người thao tác.

---

# 3. Tư duy thiết kế nghiệp vụ

## 3.1. Tồn kho là dữ liệu nhạy cảm

Sai tồn kho có thể gây hậu quả trực tiếp:

```text
Bán nhầm hàng đã hết.
Không bán được hàng đang còn.
Sai báo cáo giá trị kho.
Sai fulfillment.
Khách đặt hàng rồi bị hủy.
Nhân sự không biết cần nhập thêm hàng.
```

Vì vậy inventory không được xử lý hời hợt.

## 3.2. Không sửa số tồn kho không có log

Mọi thay đổi tồn kho phải sinh ra `StockMovement`.

Không được làm:

```text
UPDATE inventory SET available = 100 WHERE sku = 'ABC'
```

Phải làm:

```text
Tạo StockMovement type=adjust, delta=+10, reason='Manual correction'
Sau đó cập nhật InventoryItem trong cùng transaction
```

## 3.3. Frontend không phải nguồn sự thật

Frontend chỉ hiển thị và gửi ý định thao tác.

Backend phải tự kiểm tra:

```text
SKU có tồn tại không?
User có quyền không?
Số lượng hợp lệ không?
Order có hợp lệ không?
Reserve/release có trùng không?
Stock movement có idempotency không?
```

## 3.4. Inventory phải liên kết chặt với Order

Khi tạo đơn:

```text
Cart checkout
→ backend kiểm tra tồn kho
→ reserve stock
→ tạo order
→ thanh toán thành công hoặc COD
→ fulfillment xử lý
```

Khi đơn hủy:

```text
Order cancel
→ release reserved stock
→ ghi StockMovement type=release
```

Khi đơn giao thành công:

```text
Reserved stock
→ chuyển thành sold/exported
→ giảm on hand nếu chưa giảm trước đó
```

Cần chọn strategy rõ ràng cho MVP.

---

# 4. Khái niệm tồn kho chuẩn

## 4.1. InventoryItem

Một `InventoryItem` đại diện cho tồn kho của một product/variant tại một warehouse.

```text
Product + Variant + Warehouse = InventoryItem
```

Ví dụ:

```text
Product: Dell Inspiron 15
Variant: 16GB / 512GB / Silver
Warehouse: Hà Nội
```

## 4.2. Các trường số lượng

Nên dùng các trường sau:

| Field | Ý nghĩa |
|---|---|
| `on_hand_quantity` | Tổng số lượng vật lý đang có trong kho |
| `reserved_quantity` | Số lượng đã giữ cho đơn nhưng chưa hoàn tất |
| `available_quantity` | Số lượng có thể bán |
| `incoming_quantity` | Số lượng đang nhập về, chưa có trong kho |
| `damaged_quantity` | Số lượng lỗi/hỏng không bán được |
| `safety_stock_quantity` | Số lượng giữ an toàn, không nên bán hết |
| `low_stock_threshold` | Ngưỡng cảnh báo sắp hết |

Công thức gợi ý:

```text
available_quantity = on_hand_quantity - reserved_quantity - damaged_quantity - safety_stock_quantity
```

Nếu MVP đơn giản hơn:

```text
available_quantity = quantity_available
reserved_quantity = quantity_reserved
```

Nhưng UI vẫn nên thiết kế để mở rộng.

## 4.3. Stock status

Các trạng thái tồn kho:

| Status | Điều kiện gợi ý | Ý nghĩa |
|---|---|---|
| `in_stock` | available > threshold | Còn hàng |
| `low_stock` | 0 < available <= threshold | Sắp hết |
| `out_of_stock` | available = 0 | Hết hàng |
| `oversold` | available < 0 | Bán vượt tồn |
| `not_tracked` | tracking disabled | Không theo dõi tồn |
| `discontinued` | product discontinued | Ngừng kinh doanh |

## 4.4. StockMovement

`StockMovement` là lịch sử biến động kho.

Mỗi lần tồn kho thay đổi phải có một movement.

Types:

| Type | Ý nghĩa |
|---|---|
| `import` | Nhập kho |
| `export` | Xuất kho |
| `adjust` | Điều chỉnh thủ công |
| `reserve` | Giữ hàng cho đơn |
| `release` | Trả lại hàng đã giữ |
| `return` | Khách trả hàng |
| `damage` | Chuyển sang hàng lỗi |
| `transfer_out` | Chuyển kho đi |
| `transfer_in` | Nhận hàng chuyển kho |
| `stock_count` | Điều chỉnh sau kiểm kho |

---

# 5. Route admin

Route gợi ý:

```text
/admin/inventory
/admin/inventory/low-stock
/admin/inventory/out-of-stock
/admin/inventory/movements
/admin/inventory/import
/admin/inventory/export
/admin/inventory/stock-count
/admin/inventory/transfers
/admin/inventory/:inventoryItemId
/admin/inventory/:inventoryItemId/movements
/admin/inventory/:inventoryItemId/adjust
/admin/products/:productId/inventory
/admin/orders/:orderId/inventory
```

Route MVP bắt buộc:

```text
/admin/inventory
/admin/inventory/:inventoryItemId
/admin/inventory/movements
/admin/inventory/import
```

---

# 6. Admin Inventory Overview

## 6.1. Mục đích

Trang overview giúp admin nhìn nhanh tình trạng tồn kho:

```text
Có bao nhiêu SKU còn hàng?
Có bao nhiêu SKU sắp hết?
Có bao nhiêu SKU đã hết?
Có SKU nào oversold không?
Hàng nào đang bị reserved nhiều?
Hàng nào cần nhập thêm?
```

## 6.2. KPI cards

Các card nên có:

```text
Total SKUs
In Stock SKUs
Low Stock SKUs
Out of Stock SKUs
Reserved Units
Inventory Value
```

MVP có thể dùng:

```text
Total SKUs
Low Stock
Out of Stock
Reserved Units
```

## 6.3. KPI card behavior

Click vào card nên dẫn tới inventory list đã filter.

Ví dụ:

```text
Low Stock → /admin/inventory?stock_status=low_stock
Out of Stock → /admin/inventory?stock_status=out_of_stock
Reserved Units → /admin/inventory?reserved_gt=0
```

## 6.4. Warning panel

Nên có alert center nhỏ:

```text
12 variants are low stock.
3 variants are oversold.
5 best-selling products are below threshold.
2 orders cannot be fulfilled due to stock issue.
```

Alert nghiêm trọng không nên chỉ là màu đỏ. Phải có text và action.

---

# 7. Inventory List Page

## 7.1. Mục đích

Trang danh sách tồn kho là màn hình chính cho warehouse/admin.

Admin dùng để:

```text
Tìm SKU.
Xem tồn kho.
Lọc theo trạng thái.
Điều chỉnh nhanh.
Xem movement.
Đi tới product detail.
Đi tới inventory detail.
```

## 7.2. Layout desktop

```text
┌────────────────────────────────────────────────────────────────────┐
│ Admin Topbar                                                       │
├───────────────┬────────────────────────────────────────────────────┤
│ Sidebar       │ Inventory Management                               │
│               │                                                    │
│               │ KPI Cards                                          │
│               │                                                    │
│               │ [Search SKU/Product] [Warehouse] [Status] [Cat]    │
│               │ [Brand] [Reserved] [More filters] [Import] [Export]│
│               │                                                    │
│               │ Inventory Table                                    │
│               │                                                    │
│               │ Pagination                                         │
└───────────────┴────────────────────────────────────────────────────┘
```

## 7.3. Layout mobile

Mobile admin không phải chính, nhưng không được vỡ.

```text
Topbar compact
Page title
KPI cards 1 column
Search
[Filter] [Import]
Inventory cards
Pagination
```

Mobile nên dùng card list thay vì table đầy đủ.

## 7.4. Toolbar

Toolbar gồm:

```text
Search input
Warehouse select
Stock status select
Category select
Brand select
Reserved filter
Low stock quick toggle
More filters
Import button
Export button
```

Search hỗ trợ:

```text
SKU
Product name
Variant label
Brand
Model code
Barcode/serial trong tương lai
```

Placeholder:

```text
Search by SKU, product name, model...
```

Behavior:

```text
Debounce 300ms.
Enter search ngay.
Clear button.
Giữ query trên URL.
```

## 7.5. Filter nâng cao

More filters có thể mở drawer/popover:

```text
Available min/max
Reserved min/max
On hand min/max
Threshold min/max
Updated date
Has incoming stock
Has damaged stock
Track inventory enabled
Allow backorder
Supplier
Warehouse zone
```

## 7.6. Inventory table columns

Cột mặc định:

| Column | Nội dung |
|---|---|
| Checkbox | chọn dòng |
| Product / Variant | ảnh, tên, variant label |
| SKU | mã SKU, copyable |
| Warehouse | kho |
| Available | có thể bán |
| Reserved | đang giữ |
| On hand | tồn vật lý |
| Threshold | ngưỡng cảnh báo |
| Status | badge trạng thái |
| Updated | lần cập nhật cuối |
| Actions | thao tác |

Không nên nhồi toàn bộ thông tin product vào table.

## 7.7. Product / Variant cell

Hiển thị:

```text
Thumbnail
Product name
Variant label
Category/brand nhỏ
```

Ví dụ:

```text
Laptop Dell Inspiron 15 3520
16GB RAM / SSD 512GB / Silver
Laptop · Dell
```

Rule:

```text
Tên tối đa 2 dòng.
Variant label rõ.
Thumbnail 48px hoặc 56px.
Nếu thiếu ảnh, dùng placeholder.
```

## 7.8. SKU cell

SKU phải copy được.

Ví dụ:

```text
DELL-INS-3520-I5-16-512-SLV [Copy]
```

Nếu SKU trùng hoặc thiếu trong data mock, hiển thị warning.

## 7.9. Quantity cells

Các cột số lượng nên căn phải hoặc căn giữa nhất quán.

Ví dụ:

```text
Available: 3
Reserved: 2
On hand: 5
Threshold: 5
```

Nếu available <= threshold, highlight nhẹ bằng warning background.

Nếu available < 0, hiển thị danger.

## 7.10. Status badge

Badge gợi ý:

```text
In stock
Low stock
Out of stock
Oversold
Not tracked
Discontinued
```

Màu theo semantic:

```text
In stock    → success
Low stock   → warning
Out of stock→ neutral/danger
Oversold    → danger
Not tracked → neutral
```

## 7.11. Row actions

Actions chính:

```text
View detail
Adjust stock
View movements
View product
```

Actions phụ:

```text
Disable selling
Update threshold
Transfer stock
Export row
```

Action nguy hiểm cần confirm.

---

# 8. Inventory Detail Page

## 8.1. Mục đích

Trang detail giúp admin xem đầy đủ một inventory item.

Nên dùng khi:

```text
Cần xem lịch sử stock movement.
Cần biết hàng đang reserved bởi đơn nào.
Cần điều chỉnh tồn kho.
Cần xem liên kết product/variant/order.
```

## 8.2. Layout desktop

```text
Breadcrumb: Admin / Inventory / SKU
Header: SKU + status + actions

Main grid:
- Left/main: inventory summary, movement history, reserved orders
- Right/sticky: quick actions, product info, warehouse info
```

## 8.3. Header

Header gồm:

```text
Product name
Variant label
SKU
Status badge
Warehouse
Last updated
Actions: Adjust stock, Import, Export, View product
```

## 8.4. Inventory summary card

Các chỉ số:

```text
On hand
Available
Reserved
Incoming
Damaged
Safety stock
Low stock threshold
```

Nếu MVP chưa có đủ field, vẫn để slot mở rộng.

## 8.5. Reserved orders section

Hiển thị các đơn đang giữ hàng:

| Order | Customer | Quantity | Status | Reserved at | Action |
|---|---|---:|---|---|---|
| DH1024 | Nguyễn V. A | 1 | Pending confirmation | 22/06/2026 10:30 | View |

Rule:

```text
Không hiển thị full phone/email nếu không cần.
Click order mở order detail.
Nếu order đã hủy nhưng reserve chưa release, hiển thị warning.
```

## 8.6. Movement history section

Bảng lịch sử:

| Time | Type | Delta | Before | After | Reason | Related | Actor |
|---|---|---:|---:|---:|---|---|---|

Ví dụ:

```text
22/06/2026 14:35 | adjust | +10 | 2 | 12 | Nhập bù kho | — | Admin A
22/06/2026 15:10 | reserve | -1 available | 12 | 11 | Order DH1024 | DH1024 | System
```

Rule:

```text
Không cho xóa movement.
Không sửa movement sau khi tạo.
Nếu có sai, tạo movement adjust ngược lại.
```

---

# 9. Stock Adjustment Modal

## 9.1. Khi nào dùng

Dùng khi admin cần sửa tồn kho thủ công.

Ví dụ:

```text
Kiểm kho phát hiện thiếu.
Nhập bổ sung hàng nhỏ.
Sửa sai lệch sau kiểm kê.
Đánh dấu hàng lỗi.
```

## 9.2. Không dùng cho trường hợp nào

Không dùng adjustment để xử lý:

```text
Order reserve/release tự động.
Import hàng hàng loạt từ file.
Transfer giữa kho.
Return/refund có flow riêng.
```

## 9.3. Modal fields

Fields:

```text
SKU readonly
Current on hand readonly
Current available readonly
Adjustment mode
Quantity
Reason
Reference code optional
Note optional
```

Adjustment mode:

```text
Increase by quantity
Decrease by quantity
Set exact on hand quantity
Move to damaged quantity
```

MVP chỉ cần:

```text
Increase
Decrease
Set exact
```

## 9.4. Validation

Rules:

```text
Quantity required.
Quantity must be positive number.
Reason required.
Decrease không được làm on_hand âm, trừ khi role cho phép oversold/negative.
Set exact không được âm.
Nếu giảm làm available < 0, phải confirm warning.
```

## 9.5. Confirm step

Vì adjustment ảnh hưởng dữ liệu nhạy cảm, cần confirm.

Dialog nên hiển thị:

```text
Bạn sắp điều chỉnh tồn kho SKU DELL-INS-3520-I5-16-512.
On hand: 5 → 8
Available: 3 → 6
Reason: Kiểm kho bổ sung
```

Button:

```text
Cancel
Confirm adjustment
```

## 9.6. Result

Sau khi submit:

```text
Tạo StockMovement type=adjust.
Cập nhật InventoryItem.
Refresh row/card.
Hiển thị toast success.
```

Nếu fail:

```text
Giữ nguyên input.
Hiển thị lỗi rõ.
Không đóng modal nếu lỗi validation/API.
```

---

# 10. Import Stock

## 10.1. Mục đích

Import stock dùng khi nhập/cập nhật nhiều SKU.

Flow:

```text
Upload file
Validate columns
Preview rows
Show row errors
Confirm import
Process import
Show result summary
```

## 10.2. File format

MVP hỗ trợ CSV.

Mở rộng hỗ trợ XLSX.

Columns gợi ý:

```text
sku
warehouse_code
mode
quantity
reason
reference_code
note
```

`mode` có thể là:

```text
increase
decrease
set_exact
```

## 10.3. Preview table

Preview phải hiển thị:

```text
Row number
SKU
Product/Variant
Warehouse
Current quantity
New quantity
Mode
Validation status
Error message
```

## 10.4. Validation

Các lỗi cần bắt:

```text
Missing SKU
SKU not found
Warehouse not found
Invalid quantity
Invalid mode
Duplicate SKU/warehouse trong file nếu policy không cho phép
Decrease results negative stock
Permission denied for some SKU/warehouse
```

## 10.5. Import result

Sau khi import:

```text
Total rows
Success rows
Failed rows
Skipped rows
Download error report
View created stock movements
```

Rule:

```text
Không import thẳng nếu chưa preview.
Không bỏ qua lỗi silently.
Có import batch id.
Mỗi row thành công phải tạo StockMovement.
```

---

# 11. Export Stock

## 11.1. Mục đích

Export giúp admin lấy báo cáo tồn kho.

Export options:

```text
Export current filtered list
Export selected rows
Export all inventory
Export low stock only
Export movement history
```

## 11.2. Fields export

Fields:

```text
product_id
variant_id
product_name
variant_label
sku
category
brand
warehouse
on_hand_quantity
available_quantity
reserved_quantity
low_stock_threshold
stock_status
last_updated
```

Nếu có quyền cao:

```text
cost_price
inventory_value
supplier
```

Không export dữ liệu nhạy cảm cho role không có quyền.

---

# 12. Stock Movement Page

## 12.1. Mục đích

Trang movement dùng để audit.

Không chỉ là lịch sử đẹp mắt, mà là bằng chứng:

```text
Ai đã sửa?
Sửa lúc nào?
Trước bao nhiêu?
Sau bao nhiêu?
Lý do gì?
Liên quan đến đơn nào?
```

## 12.2. Filters

```text
SKU
Product
Warehouse
Movement type
Actor
Date range
Related order
Import batch
Reason keyword
```

## 12.3. Table columns

| Column | Nội dung |
|---|---|
| Time | thời gian |
| SKU | mã SKU |
| Product | sản phẩm/variant |
| Warehouse | kho |
| Type | loại movement |
| Delta | thay đổi |
| Before | trước |
| After | sau |
| Reason | lý do |
| Related | order/import/transfer |
| Actor | người thao tác |

## 12.4. Movement detail drawer

Click một movement mở drawer:

```text
Movement id
Inventory item
Before snapshot
After snapshot
Reason
Actor
Related entity
IP/device optional
Created at
```

Rule:

```text
Movement readonly.
Không có nút edit/delete.
Nếu sai, dùng Create correcting adjustment.
```

---

# 13. Reserve / Release Stock

## 13.1. Reserve stock

Reserve stock xảy ra khi đơn được tạo hoặc xác nhận, tùy strategy.

Strategy MVP khuyến nghị:

```text
Khi tạo order thành công → reserve stock ngay.
Khi order hủy/thanh toán thất bại quá hạn → release stock.
Khi order delivered/completed → chuyển reserved thành sold/export.
```

## 13.2. Reserve validation

Backend phải kiểm tra:

```text
Inventory item tồn tại.
Track inventory đang bật.
Available đủ số lượng.
Order chưa reserve trước đó.
Idempotency key hợp lệ.
```

Nếu không đủ hàng:

```text
Không tạo order hoặc chuyển order sang stock_issue.
Hiển thị lỗi cho checkout/admin.
```

## 13.3. Release stock

Release khi:

```text
Order cancelled.
Payment failed quá hạn.
Order item removed trước fulfillment.
Admin manually releases stock với quyền cao.
```

Rule:

```text
Không release quá số lượng đã reserve.
Release phải liên kết order/order item.
Release phải tạo StockMovement.
```

## 13.4. Oversell policy

MVP nên không cho oversell.

Mở rộng có thể cho phép:

```text
Allow backorder per product.
Allow oversell for preorder.
Allow negative stock for selected role.
```

Nếu bật oversell, UI phải hiển thị rõ:

```text
Oversell enabled
Available can go below 0
Expected restock date required
```

---

# 14. Low Stock Management

## 14.1. Low stock threshold

Threshold có thể nằm ở:

```text
Global default
Category default
Product default
Variant/SKU override
Warehouse override
```

Ưu tiên:

```text
SKU/Warehouse threshold
→ Variant threshold
→ Product threshold
→ Category threshold
→ Global threshold
```

## 14.2. Low stock page

Route:

```text
/admin/inventory/low-stock
```

Hiển thị danh sách SKU cần nhập thêm.

Columns:

```text
Product/variant
SKU
Warehouse
Available
Reserved
Threshold
Sales last 7 days
Suggested reorder quantity
Action
```

## 14.3. Suggested reorder quantity

MVP có thể chưa tính thông minh.

Gợi ý đơn giản:

```text
suggested_reorder_quantity = threshold * 2 - available
```

Mở rộng:

```text
Dựa trên sales velocity.
Dựa trên lead time nhà cung cấp.
Dựa trên campaign sắp tới.
```

## 14.4. Low stock alert

Alert xuất hiện ở:

```text
Admin Dashboard
Inventory Overview
Product edit side panel
Order detail nếu ảnh hưởng fulfillment
```

---

# 15. Multi-warehouse

## 15.1. MVP strategy

MVP có thể chỉ có một kho mặc định:

```text
Default Warehouse
```

Nhưng data model/UI nên có warehouse field để mở rộng.

## 15.2. Warehouse fields

```text
warehouse_id
warehouse_code
warehouse_name
address
status
is_default
```

## 15.3. Transfer stock

Mở rộng sau MVP.

Flow:

```text
Create transfer
Select source warehouse
Select destination warehouse
Add SKU/quantity
Confirm transfer out
Receive transfer in
Create movement transfer_out and transfer_in
```

Rule:

```text
Không giảm source nếu không đủ available.
Transfer pending cần status riêng.
Có thể partial receive.
```

---

# 16. Electronics-specific inventory rules

## 16.1. Variant-level inventory là bắt buộc

Đồ điện tử có nhiều cấu hình. Không được chỉ quản lý product cha.

Ví dụ sai:

```text
Dell Inspiron 15: 10 chiếc
```

Ví dụ đúng:

```text
Dell Inspiron 15 / 8GB / 512GB / Silver: 4 chiếc
Dell Inspiron 15 / 16GB / 512GB / Silver: 3 chiếc
Dell Inspiron 15 / 16GB / 1TB / Black: 3 chiếc
```

## 16.2. Serial / IMEI placeholder

MVP chưa cần quản lý serial/IMEI từng máy, nhưng UI/data nên có chỗ mở rộng.

Tương lai có thể thêm:

```text
Serial number
IMEI
Warranty activation date
Supplier batch
Device condition
```

## 16.3. High value stock

Sản phẩm giá trị cao cần kiểm soát kỹ.

Rule mở rộng:

```text
Nếu SKU có unit price >= configurable threshold,
manual adjustment cần reason chi tiết hơn hoặc approval.
```

## 16.4. Warranty connection

Khi bán đồ điện tử, inventory liên quan đến warranty.

Order item sau fulfillment có thể cần:

```text
SKU
Serial/IMEI
Warranty months
Warranty start date
Warranty provider
```

Module inventory phải không cản trở mở rộng này.

---

# 17. Data contract

## 17.1. InventoryItem object

```json
{
  "id": "inv_001",
  "product_id": "prod_001",
  "variant_id": "var_001",
  "warehouse_id": "wh_hn_001",
  "sku": "DELL-INS-3520-I5-16-512-SLV",
  "product_name": "Laptop Dell Inspiron 15 3520",
  "variant_label": "16GB RAM / SSD 512GB / Silver",
  "category_name": "Laptop",
  "brand_name": "Dell",
  "image_url": "/images/products/dell-inspiron.jpg",
  "on_hand_quantity": 12,
  "reserved_quantity": 3,
  "available_quantity": 9,
  "incoming_quantity": 0,
  "damaged_quantity": 0,
  "safety_stock_quantity": 1,
  "low_stock_threshold": 5,
  "stockStatus": "in_stock",
  "track_inventory": true,
  "allow_backorder": false,
  "last_movement_at": "2026-06-22T14:35:00+07:00",
  "updated_at": "2026-06-22T14:35:00+07:00"
}
```

## 17.2. StockMovement object

```json
{
  "id": "mov_001",
  "inventory_item_id": "inv_001",
  "sku": "DELL-INS-3520-I5-16-512-SLV",
  "warehouse_id": "wh_hn_001",
  "type": "adjust",
  "delta_quantity": 10,
  "before_on_hand": 2,
  "after_on_hand": 12,
  "before_reserved": 0,
  "after_reserved": 0,
  "reason": "Manual correction after stock count",
  "note": "Checked by warehouse team",
  "related_entity_type": null,
  "related_entity_id": null,
  "actor_id": "admin_001",
  "actor_name": "Admin A",
  "created_at": "2026-06-22T14:35:00+07:00"
}
```

## 17.3. Reserved order item

```json
{
  "order_id": "order_1024",
  "order_number": "DH1024",
  "order_status": "pending_confirmation",
  "payment_status": "bank_transfer_pending",
  "reserved_quantity": 1,
  "reserved_at": "2026-06-22T10:30:00+07:00"
}
```

## 17.4. Inventory list response

```json
{
  "items": [],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 128,
    "total_pages": 7
  },
  "summary": {
    "total_skus": 128,
    "in_stock_skus": 96,
    "low_stock_skus": 18,
    "out_of_stock_skus": 10,
    "oversold_skus": 4
  }
}
```

---

# 18. API contract

API chỉ là gợi ý, có thể đổi theo framework.

## 18.1. Inventory list

```http
GET /api/v1/admin/inventory
```

Query params:

```text
search
warehouse_id
category_id
brand_id
stock_status
track_inventory
reserved_gt
available_min
available_max
updated_from
updated_to
page
page_size
sort
```

## 18.2. Inventory detail

```http
GET /api/v1/admin/inventory/{inventory_item_id}
```

## 18.3. Stock movements

```http
GET /api/v1/admin/inventory/movements
GET /api/v1/admin/inventory/{inventory_item_id}/movements
```

Query params:

```text
sku
warehouse_id
type
actor_id
related_entity_type
related_entity_id
date_from
date_to
page
page_size
```

## 18.4. Adjust stock

```http
POST /api/v1/admin/inventory/{inventory_item_id}/adjust
```

Request:

```json
{
  "mode": "increase",
  "quantity": 10,
  "reason": "Stock count correction",
  "reference_code": "COUNT-2026-06-22",
  "note": "Checked by warehouse team",
  "idempotency_key": "adjust_inv_001_20260622_001"
}
```

Response:

```json
{
  "inventory_item": {},
  "stock_movement": {}
}
```

## 18.5. Reserve stock

```http
POST /api/v1/admin/inventory/reserve
```

Normally called by Order Service, not manually from UI.

Request:

```json
{
  "order_id": "order_1024",
  "items": [
    {
      "inventory_item_id": "inv_001",
      "quantity": 1
    }
  ],
  "idempotency_key": "reserve_order_1024"
}
```

## 18.6. Release stock

```http
POST /api/v1/admin/inventory/release
```

Request:

```json
{
  "order_id": "order_1024",
  "reason": "Order cancelled",
  "idempotency_key": "release_order_1024"
}
```

## 18.7. Import stock

```http
POST /api/v1/admin/inventory/import/preview
POST /api/v1/admin/inventory/import/confirm
GET  /api/v1/admin/inventory/import/{batch_id}
```

## 18.8. Export stock

```http
GET /api/v1/admin/inventory/export
```

Query params giống list hiện tại.

## 18.9. Update threshold

```http
PATCH /api/v1/admin/inventory/{inventory_item_id}/threshold
```

Request:

```json
{
  "low_stock_threshold": 5,
  "safety_stock_quantity": 1
}
```

---

# 19. Validation rules

## 19.1. Quantity validation

```text
Quantity phải là số nguyên.
Quantity không được âm trong input.
Decrease không được vượt on_hand nếu không có quyền oversell.
Available không được âm nếu allow_backorder=false.
Reserved không được lớn hơn on_hand nếu không cho oversold.
```

## 19.2. SKU validation

```text
SKU phải tồn tại.
SKU phải unique ở cấp variant.
Inventory item phải khớp product/variant/warehouse.
```

## 19.3. Movement validation

```text
Type required.
Reason required với manual adjustment.
Related order required với reserve/release từ order.
Actor required.
Before/after snapshot required.
```

## 19.4. Import validation

```text
File type hợp lệ.
File size trong giới hạn.
Required columns đầy đủ.
Không có row quantity invalid.
Không có SKU không tồn tại.
Không có warehouse không tồn tại.
```

---

# 20. Permissions

Role tham khảo:

```text
Super Admin
Store Manager
Inventory Manager
Warehouse Staff
Product Manager
Support Staff
Viewer
```

Permission matrix:

| Permission | Super | Store Manager | Inventory Manager | Warehouse | Product | Support | Viewer |
|---|---|---|---|---|---|---|---|
| inventory.view | yes | yes | yes | yes | yes | yes | yes |
| inventory.adjust | yes | yes | yes | yes | no | no | no |
| inventory.import | yes | yes | yes | yes | no | no | no |
| inventory.export | yes | yes | yes | yes | yes | no | yes |
| inventory.transfer | yes | yes | yes | yes | no | no | no |
| inventory.threshold.update | yes | yes | yes | no | yes | no | no |
| inventory.movement.view | yes | yes | yes | yes | yes | yes | yes |
| inventory.cost.view | yes | yes | yes | no | no | no | no |
| inventory.oversell.approve | yes | yes | no | no | no | no | no |

Rule:

```text
Frontend ẩn action nếu không có quyền.
Backend vẫn phải validate permission.
Không leak cost/inventory value cho role không có quyền.
```

---

# 21. Loading, empty, error states

## 21.1. Loading

Inventory list loading:

```text
KPI skeleton
Toolbar visible
Table row skeleton
Không dùng fake numbers
```

Adjustment loading:

```text
Button loading
Disable submit
Không disable close nếu chưa submit thành công, trừ khi request đang chạy
```

Import loading:

```text
Upload progress
Preview loading
Confirm import progress
```

## 21.2. Empty states

No inventory:

```text
No inventory records yet.
Create products and enable inventory tracking to start managing stock.
```

No low stock:

```text
All tracked SKUs are above low stock threshold.
```

No movements:

```text
No stock movements yet.
Movements will appear when stock is imported, adjusted, reserved, or released.
```

## 21.3. Error states

Common errors:

```text
Cannot load inventory.
Cannot adjust stock.
Quantity is invalid.
SKU not found.
Warehouse not found.
Permission denied.
Import file has invalid rows.
Stock changed while submitting. Please reload and try again.
```

Rule:

```text
Error phải có hành động tiếp theo.
Nếu lỗi network, có Retry.
Nếu lỗi validation, giữ input.
Nếu stock conflict, yêu cầu refresh data.
```

---

# 22. Concurrency and data consistency

Inventory dễ gặp race condition.

Ví dụ:

```text
Hai khách checkout cùng SKU cùng lúc.
Admin giảm tồn kho đúng lúc order đang reserve.
Import stock chạy cùng manual adjust.
```

Backend cần:

```text
Transaction.
Row-level lock hoặc optimistic locking.
Version field.
Idempotency key cho reserve/release/adjust.
Conflict response rõ.
```

UI cần xử lý:

```text
Nếu version conflict, show message:
Stock was updated by another action. Please reload latest data.
```

Data model nên có:

```text
version
updated_at
last_movement_id
```

---

# 23. Security and audit

## 23.1. Security

```text
Auth required cho toàn bộ admin inventory.
Permission required từng action.
Không trust quantity từ frontend.
Validate file upload.
Rate limit import nếu cần.
Không expose cost nếu role không có quyền.
```

## 23.2. Audit

Ghi log cho:

```text
Manual adjustment
Import stock
Export stock
Update threshold
Transfer stock
Enable/disable tracking
Allow/disallow backorder
Reserve/release error
```

Audit fields:

```text
actor_id
actor_role
action
target_type
target_id
before_snapshot
after_snapshot
ip_address optional
user_agent optional
created_at
```

---

# 24. Accessibility rules

Bắt buộc:

```text
Table có header semantic.
Input có label.
Quantity field có error text.
Modal trap focus.
Icon-only button có aria-label.
Badge không chỉ dựa vào màu.
Toast không phải nơi duy nhất báo lỗi validation.
Keyboard dùng được cho filter, modal, table action.
```

Adjustment modal:

```text
Focus vào field quantity khi mở.
Sau submit lỗi, focus vào error summary.
Sau đóng modal, focus về button Adjust stock.
```

---

# 25. Responsive rules

Desktop:

```text
Sidebar visible.
KPI 4 columns.
Inventory table đầy đủ.
Toolbar multi-row nếu cần.
```

Tablet:

```text
Sidebar collapsed.
KPI 2 columns.
Table horizontal scroll hoặc ẩn cột phụ.
```

Mobile:

```text
Sidebar drawer.
KPI 1 column.
Inventory list thành cards.
Filter thành drawer.
Import/adjust modal full width.
Không overflow ngang.
```

Mobile inventory card hiển thị:

```text
Product name
Variant
SKU
Warehouse
Available / Reserved / On hand
Status badge
Actions
```

---

# 26. Component structure

Tên component gợi ý:

```text
AdminInventoryPage
InventoryOverviewCards
InventoryToolbar
InventoryAdvancedFilterDrawer
InventoryTable
InventoryTableRow
InventoryMobileCard
InventoryStatusBadge
InventoryQuantityCell
InventoryDetailPage
InventorySummaryCard
ReservedOrdersTable
StockMovementTable
StockMovementDetailDrawer
StockAdjustmentModal
StockImportPage
StockImportUploader
StockImportPreviewTable
StockImportResultSummary
StockExportMenu
LowStockPage
LowStockRecommendationTable
ThresholdEditModal
WarehouseSelect
```

Shared components dùng lại:

```text
AdminLayout
AdminSidebar
AdminTopbar
DataTable
TableToolbar
Badge
Button
Input
Select
Modal
Drawer
Toast
Pagination
Skeleton
EmptyState
ErrorState
ConfirmDialog
```

---

# 27. Suggested file structure

Không phụ thuộc framework, nhưng có thể map như sau:

```text
src/
  admin/
    inventory/
      pages/
        AdminInventoryPage.*
        InventoryDetailPage.*
        StockMovementsPage.*
        StockImportPage.*
        LowStockPage.*

      components/
        InventoryOverviewCards.*
        InventoryToolbar.*
        InventoryTable.*
        InventoryMobileCard.*
        InventoryStatusBadge.*
        StockAdjustmentModal.*
        StockMovementTable.*
        ReservedOrdersTable.*
        StockImportPreviewTable.*
        ThresholdEditModal.*

      api/
        inventoryApi.*

      types/
        inventoryTypes.*

      utils/
        inventoryFormatters.*
        inventoryStatus.*
        inventoryValidation.*
        stockMovementHelpers.*

tests/
  e2e/
    admin-inventory.spec.*
  visual/
    admin-inventory.visual.spec.*
```

---

# 28. Analytics events

Admin analytics optional, nhưng nên có để hiểu vận hành.

Events:

```text
admin_inventory_viewed
admin_inventory_search_used
admin_inventory_filter_applied
admin_inventory_low_stock_clicked
admin_inventory_adjust_opened
admin_inventory_adjust_submitted
admin_inventory_adjust_failed
admin_inventory_import_started
admin_inventory_import_previewed
admin_inventory_import_confirmed
admin_inventory_export_clicked
admin_inventory_movement_viewed
admin_inventory_threshold_updated
```

Không gửi dữ liệu nhạy cảm nếu không cần.

---

# 29. Playwright test specification

## 29.1. Inventory list tests

Test cases:

```text
Admin can view inventory list.
Admin can search by SKU.
Admin can search by product name.
Admin can filter by stock status.
Admin can filter by warehouse.
Admin can open inventory detail.
Admin can view low stock list.
Inventory table shows available/reserved/on hand correctly.
Empty state appears when no inventory exists.
Error state appears when API fails.
Mobile inventory page has no horizontal overflow.
```

## 29.2. Stock adjustment tests

Test cases:

```text
Admin can open stock adjustment modal.
Quantity is required.
Reason is required.
Admin can increase stock.
Admin can decrease stock.
Admin can set exact quantity.
Adjustment creates movement row.
Adjustment updates available/on hand quantity.
Invalid decrease shows validation error.
Permission denied user cannot see adjustment action.
```

## 29.3. Movement tests

Test cases:

```text
Admin can view stock movements.
Admin can filter movements by type.
Admin can filter movements by SKU.
Admin can open movement detail drawer.
Movement detail is readonly.
```

## 29.4. Import tests

Test cases:

```text
Admin can open import page.
Invalid file type shows error.
Missing required columns shows error.
Valid file shows preview.
Rows with invalid SKU show row error.
Admin can confirm valid import.
Import result summary appears.
```

## 29.5. Reserve/release integration tests

Test cases:

```text
When order is created, stock is reserved.
When order is cancelled, stock is released.
When payment fails, reserved stock is released.
When stock is insufficient, order creation is blocked or marked as stock issue.
```

## 29.6. Permission tests

Test cases:

```text
Viewer can view inventory but cannot adjust.
Warehouse staff can adjust stock.
Product manager can update threshold but cannot adjust stock if not allowed.
Support can view reserved orders but cannot change stock.
Unauthorized user is redirected or blocked.
```

---

# 30. Visual regression checklist

Capture screenshots for:

```text
Inventory overview desktop.
Inventory overview mobile.
Inventory table with normal data.
Inventory table with low stock and out of stock.
Inventory detail page.
Stock adjustment modal.
Adjustment validation error.
Stock movement table.
Movement detail drawer.
Import preview page.
Import result summary.
Empty state.
Error state.
Permission-limited state.
```

Viewports:

```text
1440px desktop
1024px laptop
768px tablet
375px mobile
320px small mobile
```

Không approve screenshot nếu:

```text
Table overflow toàn page.
Mobile card bị cắt text nghiêm trọng.
Badge màu sai semantic.
Quantity không đọc được.
Modal vượt màn hình.
Action nguy hiểm không rõ.
```

---

# 31. Definition of Done

Module Admin Inventory Management được coi là xong khi:

## 31.1. UI

```text
Inventory overview hiển thị đúng.
Inventory list hoạt động.
Search/filter/sort/pagination hoạt động.
Inventory detail hoạt động.
Movement history hiển thị đúng.
Adjustment modal có validation.
Import preview có validation row-level.
Loading/empty/error states đầy đủ.
Responsive desktop/tablet/mobile không vỡ.
```

## 31.2. Function

```text
Tồn kho theo variant/SKU.
Adjust stock tạo StockMovement.
Reserve/release liên kết order.
Threshold low stock hoạt động.
Permission hoạt động.
Export/import hoạt động hoặc mock rõ ràng.
Không có action nguy hiểm thiếu confirm.
```

## 31.3. Data

```text
Quantity format đúng.
Status tính đúng.
Available/reserved/on hand không mâu thuẫn.
Movement có before/after snapshot.
Không mất lịch sử movement.
Không leak cost cho role không có quyền.
```

## 31.4. Test

```text
Playwright main flow pass.
Visual screenshot desktop/mobile đã kiểm tra.
Không có console error nghiêm trọng.
Không có horizontal overflow.
```

---

# 32. MVP scope

Nếu cần làm MVP trước, chỉ cần:

```text
Inventory list.
Search by SKU/product name.
Filter by stock status.
Filter by warehouse nếu có.
Available/reserved/on hand display.
Low stock badge.
Out of stock badge.
Inventory detail basic.
Movement history basic.
Manual stock adjustment.
StockMovement log.
Import CSV basic preview.
Permission basic.
Responsive mobile card.
```

Chưa cần ngay:

```text
Multi-warehouse transfer.
Serial/IMEI tracking.
Advanced reorder suggestion.
Inventory value/cost permission.
Cycle count workflow.
Supplier purchase request.
Realtime warehouse sync.
Barcode scanner.
AI demand forecast.
```

---

# 33. Future extension

Sau MVP có thể mở rộng:

```text
Serial/IMEI level inventory.
Barcode scanning.
Purchase order module.
Supplier management.
Stock transfer between warehouses.
Cycle count / stocktake.
Inventory forecast.
Dead stock report.
Inventory valuation.
Bundle inventory.
Multi-channel inventory sync.
Approval workflow for high-value adjustment.
```

---

# 34. Ghi chú cho source clone nhiều ngành hàng

Phần lõi dùng chung cho mọi ngành:

```text
InventoryItem
StockMovement
Warehouse
Reserved stock
Available stock
Low stock threshold
Import/export
Adjustment
Audit log
Permission
```

Phần riêng ngành điện tử:

```text
Variant-level stock rất quan trọng.
Serial/IMEI future extension.
Warranty linkage.
High-value product adjustment approval.
Specs/variant label rõ ràng trong inventory list.
```

Khi clone sang ngành khác:

```text
Thời trang: inventory theo size/color.
Mỹ phẩm: inventory theo batch/expiry date.
Thực phẩm: inventory theo expiry/cold storage.
Nội thất: inventory theo warehouse/bulky item.
Sản phẩm số: inventory có thể not_tracked hoặc license key.
```

Không sửa core inventory model nếu không cần. Chỉ mở rộng attribute/variant/batch-specific logic.

---

# 35. Tóm tắt cho agent

Nếu agent chỉ đọc phần này, hãy nhớ:

```text
Inventory phải theo SKU/variant.
Không sửa stock trực tiếp nếu không tạo StockMovement.
Available = on hand - reserved - damaged - safety stock.
Reserve/release phải đồng bộ với order.
Low stock phải rõ ràng và có action.
Admin UI phải gọn, ít màu, dễ scan.
Mobile không được overflow.
Adjustment cần reason và confirm.
Import cần preview và row-level validation.
Permission không chỉ xử lý ở frontend.
Playwright phải test search, filter, adjust, movement, import, mobile.
```

Prompt tiếp theo nên tạo:

```text
14-admin-promotion-management.md
```
