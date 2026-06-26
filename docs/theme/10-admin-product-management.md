# 10-admin-product-management.md

# Admin Product Management Specification

> Dự án: Electronics Store Theme  
> Khu vực: Admin Panel  
> Màn hình: Quản lý sản phẩm  
> Mục tiêu: Đặc tả đủ chi tiết để agent/frontend/backend có thể code module quản lý sản phẩm từ đầu đến cuối.  
> Phụ thuộc: `01-electronics-store-theme.md`, `09-admin-dashboard.md`  
> Không phụ thuộc công nghệ frontend/backend cụ thể.

---

## 1. Mục đích của module

Module quản lý sản phẩm là trung tâm vận hành của website bán đồ điện tử.

Admin dùng module này để:

- Tạo sản phẩm mới.
- Sửa thông tin sản phẩm.
- Quản lý ảnh sản phẩm.
- Quản lý giá bán.
- Quản lý khuyến mãi.
- Quản lý biến thể sản phẩm.
- Quản lý thông số kỹ thuật.
- Quản lý tồn kho.
- Quản lý bảo hành.
- Quản lý trạng thái hiển thị ngoài storefront.
- Tối ưu SEO cho từng sản phẩm.
- Kiểm tra preview trước khi publish.

Với web bán đồ điện tử, sản phẩm không đơn giản chỉ có tên và giá. Mỗi sản phẩm thường có nhiều thông số kỹ thuật, nhiều phiên bản, nhiều màu, nhiều dung lượng, bảo hành, tình trạng tồn kho và chính sách bán hàng.

Vì vậy, module này phải được thiết kế theo hướng **linh hoạt theo danh mục**.

Ví dụ:

- Laptop có CPU, RAM, SSD, GPU, màn hình.
- Điện thoại có chip, RAM, bộ nhớ, camera, pin.
- Tai nghe có kết nối, chống ồn, thời lượng pin.
- Màn hình có kích thước, tần số quét, độ phân giải, tấm nền.

Không được hard-code form sản phẩm theo một loại hàng duy nhất.

---

## 2. Vai trò trong toàn hệ thống

Module này ảnh hưởng trực tiếp tới các khu vực sau:

- Storefront home page.
- Product list page.
- Product detail page.
- Cart page.
- Checkout page.
- Search.
- Filter.
- Compare.
- Recommendation.
- Inventory.
- Promotion.
- SEO.
- Analytics.

Nếu dữ liệu sản phẩm sai, toàn bộ trải nghiệm mua hàng sẽ sai.

Ví dụ:

- Sai giá làm checkout sai.
- Sai tồn kho làm khách đặt được hàng hết.
- Sai thông số làm filter sai.
- Sai ảnh làm giao diện kém tin cậy.
- Sai slug làm lỗi SEO.
- Sai trạng thái publish làm sản phẩm chưa sẵn sàng vẫn hiển thị.

---

## 3. Nguyên tắc thiết kế admin product

### 3.1. Rõ ràng hơn đẹp

Admin là nơi vận hành. Giao diện cần ưu tiên:

- Dễ đọc.
- Dễ tìm.
- Dễ sửa.
- Dễ kiểm tra lỗi.
- Dễ thao tác nhiều dữ liệu.

Không ưu tiên hiệu ứng trang trí.

### 3.2. Không để admin nhập sai dễ dàng

Form phải có validation rõ ràng.

Ví dụ:

- Giá bán không được âm.
- SKU không được trùng.
- Sản phẩm publish phải có ảnh chính.
- Sản phẩm publish phải có giá.
- Sản phẩm publish phải có danh mục.
- Sản phẩm publish phải có tồn kho hoặc cấu hình cho phép bán khi hết hàng.

### 3.3. Tách dữ liệu bán hàng và dữ liệu trình bày

Một sản phẩm có các nhóm dữ liệu khác nhau:

- Dữ liệu cơ bản.
- Dữ liệu giá.
- Dữ liệu ảnh.
- Dữ liệu kỹ thuật.
- Dữ liệu tồn kho.
- Dữ liệu SEO.
- Dữ liệu vận chuyển.
- Dữ liệu bảo hành.

Không nên dồn tất cả vào một form dài không có cấu trúc.

### 3.4. Luôn có draft trước khi publish

Admin nên có khả năng lưu nháp.

Trạng thái sản phẩm:

- Draft.
- Active.
- Hidden.
- Out of stock.
- Discontinued.
- Archived.

Không nên bắt admin nhập hoàn chỉnh mọi thứ ngay từ đầu.

### 3.5. Dữ liệu kỹ thuật phải dùng attribute động

Không hard-code field như `cpu`, `ram`, `ssd` trong bảng Product nếu mục tiêu là clone source cho nhiều ngành hàng.

Nên dùng cấu trúc:

- Category.
- AttributeGroup.
- Attribute.
- AttributeOption.
- ProductAttributeValue.
- AttributeTemplate.

---

## 4. Cấu trúc route admin

```text
/admin/products
/admin/products/new
/admin/products/:id
/admin/products/:id/edit
/admin/products/:id/preview
/admin/products/import
/admin/products/export
/admin/products/bulk-update
/admin/categories
/admin/brands
/admin/attributes
/admin/attribute-templates
/admin/inventory
/admin/warranty-policies
```

### 4.1. Route chính

`/admin/products`

Dùng để xem danh sách sản phẩm, tìm kiếm, lọc, thao tác hàng loạt.

### 4.2. Route tạo mới

`/admin/products/new`

Dùng để tạo sản phẩm mới. Form có thể lưu nháp.

### 4.3. Route chỉnh sửa

`/admin/products/:id/edit`

Dùng để chỉnh sửa sản phẩm đã tồn tại.

### 4.4. Route preview

`/admin/products/:id/preview`

Dùng để xem sản phẩm sẽ hiển thị ngoài storefront như thế nào.

---

## 5. Admin product list page

## 5.1. Mục đích

Trang danh sách sản phẩm giúp admin:

- Xem toàn bộ sản phẩm.
- Tìm kiếm theo tên, SKU, slug.
- Lọc theo danh mục, thương hiệu, trạng thái, tồn kho.
- Kiểm tra nhanh giá, tồn kho, trạng thái publish.
- Thao tác nhanh như ẩn, publish, duplicate, xóa mềm.
- Đi tới form chỉnh sửa.

---

## 5.2. Layout desktop

```text
┌────────────────────────────────────────────────────────────────────┐
│ Admin Topbar                                                       │
├───────────────┬────────────────────────────────────────────────────┤
│ Sidebar       │ Product Management                                 │
│               │                                                    │
│               │ [Search by name/SKU] [Category] [Brand] [Status]   │
│               │ [Stock] [Price] [More filters] [Create product]    │
│               │                                                    │
│               │ [Bulk actions] [Export] [Import]                   │
│               │                                                    │
│               │ Product Table                                      │
│               │                                                    │
│               │ Pagination                                         │
└───────────────┴────────────────────────────────────────────────────┘
```

### Layout notes

- Sidebar admin dùng theo rule trong `09-admin-dashboard.md`.
- Content area dùng max width linh hoạt.
- Table phải có horizontal scroll nếu thiếu chiều rộng.
- Header của table nên sticky khi danh sách dài.
- Filter bar nên sticky trên desktop nếu có nhiều sản phẩm.

---

## 5.3. Layout tablet

```text
┌─────────────────────────────────────┐
│ Topbar                              │
├─────────────────────────────────────┤
│ Product Management                  │
│ Search                              │
│ Filter row scroll horizontal        │
│ Product table compact               │
└─────────────────────────────────────┘
```

Tablet vẫn có thể dùng table, nhưng nên giảm cột mặc định.

Cột hiển thị mặc định:

- Product.
- SKU.
- Price.
- Stock.
- Status.
- Actions.

Các cột phụ đưa vào column settings.

---

## 5.4. Layout mobile

Mobile admin không phải ưu tiên chính, nhưng vẫn không được vỡ layout.

```text
┌─────────────────────────────┐
│ Topbar                      │
├─────────────────────────────┤
│ Product Management          │
│ Search                      │
│ [Filter] [Create]           │
│ Product cards               │
│ Product cards               │
│ Product cards               │
└─────────────────────────────┘
```

Mobile không nên dùng table đầy đủ.

Nên chuyển mỗi sản phẩm thành card compact.

---

## 6. Product list toolbar

Toolbar gồm:

- Search input.
- Category filter.
- Brand filter.
- Status filter.
- Stock filter.
- Price range filter.
- More filters.
- Create product button.
- Import button.
- Export button.

### 6.1. Search input

Placeholder:

```text
Search by product name, SKU, slug...
```

Search nên hỗ trợ:

- Product name.
- SKU.
- Slug.
- Brand.
- Model code.

Behavior:

- Debounce 300ms.
- Enter để search ngay.
- Có nút clear.
- Giữ query trên URL.

Ví dụ URL:

```text
/admin/products?search=macbook&status=active&page=1
```

### 6.2. Category filter

Dạng tree select.

Ví dụ:

```text
Electronics
  Laptop
    Gaming Laptop
    Office Laptop
  Phone
  Tablet
  Monitor
```

Rule:

- Chọn category cha thì có thể bao gồm category con.
- Có option `Include subcategories`.
- Nếu category có nhiều cấp, phải hiển thị breadcrumb.

### 6.3. Status filter

Các trạng thái:

- All.
- Draft.
- Active.
- Hidden.
- Out of stock.
- Discontinued.
- Archived.

### 6.4. Stock filter

Các trạng thái tồn kho:

- All.
- In stock.
- Low stock.
- Out of stock.
- Oversold.
- No inventory tracking.

### 6.5. More filters

Có thể mở drawer hoặc popover.

Các filter nâng cao:

- Created date.
- Updated date.
- Price range.
- Promotion active.
- Missing image.
- Missing SEO.
- Missing specs.
- Has variants.
- Warranty policy.
- Supplier.

---

## 7. Product table

## 7.1. Cột mặc định

| Cột | Nội dung |
|---|---|
| Checkbox | chọn dòng |
| Product | ảnh + tên |
| SKU | mã sản phẩm |
| Category | danh mục |
| Brand | thương hiệu |
| Price | giá bán |
| Stock | tồn kho |
| Status | trạng thái |
| Updated | thời gian |
| Actions | thao tác |

Không đưa câu dài vào table.

---

## 7.2. Product cell

Hiển thị:

- Thumbnail.
- Product name.
- Slug hoặc model code.
- Badge nếu thiếu dữ liệu.

Ví dụ:

```text
[Image]
Laptop Dell Inspiron 15 3520
/dell-inspiron-15-3520
Badge: Missing specs
```

Rule:

- Tên tối đa 2 dòng.
- Thumbnail fixed size.
- Nếu thiếu ảnh, hiển thị placeholder.
- Nếu sản phẩm archived, row giảm opacity nhẹ.

---

## 7.3. Price cell

Hiển thị:

- Price.
- Compare at price nếu có.
- Promotion badge nếu đang sale.

Ví dụ:

```text
15,990,000₫
18,990,000₫
Sale
```

Rule:

- Giá sale nổi bật hơn.
- Giá gốc gạch ngang.
- Nếu thiếu giá và trạng thái active, hiển thị warning.

---

## 7.4. Stock cell

Hiển thị:

- Available quantity.
- Reserved quantity.
- Low stock badge.

Ví dụ:

```text
Available: 12
Reserved: 2
```

Status badge:

- In stock.
- Low stock.
- Out of stock.
- Oversold.

---

## 7.5. Status cell

Badge màu theo trạng thái:

| Status | Ý nghĩa |
|---|---|
| Draft | nháp |
| Active | đang bán |
| Hidden | ẩn |
| Out of stock | hết hàng |
| Discontinued | ngừng bán |
| Archived | lưu trữ |

---

## 7.6. Actions cell

Actions chính:

- Edit.
- Preview.
- Duplicate.
- Hide.
- Publish.
- Archive.
- Delete soft.

Rule:

- Không hard delete sản phẩm mặc định.
- Delete nên là soft delete hoặc archive.
- Nếu sản phẩm đã có đơn hàng, không được xóa vật lý.

---

## 8. Bulk actions

Khi chọn nhiều sản phẩm, hiển thị bulk action bar.

Actions:

- Publish selected.
- Hide selected.
- Archive selected.
- Update category.
- Update brand.
- Update warranty policy.
- Update low stock threshold.
- Export selected.

Rule:

- Bulk action nguy hiểm phải confirm.
- Hiển thị số sản phẩm bị ảnh hưởng.
- Nếu action một phần fail, phải có report.

Ví dụ:

```text
20 products selected
[Publish] [Hide] [Archive] [Export]
```

---

## 9. Product create/edit page

## 9.1. Mục đích

Form tạo/sửa sản phẩm cần giúp admin nhập dữ liệu đầy đủ nhưng không quá rối.

Nên chia form thành các section:

1. Basic information.
2. Media.
3. Pricing.
4. Category and brand.
5. Variants.
6. Technical specifications.
7. Inventory.
8. Shipping.
9. Warranty.
10. SEO.
11. Visibility.
12. Preview.

---

## 9.2. Layout desktop

```text
┌──────────────────────────────────────────────────────────────────┐
│ Topbar                                                           │
├──────────────┬─────────────────────────────────────┬─────────────┤
│ Sidebar      │ Main form                           │ Side panel  │
│              │                                     │             │
│              │ Basic information                   │ Status      │
│              │ Media                               │ Publish     │
│              │ Pricing                             │ Preview     │
│              │ Variants                            │ Checklist   │
│              │ Technical specs                     │             │
│              │ Inventory                           │             │
│              │ Warranty                            │             │
│              │ SEO                                 │             │
└──────────────┴─────────────────────────────────────┴─────────────┘
```

Side panel sticky.

Side panel gồm:

- Save draft.
- Publish.
- Preview.
- Product status.
- Completion checklist.
- Last updated.
- Created by.

---

## 9.3. Layout mobile

Mobile form nên chuyển thành single column.

Side panel chuyển thành bottom action bar.

```text
[Save draft] [Preview] [Publish]
```

Nếu form có lỗi, publish button mở error summary.

---

## 10. Basic information section

Fields:

- Product name.
- Slug.
- SKU.
- Model code.
- Short description.
- Full description.
- Brand.
- Category.
- Tags.
- Status.

### 10.1. Product name

Type: text input.

Rule:

- Required khi publish.
- Max 180 ký tự.
- Nên rõ model và cấu hình chính.

Ví dụ tốt:

```text
Laptop Dell Inspiron 15 3520 i5 1235U 16GB 512GB
```

Ví dụ kém:

```text
Laptop Dell đẹp giá tốt
```

### 10.2. Slug

Type: text input.

Rule:

- Auto-generate từ name.
- Admin có thể sửa.
- Unique.
- Lowercase.
- Dùng dấu gạch ngang.
- Không có ký tự đặc biệt.

Ví dụ:

```text
dell-inspiron-15-3520-i5-16gb-512gb
```

Nếu slug trùng, gợi ý:

```text
dell-inspiron-15-3520-i5-16gb-512gb-2
```

### 10.3. SKU

Type: text input.

Rule:

- Required nếu quản lý kho.
- Unique.
- Uppercase recommended.
- Không dùng khoảng trắng.

Ví dụ:

```text
DELL-INS-3520-I5-16-512
```

### 10.4. Short description

Dùng để hiển thị ở product card hoặc summary.

Rule:

- Max 300 ký tự.
- Nên gồm cấu hình chính.
- Không dùng HTML phức tạp.

### 10.5. Full description

Type: rich text editor hoặc markdown editor.

Nội dung nên gồm:

- Tổng quan sản phẩm.
- Điểm nổi bật.
- Nhu cầu phù hợp.
- Mô tả tính năng.
- Chính sách đi kèm.

Rule:

- Không nhúng script.
- Sanitize HTML.
- Ảnh trong description phải có alt text.

---

## 11. Category and brand section

### 11.1. Category

Type: tree select.

Rule:

- Required khi publish.
- Một sản phẩm cần có primary category.
- Có thể có secondary categories nếu cần.
- Khi chọn category, load attribute template tương ứng.

Ví dụ:

```text
Electronics > Laptop > Office Laptop
```

### 11.2. Brand

Type: searchable select.

Rule:

- Required với đồ điện tử.
- Nếu brand chưa có, admin có quyền có thể tạo nhanh.

Brand fields:

- Name.
- Slug.
- Logo.
- Country.
- Status.

---

## 12. Media section

## 12.1. Mục đích

Ảnh sản phẩm ảnh hưởng trực tiếp đến niềm tin mua hàng.

Với đồ điện tử, ảnh phải rõ:

- Mặt trước.
- Mặt sau.
- Cạnh bên.
- Cổng kết nối.
- Ảnh thực tế nếu có.
- Ảnh cấu hình hoặc kích thước nếu cần.

---

## 12.2. Image uploader

Yêu cầu:

- Drag and drop.
- Multiple upload.
- Reorder bằng kéo thả.
- Set primary image.
- Alt text cho từng ảnh.
- Preview ảnh lớn.
- Delete ảnh.
- Replace ảnh.

### File rule

- Format: jpg, jpeg, png, webp.
- Max size tùy cấu hình.
- Recommended ratio: 1:1 hoặc 4:3.
- Tự tạo thumbnail.
- Tự optimize ảnh.

### Validation

- Publish required ít nhất 1 ảnh chính.
- Ảnh chính không được là ảnh lỗi.
- Mỗi ảnh nên có alt text.

---

## 12.3. Video media optional

Có thể hỗ trợ video review sản phẩm.

Fields:

- Video URL.
- Thumbnail.
- Title.

Rule:

- Chỉ allow nguồn trusted.
- Không autoplay có âm thanh.

---

## 13. Pricing section

Fields:

- Base price.
- Compare at price.
- Cost price.
- Currency.
- Tax class.
- Promotion price.
- Promotion start/end.
- Installment available.

### 13.1. Base price

Rule:

- Required khi publish.
- Number >= 0.
- Format theo currency.

Ví dụ:

```text
15,990,000₫
```

### 13.2. Compare at price

Dùng làm giá gốc gạch ngang.

Rule:

- Phải lớn hơn base price nếu hiển thị sale.
- Nếu nhỏ hơn base price, show validation error.

### 13.3. Cost price

Dùng nội bộ để tính lợi nhuận.

Rule:

- Không hiển thị ngoài storefront.
- Chỉ role có quyền mới thấy.

### 13.4. Promotion price

Có thể cấu hình theo thời gian.

Fields:

- Promotion price.
- Start date.
- End date.
- Promotion badge text.

Rule:

- Nếu hết hạn, tự không áp dụng.
- Nếu start date > end date, báo lỗi.

---

## 14. Variants section

## 14.1. Mục đích

Đồ điện tử thường có biến thể:

- Màu sắc.
- RAM.
- Storage.
- CPU.
- Connectivity.
- Region.

Ví dụ iPhone:

- Color: Black, White, Blue.
- Storage: 128GB, 256GB, 512GB.

Ví dụ laptop:

- RAM: 8GB, 16GB.
- SSD: 512GB, 1TB.

---

## 14.2. Variant model

Mỗi variant có:

- Variant name.
- SKU.
- Attribute combination.
- Price override.
- Compare price override.
- Stock quantity.
- Image override.
- Weight override.
- Status.

---

## 14.3. Variant generation

Admin chọn options rồi hệ thống generate combinations.

Ví dụ:

```text
Color: Black, Silver
Storage: 512GB, 1TB
```

Sinh ra:

```text
Black / 512GB
Black / 1TB
Silver / 512GB
Silver / 1TB
```

Rule:

- Không generate duplicate.
- Có thể disable một combination.
- Có thể sửa SKU từng variant.
- SKU variant phải unique.

---

## 14.4. Variant table

| Cột | Nội dung |
|---|---|
| Variant | tổ hợp |
| SKU | mã |
| Price | giá |
| Stock | tồn |
| Image | ảnh |
| Status | trạng thái |
| Actions | thao tác |

Không để cột quá dài.

---

## 15. Technical specifications section

## 15.1. Mục đích

Đây là phần cực kỳ quan trọng với đồ điện tử.

Thông số kỹ thuật được dùng cho:

- Trang chi tiết sản phẩm.
- Quick specs ở product card.
- Filter ở product list.
- Compare products.
- SEO structured content.
- Admin kiểm tra dữ liệu.

---

## 15.2. Attribute template theo category

Khi admin chọn category, hệ thống load template.

Ví dụ category Laptop:

```text
Processor
- CPU brand
- CPU model
- CPU generation

Memory
- RAM size
- RAM type

Storage
- Storage type
- Storage capacity

Display
- Screen size
- Resolution
- Refresh rate
- Panel type

Graphics
- GPU type
- GPU model

Battery
- Battery capacity
- Charging power

Physical
- Weight
- Material

Warranty
- Warranty months
```

Ví dụ category Phone:

```text
Performance
- Chipset
- RAM
- Storage

Display
- Screen size
- Panel type
- Refresh rate

Camera
- Main camera
- Front camera

Battery
- Capacity
- Charging power

Connectivity
- 5G
- SIM
- Wi-Fi

Warranty
- Warranty months
```

---

## 15.3. Attribute field types

| Type | Ví dụ |
|---|---|
| text | CPU model |
| number | weight |
| select | brand |
| multi_select | features |
| boolean | 5G |
| unit_number | screen size |
| rich_text | description |
| date | release date |

---

## 15.4. Attribute options

Với select/multi_select, admin nên chọn từ danh sách chuẩn.

Ví dụ RAM:

```text
4GB
8GB
16GB
32GB
64GB
```

Ví dụ storage:

```text
128GB
256GB
512GB
1TB
2TB
```

Rule:

- Không để admin nhập tự do nếu field cần filter.
- Nếu nhập tự do, dữ liệu filter sẽ bẩn.
- Cho phép admin có quyền tạo option mới.

---

## 15.5. Quick specs flag

Mỗi attribute có thể có flag:

- Show in product card.
- Show in quick specs.
- Show in compare.
- Use in filter.
- Required for publish.

Ví dụ Laptop quick specs:

- CPU.
- RAM.
- Storage.
- Screen.
- GPU.

Product card chỉ nên hiển thị 3 đến 5 specs chính.

---

## 15.6. Missing specs warning

Nếu sản phẩm active nhưng thiếu attribute required, hiển thị warning.

Ví dụ:

```text
Missing required specs: CPU, RAM, Storage
```

Side panel checklist cũng phải báo.

---

## 16. Inventory section

Fields:

- Track inventory.
- Available quantity.
- Reserved quantity.
- Low stock threshold.
- Allow backorder.
- Warehouse.
- Supplier.
- Restock date.

### 16.1. Track inventory

Nếu bật:

- Checkout phải kiểm tra tồn kho.
- Admin phải nhập quantity.
- Order tạo mới phải reserve stock.

Nếu tắt:

- Sản phẩm có thể bán không giới hạn.
- Nên hiển thị warning trong admin.

### 16.2. Low stock threshold

Nếu available quantity <= threshold, sản phẩm vào cảnh báo low stock.

Ví dụ:

```text
Available: 3
Low stock threshold: 5
Status: Low stock
```

### 16.3. Reserved quantity

Reserved là số hàng đã bị giữ bởi đơn đang xử lý.

Không nên cho admin sửa trực tiếp nếu không có quyền.

### 16.4. Stock movement

Nên có lịch sử thay đổi tồn kho.

Fields:

- Type.
- Quantity.
- Before.
- After.
- Reason.
- Related order.
- Created by.
- Created at.

Types:

- Import.
- Export.
- Reserve.
- Release.
- Adjust.
- Return.

---

## 17. Shipping section

Fields:

- Weight.
- Length.
- Width.
- Height.
- Shipping class.
- Fragile flag.
- Bulky item flag.
- Free shipping eligible.

### Rule

- Weight required nếu tính phí ship theo cân nặng.
- Dimension required nếu sản phẩm cồng kềnh.
- Với đồ điện tử dễ vỡ, có thể bật fragile flag.

---

## 18. Warranty section

Fields:

- Warranty policy.
- Warranty months.
- Warranty provider.
- Warranty type.
- Return period days.
- Service note.

Warranty type:

- Manufacturer warranty.
- Store warranty.
- Extended warranty.
- No warranty.

Rule:

- Đồ điện tử nên required warranty khi publish.
- Product detail page phải hiển thị warranty rõ.
- Order detail cần lưu snapshot warranty tại thời điểm mua.

---

## 19. SEO section

Fields:

- SEO title.
- Meta description.
- Canonical URL.
- Open Graph title.
- Open Graph description.
- Open Graph image.
- Indexable.

### 19.1. SEO title

Rule:

- Nếu trống, auto-generate từ product name.
- Nên có brand và model.

Ví dụ:

```text
Laptop Dell Inspiron 15 3520 i5 16GB 512GB chính hãng
```

### 19.2. Meta description

Rule:

- Nếu trống, auto-generate từ short description.
- Không quá dài.
- Nên chứa điểm nổi bật.

### 19.3. Open Graph image

Rule:

- Mặc định dùng primary image.
- Admin có thể override.

---

## 20. Visibility section

Fields:

- Status.
- Publish date.
- Unpublish date.
- Featured product.
- Show on home.
- Show in search.
- Show in category.
- Allow compare.
- Allow review.

### Status behavior

Draft:

- Không hiển thị ngoài storefront.
- Có thể preview trong admin.

Active:

- Hiển thị ngoài storefront.
- Search index active.

Hidden:

- Không hiển thị trong list/search.
- Có thể truy cập nếu có URL tùy cấu hình.

Out of stock:

- Có thể hiển thị nhưng disable buy button.

Discontinued:

- Hiển thị thông báo ngừng kinh doanh.
- Gợi ý sản phẩm thay thế.

Archived:

- Không chỉnh sửa bình thường.
- Chỉ xem lịch sử.

---

## 21. Completion checklist

Side panel cần hiển thị checklist trước khi publish.

Checklist:

- Product name exists.
- Slug unique.
- SKU unique.
- Category selected.
- Brand selected.
- Primary image exists.
- Base price valid.
- Required specs completed.
- Inventory configured.
- Warranty configured.
- SEO title exists.
- Product status valid.

Nếu thiếu item, Publish button có thể disabled hoặc mở confirm với warning tùy policy.

---

## 22. Save, publish, preview behavior

### 22.1. Save draft

- Cho phép lưu thiếu dữ liệu bắt buộc publish.
- Validate cơ bản để tránh dữ liệu hỏng.
- Không index search.
- Không hiển thị storefront.

### 22.2. Publish

Phải validate đầy đủ.

Nếu fail:

- Scroll đến error summary.
- Highlight field lỗi.
- Hiển thị checklist còn thiếu.

### 22.3. Preview

Preview dùng dữ liệu hiện tại.

Nếu chưa save:

- Có thể preview local draft.
- Hoặc yêu cầu save trước tùy kỹ thuật.

Preview phải gần giống product detail page thật.

---

## 23. Error summary

Khi submit form fail, hiển thị error summary ở đầu form.

Ví dụ:

```text
Cannot publish product.
Please fix 4 issues:
- Product image is required.
- SKU already exists.
- Base price is required.
- Missing required specs: CPU, RAM.
```

Mỗi error click được để scroll tới field tương ứng.

---

## 24. Data contract

## 24.1. Product object

```json
{
  "id": "prod_001",
  "name": "Laptop Dell Inspiron 15 3520 i5 16GB 512GB",
  "slug": "dell-inspiron-15-3520-i5-16gb-512gb",
  "sku": "DELL-INS-3520-I5-16-512",
  "model_code": "Inspiron 15 3520",
  "short_description": "Laptop văn phòng màn hình 15.6 inch, Core i5, RAM 16GB, SSD 512GB.",
  "description": "<p>...</p>",
  "category_id": "cat_laptop_office",
  "brand_id": "brand_dell",
  "status": "draft",
  "price": 15990000,
  "compare_at_price": 18990000,
  "currency": "VND",
  "images": [],
  "variants": [],
  "attributes": [],
  "inventory": {},
  "shipping": {},
  "warranty": {},
  "seo": {},
  "visibility": {},
  "created_at": "2026-06-22T00:00:00+07:00",
  "updated_at": "2026-06-22T00:00:00+07:00"
}
```

---

## 24.2. Product attribute value

```json
{
  "attribute_id": "attr_ram",
  "attribute_code": "ram",
  "label": "RAM",
  "value": "16GB",
  "unit": null,
  "display_value": "16GB",
  "show_in_card": true,
  "show_in_compare": true,
  "use_in_filter": true
}
```

---

## 24.3. Variant object

```json
{
  "id": "var_001",
  "product_id": "prod_001",
  "sku": "DELL-INS-3520-I5-16-512-SILVER",
  "name": "Silver / 16GB / 512GB",
  "options": [
    { "name": "Color", "value": "Silver" },
    { "name": "RAM", "value": "16GB" },
    { "name": "Storage", "value": "512GB" }
  ],
  "price": 15990000,
  "compare_at_price": 18990000,
  "stock_quantity": 12,
  "image_id": "img_001",
  "status": "active"
}
```

---

## 25. API contract

API chỉ là tham khảo. Có thể đổi theo framework.

### Product list

```http
GET /api/admin/products
```

Query params:

```text
search
category_id
brand_id
status
stock_status
missing_data
price_min
price_max
created_from
created_to
updated_from
updated_to
page
page_size
sort
```

### Product detail

```http
GET /api/admin/products/{id}
```

### Create product

```http
POST /api/admin/products
```

### Update product

```http
PATCH /api/admin/products/{id}
```

### Publish product

```http
POST /api/admin/products/{id}/publish
```

### Save draft

```http
POST /api/admin/products/{id}/save-draft
```

### Duplicate product

```http
POST /api/admin/products/{id}/duplicate
```

### Archive product

```http
POST /api/admin/products/{id}/archive
```

### Upload product image

```http
POST /api/admin/products/{id}/images
```

### Reorder images

```http
PATCH /api/admin/products/{id}/images/reorder
```

### Generate variants

```http
POST /api/admin/products/{id}/variants/generate
```

### Validate product before publish

```http
POST /api/admin/products/{id}/validate-publish
```

---

## 26. Validation rules

## 26.1. Draft validation

Draft cho phép thiếu nhiều field.

Vẫn phải validate:

- Slug format nếu có.
- SKU format nếu có.
- Price không âm nếu có.
- Date range hợp lệ nếu có.
- JSON/rich text an toàn.

## 26.2. Publish validation

Publish yêu cầu:

- Name required.
- Slug required and unique.
- SKU required and unique.
- Category required.
- Brand required.
- Primary image required.
- Price required.
- Required specs completed.
- Inventory valid.
- Warranty valid.
- SEO valid hoặc auto-generated.

---

## 27. Permissions

Roles tham khảo:

- Super Admin.
- Product Manager.
- Content Editor.
- Inventory Manager.
- Support.
- Viewer.

| Permission | Super | Product | Content | Inventory | Support | Viewer |
|---|---|---|---|---|---|---|
| product.view | yes | yes | yes | yes | yes | yes |
| product.create | yes | yes | yes | no | no | no |
| product.update | yes | yes | yes | no | no | no |
| product.publish | yes | yes | no | no | no | no |
| product.archive | yes | yes | no | no | no | no |
| price.update | yes | yes | no | no | no | no |
| inventory.update | yes | no | no | yes | no | no |
| seo.update | yes | yes | yes | no | no | no |

Rule:

- Không hiển thị action nếu user không có quyền.
- Backend vẫn phải kiểm tra permission.
- UI permission không thay thế backend security.

---

## 28. Loading state

### Product list loading

- Hiển thị skeleton table rows.
- Toolbar vẫn visible.
- Không hiển thị fake data.

### Product form loading

- Hiển thị skeleton từng section.
- Nếu load detail fail, hiển thị retry.

### Image upload loading

- Mỗi ảnh có progress riêng.
- Nếu upload fail, ảnh đó hiển thị retry.

---

## 29. Empty state

### No products

Message:

```text
No products yet.
Create your first product to start selling.
```

Actions:

- Create product.
- Import products.

### No search results

Message:

```text
No products match your filters.
Try changing search keywords or clearing filters.
```

Actions:

- Clear filters.

---

## 30. Error state

Các lỗi thường gặp:

- Cannot load products.
- Cannot save product.
- SKU already exists.
- Slug already exists.
- Image upload failed.
- Invalid price.
- Publish validation failed.
- Permission denied.

Rule:

- Error phải rõ hành động tiếp theo.
- Có retry nếu lỗi network.
- Không mất dữ liệu form khi save fail.
- Với lỗi validation, giữ nguyên input của admin.

---

## 31. Confirmation dialogs

Các action cần confirm:

- Archive product.
- Delete image.
- Remove variant.
- Bulk hide.
- Bulk archive.
- Publish product thiếu một số warning không nghiêm trọng.

Dialog phải hiển thị:

- Tên action.
- Số item bị ảnh hưởng.
- Hậu quả.
- Button cancel.
- Button confirm.

Ví dụ:

```text
Archive 12 products?
Archived products will not appear in storefront.
You can restore them later.
```

---

## 32. Audit log

Module sản phẩm cần ghi lịch sử thay đổi quan trọng.

Log events:

- Product created.
- Product updated.
- Product published.
- Product hidden.
- Product archived.
- Price changed.
- Stock adjusted.
- Warranty changed.
- SEO changed.
- Image added.
- Image removed.

Log fields:

- Actor.
- Action.
- Entity.
- Before.
- After.
- Timestamp.
- IP optional.

Admin có thể xem audit log trong product detail.

---

## 33. Import / export

## 33.1. Export

Export fields:

- Product ID.
- Name.
- SKU.
- Category.
- Brand.
- Price.
- Stock.
- Status.
- Attributes.
- SEO.

Formats:

- CSV.
- XLSX optional.

## 33.2. Import

Import flow:

1. Upload file.
2. Validate columns.
3. Preview imported rows.
4. Show errors per row.
5. Confirm import.
6. Display summary.

Rule:

- Không import thẳng nếu có lỗi nghiêm trọng.
- Có dry-run preview.
- Có template download.
- Có rollback hoặc import batch log.

---

## 34. Admin UX details

### 34.1. Auto-save optional

Nếu bật auto-save:

- Chỉ auto-save draft.
- Không auto-publish.
- Hiển thị trạng thái `Saving...`, `Saved`, `Save failed`.

### 34.2. Unsaved changes guard

Nếu admin sửa form chưa lưu rồi rời trang:

- Hiển thị confirm.
- Không mất dữ liệu.

### 34.3. Keyboard shortcuts optional

Có thể hỗ trợ:

- Ctrl/Cmd + S: save draft.
- Ctrl/Cmd + Enter: publish nếu hợp lệ.
- Esc: close drawer/modal.

---

## 35. Accessibility rules

- Tất cả input có label.
- Error message liên kết với input.
- Button có accessible name.
- Table có header đúng.
- Modal trap focus.
- Upload area dùng được bằng keyboard.
- Color không phải cách duy nhất truyền đạt trạng thái.
- Form error summary focus được sau submit fail.
- Image alt text có thể nhập và validate.

---

## 36. Responsive rules

### Desktop

- Sidebar visible.
- Product table đầy đủ.
- Product form hai vùng: main form + side panel.

### Tablet

- Table compact.
- Cột phụ có thể ẩn.
- Side panel có thể nằm dưới hoặc sticky hẹp.

### Mobile

- Product list chuyển thành card.
- Filter chuyển thành drawer.
- Product form single column.
- Action bar sticky bottom.
- Không overflow ngang.

---

## 37. Performance rules

- Product list phải phân trang.
- Không load toàn bộ sản phẩm một lần.
- Search debounce.
- Ảnh dùng thumbnail trong table.
- Form edit chỉ load detail cần thiết.
- Attribute options lớn phải search async.
- Không render rich editor nếu section chưa mở nếu performance kém.
- Upload ảnh có compression hoặc backend optimization.

---

## 38. Security rules

- Sanitize rich text description.
- Validate upload file type.
- Validate upload file size.
- Không trust frontend price.
- Backend phải validate permission.
- Không để user role thấp sửa giá nếu không có quyền.
- Audit mọi thay đổi nhạy cảm.
- CSRF protection nếu dùng cookie session.
- Không leak cost price cho role không có quyền.

---

## 39. Analytics events

Admin analytics optional, nhưng nên có event để hiểu thao tác vận hành.

Events:

```text
admin_product_list_viewed
admin_product_search_used
admin_product_filter_applied
admin_product_created
admin_product_saved_draft
admin_product_published
admin_product_archived
admin_product_image_uploaded
admin_product_variant_generated
admin_product_import_started
admin_product_import_completed
```

Event payload không chứa dữ liệu nhạy cảm như cost price nếu không cần.

---

## 40. Agent implementation rules

Khi agent code module này, bắt buộc tuân thủ:

1. Đọc `01-electronics-store-theme.md` trước.
2. Đọc `09-admin-dashboard.md` trước.
3. Không tự ý đổi design token.
4. Không hard-code thông số laptop vào Product base model.
5. Dùng attribute template cho thông số kỹ thuật.
6. Tách form thành section/component nhỏ.
7. Không dồn toàn bộ vào một file component khổng lồ.
8. Không xóa validation để làm test pass.
9. Không dùng CSS global bừa bãi.
10. Không hard delete sản phẩm nếu đã có đơn hàng.
11. Không để publish sản phẩm thiếu dữ liệu bắt buộc.
12. Không làm mobile overflow ngang.
13. Nếu sửa UI, phải chạy visual hoặc screenshot test.
14. Nếu sửa form, phải test validation.
15. Nếu sửa API mapping, phải test data contract.

---

## 41. Suggested component structure

Tên component không bắt buộc, nhưng nên tách như sau:

```text
AdminProductListPage
AdminProductToolbar
AdminProductFilterDrawer
AdminProductTable
AdminProductRow
AdminProductMobileCard
AdminBulkActionBar
AdminProductFormPage
ProductBasicInfoSection
ProductMediaSection
ProductPricingSection
ProductCategoryBrandSection
ProductVariantSection
ProductSpecsSection
ProductInventorySection
ProductShippingSection
ProductWarrantySection
ProductSeoSection
ProductVisibilitySection
ProductCompletionChecklist
ProductPublishPanel
ProductErrorSummary
ProductAuditLog
```

---

## 42. Playwright test specification

## 42.1. Product list tests

Test cases:

- Admin can view product list.
- Admin can search by product name.
- Admin can search by SKU.
- Admin can filter by category.
- Admin can filter by status.
- Admin can filter by stock status.
- Admin can open product edit page.
- Admin can preview product.
- Empty state appears when no products exist.
- Error state appears when API fails.
- Mobile product list does not overflow horizontally.

## 42.2. Product create tests

Test cases:

- Admin can open create product page.
- Admin can save draft with minimal information.
- Admin cannot publish without required fields.
- Error summary appears when publish validation fails.
- Admin can create valid product draft.
- Admin can publish valid product.

## 42.3. Product edit tests

Test cases:

- Admin can edit product name.
- Admin can update price.
- Admin can update status.
- Admin can upload image.
- Admin can set primary image.
- Admin can update technical specs.
- Admin can update inventory.
- Admin can update warranty.
- Unsaved changes guard appears.

## 42.4. Variant tests

Test cases:

- Admin can add variant option.
- Admin can generate variant combinations.
- Duplicate variant SKU shows validation error.
- Admin can disable one variant.
- Variant stock appears correctly.

## 42.5. Permission tests

Test cases:

- Viewer can view but cannot edit.
- Content editor cannot update price if not allowed.
- Inventory manager can update stock but not description.
- Unauthorized user is redirected or blocked.

---

## 43. Visual regression checklist

Capture screenshots for:

- Product list desktop.
- Product list mobile.
- Empty product list.
- Product form desktop.
- Product form mobile.
- Error summary state.
- Media upload section.
- Variant table.
- Technical specs section.
- Completion checklist.
- Publish validation fail.

Viewports:

```text
1440px desktop
1024px laptop
768px tablet
375px mobile
```

Must verify:

- No horizontal overflow.
- Sticky side panel works.
- Table/card layout readable.
- Form fields aligned.
- Error messages visible.
- Buttons not covered.

---

## 44. Definition of Done

Một implementation được coi là xong khi:

- Product list page hoạt động.
- Search/filter/sort/pagination hoạt động.
- Create product form hoạt động.
- Edit product form hoạt động.
- Save draft hoạt động.
- Publish validation hoạt động.
- Image upload hoạt động hoặc có mock rõ ràng.
- Variant management hoạt động ở mức MVP.
- Technical specs dùng attribute template.
- Inventory section hoạt động.
- Warranty section hoạt động.
- SEO section hoạt động.
- Permission UI hoạt động.
- Error/loading/empty states đầy đủ.
- Responsive không vỡ layout.
- Playwright tests chính pass.
- Visual snapshots không có diff bất thường.
- Không có console error nghiêm trọng.
- Không có dữ liệu nhạy cảm bị lộ sai role.

---

## 45. MVP scope

Nếu làm MVP trước, chỉ cần:

- Product list.
- Search theo name/SKU.
- Filter category/status.
- Create/edit basic info.
- Upload primary image.
- Price.
- Category.
- Brand.
- Basic specs.
- Inventory quantity.
- Warranty months.
- Save draft.
- Publish.
- Hide.

Chưa cần ngay:

- Import/export nâng cao.
- Bulk update phức tạp.
- Auto-save.
- Multi-warehouse.
- Advanced promotion price.
- Full audit diff UI.
- AI content suggestion.
- Advanced SEO schema editor.

---

## 46. Future extension

Sau MVP có thể mở rộng:

- AI generate product description.
- AI normalize technical specs.
- Duplicate detection.
- Product bundle.
- Accessory recommendation.
- Cross-sell rule.
- Dynamic pricing.
- Supplier integration.
- Marketplace sync.
- Multi-language product content.
- Advanced product comparison templates.

---

## 47. Ghi chú cho source clone nhiều ngành hàng

Module này phải giữ phần lõi dùng chung:

- Product.
- Category.
- Brand.
- Media.
- Price.
- Variant.
- Attribute.
- Inventory.
- SEO.
- Visibility.

Phần riêng ngành hàng nằm ở:

- Attribute template.
- Category tree.
- Product card quick specs.
- Filter fields.
- Compare fields.
- Warranty policy.
- Theme copywriting.

Vì vậy, khi clone sang ngành khác như thời trang, mỹ phẩm, nội thất, không sửa core product management. Chỉ thay attribute template và theme-specific display rules.

