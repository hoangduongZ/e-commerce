# 09-admin-dashboard.md

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Spec chi tiết cho **Admin Dashboard** của web bán hàng đồ điện tử.  
> File này kế thừa từ:
>
> - `../main/ecommerce_design_language.md`
> - `01-electronics-store-theme.md`
>
> Mục tiêu: agent/frontend có thể dựa vào file này để code dashboard quản trị từ đầu đến cuối, không phụ thuộc framework.

---

# 1. Vai trò của Admin Dashboard

Admin Dashboard là màn hình đầu tiên sau khi nhân sự quản trị đăng nhập vào hệ thống.

Trang này không phải chỉ để “trang trí bằng số liệu”. Nó là **trung tâm vận hành** của shop.

Admin Dashboard cần trả lời nhanh các câu hỏi sau:

- Hôm nay shop bán được bao nhiêu?
- Có bao nhiêu đơn cần xử lý ngay?
- Có đơn nào thanh toán lỗi hoặc giao hàng lỗi không?
- Sản phẩm nào sắp hết hàng?
- Sản phẩm nào bán chạy?
- Doanh thu đang tăng hay giảm?
- Có cảnh báo vận hành nào cần xử lý không?
- Nhân sự admin nên bấm vào đâu tiếp theo?

Với shop đồ điện tử, dashboard càng quan trọng vì sản phẩm thường có giá trị cao, nhiều thông số, bảo hành, tồn kho và đơn hàng cần xác nhận kỹ.

---

# 2. Nguyên tắc thiết kế riêng cho Admin Dashboard

## 2.1. Ưu tiên hiệu quả vận hành

Dashboard admin không cần quá “marketing”. Nó cần rõ ràng, nhanh, chính xác.

Ưu tiên:

- Dữ liệu dễ scan.
- KPI nổi bật.
- Cảnh báo rõ.
- Nút hành động gần dữ liệu liên quan.
- Bảng dễ đọc.
- Filter nhanh.
- Không dùng animation gây mất tập trung.

## 2.2. Admin khác storefront

Storefront hướng tới khách mua hàng.

Admin hướng tới người vận hành.

Vì vậy:

```text
Storefront = đẹp, thuyết phục, mua hàng nhanh
Admin      = rõ, chính xác, xử lý nhanh
```

Admin Dashboard nên dùng layout chắc chắn, ít màu, nhiều khoảng trắng, dữ liệu có thứ tự.

## 2.3. Không nhồi quá nhiều dữ liệu

Dashboard chỉ hiển thị dữ liệu cần hành động ngay.

Dữ liệu phân tích sâu nên đưa sang trang report riêng.

Dashboard nên tập trung vào:

- Tổng quan hôm nay.
- Đơn cần xử lý.
- Tồn kho cần chú ý.
- Cảnh báo hệ thống.
- Xu hướng ngắn hạn.

---

# 3. User roles và quyền truy cập

## 3.1. Role chính

| Role | Mục đích |
|---|---|
| Super Admin | Toàn quyền |
| Store Manager | Quản lý vận hành |
| Sales Staff | Xử lý đơn hàng |
| Warehouse Staff | Xử lý tồn kho |
| Support Staff | Hỗ trợ khách hàng |
| Marketing Staff | Khuyến mãi và nội dung |
| Analyst | Xem báo cáo |

## 3.2. Permission gợi ý

| Permission | Mục đích |
|---|---|
| dashboard.view | Xem dashboard |
| dashboard.revenue.view | Xem doanh thu |
| dashboard.order.view | Xem KPI đơn hàng |
| dashboard.inventory.view | Xem cảnh báo tồn kho |
| dashboard.customer.view | Xem dữ liệu khách hàng |
| order.view | Xem đơn hàng |
| order.update | Cập nhật đơn hàng |
| product.view | Xem sản phẩm |
| inventory.update | Cập nhật tồn kho |
| report.view | Xem báo cáo |

## 3.3. Rule hiển thị theo role

Không phải admin nào cũng thấy cùng một dashboard.

Ví dụ:

- Sales Staff thấy đơn cần gọi xác nhận, đơn chờ xử lý, đơn thanh toán lỗi.
- Warehouse Staff thấy tồn kho thấp, đơn cần đóng gói, sản phẩm cần nhập thêm.
- Marketing Staff thấy doanh thu theo campaign, sản phẩm bán chạy, coupon usage.
- Store Manager thấy toàn bộ KPI vận hành.

Agent không được hard-code toàn bộ widget cho mọi role. Cần thiết kế dashboard widget có thể bật/tắt theo permission.

---

# 4. Layout tổng thể

## 4.1. Desktop layout

Desktop là layout chính cho admin.

Kích thước ưu tiên:

```text
1440px desktop
1366px laptop
1280px small desktop
```

Cấu trúc:

```text
┌─────────────────────────────────────────────────────────────┐
│ Top Bar                                                     │
├───────────────┬─────────────────────────────────────────────┤
│ Sidebar       │ Page Content                                │
│ Navigation    │                                             │
│               │ KPI Cards                                   │
│               │ Charts                                      │
│               │ Action Tables                               │
│               │ Alerts                                      │
└───────────────┴─────────────────────────────────────────────┘
```

## 4.2. Tablet layout

Tablet có thể thu gọn sidebar.

```text
Sidebar collapsed
Top Bar full width
Content 1 hoặc 2 cột
Table có horizontal scroll nếu cần
```

## 4.3. Mobile layout

Admin mobile không phải trải nghiệm chính, nhưng vẫn cần usable.

Rule:

- Sidebar chuyển thành drawer.
- KPI card hiển thị 1 cột.
- Chart giảm chiều cao.
- Table chuyển thành card list nếu quá hẹp.
- Action button phải đủ lớn để bấm.
- Không overflow ngang toàn page.

---

# 5. Admin shell

Admin Dashboard nằm trong Admin Shell chung.

## 5.1. Admin Shell gồm

```text
AdminLayout
├── AdminSidebar
├── AdminTopBar
├── AdminBreadcrumb
├── AdminPageHeader
├── AdminContent
└── AdminToastContainer
```

## 5.2. Sidebar

Sidebar là menu điều hướng chính.

### Menu gợi ý

```text
Dashboard
Orders
Products
Categories
Brands
Attributes
Inventory
Customers
Promotions
Reviews
Warranty
Shipping
Reports
Settings
Staff & Roles
```

### Sidebar item states

Mỗi item có trạng thái:

- default
- hover
- active
- disabled
- has notification badge
- collapsed

### Sidebar rule

- Active item phải rõ.
- Badge số lượng chỉ dùng khi có việc cần xử lý.
- Không dùng quá nhiều màu trong sidebar.
- Menu bị thiếu quyền thì ẩn, không disable nếu user không có quyền.

## 5.3. Top Bar

Top Bar chứa:

- Global search.
- Notification bell.
- Quick create button.
- Store switcher nếu có nhiều shop.
- Date range selector nếu dashboard cần.
- Admin profile menu.

### Global search

Global search cho phép tìm nhanh:

```text
Order number
Product SKU
Product name
Customer phone
Customer email
Tracking code
```

Kết quả search nên nhóm theo loại.

```text
Orders
Products
Customers
```

## 5.4. Breadcrumb

Dashboard có thể không cần breadcrumb nếu là root page.

Các trang con cần breadcrumb.

Ví dụ:

```text
Admin / Products / Create Product
```

---

# 6. Dashboard page header

## 6.1. Nội dung

Page header gồm:

- Page title: `Dashboard`
- Subtitle: mô tả ngắn tình trạng hiện tại.
- Date range selector.
- Refresh button.
- Export button nếu role có quyền.

Ví dụ:

```text
Dashboard
Tổng quan hoạt động bán hàng và vận hành hôm nay.
[Today] [Last 7 days] [Last 30 days] [Custom range] [Refresh]
```

## 6.2. Date range

Các preset cần có:

```text
Today
Yesterday
Last 7 days
Last 30 days
This month
Last month
Custom
```

## 6.3. Refresh behavior

Refresh button:

- Reload toàn bộ dashboard widgets.
- Không làm mất date range hiện tại.
- Có loading state riêng.
- Nếu một widget fail, không làm fail toàn trang.

---

# 7. Dashboard widgets tổng quan

Dashboard được ghép từ nhiều widget độc lập.

Mỗi widget cần có:

- title
- description hoặc tooltip
- data state
- loading state
- empty state
- error state
- permission rule
- refresh behavior
- link tới trang chi tiết

## 7.1. Widget priority

Dashboard không nên hiển thị widget tùy tiện.

Thứ tự ưu tiên:

1. Cảnh báo cần xử lý ngay.
2. KPI doanh thu và đơn hàng.
3. Đơn cần xử lý.
4. Tồn kho thấp.
5. Biểu đồ xu hướng.
6. Sản phẩm bán chạy.
7. Khách hàng mới.
8. Hoạt động gần đây.

---

# 8. KPI cards

## 8.1. KPI card bắt buộc

Các KPI card cơ bản:

```text
Revenue
Orders
Average Order Value
Conversion Rate
Pending Orders
Low Stock Products
```

Với MVP, nếu chưa có tracking conversion thì có thể bỏ `Conversion Rate`.

## 8.2. KPI card structure

Mỗi KPI card gồm:

```text
Icon
Label
Main value
Comparison value
Trend indicator
Short helper text
Click target
```

Ví dụ:

```text
Revenue
₫128,500,000
+12.4% vs yesterday
View report
```

## 8.3. KPI card visual rule

- Main value lớn nhất trong card.
- Label nhỏ hơn và rõ nghĩa.
- Trend màu xanh nếu tăng tốt.
- Trend màu đỏ nếu giảm xấu.
- Với một số KPI, tăng không phải lúc nào cũng tốt.

Ví dụ:

```text
Revenue tăng     = tốt
Orders tăng      = tốt
Refunds tăng     = xấu
Cancelled tăng   = xấu
Low stock tăng   = xấu
```

Agent phải biết KPI nào là positive metric, KPI nào là negative metric.

## 8.4. KPI card states

### Loading

Dùng skeleton.

Không hiển thị số giả như `0` khi data đang loading.

### Empty

Hiển thị:

```text
No data yet
```

Kèm helper text.

### Error

Hiển thị:

```text
Could not load this metric
[Retry]
```

Không làm sập toàn dashboard.

---

# 9. Revenue chart

## 9.1. Mục đích

Revenue chart cho admin biết doanh thu đang tăng hay giảm theo thời gian.

## 9.2. Chart type

Ưu tiên:

```text
Line chart
Bar chart
```

Không dùng pie chart cho doanh thu theo thời gian.

## 9.3. Data granularity

Theo date range:

| Date range | Granularity |
|---|---|
| Today | Hourly |
| Last 7 days | Daily |
| Last 30 days | Daily |
| This month | Daily |
| Last month | Daily |
| Custom long range | Weekly / Monthly |

## 9.4. Chart controls

Chart nên có toggle:

```text
Revenue
Orders
AOV
```

MVP chỉ cần `Revenue` và `Orders`.

## 9.5. Tooltip

Tooltip cần hiển thị:

```text
Date/time
Revenue
Orders
AOV nếu có
```

Ví dụ:

```text
22 Jun 2026
Revenue: ₫18,500,000
Orders: 32
AOV: ₫578,125
```

## 9.6. Chart accessibility

Chart không được là nguồn thông tin duy nhất.

Phía dưới hoặc trong summary cần có số tổng.

Screen reader cần có mô tả:

```text
Revenue increased by 12.4% compared with previous period.
```

---

# 10. Order status overview

## 10.1. Mục đích

Hiển thị số lượng đơn theo trạng thái để admin biết bottleneck nằm ở đâu.

## 10.2. Status gợi ý

```text
Pending confirmation
Confirmed
Packing
Shipping
Delivered
Cancelled
Returned
Payment failed
```

## 10.3. Layout

Có thể dùng card grid hoặc horizontal status bar.

Mỗi status hiển thị:

```text
Status label
Count
Warning badge nếu cần
Click to filtered orders page
```

Ví dụ:

```text
Pending confirmation: 18
Payment failed: 3
Packing: 9
Shipping: 42
```

## 10.4. Color rule

| Status | Tone |
|---|---|
| Pending | warning |
| Confirmed | info |
| Packing | info |
| Shipping | primary |
| Delivered | success |
| Cancelled | neutral/danger |
| Returned | warning |
| Payment failed | danger |

Không dùng quá nhiều màu bão hòa. Trạng thái cần cảnh báo mới dùng màu mạnh.

---

# 11. Orders requiring action

## 11.1. Mục đích

Đây là widget quan trọng nhất cho vận hành.

Nó hiển thị các đơn cần admin xử lý ngay.

Ví dụ:

```text
Đơn mới chưa xác nhận
Đơn thanh toán lỗi
Đơn đang chờ chuyển khoản
Đơn giao hàng bị lỗi
Đơn khách yêu cầu hủy
Đơn hoàn trả
```

## 11.2. Table columns

| Column | Nội dung |
|---|---|
| Order | Mã đơn |
| Customer | Tên / SĐT |
| Amount | Tổng tiền |
| Payment | Trạng thái thanh toán |
| Fulfillment | Trạng thái xử lý |
| Age | Thời gian chờ |
| Action | Hành động |

Không đưa câu dài vào bảng.

## 11.3. Row action

Row action gợi ý:

```text
View
Confirm
Call customer
Mark paid
Cancel
```

Không hiển thị quá nhiều action cùng lúc. Action phụ đưa vào menu `More`.

## 11.4. Row warning

Các trường hợp cần highlight:

- Chờ xác nhận quá SLA.
- Chờ chuyển khoản quá lâu.
- Thanh toán failed.
- Địa chỉ thiếu thông tin.
- Sản phẩm trong đơn hết hàng.

## 11.5. SLA gợi ý

| Case | SLA |
|---|---|
| New order | 30 phút |
| Bank transfer pending | 24 giờ |
| Packing | 4 giờ |
| Shipping issue | ngay |
| Cancel request | 2 giờ |

SLA nên configurable từ admin settings.

---

# 12. Low stock widget

## 12.1. Mục đích

Shop đồ điện tử cần kiểm soát tồn kho chặt vì sản phẩm giá trị cao.

Widget này giúp admin biết sản phẩm cần nhập thêm.

## 12.2. Table columns

| Column | Nội dung |
|---|---|
| Product | Tên / SKU |
| Category | Danh mục |
| Available | Có thể bán |
| Reserved | Đang giữ |
| Threshold | Ngưỡng cảnh báo |
| Action | Hành động |

## 12.3. Electronics-specific rule

Với đồ điện tử, tồn kho nên xét theo variant.

Ví dụ:

```text
iPhone 15 Pro / 256GB / Natural Titanium
Laptop A / 16GB RAM / 512GB SSD
```

Không chỉ cảnh báo theo product cha.

## 12.4. Action

Action gợi ý:

```text
View product
Adjust stock
Create purchase request
Disable selling
```

MVP có thể chỉ cần:

```text
View product
Adjust stock
```

---

# 13. Best selling products widget

## 13.1. Mục đích

Cho admin biết sản phẩm nào đang bán tốt.

Dùng để:

- Ưu tiên nhập hàng.
- Tạo campaign.
- Đưa lên homepage.
- Gợi ý phụ kiện liên quan.

## 13.2. Metrics

Có thể sort theo:

```text
Revenue
Units sold
Gross profit
View-to-cart rate
Cart-to-order rate
```

MVP chỉ cần:

```text
Units sold
Revenue
```

## 13.3. Product row

Mỗi row hiển thị:

```text
Product image
Product name
SKU
Units sold
Revenue
Stock left
```

Nếu stock còn thấp nhưng sản phẩm bán chạy, hiển thị warning.

---

# 14. Inventory health summary

## 14.1. Mục đích

Cho admin nhìn nhanh tình trạng kho.

## 14.2. Các chỉ số

```text
Products in stock
Low stock products
Out of stock products
Reserved quantity
Dead stock products
```

## 14.3. Dead stock

Dead stock là sản phẩm tồn kho nhưng không bán được trong một khoảng thời gian.

Ví dụ:

```text
No sales in 60 days
```

Rule này nên configurable.

---

# 15. Payment summary

## 15.1. Mục đích

Theo dõi tình trạng thanh toán.

## 15.2. Metrics

```text
Paid orders
COD orders
Bank transfer pending
Payment failed
Refund pending
Refunded
```

## 15.3. Electronics-specific note

Với đơn giá trị cao, đơn chuyển khoản và trả góp cần được hiển thị rõ.

Nếu hệ thống chưa tích hợp trả góp, vẫn nên để chỗ mở rộng.

---

# 16. Warranty and service widget

## 16.1. Mục đích

Đồ điện tử thường gắn với bảo hành.

Dashboard nên hiển thị cảnh báo liên quan bảo hành nếu module đã có.

## 16.2. Metrics

```text
Warranty requests pending
Service tickets open
Replacement requests
Return requests
```

## 16.3. MVP scope

MVP có thể chỉ hiển thị:

```text
Return requests
```

Warranty module có thể làm sau.

---

# 17. Customer summary

## 17.1. Mục đích

Theo dõi khách hàng mới, khách quay lại và khách có giá trị cao.

## 17.2. Metrics

```text
New customers
Returning customers
Top customers
Abandoned carts
```

## 17.3. Privacy rule

Không hiển thị quá nhiều thông tin cá nhân trên dashboard.

Ví dụ, nên hiển thị:

```text
Nguyễn V. A
090****000
```

Không hiển thị đầy đủ nếu không cần.

---

# 18. Activity feed

## 18.1. Mục đích

Hiển thị hoạt động vận hành gần đây.

Ví dụ:

```text
Order #DH1024 was confirmed by Minh.
Product SKU LAP-DELL-001 stock adjusted from 3 to 10.
Coupon SUMMER10 was created by Admin.
```

## 18.2. Feed item structure

```text
Icon
Action summary
Actor
Timestamp
Link to detail
```

## 18.3. Rule

- Không hiển thị dữ liệu nhạy cảm.
- Không dùng feed thay thế audit log chính thức.
- Feed chỉ là tóm tắt.

---

# 19. Alert center

## 19.1. Mục đích

Alert center gom các cảnh báo cần xử lý.

Ví dụ:

```text
3 payment failures today
12 products are low stock
5 orders exceed confirmation SLA
2 shipping issues reported
```

## 19.2. Alert severity

| Severity | Mục đích |
|---|---|
| Info | Thông tin |
| Warning | Cần chú ý |
| Danger | Cần xử lý ngay |
| Success | Hoàn tất |

## 19.3. Alert behavior

- Có thể dismiss alert nếu là thông báo nhẹ.
- Alert nghiêm trọng không được dismiss nếu chưa xử lý.
- Alert phải link tới trang chi tiết.

---

# 20. Data contract

## 20.1. Dashboard response shape

API có thể trả một payload tổng hợp.

Ví dụ:

```json
{
  "dateRange": {
    "from": "2026-06-22T00:00:00+07:00",
    "to": "2026-06-22T23:59:59+07:00",
    "preset": "today"
  },
  "kpis": {
    "revenue": {
      "value": 128500000,
      "currency": "VND",
      "comparisonPercent": 12.4,
      "direction": "up"
    },
    "orders": {
      "value": 156,
      "comparisonPercent": 8.1,
      "direction": "up"
    },
    "averageOrderValue": {
      "value": 823717,
      "currency": "VND",
      "comparisonPercent": -2.3,
      "direction": "down"
    },
    "pendingOrders": {
      "value": 18,
      "severity": "warning"
    },
    "lowStockProducts": {
      "value": 12,
      "severity": "danger"
    }
  },
  "revenueChart": [
    {
      "label": "08:00",
      "revenue": 12000000,
      "orders": 14
    }
  ],
  "orderStatus": {
    "pending_confirmation": 18,
    "confirmed": 24,
    "packing": 9,
    "shipping": 42,
    "delivered": 63,
    "cancelled": 4,
    "payment_failed": 3
  },
  "ordersRequiringAction": [],
  "lowStockProducts": [],
  "bestSellingProducts": [],
  "alerts": []
}
```

## 20.2. Order requiring action item

```json
{
  "id": "order_1024",
  "orderNumber": "DH1024",
  "customerName": "Nguyễn Văn A",
  "customerPhoneMasked": "090****000",
  "amount": 18500000,
  "currency": "VND",
  "paymentStatus": "bank_transfer_pending",
  "fulfillmentStatus": "pending_confirmation",
  "ageMinutes": 42,
  "slaExceeded": true,
  "actions": ["view", "confirm", "mark_paid"]
}
```

## 20.3. Low stock product item

```json
{
  "productId": "product_1",
  "variantId": "variant_1",
  "sku": "LAP-DELL-INS-16-512",
  "name": "Dell Inspiron 15",
  "variantLabel": "16GB RAM / SSD 512GB",
  "categoryName": "Laptop",
  "availableQuantity": 2,
  "reservedQuantity": 1,
  "threshold": 5,
  "imageUrl": "/images/products/dell-inspiron.jpg"
}
```

## 20.4. Alert item

```json
{
  "id": "alert_1",
  "severity": "danger",
  "title": "12 products are low stock",
  "message": "Some best-selling laptop variants are below threshold.",
  "targetUrl": "/admin/inventory?status=low_stock",
  "dismissible": false,
  "createdAt": "2026-06-22T09:30:00+07:00"
}
```

---

# 21. API contract gợi ý

## 21.1. Dashboard overview

```http
GET /api/v1/admin/dashboard?range=today
GET /api/v1/admin/dashboard?from=2026-06-01&to=2026-06-22
```

## 21.2. Widget-specific APIs

Có thể tách API theo widget để dashboard không bị fail toàn bộ.

```http
GET /api/v1/admin/dashboard/kpis?range=today
GET /api/v1/admin/dashboard/revenue-chart?range=last_7_days
GET /api/v1/admin/dashboard/order-status?range=today
GET /api/v1/admin/dashboard/orders-requiring-action
GET /api/v1/admin/dashboard/low-stock-products
GET /api/v1/admin/dashboard/best-selling-products?range=last_30_days
GET /api/v1/admin/dashboard/alerts
```

## 21.3. Recommended strategy

Với MVP, có thể dùng một API tổng hợp:

```http
GET /api/v1/admin/dashboard
```

Khi hệ thống lớn hơn, tách từng widget để:

- Load song song.
- Retry từng phần.
- Permission từng widget dễ hơn.
- Cache riêng từng loại dữ liệu.

---

# 22. Loading, empty, error states

## 22.1. Page loading

Khi vào dashboard lần đầu:

- Hiển thị skeleton cho KPI cards.
- Hiển thị skeleton chart.
- Hiển thị skeleton table rows.
- Không hiển thị spinner toàn màn hình quá lâu.

## 22.2. Widget loading

Mỗi widget có loading state riêng.

Nếu chart đang loading nhưng KPI đã có data, KPI vẫn hiển thị.

## 22.3. Empty state

Ví dụ dashboard shop mới chưa có đơn:

```text
No orders yet
Once customers place orders, they will appear here.
```

Nên có CTA:

```text
Add products
Configure homepage
Create promotion
```

## 22.4. Error state

Nếu một widget fail:

```text
Could not load revenue chart
[Retry]
```

Không được làm crash toàn dashboard.

## 22.5. Permission empty state

Nếu user không có quyền xem doanh thu:

```text
You do not have permission to view revenue data.
```

Hoặc ẩn widget tùy cấu hình.

---

# 23. Formatting rules

## 23.1. Currency

Tiền Việt Nam:

```text
₫18,500,000
18.500.000đ
```

Chọn một format và dùng nhất quán toàn hệ thống.

Khuyến nghị cho UI Việt Nam:

```text
18.500.000đ
```

Với code, nên format qua helper.

Không format thủ công ở nhiều component.

## 23.2. Number

Dùng separator rõ ràng:

```text
1,250
12,500
128,500
```

Hoặc theo locale Việt Nam:

```text
1.250
12.500
128.500
```

Phải thống nhất theo locale.

## 23.3. Date/time

Admin nên hiển thị rõ ngày giờ.

Ví dụ:

```text
22/06/2026 14:35
```

Với relative time:

```text
42 minutes ago
```

Có thể dùng cả hai trong tooltip.

## 23.4. Percent

```text
+12.4%
-2.3%
```

Không hiển thị quá nhiều chữ số thập phân.

---

# 24. Responsive rules

## 24.1. Desktop

```text
Sidebar: 240px
Content max width: fluid
Grid gap: 24px
KPI cards: 4 columns
Main chart: 2/3 width
Side widgets: 1/3 width
```

## 24.2. Laptop

```text
Sidebar: 220px
KPI cards: 3 columns
Tables remain table
```

## 24.3. Tablet

```text
Sidebar collapsed
KPI cards: 2 columns
Chart full width
Tables may scroll horizontally
```

## 24.4. Mobile

```text
Sidebar drawer
KPI cards: 1 column
Chart full width
Tables become cards where possible
Top Bar compact
```

## 24.5. No horizontal overflow

Toàn page không được overflow ngang.

Nếu table rộng, chỉ table container được scroll ngang.

---

# 25. Accessibility rules

Admin Dashboard phải accessible vì nhân sự có thể dùng nhiều giờ mỗi ngày.

## 25.1. Keyboard navigation

- Sidebar item focus được bằng keyboard.
- Dropdown, date picker, action menu sử dụng được bằng keyboard.
- Table row action có focus rõ.
- Modal không bị thoát focus.

## 25.2. Color contrast

- Text chính phải đủ tương phản.
- Badge trạng thái không chỉ dựa vào màu.
- Alert cần có icon và text.

## 25.3. Screen reader

- KPI card có `aria-label` mô tả đủ.
- Chart cần text summary.
- Table header phải đúng semantic.
- Button icon-only cần accessible name.

## 25.4. Motion

- Không dùng animation liên tục.
- Tôn trọng `prefers-reduced-motion`.

---

# 26. Performance rules

Admin Dashboard thường gọi nhiều dữ liệu.

## 26.1. Load strategy

Khuyến nghị:

1. Load shell trước.
2. Load KPI trước.
3. Load orders requiring action.
4. Load chart và widgets còn lại song song.

## 26.2. Cache

Có thể cache ngắn hạn:

```text
KPI: 30-60 seconds
Chart: 60-300 seconds
Low stock: 60 seconds
Alerts: 30 seconds
```

Dữ liệu critical như đơn cần xử lý nên refresh thường xuyên hơn.

## 26.3. Heavy chart

Không render chart quá nặng.

Nếu data points nhiều:

- Group data theo ngày/tuần.
- Limit range.
- Dùng lazy render.

## 26.4. Image

Product image trong admin dùng thumbnail.

Không load ảnh gốc.

---

# 27. Security rules

## 27.1. Auth guard

Dashboard chỉ truy cập được khi user đã đăng nhập admin.

Nếu chưa login:

```text
Redirect to /admin/login
```

## 27.2. Permission guard

Frontend chỉ ẩn UI để tăng UX.

Backend vẫn phải kiểm tra permission.

Không tin dữ liệu từ client.

## 27.3. Sensitive data

Dashboard không hiển thị đầy đủ:

- Số điện thoại.
- Email.
- Địa chỉ.
- Mã thanh toán.

Trừ khi role có quyền.

## 27.4. Audit

Các action từ dashboard như xác nhận đơn, mark paid, adjust stock phải ghi audit log.

Audit log cần có:

```text
actor
action
target
before_value
after_value
timestamp
ip/device nếu có
```

## 27.5. Dangerous action

Các action nguy hiểm cần confirm:

```text
Cancel order
Mark as paid
Adjust stock
Disable product
```

Không confirm bằng `window.confirm` nếu app có design system modal.

---

# 28. Admin Dashboard components

## 28.1. `AdminDashboardPage`

Container chính.

Nhiệm vụ:

- Quản lý date range.
- Gọi data dashboard.
- Render widget theo permission.
- Handle refresh.
- Handle partial error.

Không đặt quá nhiều logic formatting trong component này.

## 28.2. `DashboardKpiCard`

Props:

```text
title
value
valueType
comparisonPercent
trendDirection
metricPolarity
icon
loading
error
href
```

`metricPolarity`:

```text
positive_when_up
negative_when_up
neutral
```

## 28.3. `RevenueChartWidget`

Props:

```text
data
range
metric
loading
error
onRetry
```

## 28.4. `OrderStatusOverview`

Props:

```text
statusCounts
loading
error
onStatusClick
```

## 28.5. `OrdersRequiringActionTable`

Props:

```text
orders
actions
loading
error
onAction
```

## 28.6. `LowStockProductsTable`

Props:

```text
products
loading
error
onAdjustStock
onViewProduct
```

## 28.7. `AdminAlertCenter`

Props:

```text
alerts
loading
error
onDismiss
```

## 28.8. `ActivityFeed`

Props:

```text
items
loading
error
```

---

# 29. Page structure recommendation

```text
AdminDashboardPage
├── AdminPageHeader
│   ├── DateRangeSelector
│   ├── RefreshButton
│   └── ExportButton
├── AlertCenter
├── KpiCardGrid
│   ├── RevenueKpiCard
│   ├── OrdersKpiCard
│   ├── AverageOrderValueKpiCard
│   ├── PendingOrdersKpiCard
│   └── LowStockKpiCard
├── DashboardMainGrid
│   ├── RevenueChartWidget
│   └── OrderStatusOverview
├── OperationsGrid
│   ├── OrdersRequiringActionTable
│   └── LowStockProductsTable
├── InsightsGrid
│   ├── BestSellingProductsWidget
│   ├── PaymentSummaryWidget
│   └── CustomerSummaryWidget
└── ActivityFeed
```

---

# 30. State management rules

## 30.1. Date range state

Date range nên nằm ở page level.

Khi date range đổi:

- KPI reload.
- Chart reload.
- Best sellers reload.
- Order status reload.

Không nhất thiết reload low stock nếu low stock là realtime theo hiện tại.

## 30.2. Widget state

Mỗi widget có state riêng:

```text
idle
loading
success
empty
error
```

## 30.3. Refresh state

Có hai loại refresh:

```text
page refresh
widget refresh
```

Page refresh reload tất cả widget.

Widget refresh chỉ reload widget đó.

## 30.4. Optimistic update

Với action như confirm order, có thể optimistic update nếu API ổn định.

Nhưng MVP nên dùng flow an toàn:

```text
Click action
Show loading on action
Call API
Success -> refresh relevant widgets
Error -> show toast
```

---

# 31. Interaction rules

## 31.1. Click KPI card

KPI card nên link tới trang chi tiết tương ứng.

Ví dụ:

```text
Revenue -> /admin/reports/revenue
Orders -> /admin/orders
Pending Orders -> /admin/orders?status=pending_confirmation
Low Stock -> /admin/inventory?status=low_stock
```

## 31.2. Click order row

Click vào order number hoặc row chính:

```text
/admin/orders/{orderId}
```

Action button không được bị kích hoạt khi click row nếu user click vào button.

## 31.3. Confirm order action

Flow:

```text
Click Confirm
Open confirmation modal
Confirm
Call API
Show success toast
Refresh orders requiring action
Refresh order status overview
```

## 31.4. Mark paid action

Đây là action nhạy cảm.

Cần modal xác nhận.

Modal nên yêu cầu:

```text
Payment method
Payment reference optional
Note optional
```

MVP có thể chỉ confirm đơn giản.

## 31.5. Adjust stock action

Click từ low stock table:

```text
Open stock adjustment modal
Input new quantity or adjustment delta
Reason required
Submit
Refresh inventory widgets
```

---

# 32. Empty dashboard for new store

Khi shop mới chưa có dữ liệu, dashboard không nên trống khó hiểu.

## 32.1. Empty onboarding state

Hiển thị checklist:

```text
Complete your store setup
1. Add categories
2. Add products
3. Configure payment methods
4. Configure shipping methods
5. Publish homepage
```

## 32.2. CTA

```text
Add first product
Create category
Configure shipping
```

## 32.3. Rule

Nếu store chưa có sản phẩm, ưu tiên onboarding hơn KPI rỗng.

---

# 33. Electronics-specific dashboard logic

## 33.1. Variant-level inventory

Tồn kho phải theo biến thể.

Ví dụ:

```text
Laptop A / RAM 16GB / SSD 512GB
Laptop A / RAM 32GB / SSD 1TB
```

Dashboard low stock không được gộp thành `Laptop A` chung chung.

## 33.2. Warranty visibility

Sản phẩm điện tử có bảo hành, nên dashboard cần chỗ để mở rộng warranty/service.

MVP có thể chưa làm, nhưng component slot cần có.

## 33.3. High value order warning

Đơn giá trị cao nên có warning để xác nhận kỹ.

Ví dụ rule:

```text
Order amount >= 20,000,000đ
```

Hiển thị badge:

```text
High value
```

Ngưỡng cần configurable.

## 33.4. Bank transfer attention

Đơn chuyển khoản cần dễ theo dõi.

Widget orders requiring action phải có trạng thái:

```text
Bank transfer pending
Bank transfer verification required
```

## 33.5. Serial number / IMEI placeholder

Với điện thoại/laptop, sau này có thể cần serial/IMEI.

Dashboard không cần hiển thị ở MVP, nhưng order/warehouse flow nên để khả năng mở rộng.

---

# 34. Analytics events

Dashboard nên emit analytics cho hành vi admin nếu cần cải thiện UX.

## 34.1. Events

```text
admin_dashboard_viewed
admin_dashboard_date_range_changed
admin_dashboard_widget_retry_clicked
admin_dashboard_order_confirm_clicked
admin_dashboard_low_stock_product_clicked
admin_dashboard_alert_clicked
admin_dashboard_export_clicked
```

## 34.2. Event payload

```json
{
  "adminId": "admin_1",
  "role": "store_manager",
  "dateRange": "today",
  "widget": "orders_requiring_action"
}
```

Không gửi dữ liệu cá nhân không cần thiết.

---

# 35. SEO rules

Admin page không cần SEO công khai.

Rule:

- Không index admin pages.
- Không expose admin route trong sitemap.
- Có meta title nội bộ.

Ví dụ:

```text
Dashboard | Admin
```

---

# 36. Error handling

## 36.1. Unauthorized

Nếu token hết hạn:

```text
Redirect to admin login
Show message: Session expired. Please log in again.
```

## 36.2. Forbidden

Nếu thiếu quyền:

```text
Show 403 page or hide restricted widget
```

## 36.3. API partial failure

Nếu KPI load được nhưng chart lỗi:

- KPI vẫn hiển thị.
- Chart hiển thị error card.
- Có retry.

## 36.4. Network offline

Nếu mất mạng:

```text
You are offline. Some dashboard data may be outdated.
```

Nếu có cached data, hiển thị timestamp:

```text
Last updated: 22/06/2026 14:35
```

---

# 37. Testing strategy

## 37.1. Unit test

Test các helper:

```text
formatCurrency
formatPercent
formatDateTime
getTrendTone
getMetricTrendState
maskPhoneNumber
buildDashboardLink
```

## 37.2. Component test

Test các component:

```text
DashboardKpiCard
RevenueChartWidget
OrderStatusOverview
OrdersRequiringActionTable
LowStockProductsTable
AdminAlertCenter
DateRangeSelector
```

## 37.3. E2E test

Dùng Playwright để test flow admin.

---

# 38. Playwright test specification

## 38.1. Admin can view dashboard

Test:

```text
Given admin logged in
When admin opens /admin/dashboard
Then dashboard title is visible
And KPI cards are visible
And orders requiring action widget is visible
```

## 38.2. Date range changes dashboard data

Test:

```text
Given admin is on dashboard
When admin selects Last 7 days
Then KPI widgets reload
And revenue chart displays Last 7 days data
```

## 38.3. Widget partial error

Test:

```text
Given revenue chart API fails
When dashboard loads
Then KPI cards still render
And revenue chart shows retry error state
```

## 38.4. Admin can open pending orders

Test:

```text
Given dashboard has pending orders
When admin clicks Pending Orders KPI
Then admin navigates to /admin/orders?status=pending_confirmation
```

## 38.5. Admin can confirm order from dashboard

Test:

```text
Given dashboard has an order requiring action
When admin clicks Confirm
And confirms modal
Then success toast appears
And order disappears from requiring action list
```

## 38.6. Low stock product links to inventory

Test:

```text
Given dashboard has low stock products
When admin clicks a product row
Then admin navigates to product or inventory detail
```

## 38.7. Permission hides revenue

Test:

```text
Given admin lacks dashboard.revenue.view
When dashboard loads
Then revenue KPI is hidden or permission state is shown
```

## 38.8. Mobile dashboard has no horizontal overflow

Test:

```text
Given viewport is 375px
When admin opens dashboard
Then page has no horizontal overflow
And sidebar is accessible as drawer
```

---

# 39. Visual regression checklist

Chụp screenshot cho:

```text
Desktop 1440px
Laptop 1366px
Tablet 768px
Mobile 375px
Dark mode nếu có
Empty store state
Partial error state
Low stock warning state
Permission-limited state
```

Không được approve screenshot mới nếu:

- KPI card lệch grid.
- Chart bị méo.
- Table overflow toàn page.
- Sidebar che content.
- Mobile không bấm được action.
- Text số liệu bị cắt.

---

# 40. Agent implementation rules

Khi agent implement Admin Dashboard, bắt buộc tuân thủ:

1. Đọc file này trước khi code.
2. Đọc `../main/ecommerce_design_language.md` để lấy design token gốc.
3. Đọc `01-electronics-store-theme.md` để lấy tone ngành đồ điện tử.
4. Không hard-code màu ngoài design tokens.
5. Không hard-code role/permission trong UI nếu có permission config.
6. Không để một widget fail làm crash toàn dashboard.
7. Không hiển thị dữ liệu khách hàng nhạy cảm nếu không cần.
8. Không dùng table quá rộng gây overflow toàn page.
9. Không dùng chart mà thiếu text summary.
10. Không kết luận xong nếu chưa test desktop và mobile.

## 40.1. Required report after coding

Agent sau khi code phải báo cáo:

```text
Files changed
Components created/updated
APIs integrated/mocked
Tests added/updated
Tests run
Screenshots checked
Known limitations
```

## 40.2. Không được làm

- Không xóa test để pass.
- Không tắt warning console mà không xử lý nguyên nhân.
- Không dùng fake static data trong production component nếu API đã có.
- Không hiển thị full phone/email trên dashboard mặc định.
- Không thêm animation nặng.
- Không sửa global CSS bừa bãi.

---

# 41. Definition of Done

Admin Dashboard được coi là hoàn thành khi đạt các điều kiện sau:

## 41.1. UI

- Layout desktop đúng spec.
- Tablet usable.
- Mobile không overflow ngang.
- Sidebar và top bar hoạt động đúng.
- KPI card hiển thị đúng trạng thái.
- Chart hiển thị đúng dữ liệu hoặc error state.
- Table có loading, empty, error state.

## 41.2. Function

- Date range hoạt động.
- Refresh hoạt động.
- KPI click link đúng trang.
- Order action hoạt động hoặc có placeholder rõ nếu chưa implement.
- Permission được xử lý.
- Partial error không crash page.

## 41.3. Data

- Currency format đúng.
- Date/time format đúng.
- Percent format đúng.
- Phone/email được mask nếu cần.
- Tồn kho theo variant.

## 41.4. Accessibility

- Keyboard dùng được.
- Icon-only button có label.
- Table header semantic đúng.
- Chart có text summary.
- Color contrast đạt yêu cầu.

## 41.5. Test

- Unit/component test pass.
- Playwright admin dashboard test pass.
- Visual screenshot desktop/mobile đã kiểm tra.
- Không có console error nghiêm trọng.

---

# 42. MVP scope

Để build nhanh, MVP Admin Dashboard chỉ cần:

```text
Admin shell
Page header
Date range preset
Revenue KPI
Orders KPI
Pending Orders KPI
Low Stock KPI
Revenue chart
Order status overview
Orders requiring action table
Low stock products table
Basic alerts
Loading / empty / error states
Permission guard cơ bản
```

Chưa cần làm ngay:

```text
Advanced analytics
Warranty widget
Customer segmentation
AI insights
Dead stock prediction
Export report nâng cao
Custom dashboard layout
Multi-store switcher
```

---

# 43. Suggested file structure

Framework nào cũng có thể map theo cấu trúc tương tự.

```text
src/
  admin/
    layout/
      AdminLayout.*
      AdminSidebar.*
      AdminTopBar.*

    dashboard/
      pages/
        AdminDashboardPage.*

      components/
        DashboardKpiCard.*
        DashboardKpiGrid.*
        RevenueChartWidget.*
        OrderStatusOverview.*
        OrdersRequiringActionTable.*
        LowStockProductsTable.*
        AdminAlertCenter.*
        ActivityFeed.*
        DateRangeSelector.*

      api/
        dashboardApi.*

      types/
        dashboardTypes.*

      utils/
        dashboardFormatters.*
        dashboardLinks.*
        metricTrend.*

tests/
  e2e/
    admin-dashboard.spec.*

  visual/
    admin-dashboard.visual.spec.*
```

---

# 44. Future extensions

Sau MVP, có thể mở rộng:

- Custom dashboard widget theo role.
- Realtime order notifications.
- Drill-down report từ KPI.
- Forecast tồn kho.
- Profit margin dashboard.
- Campaign performance dashboard.
- Warranty/service operations dashboard.
- Multi-warehouse dashboard.
- Multi-store dashboard.

---

# 45. Tóm tắt cho agent

Admin Dashboard cho shop đồ điện tử là màn hình vận hành.

Ưu tiên của nó là:

```text
Nhanh
Rõ
Chính xác
Có hành động
Không crash toàn trang
Không lộ dữ liệu nhạy cảm
Không vỡ responsive
```

Nếu phải chọn ít component nhất cho MVP, hãy làm:

```text
KPI cards
Revenue chart
Order status overview
Orders requiring action
Low stock products
Alert center
```

Các component này phải độc lập, có loading/empty/error state riêng, hỗ trợ permission, và có test tự động.
