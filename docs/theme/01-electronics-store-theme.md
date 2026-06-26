# 01 - Electronics Store Theme

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Tài liệu này là **lớp theme chuyên ngành đồ điện tử** dựa trên file gốc `00 - Ngôn ngữ thiết kế chung cho website bán hàng và trang admin`.
>
> Mục tiêu: khi đưa tài liệu này cho coding agent/frontend agent, agent có thể biến design system chung thành giao diện bán đồ điện tử có tính **hiện đại, rõ thông số, đáng tin, dễ so sánh, dễ mua**, đồng thời vẫn tái sử dụng được source cho laptop, điện thoại, linh kiện, phụ kiện, thiết bị gia dụng điện tử.

---

## 0. Cách dùng tài liệu này

Tài liệu này **không thay thế** design system gốc. Nó chỉ là lớp mở rộng theo ngành hàng.

Thứ tự ưu tiên khi agent/code đọc tài liệu:

```text
../main/ecommerce_design_language.md
↓
01-electronics-store-theme.md
↓
Page spec cụ thể
↓
Component spec cụ thể
↓
Test spec cụ thể
```

Nếu có xung đột:

```text
Rule accessibility, responsive, spacing core của file gốc được ưu tiên.
Rule ngành hàng điện tử trong file này dùng để override màu, layout density, product information, filter, specs, compare.
Rule trong page spec cụ thể được ưu tiên cho màn hình đó.
```

---

## 1. Định vị giao diện ngành đồ điện tử

### 1.1. Cảm giác thương hiệu

Giao diện bán đồ điện tử phải tạo cảm giác:

```text
Hiện đại
Rõ ràng
Đáng tin
Có tính công nghệ
Nhiều thông tin nhưng dễ quét mắt
Dễ so sánh giữa các model
Dễ kiểm tra giá, khuyến mãi, bảo hành
```

Khách mua đồ điện tử thường ra quyết định dựa trên:

```text
Giá
Thông số kỹ thuật
Thương hiệu
Bảo hành
Tình trạng còn hàng
Khuyến mãi
Đánh giá
Nhu cầu sử dụng
Khả năng so sánh model
```

Vì vậy UI không được chỉ đẹp kiểu cảm xúc. UI phải giúp khách trả lời nhanh:

```text
Sản phẩm này mạnh không?
Có đúng nhu cầu của tôi không?
Giá này có tốt không?
Có còn hàng không?
Bảo hành thế nào?
Có trả góp không?
Có sản phẩm nào tương tự tốt hơn không?
```

### 1.2. Không khí thị giác

Electronics theme nên đi theo hướng:

```text
Tech clean
Data-rich
High contrast vừa phải
Card rõ khối
CTA nổi bật
Spec table dễ đọc
Filter mạnh
Admin gọn và chính xác
```

Không nên đi theo hướng:

```text
Quá nhiều gradient
Quá nhiều animation
Quá cute
Quá luxury tối giản đến mức thiếu thông tin
Quá nhiều màu sale làm rối
Layout quá thoáng khiến đọc thông số chậm
```

### 1.3. Nguyên tắc quan trọng nhất

```text
Đồ điện tử = mua bằng thông tin + niềm tin.
```

Vì vậy mỗi màn hình phải ưu tiên:

1. **Thông tin sản phẩm rõ**.
2. **Giá và ưu đãi dễ thấy**.
3. **Thông số kỹ thuật dễ scan**.
4. **Tồn kho và bảo hành minh bạch**.
5. **So sánh sản phẩm dễ thao tác**.
6. **Checkout không rối**.

---

## 2. Phạm vi ngành hàng

Theme này áp dụng tốt cho:

```text
Laptop
Điện thoại
Máy tính bảng
Màn hình
Tai nghe
Loa
Phụ kiện máy tính
Linh kiện PC
Thiết bị mạng
Camera
Máy in
Smart home
Đồ gia dụng điện tử
```

### 2.1. MVP khuyến nghị

Để bắt đầu code source clone, nên chọn MVP hẹp:

```text
Laptop Store MVP
```

Lý do laptop có đủ bài toán:

```text
Danh mục
Thương hiệu
Thông số kỹ thuật
Biến thể
Bảo hành
Tồn kho
Giá sale
Trả góp
So sánh
Checkout
Admin quản lý thuộc tính
```

Khi làm tốt laptop, có thể clone sang điện thoại, màn hình, phụ kiện bằng cách đổi category attribute template.

---

## 3. Design token override cho Electronics Theme

### 3.1. Ý tưởng token

Không sửa trực tiếp component. Theme chỉ override token.

Ví dụ không viết:

```css
.product-price {
  color: #dc2626;
}
```

Phải viết:

```css
.product-price {
  color: var(--commerce-price-strong);
}
```

Theme điện tử sẽ định nghĩa lại token:

```css
:root[data-theme="electronics"] {
  --commerce-price-strong: #dc2626;
}
```

---

## 4. Hệ màu cho Electronics Theme

### 4.1. Mục tiêu màu

Màu của web điện tử cần làm được 4 việc:

1. Tạo cảm giác công nghệ và tin cậy.
2. Làm nổi CTA mua hàng.
3. Làm rõ giá, sale, bảo hành, tồn kho.
4. Không làm rối các bảng thông số.

### 4.2. Bảng màu chủ đạo

| Nhóm | Token | Giá trị |
|---|---|---:|
| Brand | brand-50 | #eff6ff |
| Brand | brand-100 | #dbeafe |
| Brand | brand-200 | #bfdbfe |
| Brand | brand-300 | #93c5fd |
| Brand | brand-400 | #60a5fa |
| Brand | brand-500 | #3b82f6 |
| Brand | brand-600 | #2563eb |
| Brand | brand-700 | #1d4ed8 |
| Brand | brand-800 | #1e40af |
| Brand | brand-900 | #1e3a8a |
| Navy | navy-900 | #0f172a |
| Navy | navy-800 | #1e293b |
| Navy | navy-700 | #334155 |
| Neutral | neutral-50 | #f8fafc |
| Neutral | neutral-100 | #f1f5f9 |
| Neutral | neutral-200 | #e2e8f0 |
| Neutral | neutral-300 | #cbd5e1 |
| Neutral | neutral-500 | #64748b |
| Neutral | neutral-700 | #334155 |
| Neutral | neutral-900 | #0f172a |
| Sale | sale-500 | #ef4444 |
| Sale | sale-600 | #dc2626 |
| Promo | promo-500 | #f97316 |
| Success | success-500 | #22c55e |
| Warning | warning-500 | #f59e0b |
| Danger | danger-500 | #ef4444 |
| Info | info-500 | #0ea5e9 |

### 4.3. Semantic usage

| Mục đích | Token dùng |
|---|---|
| CTA chính | brand-600 |
| CTA hover | brand-700 |
| Header dark | navy-900 |
| Link | brand-600 |
| Giá bán | sale-600 |
| Giá gốc | neutral-500 |
| Badge sale | sale-600 |
| Badge trả góp | promo-500 |
| Còn hàng | success-500 |
| Sắp hết hàng | warning-500 |
| Hết hàng | neutral-500 |
| Lỗi | danger-500 |
| Border card | neutral-200 |
| Nền page | neutral-50 |
| Text chính | neutral-900 |
| Text phụ | neutral-500 |

### 4.4. Token CSS gợi ý

```css
:root[data-theme="electronics"] {
  /* Brand */
  --color-brand-50: #eff6ff;
  --color-brand-100: #dbeafe;
  --color-brand-200: #bfdbfe;
  --color-brand-300: #93c5fd;
  --color-brand-400: #60a5fa;
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;
  --color-brand-800: #1e40af;
  --color-brand-900: #1e3a8a;

  /* Neutral */
  --color-bg-page: #f8fafc;
  --color-bg-surface: #ffffff;
  --color-bg-muted: #f1f5f9;
  --color-border-default: #e2e8f0;
  --color-border-strong: #cbd5e1;
  --color-text-primary: #0f172a;
  --color-text-secondary: #334155;
  --color-text-muted: #64748b;
  --color-text-inverse: #ffffff;

  /* Commerce */
  --commerce-price-strong: #dc2626;
  --commerce-price-normal: #0f172a;
  --commerce-price-old: #64748b;
  --commerce-sale-bg: #fee2e2;
  --commerce-sale-text: #dc2626;
  --commerce-promo-bg: #ffedd5;
  --commerce-promo-text: #c2410c;
  --commerce-stock-in: #16a34a;
  --commerce-stock-low: #d97706;
  --commerce-stock-out: #64748b;

  /* Admin */
  --admin-sidebar-bg: #0f172a;
  --admin-sidebar-active-bg: #1d4ed8;
  --admin-header-bg: #ffffff;
  --admin-table-header-bg: #f8fafc;
}
```

### 4.5. Rule không dùng màu

Không dùng màu theo kiểu cảm tính:

```text
Đỏ cho mọi thứ nổi bật
Cam cho mọi nút
Gradient nhiều màu ở card sản phẩm
Background tối toàn trang storefront
```

Rule:

```text
CTA chính dùng brand.
Giá sale dùng sale.
Promotion dùng promo.
Status dùng semantic.
Thông số dùng neutral.
```

---

## 5. Typography cho Electronics Theme

### 5.1. Mục tiêu typography

Web đồ điện tử có nhiều số và thông số, nên typography phải:

```text
Rõ số
Dễ đọc chữ nhỏ
Không quá trang trí
Có phân cấp mạnh giữa tên sản phẩm, giá và specs
```

### 5.2. Font gợi ý

Không ép framework. Có thể dùng một trong các nhóm:

```text
Inter
Roboto
System UI
Noto Sans
```

Nếu web phục vụ tiếng Việt, nên kiểm tra dấu tiếng Việt ở:

```text
Tên sản phẩm dài
Mô tả kỹ thuật
Button
Badge
Bảng thông số
```

### 5.3. Quy mô chữ storefront

| Element | Size | Weight |
|---|---:|---:|
| Page title | 28-36px | 700 |
| Section title | 20-24px | 700 |
| Product name detail | 24-30px | 700 |
| Product card name | 14-16px | 600 |
| Price detail | 28-34px | 800 |
| Price card | 18-22px | 800 |
| Old price | 13-15px | 400 |
| Quick specs | 12-13px | 400-500 |
| Body text | 14-16px | 400 |
| Badge | 11-12px | 600 |
| Button | 14-16px | 600 |

### 5.4. Quy mô chữ admin

| Element | Size | Weight |
|---|---:|---:|
| Admin page title | 22-28px | 700 |
| Table header | 12-13px | 700 |
| Table cell | 13-14px | 400-500 |
| Form label | 13-14px | 600 |
| Helper text | 12-13px | 400 |
| Status badge | 11-12px | 600 |
| Metric number | 24-32px | 800 |

### 5.5. Product name rule

Tên sản phẩm đồ điện tử thường dài:

```text
Laptop Dell Inspiron 15 3520 Intel Core i5-1235U 16GB 512GB SSD 15.6 inch FHD Windows 11
```

Rule:

```text
Product card desktop: tối đa 2 dòng.
Product card mobile: tối đa 2 dòng.
Product detail: cho phép 2-3 dòng.
Admin table: ellipsis 1 dòng, có tooltip hoặc mở detail.
Search suggestion: 1-2 dòng.
```

Tên sản phẩm không được phá layout card.

---

## 6. Spacing, density và layout feeling

### 6.1. Density

Electronics theme dùng density trung bình-cao.

Không quá thoáng như luxury. Không quá chật như admin dữ liệu.

```text
Storefront: medium density
Admin: compact-medium density
Product specs: compact but readable
Checkout: medium, ưu tiên dễ điền form
```

### 6.2. Spacing gợi ý

| Context | Spacing |
|---|---:|
| Page section gap | 32-48px |
| Card gap desktop | 16-20px |
| Card gap mobile | 12-16px |
| Product card padding | 12-16px |
| Detail page column gap | 32-48px |
| Form field gap | 12-16px |
| Admin table cell padding | 10-14px |
| Spec table row padding | 10-12px |

### 6.3. Radius

Electronics theme nên dùng radius vừa phải.

| Component | Radius |
|---|---:|
| Button | 8-10px |
| Product card | 12-16px |
| Input | 8-10px |
| Badge | 999px hoặc 6px |
| Modal | 16px |
| Admin table container | 12px |
| Image container | 12px |

Rule:

```text
Không dùng radius quá tròn cho toàn bộ UI.
Không dùng card vuông cứng quá mức nếu không có chủ đích B2B.
```

### 6.4. Shadow

Electronics storefront cần shadow nhẹ để phân card.

```text
Default card: shadow rất nhẹ hoặc border.
Hover card: shadow rõ hơn + border brand nhẹ.
Admin: ưu tiên border hơn shadow.
```

---

## 7. Icon language

### 7.1. Phong cách icon

Icon nên là line icon hoặc minimal solid icon.

Nên dùng icon rõ nghĩa cho:

```text
Cart
Search
User
Compare
Wishlist
Warranty
Delivery
Installment
Stock
Specs
Filter
Sort
Order status
Payment
```

### 7.2. Rule icon

```text
Icon không thay thế text ở chức năng quan trọng.
Icon trong button chính cần có label.
Icon đơn độc phải có tooltip hoặc aria-label.
Icon kỹ thuật không được quá decorative.
```

Ví dụ nút so sánh:

```text
Desktop: [icon compare] So sánh
Mobile: icon + aria-label, hoặc label ngắn nếu đủ chỗ
```

---

## 8. Storefront layout chính

### 8.1. Header storefront

Header là trung tâm điều hướng mua hàng.

#### Desktop header

Cấu trúc khuyến nghị:

```text
Top bar
- Hotline
- Chính sách bảo hành
- Tra cứu đơn hàng
- Hệ thống cửa hàng

Main header
- Logo
- Search bar lớn
- Account
- Compare
- Cart

Navigation
- Danh mục sản phẩm
- Laptop
- Điện thoại
- Phụ kiện
- Khuyến mãi
- Hàng mới
```

#### Mobile header

Cấu trúc:

```text
Row 1: Menu icon + Logo + Cart
Row 2: Search bar full width
```

Rule mobile:

```text
Search phải dễ bấm.
Cart luôn thấy được.
Menu mở drawer.
Compare có thể nằm trong account/menu nếu thiếu chỗ.
```

### 8.2. Mega menu danh mục

Web điện tử nên dùng mega menu vì nhiều nhóm hàng.

Ví dụ:

```text
Laptop
- Laptop văn phòng
- Laptop gaming
- Laptop đồ họa
- Laptop mỏng nhẹ
- MacBook
- Laptop theo giá

Điện thoại
- iPhone
- Samsung
- Xiaomi
- Oppo
- Điện thoại dưới 5 triệu

Phụ kiện
- Chuột
- Bàn phím
- Tai nghe
- Sạc
- Cáp
```

Rule:

```text
Danh mục không được quá sâu trên menu chính.
Tối đa 2-3 cấp trên giao diện.
Danh mục sâu hơn để trong trang category/filter.
```

### 8.3. Search bar

Search là chức năng cực quan trọng.

Search bar desktop:

```text
Chiếm trung tâm header.
Có placeholder cụ thể.
Có icon search.
Có suggest dropdown.
```

Placeholder gợi ý:

```text
Tìm laptop, điện thoại, linh kiện...
```

Search suggestion nên có:

```text
Từ khóa phổ biến
Sản phẩm gợi ý
Danh mục gợi ý
Thương hiệu gợi ý
```

Rule:

```text
Không để search quá nhỏ.
Không ẩn search sau icon ở desktop.
Mobile được phép có search row riêng.
```

---

## 9. Trang chủ Electronics Store

### 9.1. Mục tiêu trang chủ

Trang chủ phải trả lời nhanh:

```text
Shop bán gì?
Có chương trình gì nổi bật?
Danh mục chính là gì?
Sản phẩm nào đáng mua?
Shop có đáng tin không?
```

### 9.2. Cấu trúc trang chủ khuyến nghị

```text
Header
Hero banner / campaign banner
Category shortcut
Flash sale / deal nổi bật
Laptop nổi bật
Điện thoại nổi bật
Phụ kiện nổi bật
Brand strip
Lý do mua tại shop
Review / trust section
Footer
```

### 9.3. Hero banner

Hero banner nên rõ campaign:

```text
Back to School
Gaming Laptop Sale
iPhone New Arrival
Deal cuối tuần
Trả góp 0%
```

Hero không nên quá nhiều chữ.

Hero nên có:

```text
Headline
Sub headline
CTA
Ảnh sản phẩm/campaign
Badge sale nếu có
```

CTA gợi ý:

```text
Mua ngay
Xem ưu đãi
Khám phá laptop gaming
```

### 9.4. Category shortcut

Nên dùng icon/card danh mục:

```text
Laptop
Điện thoại
Màn hình
Tai nghe
Chuột bàn phím
Linh kiện PC
Phụ kiện
Thiết bị mạng
```

Rule:

```text
Desktop: 6-8 item / row.
Mobile: horizontal scroll hoặc 4 item / row.
Card danh mục phải có icon/ảnh rõ.
```

### 9.5. Product section

Mỗi product section nên có:

```text
Title
Subtitle ngắn
Tab filter nhỏ nếu cần
Product grid
View all link
```

Ví dụ:

```text
Laptop bán chạy
- Văn phòng
- Gaming
- Mỏng nhẹ
- Đồ họa
```

---

## 10. Product Card cho đồ điện tử

### 10.1. Vai trò

Product card là component quan trọng nhất của storefront.

Card phải giúp khách quyết định trong 3-5 giây:

```text
Đây là sản phẩm gì?
Giá bao nhiêu?
Có sale không?
Thông số chính là gì?
Có còn hàng không?
Có đáng xem chi tiết không?
```

### 10.2. Cấu trúc Product Card

```text
ProductCard
- Image area
- Top badge area
- Product name
- Quick specs
- Rating/review
- Price block
- Promotion summary
- Stock/warranty line
- Action row
```

### 10.3. Thứ tự ưu tiên thông tin

```text
1. Ảnh
2. Tên sản phẩm
3. Giá bán
4. Thông số nhanh
5. Sale/promo
6. Rating
7. Tồn kho
8. Action
```

Trong một số campaign sale:

```text
Sale badge có thể đứng trên ảnh.
Giá vẫn phải là thông tin nổi bật nhất sau tên sản phẩm.
```

### 10.4. Image area

Rule:

```text
Tỷ lệ ảnh card: 1:1 hoặc 4:3.
Ảnh phải object-fit contain.
Nền ảnh trắng hoặc neutral-50.
Không crop mất sản phẩm.
Có placeholder khi ảnh lỗi.
Có skeleton khi loading.
```

Với đồ điện tử, không nên dùng object-fit cover làm mất chi tiết sản phẩm.

### 10.5. Product name

Rule:

```text
Tối đa 2 dòng.
Dùng line clamp.
Không để tên kéo dài làm lệch card grid.
Không viết toàn chữ hoa.
```

Ví dụ tốt:

```text
Laptop Dell Inspiron 15 3520 i5 16GB 512GB
```

Ví dụ quá dài cần clamp:

```text
Laptop Dell Inspiron 15 3520 Intel Core i5-1235U 16GB RAM 512GB SSD 15.6 inch FHD Windows 11
```

### 10.6. Quick specs

Quick specs là điểm khác biệt của web điện tử.

Laptop quick specs:

```text
Core i5
16GB RAM
SSD 512GB
15.6" FHD
```

Điện thoại quick specs:

```text
A17 Pro
8GB RAM
256GB
Camera 48MP
```

Màn hình quick specs:

```text
27"
2K
165Hz
IPS
```

Rule:

```text
Hiển thị 3-5 specs quan trọng nhất.
Mỗi spec ngắn.
Không hiển thị mô tả dài trong card.
Nếu thiếu specs, ẩn vùng quick specs hoặc dùng fallback ngắn.
```

### 10.7. Price block

Price block gồm:

```text
Sale price
Original price
Discount percent
Installment info nếu có
```

Rule:

```text
Sale price dùng màu commerce-price-strong.
Original price nhỏ hơn, gạch ngang.
Discount badge ngắn.
Không dùng quá 2 màu trong price block.
Nếu không có sale, giá thường dùng text primary hoặc price normal.
```

Ví dụ:

```text
15.990.000đ
18.990.000đ  -16%
Trả góp 0%
```

### 10.8. Promotion summary

Nên hiển thị ưu đãi ngắn:

```text
Tặng balo
Giảm thêm 500K
Trả góp 0%
Bảo hành 24 tháng
```

Rule:

```text
Tối đa 1-2 dòng.
Không nhồi toàn bộ khuyến mãi vào card.
Chi tiết nằm ở product detail.
```

### 10.9. Stock/warranty line

Ví dụ:

```text
Còn hàng
Sắp hết hàng
Hết hàng
Bảo hành 24 tháng
```

Rule:

```text
Còn hàng dùng success.
Sắp hết hàng dùng warning.
Hết hàng dùng muted.
Warranty không dùng màu quá nổi.
```

### 10.10. Action row

Desktop:

```text
[Thêm vào giỏ] [So sánh]
```

Mobile:

```text
[Thêm] [icon compare]
```

Rule:

```text
Primary action là thêm vào giỏ hoặc mua ngay tùy page.
Compare là secondary.
Wishlist nếu có thì icon riêng.
Hết hàng thì disable thêm vào giỏ.
```

### 10.11. Product card states

Bắt buộc có đủ state:

```text
Default
Hover
Focus
Loading
Image error
Out of stock
Sale
Selected for compare
Disabled action
Long name
Missing specs
```

### 10.12. Product card acceptance criteria

```text
Không vỡ layout với tên dài.
Không vỡ layout với ảnh sai tỷ lệ.
Giá sale nổi bật.
Quick specs dễ đọc.
Mobile 375px không overflow.
Card cùng grid có chiều cao ổn định.
Hết hàng disable action rõ ràng.
```

---

## 11. Trang danh sách sản phẩm / Category Page

### 11.1. Mục tiêu

Trang category phải giúp khách:

```text
Lọc nhanh theo nhu cầu
So sánh nhiều model
Nhận biết sản phẩm tốt trong tầm giá
Không bị quá tải thông tin
```

### 11.2. Layout desktop

```text
Breadcrumb
Category title + count
Optional category description
Filter sidebar bên trái
Toolbar bên trên grid
Product grid bên phải
Pagination / load more
SEO content cuối trang nếu cần
```

Desktop grid:

```text
>= 1440px: 4-5 cột
1024-1439px: 3-4 cột
```

### 11.3. Layout mobile

```text
Breadcrumb ngắn hoặc ẩn bớt
Category title
Toolbar: filter button + sort button
Product grid 2 cột hoặc 1 cột tùy card density
Filter mở bottom sheet/drawer
Pagination/load more
```

Rule mobile:

```text
Filter không chiếm màn hình liên tục.
Sort dễ đổi.
Product card phải đủ thông tin nhưng không quá cao.
```

### 11.4. Filter sidebar

Filter cho đồ điện tử phải dựa trên attribute động.

Không hard-code trong UI:

```text
Nếu category = laptop thì render CPU/RAM/SSD.
Nếu category = phone thì render Chip/RAM/Storage.
```

Thay vào đó:

```text
Backend/config trả về attribute filters theo category.
Frontend render theo type.
```

Filter types:

```text
checkbox
radio
range price
range numeric
select
multi-select
toggle
rating
availability
```

### 11.5. Filter group cho Laptop

Ví dụ:

```text
Thương hiệu
Khoảng giá
Nhu cầu sử dụng
CPU
RAM
Ổ cứng
Card đồ họa
Kích thước màn hình
Tần số quét
Trọng lượng
Bảo hành
Tình trạng hàng
Khuyến mãi
Đánh giá
```

### 11.6. Filter group cho Điện thoại

Ví dụ:

```text
Thương hiệu
Khoảng giá
Dung lượng
RAM
Chip
Kích thước màn hình
Camera
Pin
Sạc nhanh
5G
SIM
Tình trạng hàng
Khuyến mãi
Đánh giá
```

### 11.7. Filter behavior

Rule:

```text
Có nút clear all.
Có active filter chips.
Có count kết quả.
Filter mobile có Apply button.
Desktop có thể apply tức thì hoặc có Apply button.
Khi loading filter, grid hiển thị skeleton.
URL phải phản ánh filter để share được.
```

Ví dụ URL:

```text
/laptops?brand=dell,hp&ram=16gb&price_min=10000000&price_max=20000000&sort=price_asc
```

### 11.8. Sort options

Sort nên có:

```text
Phù hợp nhất
Bán chạy
Giá thấp đến cao
Giá cao đến thấp
Mới nhất
Đánh giá cao
Khuyến mãi nhiều
```

### 11.9. Empty state

Khi không có sản phẩm:

```text
Không tìm thấy sản phẩm phù hợp
Gợi ý bỏ bớt bộ lọc
Nút Xóa bộ lọc
Có thể hiển thị sản phẩm gợi ý
```

Không chỉ hiển thị màn trắng.

---

## 12. Trang chi tiết sản phẩm

### 12.1. Mục tiêu

Product Detail Page phải giúp khách:

```text
Hiểu sản phẩm rõ nhất
Tin tưởng thông tin
Chọn đúng biến thể
Biết giá cuối cùng
Biết bảo hành/giao hàng
Có thể so sánh
Đặt mua dễ dàng
```

### 12.2. Layout desktop

```text
Breadcrumb
Product title row
Main section 2 columns hoặc 3 columns
- Left: Product gallery
- Middle: Product summary/specs/variant
- Right: Price/promotion/buy box
Below sections
- Technical specs
- Product description
- Reviews
- Compare similar
- Related products
```

Với web nhỏ, có thể dùng 2 columns:

```text
Left: Gallery
Right: Info + Buy box
Below: Specs + Description + Reviews
```

### 12.3. Layout mobile

```text
Gallery
Product title
Rating/brand/SKU
Price block
Promotion box
Variant selector
Warranty/delivery info
Sticky buy bar
Specs accordion
Description accordion
Reviews
Related products
```

Sticky buy bar mobile:

```text
Price short + [Add to cart] + [Buy now]
```

Rule:

```text
Sticky bar không che form quan trọng.
Có safe-area cho iOS.
Không làm overflow ngang.
```

### 12.4. Product gallery

Gallery cần:

```text
Main image
Thumbnail list
Zoom/lightbox
Video badge nếu có
360 image nếu có
Alt text
Placeholder khi ảnh lỗi
```

Rule:

```text
Ảnh object-fit contain.
Không crop sản phẩm.
Thumbnail active rõ.
Mobile swipe được.
```

### 12.5. Product summary

Thông tin trên cùng:

```text
Tên sản phẩm
Brand
SKU/model
Rating + review count
Sold count nếu có
Compare button
Wishlist button
```

Ví dụ:

```text
Laptop Dell Inspiron 15 3520 i5 16GB 512GB
Dell | SKU: DELL3520-I5-16-512
★★★★☆ 4.7 (128 đánh giá) | Đã bán 320
```

### 12.6. Price & Promotion Box

Buy box cần rõ:

```text
Giá bán
Giá gốc
Discount
Trả góp
Coupon khả dụng
Quà tặng
Thời gian khuyến mãi nếu có
```

Rule:

```text
Giá bán lớn nhất trong buy box.
Thông tin ưu đãi ngắn gọn.
Chi tiết quà tặng có thể mở modal/accordion.
Không làm khách rối trước CTA.
```

### 12.7. Variant selector

Đồ điện tử có thể có variant:

```text
RAM
Storage
Color
CPU
GPU
Bundle
Warranty package
```

Rule:

```text
Variant bắt buộc phải chọn trước khi add to cart.
Variant hết hàng phải disabled.
Variant đang chọn phải active rõ.
Giá thay đổi theo variant phải update ngay.
SKU phải update theo variant.
```

Ví dụ:

```text
RAM: [8GB] [16GB] [32GB]
SSD: [512GB] [1TB]
Màu: [Silver] [Black]
```

### 12.8. Quick specs block

Quick specs nằm gần buy decision.

Laptop:

```text
CPU: Intel Core i5-1235U
RAM: 16GB DDR4
SSD: 512GB NVMe
Màn hình: 15.6" FHD
GPU: Intel Iris Xe
```

Điện thoại:

```text
Chip: A17 Pro
RAM: 8GB
Dung lượng: 256GB
Camera: 48MP
Pin: 4441mAh
```

Rule:

```text
Hiển thị 5-8 thông số chính.
Có link Xem cấu hình chi tiết.
Không dồn toàn bộ specs vào đầu trang.
```

### 12.9. Warranty box

Warranty box cần rõ:

```text
Bảo hành chính hãng 12/24 tháng
Đổi mới trong X ngày nếu lỗi
Hỗ trợ kỹ thuật
Xuất hóa đơn VAT nếu có
```

Rule:

```text
Thông tin bảo hành phải gần buy box.
Không giấu trong footer.
```

### 12.10. Delivery box

Delivery box nên có:

```text
Nhập/chọn khu vực giao hàng
Dự kiến thời gian nhận
Phí vận chuyển
Có hàng tại cửa hàng nào
```

MVP có thể đơn giản:

```text
Giao hàng toàn quốc
Miễn phí ship từ X
Nhận tại cửa hàng
```

### 12.11. Technical specs section

Bảng thông số là phần bắt buộc.

Cấu trúc:

```text
Group: Bộ xử lý
- CPU
- Số nhân
- Tốc độ

Group: Bộ nhớ
- RAM
- Loại RAM
- Số khe

Group: Lưu trữ
- SSD
- Khả năng nâng cấp
```

Rule:

```text
Specs nên được group.
Mỗi row có label và value.
Label không quá dài.
Value có thể dài nhưng phải wrap đẹp.
Có nút Xem thêm nếu specs dài.
Mobile dùng accordion hoặc table responsive.
```

### 12.12. Product description

Description không thay thế specs.

Description nên nói:

```text
Sản phẩm phù hợp với ai
Điểm nổi bật
Trải nghiệm sử dụng
Thiết kế
Hiệu năng
Màn hình
Pin
Kết nối
```

Rule:

```text
Không viết mô tả quá chung chung.
Không chỉ copy specs.
Có heading ngắn.
Có ảnh minh họa nếu có.
```

### 12.13. Reviews

Review section nên có:

```text
Average rating
Rating distribution
Review list
Filter review theo sao
Ảnh/video từ khách nếu có
Verified purchase badge
```

Rule:

```text
Review thật giúp tăng niềm tin.
Không để review spam phá layout.
Review dài cần collapse.
```

### 12.14. Related/Similar products

Nên có:

```text
Sản phẩm cùng tầm giá
Sản phẩm cùng thương hiệu
Sản phẩm cấu hình cao hơn
Phụ kiện mua kèm
```

---

## 13. So sánh sản phẩm

### 13.1. Vai trò

Đồ điện tử rất cần so sánh.

So sánh giúp khách trả lời:

```text
Model nào mạnh hơn?
Giá nào hợp lý hơn?
Khác nhau ở CPU/RAM/màn hình/bảo hành?
Nên mua cái nào?
```

### 13.2. Compare entry points

Nút so sánh xuất hiện ở:

```text
Product card
Product detail
Search result
Related product
Sticky compare bar
```

### 13.3. Compare bar

Khi khách chọn sản phẩm để so sánh:

```text
Sticky compare bar hiện ở cuối màn hình.
Hiển thị số sản phẩm đã chọn.
Có nút Xem so sánh.
Có nút Xóa tất cả.
```

Rule:

```text
Tối đa 3-4 sản phẩm một lần.
Không cho so sánh sản phẩm quá khác danh mục nếu không hỗ trợ.
Mobile có thể dùng bottom sheet.
```

### 13.4. Compare page

Compare page nên có:

```text
Cột sản phẩm
Ảnh
Tên
Giá
CTA mua
Thông số group
Highlight khác biệt
```

Rule:

```text
Có toggle Chỉ hiển thị điểm khác nhau.
Specs phải cùng format.
Mobile có horizontal scroll.
CTA mua phải sticky trong khu so sánh hoặc xuất hiện lại sau group dài.
```

---

## 14. Cart và Checkout cho Electronics Store

### 14.1. Cart page

Cart item đồ điện tử nên hiển thị:

```text
Ảnh
Tên sản phẩm
Variant/spec ngắn
SKU nếu cần
Giá
Số lượng
Bảo hành
Khuyến mãi đi kèm
Tồn kho warning nếu sắp hết
```

Rule:

```text
Không chỉ hiển thị tên và giá.
Variant phải rõ để tránh mua nhầm.
Nếu item hết hàng trong cart, cảnh báo rõ.
```

### 14.2. Cart summary

Nên có:

```text
Tạm tính
Giảm giá
Phí ship ước tính
Tổng cộng
Coupon input
CTA checkout
```

### 14.3. Checkout page

Checkout nên ít bước.

MVP gợi ý:

```text
1. Thông tin nhận hàng
2. Phương thức vận chuyển
3. Phương thức thanh toán
4. Xác nhận đơn hàng
```

Với đồ điện tử, cần thêm:

```text
Xuất hóa đơn VAT
Ghi chú giao hàng
Chọn cửa hàng nhận hàng nếu có
Chọn gói bảo hành mở rộng nếu có
```

### 14.4. Payment methods

MVP:

```text
COD
Chuyển khoản ngân hàng
```

Mở rộng:

```text
VNPay
MoMo
ZaloPay
Thẻ quốc tế
Trả góp
```

Rule:

```text
Thanh toán phải thể hiện phí/phụ phí nếu có.
Trả góp phải nói rõ điều kiện.
Không làm khách hiểu nhầm tổng tiền.
```

### 14.5. Order success

Sau khi đặt hàng:

```text
Mã đơn hàng
Tổng tiền
Phương thức thanh toán
Thông tin nhận hàng
Sản phẩm đã mua
Trạng thái ban đầu
Hướng dẫn thanh toán nếu chuyển khoản
Link theo dõi đơn
```

Nếu là chuyển khoản:

```text
Hiển thị số tài khoản
Nội dung chuyển khoản
QR nếu có
Thời hạn giữ đơn
```

---

## 15. Account và Order tracking

### 15.1. Customer account

Customer account nên có:

```text
Thông tin cá nhân
Sổ địa chỉ
Lịch sử đơn hàng
Theo dõi bảo hành
Sản phẩm đã xem
Wishlist
Compare history nếu cần
```

### 15.2. Order detail

Order detail nên có:

```text
Timeline trạng thái
Danh sách sản phẩm
Thông tin giao hàng
Thông tin thanh toán
Mã vận đơn nếu có
Thông tin bảo hành
Nút hỗ trợ/đổi trả
```

### 15.3. Warranty tracking

Đồ điện tử có bảo hành, nên thêm:

```text
Tra cứu bảo hành theo order/SKU/serial
Ngày mua
Thời hạn bảo hành
Trạng thái bảo hành
```

MVP có thể chưa cần serial-level, nhưng UI nên có vị trí mở rộng.

---

## 16. Admin theme cho Electronics Store

### 16.1. Mục tiêu admin

Admin của web điện tử phải xử lý nhiều dữ liệu:

```text
Nhiều SKU
Nhiều biến thể
Nhiều thông số
Nhiều giá khuyến mãi
Nhiều đơn hàng
Nhiều trạng thái tồn kho
```

Admin ưu tiên:

```text
Nhanh
Rõ
Ít màu
Dữ liệu dễ lọc
Form dài nhưng dễ quản lý
Tránh nhập nhầm thông số
```

### 16.2. Admin navigation

Sidebar admin gợi ý:

```text
Dashboard
Orders
Products
Categories
Brands
Attribute Templates
Inventory
Promotions
Customers
Reviews
Warranty
Reports
Settings
```

### 16.3. Admin dashboard

Metric cards:

```text
Doanh thu hôm nay
Đơn hàng mới
Đơn chờ xử lý
Sản phẩm sắp hết hàng
Tồn kho thấp
Sản phẩm bán chạy
Tỷ lệ hoàn/hủy
```

Rule:

```text
Metric chính dùng số lớn.
Status cần semantic color.
Chart không quá nhiều màu.
Data stale cần hiển thị thời gian cập nhật.
```

### 16.4. Product list admin

Bảng sản phẩm nên có cột:

```text
Ảnh
Tên sản phẩm
SKU
Danh mục
Brand
Giá
Tồn kho
Trạng thái
Ngày cập nhật
Actions
```

Filter admin:

```text
Search name/SKU
Category
Brand
Status
Stock status
Price range
Created date
Updated date
```

Rule:

```text
Tên dài ellipsis.
SKU copy được.
Tồn kho thấp có badge warning.
Không nhồi toàn bộ specs vào bảng list.
Actions có view/edit/duplicate/hide.
```

### 16.5. Product form admin

Product form đồ điện tử nên chia section:

```text
Basic information
Media
Pricing
Category & brand
Variants
Technical specifications
Inventory
Warranty
Shipping
SEO
Visibility
```

Không nên để toàn bộ field trên một màn hình không phân nhóm.

### 16.6. Basic information

Fields:

```text
Product name
Slug
SKU base
Brand
Category
Short description
Description
Status
```

Rule:

```text
Slug auto-generate nhưng sửa được.
SKU không được trùng.
Category bắt buộc.
Brand nên bắt buộc với đồ điện tử.
```

### 16.7. Media management

Fields:

```text
Main image
Gallery images
Video URL nếu có
Alt text
Sort order
```

Rule:

```text
Có drag sort.
Có preview.
Có validate kích thước file.
Có ảnh chính bắt buộc.
```

### 16.8. Pricing

Fields:

```text
Base price
Sale price
Cost price nếu cần
Currency
Sale start/end
Tax class
Installment enabled
```

Rule:

```text
Sale price phải nhỏ hơn base price.
Sale date phải hợp lệ.
Hiển thị preview price block.
```

### 16.9. Variants

Variant UI cần hỗ trợ:

```text
RAM
Storage
Color
CPU
GPU
Bundle
```

Admin flow:

```text
Chọn variant attributes
Generate variant combinations
Nhập SKU/price/stock cho từng variant
Bật/tắt variant
```

Rule:

```text
Variant SKU unique.
Variant có thể override price.
Variant có tồn kho riêng.
Variant disabled không hiển thị trên storefront.
```

### 16.10. Technical specifications admin

Đây là phần quan trọng nhất của admin electronics.

Không để admin tự nhập specs bằng text area tự do nếu muốn filter/compare tốt.

Nên có:

```text
Attribute Template theo category
Attribute group
Attribute field type
Attribute value
Unit
Display priority
Filterable flag
Comparable flag
Quick spec flag
```

Ví dụ attribute:

```text
CPU | text/select | quick spec | comparable | filterable
RAM | select | quick spec | comparable | filterable
SSD | select | quick spec | comparable | filterable
Screen Size | number + unit inch | comparable | filterable
Warranty | select | quick spec | comparable
```

### 16.11. Attribute Templates admin

Attribute template giúp source clone.

Entity concept:

```text
AttributeTemplate
AttributeGroup
AttributeDefinition
CategoryAttributeTemplate
```

Admin cần màn:

```text
Danh sách templates
Tạo template
Sửa group
Thêm attribute
Cấu hình type
Cấu hình filterable/comparable/quick_spec
Gán template cho category
```

### 16.12. Inventory admin

Inventory list nên có:

```text
Product/SKU
Warehouse
Available
Reserved
Low stock threshold
Status
Last updated
```

Actions:

```text
Adjust stock
Import stock
Reserve/release history
View movements
```

Rule:

```text
Không sửa tồn kho trực tiếp không có log.
Mọi thay đổi tạo StockMovement.
```

### 16.13. Order admin

Order list cột:

```text
Order number
Customer
Phone
Total
Payment status
Order status
Shipping status
Created at
Actions
```

Order detail cần:

```text
Customer info
Shipping address
Payment info
Order items
Status timeline
Internal notes
Warranty info
Fraud/risk note nếu có
```

Rule:

```text
Cập nhật trạng thái phải có confirm nếu irreversible.
Hủy đơn cần lý do.
Hoàn tiền/đổi trả cần quyền riêng.
```

### 16.14. Warranty admin

Fields:

```text
Product
SKU/serial
Order number
Customer
Warranty start
Warranty end
Warranty status
Service history
```

MVP có thể chỉ dùng order date + warranty months.

---

## 17. Category attribute templates

### 17.1. Nguyên tắc

Tất cả ngành hàng điện tử phải đi qua attribute template.

Không hard-code field trong product form.

Frontend product detail, filter, compare đều đọc từ attribute definitions.

### 17.2. Laptop template

Groups:

```text
Processor
Memory
Storage
Display
Graphics
Connectivity
Battery
Physical
Software
Warranty
```

Attributes gợi ý:

```text
CPU brand
CPU model
CPU generation
CPU cores
RAM size
RAM type
RAM upgradeable
Storage type
Storage capacity
Screen size
Resolution
Refresh rate
Panel type
GPU model
Wi-Fi
Bluetooth
Ports
Battery capacity
Weight
Operating system
Warranty months
```

Quick specs nên chọn:

```text
CPU model
RAM size
Storage capacity
Screen size
GPU model
```

Filterable nên chọn:

```text
Brand
Price
CPU brand
CPU model group
RAM size
Storage capacity
Screen size
GPU type
Weight range
Warranty months
```

Comparable nên chọn:

```text
CPU model
RAM size
Storage capacity
Screen size
Resolution
Refresh rate
GPU model
Weight
Battery capacity
Warranty months
```

### 17.3. Phone template

Groups:

```text
Performance
Display
Camera
Battery
Storage
Connectivity
Body
Software
Warranty
```

Attributes gợi ý:

```text
Chipset
RAM
Storage
Screen size
Screen type
Refresh rate
Rear camera
Front camera
Battery capacity
Charging power
5G support
SIM type
Water resistance
Weight
Operating system
Warranty months
```

Quick specs:

```text
Chipset
RAM
Storage
Camera
Battery capacity
```

### 17.4. Monitor template

Groups:

```text
Display
Performance
Connectivity
Ergonomics
Physical
Warranty
```

Attributes:

```text
Screen size
Resolution
Refresh rate
Panel type
Response time
Brightness
Color gamut
HDR
Ports
VESA
Stand adjustment
Warranty months
```

Quick specs:

```text
Screen size
Resolution
Refresh rate
Panel type
Response time
```

### 17.5. Accessory template

Groups:

```text
Compatibility
Specification
Material
Connectivity
Power
Warranty
```

Attributes:

```text
Compatible devices
Connection type
Material
Cable length
Battery life
Color
Warranty months
```

---

## 18. Badge system cho Electronics Store

### 18.1. Badge types

| Badge | Mục đích |
|---|---|
| Sale | Giảm giá |
| New | Hàng mới |
| Hot | Bán chạy |
| Installment | Trả góp |
| Warranty | Bảo hành |
| In stock | Còn hàng |
| Low stock | Sắp hết |
| Out of stock | Hết hàng |
| Official | Chính hãng |
| Gift | Quà tặng |

### 18.2. Badge rules

```text
Không hiển thị quá 3 badge trên product card.
Badge quan trọng nhất: Sale > Out of stock > New/Hot > Installment.
Product detail có thể hiển thị nhiều thông tin hơn card.
Badge text phải ngắn.
```

Ví dụ tốt:

```text
-16%
Trả góp 0%
Chính hãng
```

Ví dụ không tốt:

```text
Sản phẩm này đang có chương trình khuyến mãi cực sốc trong tuần này
```

---

## 19. Trust elements cho Electronics Store

Đồ điện tử có giá trị cao, nên trust rất quan trọng.

### 19.1. Trust block storefront

Nên có các trust item:

```text
Hàng chính hãng
Bảo hành rõ ràng
Đổi trả dễ dàng
Giao hàng toàn quốc
Hỗ trợ kỹ thuật
Thanh toán an toàn
```

### 19.2. Vị trí trust element

Trust xuất hiện ở:

```text
Header top bar
Product detail buy box
Footer
Checkout
Order success
```

### 19.3. Rule

```text
Không phóng đại cam kết nếu shop không làm được.
Chính sách phải có link chi tiết.
Text ngắn, dễ hiểu.
```

---

## 20. Content tone cho Electronics Store

### 20.1. Tone chung

Content nên:

```text
Rõ ràng
Có tính tư vấn
Không quá khoa trương
Không dùng thuật ngữ quá khó nếu không giải thích
```

### 20.2. CTA text

CTA tốt:

```text
Mua ngay
Thêm vào giỏ
So sánh
Xem cấu hình
Xem ưu đãi
Nhận tư vấn
```

CTA không nên mơ hồ:

```text
Khám phá ngay mọi điều tuyệt vời
Sở hữu siêu phẩm đỉnh cao không tưởng
```

### 20.3. Microcopy

Variant error:

```text
Vui lòng chọn dung lượng trước khi thêm vào giỏ.
```

Out of stock:

```text
Sản phẩm hiện hết hàng. Bạn có thể để lại thông tin để được báo khi có hàng.
```

Coupon invalid:

```text
Mã giảm giá không hợp lệ hoặc đã hết hạn.
```

Checkout stock changed:

```text
Số lượng tồn kho đã thay đổi. Vui lòng kiểm tra lại giỏ hàng.
```

---

## 21. Responsive rules riêng cho Electronics Theme

### 21.1. Breakpoint behavior

| Breakpoint | Storefront |
|---|---|
| 1440px+ | Grid rộng, filter sidebar |
| 1024px | Grid 3-4 cột |
| 768px | Filter drawer, grid 2-3 cột |
| 375px | Grid 2 cột hoặc list compact |
| 320px | Ưu tiên 1 cột nếu card quá chật |

### 21.2. Product card mobile

Rule:

```text
Tên tối đa 2 dòng.
Quick specs tối đa 2-3 item.
Promotion text tối đa 1 dòng.
Button có thể ngắn hơn.
Không hiển thị quá nhiều badge.
```

### 21.3. Product detail mobile

Rule:

```text
Gallery lên đầu.
Price block phải xuất hiện sớm.
CTA sticky ở cuối.
Specs dùng accordion.
Compare/wishlist không chiếm CTA chính.
```

### 21.4. Admin responsive

Admin ưu tiên desktop/tablet, nhưng mobile không được vỡ.

Rule:

```text
Sidebar collapsible.
Table có horizontal scroll.
Form stack 1 cột trên mobile.
Actions chuyển vào menu.
```

---

## 22. Accessibility rules

### 22.1. Storefront

```text
Mọi button có accessible name.
Mọi input có label.
Màu không là tín hiệu duy nhất cho trạng thái.
Focus state rõ.
Ảnh sản phẩm có alt text.
Carousel dùng được bằng keyboard.
Filter drawer có focus trap.
Modal có Esc close.
```

### 22.2. Product specs

```text
Spec table phải đọc được bởi screen reader.
Label và value có cấu trúc rõ.
Không render specs chỉ bằng ảnh.
```

### 22.3. Admin

```text
Form lỗi phải liên kết với field.
Table có header đúng.
Action destructive cần confirm.
Keyboard navigation hoạt động.
```

---

## 23. Loading, empty, error states

### 23.1. Product listing loading

Dùng skeleton:

```text
Filter skeleton
Product card skeleton grid
Toolbar disabled nhẹ
```

Không dùng spinner toàn màn nếu có thể dùng skeleton.

### 23.2. Product detail loading

Skeleton gồm:

```text
Gallery skeleton
Title skeleton
Price skeleton
Specs skeleton
Button skeleton
```

### 23.3. Empty category

Message:

```text
Chưa có sản phẩm trong danh mục này.
```

Action:

```text
Xem tất cả sản phẩm
Quay lại trang chủ
```

### 23.4. No search result

Message:

```text
Không tìm thấy sản phẩm phù hợp.
```

Suggestion:

```text
Kiểm tra lại từ khóa
Xóa bớt bộ lọc
Thử tìm theo thương hiệu hoặc dòng sản phẩm
```

### 23.5. API error

Message:

```text
Không thể tải dữ liệu. Vui lòng thử lại.
```

Action:

```text
Thử lại
```

### 23.6. Admin error

Admin error cần rõ hơn:

```text
Không thể lưu sản phẩm.
SKU đã tồn tại.
Giá khuyến mãi phải nhỏ hơn giá gốc.
Có biến thể chưa nhập tồn kho.
```

---

## 24. Motion và interaction

### 24.1. Motion style

Electronics theme dùng motion nhẹ:

```text
Hover card nâng nhẹ
Button transition 150-200ms
Drawer slide
Accordion expand/collapse
Toast fade
```

Không nên dùng:

```text
Animation bounce mạnh
Parallax nặng
Product card xoay lật
Loading quá cầu kỳ
```

### 24.2. Interaction feedback

```text
Add to cart thành công: toast + cart count update.
Compare selected: card có selected state.
Filter applied: chip active hiện rõ.
Variant selected: button active rõ.
Admin save: loading state + success toast.
```

---

## 25. SEO và content structure

### 25.1. Category page SEO

Category nên có:

```text
H1 rõ
Description ngắn phía trên hoặc cuối trang
Internal links tới category con
Product list crawl được nếu SSR/SSG hỗ trợ
```

### 25.2. Product detail SEO

Product detail nên có:

```text
H1 là tên sản phẩm
Meta title
Meta description
Structured data Product nếu có
Ảnh có alt text
Canonical URL
```

### 25.3. Slug rule

Slug nên:

```text
Ngắn
Có tên/model chính
Không chứa ký tự lạ
Không đổi liên tục
```

Ví dụ:

```text
/laptop-dell-inspiron-15-3520-i5-16gb-512gb
```

---

## 26. Component list bắt buộc cho Electronics Store

### 26.1. Storefront components

```text
StoreHeader
TopBar
MegaMenu
SearchBox
SearchSuggestion
CategoryShortcut
HeroBanner
CampaignBanner
ProductCard
ProductGrid
ProductListToolbar
FilterSidebar
FilterDrawer
ActiveFilterChips
SortSelect
PriceBlock
PromoBadge
StockBadge
WarrantyBadge
QuickSpecs
ProductGallery
VariantSelector
QuantitySelector
BuyBox
PromotionBox
WarrantyBox
DeliveryBox
SpecTable
SpecAccordion
ReviewSummary
ReviewList
RelatedProducts
CompareButton
CompareBar
ComparePageTable
CartItem
CartSummary
CheckoutForm
PaymentMethodSelector
OrderSuccessSummary
Footer
```

### 26.2. Admin components

```text
AdminLayout
AdminSidebar
AdminTopbar
MetricCard
DataTable
TableFilterBar
StatusBadge
ProductAdminForm
ProductMediaUploader
VariantMatrixEditor
AttributeTemplateEditor
SpecEditor
InventoryAdjustModal
OrderStatusTimeline
OrderDetailPanel
CouponForm
ReviewModerationTable
WarrantyRecordForm
```

---

## 27. Agent implementation rules

### 27.1. Rule chung

Agent không được tự ý phá design system gốc.

Khi code UI electronics, agent phải:

```text
Đọc file 00 design language.
Đọc file 01 electronics theme.
Xác định component/page bị ảnh hưởng.
Dùng token thay vì hard-code màu.
Tạo đủ state loading/empty/error.
Kiểm tra responsive.
Không bỏ qua accessibility.
```

### 27.2. Rule product card

Agent phải đảm bảo:

```text
Long product name không vỡ card.
Missing image có placeholder.
Missing specs không làm trống xấu.
Out of stock disable CTA.
Sale price hiển thị đúng thứ tự.
Mobile không overflow.
```

### 27.3. Rule product detail

Agent phải đảm bảo:

```text
Gallery không crop sản phẩm.
Price xuất hiện sớm.
Variant required được validate.
Specs có group.
Buy CTA rõ.
Mobile có sticky buy bar nếu page spec yêu cầu.
```

### 27.4. Rule admin

Agent phải đảm bảo:

```text
Form dài được chia section.
Table không vỡ với tên dài.
SKU unique error hiển thị rõ.
Specs nhập theo attribute template.
Không xóa StockMovement log.
Destructive actions có confirm.
```

### 27.5. Không được làm

```text
Không hard-code field specs theo laptop trong component chung.
Không dùng màu trực tiếp trong component.
Không dùng text sale quá dài trong badge.
Không dùng object-fit cover cho ảnh sản phẩm chính.
Không ẩn lỗi form.
Không bỏ qua mobile 375px.
Không xóa test để pass.
```

---

## 28. Test checklist cho Electronics Theme

### 28.1. Visual tests

Bắt buộc có visual test cho:

```text
Home page
Category page
Product card grid
Product detail
Cart
Checkout
Admin product list
Admin product form
Admin order detail
```

### 28.2. E2E tests

Flow bắt buộc:

```text
Search product
Filter laptop by RAM/price/brand
Open product detail
Select variant
Add to cart
Update quantity
Checkout COD
View order success
Admin create product
Admin update stock
Admin update order status
```

### 28.3. Responsive tests

Viewport bắt buộc:

```text
1440x900
1024x768
768x1024
390x844
375x812
320x568
```

### 28.4. Edge case tests

```text
Product name rất dài
Product không có ảnh
Product hết hàng
Product thiếu quick specs
Product có nhiều variants
Filter không có kết quả
Cart item hết hàng
Checkout thiếu phone
Admin SKU trùng
Admin sale price lớn hơn base price
```

---

## 29. Definition of Done cho UI Electronics

Một màn hình Electronics Store chỉ được coi là xong khi:

```text
Đúng design token.
Đúng layout desktop/mobile.
Không overflow ngang ở mobile.
Có loading state.
Có empty state nếu phù hợp.
Có error state nếu gọi API.
Có keyboard/focus state.
Ảnh sản phẩm không crop sai.
Giá/sale/tồn kho hiển thị đúng rule.
Product specs dễ đọc.
Agent đã chạy test liên quan.
Có screenshot hoặc visual test cho màn quan trọng.
```

Admin UI chỉ được coi là xong khi:

```text
Bảng xử lý được data dài.
Form có validation rõ.
Action nguy hiểm có confirm.
Status badge đúng semantic color.
Có loading khi submit.
Có thông báo save success/fail.
Không mất dữ liệu khi validate fail.
```

---

## 30. Bộ page spec (đã có)

Các file spec màn hình kế thừa file này, theo đúng tên & thứ tự sau:

```text
# Storefront
02-storefront-home-page.md
03-storefront-product-list-page.md
04-storefront-product-detail-page.md
05-storefront-cart-page.md
06-storefront-checkout-page.md
07-storefront-order-success-page.md
08-storefront-customer-account-page.md
# Admin
09-admin-dashboard.md
10-admin-product-management.md
11-admin-category-attribute-management.md
12-admin-order-management.md
13-admin-inventory-management.md
14-admin-promotion-management.md
15-admin-warranty-service-management.md
16-admin-shipping-management.md
17-payment-design.md
```

Mục tiêu là đi từ:

```text
Design language gốc
↓
Electronics theme
↓
Page spec
↓
Component spec
↓
Playwright test spec
↓
Code
```

---

## 31. Tóm tắt cho agent

Nếu agent chỉ được đọc phần này, hãy nhớ:

```text
Electronics UI phải rõ thông số, rõ giá, rõ tồn kho, rõ bảo hành.
Product card cần quick specs.
Product detail cần specs table và buy box mạnh.
Category page cần filter động theo attribute template.
Admin cần quản lý attribute/spec/variant/tồn kho kỹ.
Không hard-code ngành laptop vào component chung.
Dùng token, không dùng màu trực tiếp.
Mobile 375px không được overflow.
UI phải có loading/empty/error states.
```

