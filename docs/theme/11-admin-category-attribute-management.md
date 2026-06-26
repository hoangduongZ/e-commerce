# 11-admin-category-attribute-management.md

# Admin Category, Brand & Attribute Management Specification

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Module: Quản lý danh mục, thương hiệu, thuộc tính kỹ thuật, attribute template  
> Mục tiêu: Đặc tả đủ chi tiết để agent/frontend/backend có thể code module quản lý cấu trúc sản phẩm điện tử từ đầu đến cuối.  
> Phụ thuộc: `01-electronics-store-theme.md`, `09-admin-dashboard.md`, `10-admin-product-management.md`  
> Không phụ thuộc công nghệ frontend/backend cụ thể.

---

## 1. Vai trò của module này

Module này là phần **xương sống tái sử dụng** của source web bán hàng.

Nếu chỉ bán một loại hàng cố định, bạn có thể hard-code thông số sản phẩm trong form.

Ví dụ:

```text
CPU
RAM
SSD
GPU
Screen
Battery
Warranty
```

Nhưng nếu muốn source clone được cho nhiều ngành hàng, không được hard-code như vậy.

Cần có hệ thống động gồm:

```text
Category
Brand
Attribute
Attribute Group
Attribute Template
Attribute Option
Product Attribute Value
```

Nhờ đó:

- Laptop có bộ thông số riêng.
- Điện thoại có bộ thông số riêng.
- Màn hình có bộ thông số riêng.
- Tai nghe có bộ thông số riêng.
- Sau này clone sang thời trang, mỹ phẩm, nội thất vẫn dùng được cùng một logic.

Module này quyết định:

- Admin tạo sản phẩm sẽ thấy field nào.
- Storefront product detail hiển thị thông số nào.
- Trang danh sách sản phẩm có filter nào.
- Product card hiển thị quick specs nào.
- Compare page so sánh theo thông số nào.
- Search index chứa field nào.
- SEO category có cấu trúc ra sao.

---

## 2. Mục tiêu thiết kế

### 2.1. Mục tiêu nghiệp vụ

Admin có thể:

- Tạo cây danh mục sản phẩm.
- Quản lý thương hiệu.
- Tạo thuộc tính kỹ thuật dùng chung.
- Tạo nhóm thuộc tính để UI dễ đọc.
- Tạo template thông số theo từng danh mục.
- Chọn thuộc tính nào dùng làm filter.
- Chọn thuộc tính nào hiển thị ở product card.
- Chọn thuộc tính nào dùng trong compare.
- Sắp xếp thứ tự hiển thị thông số.
- Ẩn/hiện danh mục ngoài storefront.
- Cấu hình SEO cho category/brand.
- Kiểm soát dữ liệu sản phẩm nhập vào đúng định dạng.

### 2.2. Mục tiêu kỹ thuật

Hệ thống cần:

- Không hard-code attribute theo ngành hàng.
- Dữ liệu có thể mở rộng.
- Attribute dùng được cho nhiều category.
- Một category có thể kế thừa template từ category cha.
- Form tạo sản phẩm tự sinh field theo category.
- Filter storefront tự sinh từ attribute được bật filter.
- Compare table tự sinh từ attribute được bật compare.
- Có audit log cho thay đổi quan trọng.
- Có cơ chế tránh xóa nhầm dữ liệu đang được sản phẩm dùng.

### 2.3. Mục tiêu trải nghiệm admin

Admin không cần hiểu database.

Admin chỉ cần làm theo luồng:

```text
Tạo danh mục
→ Tạo thương hiệu
→ Tạo thuộc tính
→ Gom thuộc tính vào nhóm
→ Gắn template cho danh mục
→ Tạo sản phẩm
```

Khi tạo sản phẩm Laptop, form tự hiện field Laptop.

Khi tạo sản phẩm Phone, form tự hiện field Phone.

---

## 3. Phạm vi module

Module này bao gồm các màn:

```text
Category List
Category Create/Edit
Category Tree Manager
Brand List
Brand Create/Edit
Attribute List
Attribute Create/Edit
Attribute Option Manager
Attribute Group Manager
Attribute Template List
Attribute Template Builder
Category Template Assignment
SEO Settings
Audit Log
```

Module này không bao gồm:

- Tạo sản phẩm chi tiết.
- Quản lý đơn hàng.
- Quản lý tồn kho chi tiết.
- Upload ảnh sản phẩm hàng loạt.
- Quản lý khuyến mãi.

Những phần đó nằm ở các file spec khác.

---

## 4. Khái niệm cốt lõi

### 4.1. Category

Category là nhóm sản phẩm.

Ví dụ:

```text
Laptop
Phone
Tablet
Monitor
Headphone
Keyboard
Mouse
PC Component
Smart Home
```

Category có thể có cấp cha/con.

Ví dụ:

```text
Laptop
├── Gaming Laptop
├── Office Laptop
├── Ultrabook
└── Workstation Laptop

PC Component
├── CPU
├── RAM
├── SSD
├── Mainboard
└── GPU
```

### 4.2. Brand

Brand là thương hiệu.

Ví dụ:

```text
Apple
Dell
HP
Lenovo
Asus
Acer
MSI
Samsung
Sony
Logitech
```

Brand có thể dùng chung cho nhiều category.

Ví dụ Apple có thể thuộc:

```text
Phone
Laptop
Tablet
Headphone
Smartwatch
```

### 4.3. Attribute

Attribute là một thuộc tính kỹ thuật hoặc thuộc tính bán hàng.

Ví dụ:

```text
CPU
RAM
Storage
GPU
Screen Size
Battery Capacity
Color
Warranty Period
Connectivity
Refresh Rate
```

Attribute không thuộc riêng một sản phẩm cụ thể.

Attribute là định nghĩa dùng lại.

### 4.4. Attribute Option

Attribute Option là danh sách lựa chọn của attribute.

Ví dụ attribute `RAM` có option:

```text
4GB
8GB
16GB
32GB
64GB
```

Attribute `CPU Brand` có option:

```text
Intel
AMD
Apple
Qualcomm
MediaTek
```

### 4.5. Attribute Group

Attribute Group dùng để gom nhiều attribute thành một nhóm dễ đọc.

Ví dụ Laptop có group:

```text
Performance
Display
Memory & Storage
Graphics
Connectivity
Physical Design
Warranty
```

Phone có group:

```text
Performance
Display
Camera
Battery
Connectivity
Design
Warranty
```

Group giúp product detail hiển thị bảng thông số rõ hơn.

### 4.6. Attribute Template

Attribute Template là bộ cấu hình attribute cho một category.

Ví dụ template `Laptop Specs Template` gồm:

```text
CPU
RAM
Storage
GPU
Screen Size
Screen Resolution
Battery
Weight
Warranty Period
```

Template quyết định:

- Product form hiện field nào.
- Field nào bắt buộc.
- Field nào dùng làm filter.
- Field nào dùng làm quick spec.
- Field nào dùng trong compare.
- Field nào được index search.
- Field nào hiển thị ở product detail.

### 4.7. Product Attribute Value

Product Attribute Value là giá trị thật của attribute trên sản phẩm.

Ví dụ với sản phẩm Dell Inspiron 15:

```text
CPU = Intel Core i5-1335U
RAM = 16GB
Storage = 512GB SSD
Screen Size = 15.6 inch
Warranty = 24 months
```

---

## 5. Navigation trong Admin

Trong sidebar admin, module này nên nằm dưới nhóm `Catalog`.

```text
Admin
├── Dashboard
├── Catalog
│   ├── Products
│   ├── Categories
│   ├── Brands
│   ├── Attributes
│   ├── Attribute Groups
│   └── Attribute Templates
├── Orders
├── Inventory
├── Customers
├── Promotions
├── Warranty
└── Settings
```

Nếu admin đang ở module này, sidebar active vào `Catalog`.

---

## 6. Permission

### 6.1. Role đề xuất

```text
Super Admin
Catalog Manager
Product Editor
Viewer
```

### 6.2. Permission key

```text
category.view
category.create
category.update
category.delete
category.publish

brand.view
brand.create
brand.update
brand.delete

attribute.view
attribute.create
attribute.update
attribute.delete

template.view
template.create
template.update
template.delete
template.assign
```

### 6.3. Rule phân quyền

Super Admin có toàn quyền.

Catalog Manager có quyền quản lý category, brand, attribute, template.

Product Editor có quyền xem category/brand/attribute/template để tạo sản phẩm, nhưng không được sửa cấu trúc.

Viewer chỉ được xem.

Nếu user không có quyền, UI phải:

- Ẩn nút tạo/sửa/xóa.
- Disable action không được phép.
- Backend vẫn phải kiểm tra quyền.
- Không chỉ tin frontend.

---

## 7. Category Management

## 7.1. Category List Page

### Mục đích

Cho admin xem, tìm, lọc, tạo, sửa, ẩn/hiện và sắp xếp danh mục.

### Route đề xuất

```text
/admin/categories
```

### Layout desktop

```text
Admin Shell
├── Page Header
│   ├── Title: Categories
│   ├── Description
│   ├── Button: Create Category
│   └── Button: Manage Tree
├── Filter Bar
│   ├── Search input
│   ├── Status filter
│   ├── Parent category filter
│   └── Visibility filter
├── Category Table
└── Pagination
```

### Table columns

```text
Category
Slug
Parent
Products
Template
Visible
Status
Sort Order
Updated At
Actions
```

Không đưa câu mô tả dài vào table.

Các column nên ngắn.

### Row action

```text
View
Edit
Duplicate
Manage SEO
Assign Template
Hide/Show
Delete
```

### Filter

Filter cần có:

```text
Keyword
Status
Visible / Hidden
Has Template / No Template
Parent Category
```

### Search behavior

Search theo:

```text
name
slug
description
seo_title
```

Search không phân biệt hoa thường.

### Empty state

Nếu chưa có category:

```text
Title: No categories yet
Description: Create your first category to organize products.
Primary action: Create Category
```

### Error state

Nếu tải lỗi:

```text
Title: Cannot load categories
Description: Please retry or contact admin.
Action: Retry
```

---

## 7.2. Category Tree Manager

### Mục đích

Cho admin quản lý cây danh mục bằng kéo thả hoặc nút move.

### Route đề xuất

```text
/admin/categories/tree
```

### UI layout

```text
Page Header
├── Save Order button
├── Expand All
├── Collapse All
└── Reset Changes

Tree Area
├── Category node
│   ├── Drag handle
│   ├── Category name
│   ├── Product count
│   ├── Visibility badge
│   └── Actions
```

### Node action

```text
Add child
Edit
Hide
Move up
Move down
```

### Rule kéo thả

- Không cho category thành con của chính nó.
- Không cho tạo vòng lặp.
- Nếu category cha hidden, category con vẫn có thể visible trong DB nhưng storefront phải không show nếu cha bị hidden.
- Nếu di chuyển category có nhiều product, phải confirm.
- Sau khi kéo thả, cần nhấn `Save Order` mới lưu.

### Unsaved changes

Nếu admin thay đổi cây mà chưa lưu:

- Hiển thị badge `Unsaved changes`.
- Khi rời trang, hỏi confirm.
- Nút `Reset Changes` khôi phục trạng thái trước khi sửa.

---

## 7.3. Category Create/Edit Page

### Route đề xuất

```text
/admin/categories/new
/admin/categories/{id}/edit
```

### Form section

```text
Basic Information
Hierarchy
Media
Storefront Visibility
Template Assignment
SEO
Advanced
```

### Basic Information fields

```text
Name
Slug
Description
Short Description
Status
```

### Field rule

`Name` bắt buộc.

`Slug` tự sinh từ name, nhưng admin có thể sửa.

`Slug` phải unique.

`Description` dùng cho category page.

`Short Description` dùng cho meta hoặc block nhỏ.

### Hierarchy fields

```text
Parent Category
Sort Order
```

Rule:

- Parent có thể null.
- Không cho chọn chính category hiện tại làm parent.
- Không cho chọn category con của nó làm parent.

### Media fields

```text
Thumbnail Image
Banner Image
Icon
```

Usage:

- Thumbnail dùng ở category grid.
- Banner dùng ở category landing page.
- Icon dùng trong mega menu.

### Visibility fields

```text
Show in Header Menu
Show in Home Category Grid
Show in Footer
Show in Sitemap
Is Featured
```

### Template Assignment

Fields:

```text
Attribute Template
Inherit Template From Parent
```

Rule:

- Nếu `Inherit Template From Parent = true`, không cho chọn template riêng.
- Nếu category có template riêng, ưu tiên template riêng.
- Nếu không có template riêng, lấy từ category cha gần nhất.
- Nếu không có template nào, product form chỉ có basic fields.

### SEO fields

```text
SEO Title
SEO Description
Canonical URL
Meta Robots
Open Graph Image
```

Rule:

- SEO Title tối đa khuyến nghị 60 ký tự.
- SEO Description tối đa khuyến nghị 160 ký tự.
- Nếu bỏ trống, tự fallback từ name và description.

### Advanced fields

```text
Custom URL Path
Custom Sort Strategy
Filter Layout Type
Landing Page Content
```

`Filter Layout Type` có thể là:

```text
sidebar
horizontal
drawer_only
```

### Validation

```text
Name required
Slug required
Slug unique
Parent cannot create circular reference
Sort order must be number
SEO title max length warning
SEO description max length warning
```

---

## 8. Brand Management

## 8.1. Brand List Page

### Route đề xuất

```text
/admin/brands
```

### Layout

```text
Page Header
├── Title: Brands
├── Button: Create Brand

Filter Bar
├── Search
├── Status
├── Featured

Brand Table
```

### Table columns

```text
Logo
Name
Slug
Products
Featured
Status
Updated At
Actions
```

### Row action

```text
Edit
View Products
Feature/Unfeature
Hide/Show
Delete
```

---

## 8.2. Brand Create/Edit Page

### Route đề xuất

```text
/admin/brands/new
/admin/brands/{id}/edit
```

### Fields

```text
Name
Slug
Logo
Banner
Description
Website URL
Country
Status
Featured
SEO Title
SEO Description
Open Graph Image
```

### Storefront usage

Brand được dùng ở:

- Product detail.
- Product list filter.
- Brand landing page.
- Home brand showcase.
- Product compare.

### Validation

```text
Name required
Slug required
Slug unique
Website URL must be valid URL
Logo should have alt text
```

### Delete rule

Không cho hard delete brand nếu đang có product dùng brand đó.

Thay vào đó:

- Cho phép archive.
- Cho phép merge brand.
- Cho phép replace brand cho sản phẩm.

---

## 9. Attribute Management

## 9.1. Attribute List Page

### Route đề xuất

```text
/admin/attributes
```

### Mục đích

Quản lý tất cả thuộc tính kỹ thuật và thuộc tính bán hàng dùng cho sản phẩm.

### Layout

```text
Page Header
├── Title: Attributes
├── Button: Create Attribute
├── Button: Import Attributes

Filter Bar
├── Search
├── Data Type
├── Filterable
├── Comparable
├── Status

Attribute Table
```

### Table columns

```text
Name
Code
Data Type
Unit
Options
Filterable
Comparable
Templates
Status
Actions
```

### Attribute examples

```text
CPU
RAM
Storage
GPU
Screen Size
Screen Resolution
Refresh Rate
Battery Capacity
Charging Power
Camera Resolution
Weight
Warranty Period
Color
```

---

## 9.2. Attribute Create/Edit Page

### Route đề xuất

```text
/admin/attributes/new
/admin/attributes/{id}/edit
```

### Form sections

```text
Basic Information
Data Type
Options
Display Rules
Storefront Usage
Validation Rules
Advanced
```

### Basic Information fields

```text
Name
Code
Description
Status
```

`Code` là key kỹ thuật.

Ví dụ:

```text
cpu
ram
storage
screen_size
battery_capacity
warranty_period
```

Rule:

- Code chỉ dùng lowercase, number, underscore.
- Code không đổi nếu đã có product value dùng.
- Name có thể đổi.

### Data Type

Attribute type gồm:

```text
text
number
select
multi_select
boolean
color
date
range
rich_text
```

### Khi nào dùng type nào

`text` dùng cho chuỗi tự do.

Ví dụ:

```text
Intel Core i5-1335U
Apple M3 Pro
```

`number` dùng khi cần sort/filter theo số.

Ví dụ:

```text
battery_capacity = 5000
weight = 1.35
screen_size = 15.6
```

`select` dùng khi chọn một option.

Ví dụ:

```text
CPU Brand = Intel
Panel Type = IPS
```

`multi_select` dùng khi chọn nhiều option.

Ví dụ:

```text
Connectivity = Wi-Fi 6, Bluetooth 5.3, USB-C
```

`boolean` dùng cho có/không.

Ví dụ:

```text
Touch Screen = Yes
Backlit Keyboard = Yes
5G Support = No
```

`color` dùng cho màu sản phẩm.

`range` dùng cho giá trị khoảng.

Ví dụ:

```text
Screen Refresh Rate Range
Operating Temperature
```

### Unit fields

```text
Unit
Unit Position
Decimal Places
```

Unit example:

```text
GB
TB
inch
Hz
W
Wh
kg
gram
month
year
```

Unit position:

```text
after
before
```

Ví dụ:

```text
16 GB
15.6 inch
24 months
```

### Options

Nếu type là `select` hoặc `multi_select`, hiển thị option manager.

Option fields:

```text
Label
Value
Sort Order
Color Hex
Description
Status
```

Ví dụ option của RAM:

```text
8GB
16GB
32GB
64GB
```

Rule:

- Option label có thể sửa.
- Option value nên ổn định.
- Không xóa option đang được product dùng.
- Có thể archive option.

### Display Rules

Fields:

```text
Display Name
Help Text
Placeholder
Default Value
Show Unit
Use As Badge
```

`Display Name` là tên hiển thị ngoài storefront.

Ví dụ admin name là `Storage`, display name có thể là `Ổ cứng`.

### Storefront Usage

Fields:

```text
Show On Product Detail
Show On Product Card
Show As Filter
Show In Compare
Show In Search Index
Show In Admin Table
```

Rule:

- `Show On Product Card` chỉ nên bật cho thông số ngắn.
- `Show As Filter` chỉ nên bật với select, multi_select, boolean, number/range.
- `Show In Compare` nên bật cho thông số quan trọng.
- `Show In Search Index` nên bật cho attribute khách hay search.

### Validation Rules

Fields:

```text
Required
Min Value
Max Value
Min Length
Max Length
Regex Pattern
Allowed Units
Unique Per Product
```

Ví dụ:

```text
RAM required for Laptop
Weight min 0
Warranty Period min 0
Screen Size max 100
```

### Advanced

Fields:

```text
Sort Weight
Search Boost
Filter UI Type
Compare Group
Internal Note
```

Filter UI Type:

```text
checkbox
radio
range_slider
min_max_input
color_swatch
boolean_toggle
```

---

## 10. Attribute Group Management

## 10.1. Attribute Group List

### Route đề xuất

```text
/admin/attribute-groups
```

### Table columns

```text
Name
Code
Attributes
Templates
Sort Order
Status
Actions
```

### Common groups for electronics

```text
Performance
Display
Memory & Storage
Graphics
Camera
Battery
Connectivity
Design
Audio
Warranty
Package
```

---

## 10.2. Attribute Group Create/Edit

### Fields

```text
Name
Code
Description
Icon
Sort Order
Status
```

### Rule

Group chỉ là cách gom UI.

Group không quyết định product value.

Một attribute có thể nằm trong group khác nhau tùy template.

Ví dụ:

- `Resolution` trong Monitor thuộc group Display.
- `Resolution` trong Camera có thể thuộc group Camera.

---

## 11. Attribute Template Management

## 11.1. Attribute Template List Page

### Route đề xuất

```text
/admin/attribute-templates
```

### Mục đích

Quản lý các bộ thông số theo từng loại sản phẩm.

### Table columns

```text
Name
Code
Category
Attributes
Used By Products
Status
Updated At
Actions
```

### Example templates

```text
Laptop Specs Template
Phone Specs Template
Tablet Specs Template
Monitor Specs Template
Headphone Specs Template
Keyboard Specs Template
Mouse Specs Template
GPU Specs Template
SSD Specs Template
```

---

## 11.2. Attribute Template Builder

### Route đề xuất

```text
/admin/attribute-templates/new
/admin/attribute-templates/{id}/edit
```

### Mục đích

Cho admin build form thông số động bằng cách chọn attribute và cấu hình rule.

### Layout desktop

```text
Page Header
├── Template name
├── Save Draft
├── Publish
└── Preview Product Form

Main Layout
├── Left Panel: Available Attributes
├── Center: Template Structure
└── Right Panel: Selected Attribute Settings
```

### Left Panel

Hiển thị danh sách attribute có thể thêm.

Có search và filter:

```text
Search attribute
Data type
Group
Status
Used / Unused
```

### Center Panel

Hiển thị cấu trúc template theo group.

Ví dụ:

```text
Performance
├── CPU
├── CPU Brand
├── RAM
└── GPU

Display
├── Screen Size
├── Resolution
├── Refresh Rate
└── Panel Type

Warranty
└── Warranty Period
```

Cho phép:

- Kéo thả attribute vào group.
- Kéo thả đổi thứ tự group.
- Kéo thả đổi thứ tự attribute.
- Thêm group mới.
- Xóa attribute khỏi template.
- Duplicate group.
- Collapse/expand group.

### Right Panel

Khi chọn một attribute trong template, hiển thị setting riêng cho template.

Fields:

```text
Required In This Category
Show On Product Detail
Show On Product Card
Show As Filter
Show In Compare
Show In Search Index
Filter UI Type
Display Priority
Quick Spec Priority
Compare Priority
Default Value
Help Text Override
Unit Override
Validation Override
```

### Template-level fields

```text
Template Name
Template Code
Description
Applies To Category
Inherit From Template
Status
Version
```

### Required rule

Một attribute có thể required trong Laptop nhưng không required trong Accessories.

Ví dụ:

```text
RAM required for Laptop
RAM optional for Mini PC Accessory
```

### Quick spec rule

Quick specs là thông số ngắn hiển thị ở product card.

Mỗi category nên có tối đa 3-5 quick specs.

Laptop quick specs:

```text
CPU
RAM
Storage
Screen Size
GPU
```

Phone quick specs:

```text
Chip
RAM
Storage
Battery
Camera
```

Monitor quick specs:

```text
Screen Size
Resolution
Refresh Rate
Panel Type
Connectivity
```

### Filter rule

Filter nên bật cho attribute giúp khách ra quyết định.

Laptop filters:

```text
Brand
Price
CPU Brand
CPU Series
RAM
Storage
GPU
Screen Size
Use Case
Warranty
```

Phone filters:

```text
Brand
Price
Storage
RAM
Screen Size
Battery Capacity
5G Support
Camera
```

Monitor filters:

```text
Brand
Price
Screen Size
Resolution
Refresh Rate
Panel Type
Ports
```

### Compare rule

Compare nên bật cho attribute có giá trị so sánh.

Không cần đưa mô tả dài vào compare.

Nên đưa:

```text
CPU
RAM
Storage
GPU
Screen
Battery
Weight
Warranty
```

Không nên đưa:

```text
Long Description
Marketing Text
Internal Note
```

---

## 12. Template Versioning

### Mục đích

Tránh việc sửa template làm hỏng form sản phẩm cũ.

### Rule

Mỗi template có version.

Ví dụ:

```text
Laptop Template v1
Laptop Template v2
```

Khi sửa nhỏ như đổi label hoặc sort order, có thể update cùng version.

Khi sửa lớn như xóa required attribute, đổi data type, cần tạo version mới.

### Product binding

Product đã tạo có thể gắn với template version.

Khi template update:

- Product mới dùng version mới.
- Product cũ có thể giữ version cũ.
- Admin có action `Migrate products to latest template`.

### Migration warning

Khi migrate, cần báo:

```text
New required attributes missing
Removed attributes with existing values
Data type conflict
Option value no longer available
```

---

## 13. Category Template Assignment

### Mục đích

Gắn template với category.

### UI

Trong category edit có section `Template Assignment`.

Ngoài ra có thể có trang riêng:

```text
/admin/categories/template-assignment
```

### Layout trang riêng

```text
Category Tree
├── Category node
│   ├── Category name
│   ├── Assigned template
│   ├── Inherited from
│   └── Action: Assign / Change / Remove

Right Panel
├── Template detail
├── Preview product form
└── Affected products
```

### Rule

- Category con có thể inherit template từ category cha.
- Category con có thể override bằng template riêng.
- Nếu remove template ở category cha, category con đang inherit sẽ mất template.
- Cần cảnh báo số category/product bị ảnh hưởng.

### Affected products preview

Trước khi đổi template, hiển thị:

```text
Products using current template
Products missing required fields after change
Attributes that will be removed from form
Attributes that will be added
```

---

## 14. Storefront Integration Rules

Module admin này không chỉ phục vụ admin.

Nó phải đẩy cấu hình ra storefront.

### 14.1. Product List Filter

Trang product list lấy filter từ category template.

Rule:

- Chỉ attribute `Show As Filter = true` mới hiện filter.
- Filter order theo `Display Priority`.
- Filter UI theo `Filter UI Type`.
- Attribute number có thể dùng range slider.
- Attribute select/multi_select dùng checkbox hoặc radio.
- Attribute color dùng color swatch.

### 14.2. Product Card

Product card lấy quick specs từ template.

Rule:

- Chỉ attribute `Show On Product Card = true` mới hiện.
- Tối đa 3-5 dòng quick spec.
- Nếu thiếu value, bỏ qua dòng đó.
- Không hiện `N/A` ở card.

### 14.3. Product Detail Specs Table

Product detail specs table lấy group và attribute order từ template.

Rule:

- Group hiển thị theo sort order.
- Attribute hiển thị theo sort order trong group.
- Nếu attribute không có value, có thể ẩn.
- Attribute required nhưng thiếu value cần cảnh báo trong admin, không nên hiện lỗi cho khách.

### 14.4. Compare Page

Compare lấy attribute có `Show In Compare = true`.

Rule:

- Group giống product detail.
- Nếu sản phẩm khác category, chỉ so sánh common attributes trước.
- Nếu attribute không cùng unit, cần normalize hoặc hiển thị rõ unit.

### 14.5. Search Index

Search index lấy attribute có `Show In Search Index = true`.

Ví dụ nên index:

```text
Brand
CPU
RAM
Storage
Model
Series
Color
Use Case
```

Không nên index quá nhiều field rác vì search có thể nhiễu.

---

## 15. Data Model đề xuất

Đây là data model logic, không phụ thuộc database cụ thể.

### 15.1. Category

```json
{
  "id": "cat_laptop",
  "parent_id": null,
  "name": "Laptop",
  "slug": "laptop",
  "description": "Laptop for work, gaming and study.",
  "thumbnail_url": "/images/categories/laptop.png",
  "banner_url": "/images/banners/laptop.jpg",
  "icon": "laptop",
  "status": "active",
  "is_visible": true,
  "show_in_header": true,
  "show_in_home": true,
  "sort_order": 10,
  "attribute_template_id": "tpl_laptop_v1",
  "inherit_template": false,
  "seo": {
    "title": "Laptop chính hãng",
    "description": "Mua laptop chính hãng, bảo hành đầy đủ."
  },
  "created_at": "2026-06-22T00:00:00Z",
  "updated_at": "2026-06-22T00:00:00Z"
}
```

### 15.2. Brand

```json
{
  "id": "brand_dell",
  "name": "Dell",
  "slug": "dell",
  "logo_url": "/images/brands/dell.svg",
  "banner_url": "/images/brands/dell-banner.jpg",
  "description": "Dell official products.",
  "website_url": "https://example.com",
  "country": "US",
  "status": "active",
  "is_featured": true,
  "seo": {
    "title": "Dell chính hãng",
    "description": "Laptop Dell chính hãng."
  }
}
```

### 15.3. Attribute

```json
{
  "id": "attr_ram",
  "name": "RAM",
  "code": "ram",
  "description": "Memory capacity.",
  "data_type": "select",
  "unit": "GB",
  "unit_position": "after",
  "options": [
    { "label": "8GB", "value": "8" },
    { "label": "16GB", "value": "16" },
    { "label": "32GB", "value": "32" }
  ],
  "status": "active"
}
```

### 15.4. Attribute Group

```json
{
  "id": "group_memory_storage",
  "name": "Memory & Storage",
  "code": "memory_storage",
  "description": "RAM and storage specifications.",
  "sort_order": 30,
  "status": "active"
}
```

### 15.5. Attribute Template

```json
{
  "id": "tpl_laptop_v1",
  "name": "Laptop Specs Template",
  "code": "laptop_specs",
  "version": 1,
  "status": "active",
  "applies_to_category_id": "cat_laptop",
  "groups": [
    {
      "group_id": "group_performance",
      "sort_order": 10,
      "attributes": [
        {
          "attribute_id": "attr_cpu",
          "required": true,
          "show_on_detail": true,
          "show_on_card": true,
          "show_as_filter": true,
          "show_in_compare": true,
          "show_in_search_index": true,
          "filter_ui_type": "checkbox",
          "sort_order": 10
        }
      ]
    }
  ]
}
```

### 15.6. Product Attribute Value

```json
{
  "product_id": "prod_dell_inspiron_15",
  "template_id": "tpl_laptop_v1",
  "values": {
    "cpu": "Intel Core i5-1335U",
    "ram": "16",
    "storage": "512GB SSD",
    "screen_size": 15.6,
    "warranty_period": 24
  }
}
```

---

## 16. API Contract đề xuất

API chỉ là gợi ý logic.

Có thể dùng REST, GraphQL, RPC hoặc backend framework nào cũng được.

### 16.1. Category API

```http
GET    /api/admin/categories
GET    /api/admin/categories/tree
GET    /api/admin/categories/{id}
POST   /api/admin/categories
PATCH  /api/admin/categories/{id}
DELETE /api/admin/categories/{id}
POST   /api/admin/categories/reorder
POST   /api/admin/categories/{id}/assign-template
```

### 16.2. Brand API

```http
GET    /api/admin/brands
GET    /api/admin/brands/{id}
POST   /api/admin/brands
PATCH  /api/admin/brands/{id}
DELETE /api/admin/brands/{id}
POST   /api/admin/brands/{id}/merge
```

### 16.3. Attribute API

```http
GET    /api/admin/attributes
GET    /api/admin/attributes/{id}
POST   /api/admin/attributes
PATCH  /api/admin/attributes/{id}
DELETE /api/admin/attributes/{id}
POST   /api/admin/attributes/import
```

### 16.4. Attribute Group API

```http
GET    /api/admin/attribute-groups
GET    /api/admin/attribute-groups/{id}
POST   /api/admin/attribute-groups
PATCH  /api/admin/attribute-groups/{id}
DELETE /api/admin/attribute-groups/{id}
```

### 16.5. Attribute Template API

```http
GET    /api/admin/attribute-templates
GET    /api/admin/attribute-templates/{id}
POST   /api/admin/attribute-templates
PATCH  /api/admin/attribute-templates/{id}
DELETE /api/admin/attribute-templates/{id}
POST   /api/admin/attribute-templates/{id}/publish
POST   /api/admin/attribute-templates/{id}/duplicate
POST   /api/admin/attribute-templates/{id}/preview-product-form
POST   /api/admin/attribute-templates/{id}/migrate-products
```

### 16.6. Storefront config API

```http
GET /api/storefront/categories
GET /api/storefront/categories/{slug}/filters
GET /api/storefront/categories/{slug}/template
GET /api/storefront/products/{slug}/specs
GET /api/storefront/compare/config?category_id=cat_laptop
```

---

## 17. Validation and Business Rules

### 17.1. Category validation

```text
Name required
Slug required
Slug unique
Parent cannot be itself
Parent cannot be descendant
Sort order must be number
Cannot delete category with active products
Cannot delete category with child categories
```

Nếu category có product, action delete phải chuyển thành archive/hide.

### 17.2. Brand validation

```text
Name required
Slug required
Slug unique
Website URL valid
Cannot delete brand used by active products
```

### 17.3. Attribute validation

```text
Name required
Code required
Code format valid
Code unique
Data type required
Select type must have options
Number type can have min/max
Cannot change data type if values exist
Cannot delete attribute used by template or product
```

### 17.4. Template validation

```text
Name required
Code required
Code unique
At least one group required
At least one attribute required
Required attribute cannot be archived
Filterable attribute must have valid filter UI type
Quick spec count should not exceed category limit
Duplicate attribute in same template not allowed
```

### 17.5. Assignment validation

```text
Cannot assign inactive template to active category
Cannot remove inherited template without warning affected categories
Cannot assign template with archived required attribute
```

---

## 18. Delete, Archive and Merge Rules

### 18.1. Soft delete ưu tiên

Không hard delete dữ liệu cấu trúc nếu đang được sử dụng.

Dùng trạng thái:

```text
active
inactive
archived
```

### 18.2. Category delete

Nếu category có child hoặc product:

- Không cho hard delete.
- Cho phép hide khỏi storefront.
- Cho phép archive nếu không có active product.
- Cho phép move products sang category khác trước khi archive.

### 18.3. Brand delete

Nếu brand đang có product:

- Không cho hard delete.
- Cho phép merge brand.

Ví dụ:

```text
Delll -> Dell
SamSung -> Samsung
```

### 18.4. Attribute delete

Nếu attribute đang nằm trong template hoặc product values:

- Không cho hard delete.
- Cho phép archive.
- Archive attribute thì không cho thêm vào template mới.
- Product cũ vẫn có thể giữ value để hiển thị nếu template còn dùng.

### 18.5. Option delete

Nếu option đang được product dùng:

- Không cho hard delete.
- Cho phép archive option.
- Có thể replace option bằng option khác.

---

## 19. Import / Export

### 19.1. Category import

Cho phép import category từ CSV/Excel.

Columns đề xuất:

```text
name
slug
parent_slug
status
sort_order
show_in_header
show_in_home
seo_title
seo_description
```

### 19.2. Brand import

Columns đề xuất:

```text
name
slug
logo_url
website_url
country
status
is_featured
```

### 19.3. Attribute import

Columns đề xuất:

```text
name
code
data_type
unit
options
status
```

Options có thể dùng format:

```text
8GB:8|16GB:16|32GB:32
```

### 19.4. Template export

Template có thể export thành JSON để tái sử dụng ở project khác.

Ví dụ:

```text
Export Laptop Template
Import sang project Electronics Store khác
```

### 19.5. Import validation report

Sau khi import, hiển thị report:

```text
Total rows
Created
Updated
Skipped
Errors
```

Mỗi error cần có:

```text
Row number
Field
Error message
Suggested fix
```

---

## 20. Audit Log

Các action quan trọng phải ghi log.

### 20.1. Log events

```text
category.created
category.updated
category.archived
category.reordered
category.template_assigned

brand.created
brand.updated
brand.archived
brand.merged

attribute.created
attribute.updated
attribute.archived
attribute.option_updated

template.created
template.updated
template.published
template.duplicated
template.migrated
```

### 20.2. Log fields

```text
actor_id
action
entity_type
entity_id
before_snapshot
after_snapshot
created_at
ip_address
user_agent
```

### 20.3. Audit UI

Route đề xuất:

```text
/admin/catalog/audit-log
```

Filter:

```text
Actor
Action
Entity Type
Date Range
Keyword
```

---

## 21. Loading, Empty and Error States

## 21.1. Loading state

List page:

- Hiển thị skeleton table.
- Không nhảy layout.
- Filter bar vẫn giữ trạng thái.

Form page:

- Hiển thị skeleton input.
- Disable save button khi đang load.

Template builder:

- Hiển thị loading ở left panel và center panel.
- Không cho kéo thả khi data chưa load xong.

## 21.2. Empty state

Category empty:

```text
No categories yet
Create categories to organize your products.
```

Brand empty:

```text
No brands yet
Create brands to help customers filter products.
```

Attribute empty:

```text
No attributes yet
Create attributes to build dynamic product specifications.
```

Template empty:

```text
No templates yet
Create a template to generate product specification forms.
```

## 21.3. Error state

Error cần có:

- Message rõ ràng.
- Retry action.
- Không mất form data nếu save lỗi.
- Nếu validation lỗi, focus vào field đầu tiên bị lỗi.

---

## 22. Responsive Rules

Admin chủ yếu dùng desktop.

Nhưng vẫn cần tablet support.

### Desktop >= 1280px

- Sidebar cố định.
- Table full width.
- Template builder dùng 3 panel.
- Filter bar nằm ngang.

### Tablet 768px - 1279px

- Sidebar có thể collapse.
- Table có horizontal scroll.
- Template builder đổi thành 2 panel.
- Right settings panel mở dạng drawer.

### Mobile < 768px

Admin mobile không phải primary use case.

Nhưng vẫn cần không vỡ layout.

Rule:

- Sidebar thành drawer.
- Table thành card list nếu cần.
- Action menu gom vào kebab menu.
- Template builder có thể hiển thị message: `For best experience, use tablet or desktop.`
- Không được overflow ngang toàn page.

---

## 23. Accessibility Rules

### 23.1. Keyboard

Admin phải dùng được bằng keyboard cho các action cơ bản:

```text
Tab through inputs
Enter submit search
Esc close modal/drawer
Arrow key in tree if implemented
Space toggle checkbox
```

### 23.2. Form label

Mọi input phải có label rõ ràng.

Không dùng placeholder thay label.

### 23.3. Error message

Error phải gắn với field.

Ví dụ:

```text
Slug already exists.
```

Không chỉ hiện toast chung chung.

### 23.4. Color

Không chỉ dùng màu để biểu thị trạng thái.

Ví dụ badge phải có text:

```text
Active
Hidden
Archived
Required
Filterable
```

### 23.5. Drag and drop fallback

Nếu có drag and drop, cần có fallback:

```text
Move up
Move down
Move to group
```

---

## 24. Security Rules

- Tất cả API admin cần auth.
- Tất cả write API cần permission.
- Không tin dữ liệu từ frontend.
- Validate slug, code, URL ở backend.
- Sanitize rich text description.
- Không cho upload file nguy hiểm.
- Audit log action quan trọng.
- Rate limit API import.
- Import file cần giới hạn size.

---

## 25. Performance Rules

### 25.1. List page

- Có pagination.
- Không load toàn bộ category/attribute nếu dữ liệu lớn.
- Search debounce.
- Cache filter options nếu ít thay đổi.

### 25.2. Tree manager

- Category tree có thể lazy load node con.
- Nếu dưới 500 category, có thể load full tree.
- Nếu nhiều hơn, cần virtual tree hoặc lazy load.

### 25.3. Template builder

- Attribute list có search.
- Không render toàn bộ 1000 attributes cùng lúc nếu quá lớn.
- Kéo thả không làm lag.

### 25.4. Storefront sync

Khi template/category/attribute thay đổi:

- Invalidate cache category filter.
- Invalidate product specs cache nếu cần.
- Reindex search nếu attribute index thay đổi.

---

## 26. Analytics Events

Admin analytics giúp hiểu cách vận hành.

Events đề xuất:

```text
admin_category_created
admin_category_updated
admin_brand_created
admin_attribute_created
admin_attribute_updated
admin_template_created
admin_template_published
admin_template_assigned
admin_template_migration_started
admin_template_migration_completed
admin_import_completed
```

Event payload không chứa dữ liệu nhạy cảm.

---

## 27. Agent Implementation Rules

Khi agent implement module này, bắt buộc tuân thủ:

### 27.1. Không hard-code ngành hàng

Không code kiểu:

```text
if category == laptop then show CPU, RAM, SSD
```

Phải lấy từ template.

Đúng:

```text
Load category template
Render attributes by group and sort order
```

### 27.2. Không hard-code filter

Không code filter cố định theo Laptop.

Đúng:

```text
GET category filters
Render filter by attribute filter_ui_type
```

### 27.3. Không xóa dữ liệu để test pass

Không được xóa validation, permission, hoặc test để làm pass.

### 27.4. Form phải bảo toàn dữ liệu

Nếu save lỗi, form data vẫn còn.

Không làm mất input của admin.

### 27.5. UI phải có state đầy đủ

Mỗi page phải có:

```text
Loading
Empty
Error
Success
Permission denied
Validation error
Unsaved changes
```

### 27.6. Cần test tree và builder

Nếu implement category tree hoặc template builder, cần test:

- Reorder.
- Prevent circular parent.
- Unsaved changes.
- Save success.
- Save error.

---

## 28. Playwright Test Specification

## 28.1. Category tests

```text
Admin can view category list
Admin can search category
Admin can create root category
Admin can create child category
Admin can edit category name and slug
Admin cannot create duplicate slug
Admin cannot select itself as parent
Admin can assign template to category
Admin can hide category from storefront
Admin sees warning when deleting category with products
```

## 28.2. Category tree tests

```text
Admin can open category tree
Admin can expand and collapse nodes
Admin can move category up/down
Admin can drag category to another parent
System prevents circular parent
Unsaved changes warning appears
Save order persists after reload
```

## 28.3. Brand tests

```text
Admin can view brand list
Admin can create brand
Admin can upload logo
Admin can mark brand as featured
Admin cannot create duplicate slug
Admin cannot delete brand used by products
Admin can merge duplicate brand
```

## 28.4. Attribute tests

```text
Admin can view attribute list
Admin can create text attribute
Admin can create number attribute with unit
Admin can create select attribute with options
Admin cannot create select attribute without options
Admin cannot create duplicate code
Admin cannot change data type when product values exist
Admin can archive attribute
```

## 28.5. Attribute group tests

```text
Admin can create attribute group
Admin can edit group sort order
Admin can archive group
Template builder can use group
```

## 28.6. Template builder tests

```text
Admin can create attribute template
Admin can add group to template
Admin can add attribute to group
Admin can reorder attributes
Admin can mark attribute as required
Admin can mark attribute as filterable
Admin can mark attribute as quick spec
Admin can mark attribute as comparable
Admin can preview generated product form
Admin can publish template
Admin can duplicate template
```

## 28.7. Template assignment tests

```text
Admin can assign template to category
Child category can inherit parent template
Child category can override template
Changing parent template warns affected categories
Product form uses assigned category template
```

## 28.8. Storefront integration tests

```text
Category page renders filters from template
Product card renders quick specs from template
Product detail renders specs table grouped by template
Compare page renders comparable attributes
Archived attribute is not shown in new filter config
```

---

## 29. Visual Regression Checklist

Cần chụp screenshot cho:

```text
Category list desktop
Category tree desktop
Category create form
Brand list
Attribute list
Attribute create form
Template list
Template builder desktop
Template builder tablet
Category template assignment
Validation error state
Empty state
Permission denied state
```

Responsive screenshot:

```text
1440px desktop
1024px tablet
768px tablet
375px mobile sanity check
```

Mobile admin chỉ cần không vỡ layout, không cần trải nghiệm hoàn hảo như storefront.

---

## 30. Definition of Done

Module này chỉ được coi là xong khi:

```text
Category CRUD hoạt động
Category tree hoạt động hoặc có fallback move up/down
Brand CRUD hoạt động
Attribute CRUD hoạt động
Attribute option manager hoạt động
Attribute group CRUD hoạt động
Template builder hoạt động
Template assignment hoạt động
Product form có thể đọc template
Storefront filter có thể đọc template
Product detail specs có thể đọc template
Compare config có thể đọc template
Validation đầy đủ
Permission đầy đủ
Loading/empty/error đầy đủ
Audit log action quan trọng
Playwright tests pass
Visual tests không vỡ layout
Không hard-code attribute theo ngành hàng
```

---

## 31. MVP Scope

Nếu làm MVP, ưu tiên:

```text
Category CRUD
Brand CRUD
Attribute CRUD
Attribute options
Attribute template basic
Assign template to category
Product form render theo template
Product list filter render theo template
Product detail specs render theo template
```

Có thể hoãn:

```text
Template versioning nâng cao
Template migration nâng cao
Import/export
Audit log chi tiết
Drag and drop tree phức tạp
Brand merge
Advanced search boost
Advanced SEO per attribute
```

---

## 32. Kết luận

Module Category, Brand & Attribute Management là phần giúp source web bán hàng không bị khóa cứng vào một ngành hàng.

Với web đồ điện tử, module này giải quyết bài toán thông số kỹ thuật phức tạp.

Với source clone, module này là nền tảng để chuyển từ bán laptop sang điện thoại, tai nghe, linh kiện, đồ gia dụng, hoặc các ngành khác.

Nguyên tắc quan trọng nhất:

```text
Category quyết định sản phẩm thuộc nhóm nào.
Attribute định nghĩa thông tin có thể nhập.
Template quyết định category dùng attribute nào.
Product chỉ lưu giá trị thực tế.
Storefront đọc template để render UI.
Admin đọc template để render form.
```

Nếu agent code đúng theo nguyên tắc này, source sẽ tái sử dụng tốt và không bị phải sửa code mỗi khi thêm loại sản phẩm mới.
