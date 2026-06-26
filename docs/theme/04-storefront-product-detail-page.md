# 04 - Storefront Product Detail Page Specification

> **⚠️ Chuẩn đồng bộ — đọc trước:** Hợp đồng API theo [`../main/api-conventions.md`](../main/api-conventions.md) · Enum & trạng thái theo [`../main/domain-enums.md`](../main/domain-enums.md) · Design token theo [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md).
> Khi ví dụ trong file này khác tài liệu chuẩn → **tài liệu chuẩn thắng**: base path `/api/v1`, envelope `{ success, data, error, meta }`, field JSON **camelCase**, giá trị enum **snake_case** (vd `"orderStatus": "pending_confirmation"`, `"stockStatus": "in_stock"`). FE chuẩn của dự án: **Nuxt 3 + TypeScript + Pinia + Tailwind**.

> Theme: Electronics Store  
> Depends on: `../main/ecommerce_design_language.md`, `01-electronics-store-theme.md`, `02-storefront-home-page.md`, `03-storefront-product-list-page.md`  
> Page type: Public storefront page  
> Primary users: Guest customer, logged-in customer  
> Goal: Giúp khách hiểu sản phẩm, tin tưởng thông tin, chọn đúng biến thể, so sánh nhanh, thêm vào giỏ hoặc mua ngay.

---

## 1. Mục đích của trang

Trang chi tiết sản phẩm là trang quan trọng nhất trong storefront.

Với web bán đồ điện tử, khách thường không quyết định chỉ vì ảnh. Họ cần kiểm tra nhiều yếu tố:

```text
Giá
Khuyến mãi
Thông số kỹ thuật
Tình trạng hàng
Bảo hành
Chính sách đổi trả
Giao hàng
Đánh giá
So sánh với sản phẩm khác
```

Trang này phải giúp khách trả lời nhanh 6 câu hỏi:

1. Đây có đúng sản phẩm tôi đang cần không?
2. Cấu hình/thông số có phù hợp không?
3. Giá này có tốt không?
4. Sản phẩm còn hàng không?
5. Mua ở đây có đáng tin không?
6. Tôi nên thêm vào giỏ, mua ngay, hay so sánh thêm?

---

## 2. Vai trò trong storefront

```text
Product List Page
  -> Product Detail Page
      -> Add To Cart
      -> Buy Now
      -> Compare Page
      -> Cart Page
      -> Checkout Page
      -> Review Section
      -> Related Product Detail
      -> Warranty/Support Page
```

Trang Product Detail nhận traffic từ:

```text
Home section
Category page
Search result
Promotion landing page
Brand page
Compare page
Recently viewed
SEO từ Google
Direct URL
```

Trang này phải tối ưu cả cho người đã biết model cụ thể và người đang so sánh nhiều model.

---

## 3. Nguyên tắc thiết kế riêng cho đồ điện tử

### 3.1. Thông tin quan trọng phải nổi bật

Thứ tự ưu tiên hiển thị:

```text
Ảnh sản phẩm
Tên sản phẩm
Giá bán
Khuyến mãi
Tình trạng hàng
Biến thể
CTA mua hàng
Thông số nhanh
Bảo hành
Giao hàng
Thông số chi tiết
Review
```

Không để mô tả marketing dài che mất giá, thông số, bảo hành và nút mua.

### 3.2. Không làm trang quá “banner quảng cáo”

Web đồ điện tử cần cảm giác chuyên nghiệp. Có thể có khuyến mãi, nhưng không được làm khách khó đọc thông tin.

Ưu tiên:

```text
Rõ ràng
Có cấu trúc
Dễ scan
Dễ so sánh
Dễ hành động
```

### 3.3. Product detail phải hỗ trợ quyết định mua

Trang này không chỉ để hiển thị data. Nó phải dẫn người dùng qua chuỗi quyết định:

```text
Nhìn ảnh
-> đọc tên/model
-> xem giá
-> xem cấu hình nhanh
-> chọn biến thể
-> kiểm tra còn hàng/bảo hành
-> thêm giỏ hoặc mua ngay
```

---

## 4. Route và URL

### 4.1. Route chính

```text
/products/{slug}
```

Ví dụ:

```text
/products/laptop-dell-inspiron-15-3520-i5-1235u-16gb-512gb
/products/iphone-15-pro-max-256gb
/products/tai-nghe-sony-wh-1000xm5
```

### 4.2. Route có query optional

```text
/products/{slug}?variant=sku-001
/products/{slug}?ref=home-flash-sale
/products/{slug}?compare=true
```

### 4.3. Rule slug

Slug phải:

```text
Dễ đọc
Có keyword SEO
Không chứa ký tự đặc biệt phức tạp
Không phụ thuộc id nội bộ
Có thể redirect nếu sản phẩm đổi tên
```

Nếu slug cũ được truy cập, hệ thống nên redirect 301 sang slug mới nếu có dữ liệu mapping.

---

## 5. Layout tổng quan

### 5.1. Desktop layout

Màn hình từ `1024px` trở lên.

```text
Header
Breadcrumb
Main product area
  Left: Product gallery
  Right: Product purchase panel
Below main area
  Quick specs / Promotion / Warranty blocks
  Product description
  Technical specifications
  Review section
  Q&A section
  Related products
  Recently viewed
Footer
```

Layout chính:

```text
┌──────────────────────────────────────────────┐
│ Header                                       │
├──────────────────────────────────────────────┤
│ Breadcrumb                                   │
├───────────────────┬──────────────────────────┤
│ Gallery           │ Product Info + Purchase  │
│                   │                          │
│ Image             │ Name                     │
│ Thumbnails        │ Rating / SKU / Brand     │
│                   │ Price                    │
│                   │ Promotions               │
│                   │ Variants                 │
│                   │ Stock                    │
│                   │ CTA                      │
│                   │ Warranty / Delivery      │
├───────────────────┴──────────────────────────┤
│ Quick Specs                                   │
├──────────────────────────────────────────────┤
│ Description / Specs / Reviews / Related       │
└──────────────────────────────────────────────┘
```

Recommended container:

```text
Max width: 1200px hoặc 1280px
Main grid: 48% gallery / 52% info
Gap: 24px - 32px
```

### 5.2. Tablet layout

Màn hình `768px` đến `1023px`.

```text
Gallery ở trên hoặc bên trái tùy không gian
Product info ở dưới hoặc bên phải
CTA vẫn phải dễ thấy
Thông số nhanh hiển thị dạng grid 2 cột
```

Nếu layout 2 cột bị chật, chuyển sang 1 cột sớm.

### 5.3. Mobile layout

Màn hình dưới `768px`.

```text
Header compact
Breadcrumb rút gọn
Gallery full width
Product info full width
Price block
Variant selector
Sticky bottom CTA
Accordion cho mô tả/specs/reviews
Footer rút gọn
```

Mobile layout:

```text
┌─────────────────────┐
│ Mobile Header        │
├─────────────────────┤
│ Breadcrumb compact   │
├─────────────────────┤
│ Gallery              │
├─────────────────────┤
│ Product name         │
│ Rating / SKU         │
│ Price                │
│ Promotion            │
│ Variant selector     │
│ Stock / Warranty     │
├─────────────────────┤
│ Quick specs          │
├─────────────────────┤
│ Accordion sections   │
├─────────────────────┤
│ Related products     │
├─────────────────────┤
│ Sticky CTA bottom    │
└─────────────────────┘
```

Sticky CTA trên mobile:

```text
Left: Add to cart
Right: Buy now
Optional: quantity selector
```

Không được che nội dung cuối trang. Body cần có padding-bottom tương ứng chiều cao sticky CTA.

---

## 6. Page sections

## 6.1. Header

Dùng lại từ Home/Product List.

Header trên product detail phải có:

```text
Logo
Search input
Category menu
Compare shortcut
Cart icon
Account shortcut
```

Search phải luôn dễ truy cập vì người xem sản phẩm thường muốn so sánh model khác.

### Header behavior

```text
Desktop: full header
Tablet: header rút gọn
Mobile: logo + search icon + cart + menu
```

Nếu header sticky, không được che breadcrumb hoặc ảnh sản phẩm khi scroll tới anchor.

---

## 6.2. Breadcrumb

### Mục đích

Giúp khách biết sản phẩm thuộc danh mục nào và quay lại danh sách.

### Nội dung

```text
Home > Laptop > Laptop Dell > Dell Inspiron 15
```

### Rule

- Desktop hiển thị đầy đủ nếu đủ chỗ.
- Mobile chỉ hiển thị tối đa 2-3 cấp hoặc dùng dạng rút gọn.
- Breadcrumb item cuối không click.
- Breadcrumb cần có schema data nếu làm SEO.

### Empty case

Nếu sản phẩm thiếu category:

```text
Home > Sản phẩm > Product name
```

---

## 6.3. Product Gallery

### Mục đích

Hiển thị hình ảnh sản phẩm rõ ràng, giúp khách xem thiết kế, cổng kết nối, màu sắc, phụ kiện đi kèm.

### Component

```text
ProductGallery
ProductImageViewer
ProductThumbnailList
ProductImageZoom
ProductVideoPreview
```

### Data

```ts
interface ProductMedia {
  id: string;
  type: 'image' | 'video';
  url: string;
  thumbnailUrl?: string;
  alt: string;
  sortOrder: number;
  isPrimary: boolean;
}
```

### Layout desktop

```text
Main image lớn
Thumbnail list bên dưới hoặc bên trái
Có nút previous/next nếu nhiều ảnh
Có zoom khi hover hoặc click
```

### Layout mobile

```text
Swipe gallery
Indicator dot hoặc counter: 1/6
Thumbnail optional
Tap để mở full-screen image viewer
```

### Rule hiển thị

- Ảnh đầu tiên là ảnh chính.
- Ảnh phải giữ tỷ lệ, không méo.
- Nền ảnh nên dùng neutral surface.
- Nếu ảnh lỗi, hiển thị placeholder.
- Alt text phải mô tả sản phẩm.
- Video không được autoplay có âm thanh.

### Placeholder

Nếu không có ảnh:

```text
Hiển thị product placeholder
Text: Chưa có hình ảnh
```

### Interaction

```text
Click thumbnail -> đổi ảnh chính
Keyboard left/right -> chuyển ảnh nếu focus trong gallery
Click zoom -> mở modal ảnh lớn
Esc -> đóng modal
```

---

## 6.4. Product Summary

### Mục đích

Hiển thị thông tin nhận dạng sản phẩm.

### Nội dung

```text
Brand
Product name
Model code
SKU
Rating summary
Review count
Sold count optional
Compare action
Wishlist action
Share action
```

### Product name rule

Tên sản phẩm phải đủ rõ model và cấu hình chính.

Ví dụ tốt:

```text
Laptop Dell Inspiron 15 3520 i5-1235U / 16GB / 512GB SSD / 15.6" FHD
```

Ví dụ không tốt:

```text
Laptop Dell mới nhất
```

### Typography

```text
Product name: heading lớn, dễ đọc
Brand/SKU: text nhỏ, màu neutral
Rating: text nhỏ nhưng dễ click
```

### Rating summary

```text
★★★★☆ 4.7 (128 đánh giá)
```

Click vào rating thì scroll đến Review Section.

### SKU / model

Hiển thị nhỏ nhưng cần có để khách đối chiếu:

```text
SKU: LAP-DELL-3520-I5-16-512
Model: Inspiron 15 3520
```

---

## 6.5. Price Block

### Mục đích

Giá là yếu tố quyết định. Phải nổi bật, rõ ràng, không gây hiểu lầm.

### Data

```ts
interface ProductPrice {
  currency: string;
  regularPrice: number;
  salePrice?: number;
  discountPercent?: number;
  installmentText?: string;
  taxIncluded?: boolean;
}
```

### Rule hiển thị

Nếu có sale price:

```text
Giá sale: nổi bật nhất
Giá gốc: gạch ngang
Discount badge: -12%
```

Ví dụ:

```text
15.990.000đ
18.990.000đ  -16%
Trả góp từ 1.599.000đ/tháng
```

Nếu không có sale:

```text
15.990.000đ
```

Nếu giá liên hệ:

```text
Liên hệ để nhận báo giá
Button: Liên hệ tư vấn
```

### Price display rule

- Không hiển thị giá bằng số thô chưa format.
- Luôn có đơn vị tiền tệ.
- Không để nhiều mức giá gây rối nếu không giải thích rõ.
- Nếu biến thể có giá khác nhau, khi chọn biến thể phải update giá.

### Error case

Nếu thiếu giá:

```text
Tạm thời chưa có giá
```

Không được hiển thị `0đ` trừ khi sản phẩm thực sự miễn phí.

---

## 6.6. Promotion Box

### Mục đích

Làm nổi bật ưu đãi nhưng không phá layout.

### Nội dung có thể có

```text
Giảm giá trực tiếp
Coupon
Quà tặng kèm
Miễn phí giao hàng
Trả góp 0%
Gia hạn bảo hành
Ưu đãi thanh toán qua ngân hàng/ví điện tử
```

### Layout

```text
Box nền nhẹ
Title: Khuyến mãi
Danh sách ưu đãi dạng bullet ngắn
CTA phụ nếu cần: Xem điều kiện
```

### Rule

- Tối đa 3 ưu đãi chính hiển thị luôn.
- Nếu nhiều hơn, dùng “Xem thêm ưu đãi”.
- Điều kiện khuyến mãi phải rõ.
- Không dùng quá nhiều màu đỏ/cam.

### Example

```text
Khuyến mãi
- Giảm thêm 500.000đ khi thanh toán chuyển khoản
- Tặng balo laptop trị giá 399.000đ
- Trả góp 0% qua thẻ tín dụng
```

---

## 6.7. Variant Selector

### Mục đích

Cho phép khách chọn biến thể như màu sắc, RAM, dung lượng, cấu hình, bảo hành.

### Data

```ts
interface ProductVariantOption {
  attributeCode: string;
  attributeName: string;
  values: Array<{
    value: string;
    label: string;
    disabled: boolean;
    colorHex?: string;
  }>;
}
```

### Loại biến thể phổ biến

Laptop:

```text
RAM
Storage
CPU
Color
Warranty package
```

Điện thoại:

```text
Storage
Color
Region/Version
Warranty
```

Tai nghe:

```text
Color
Connection type
```

### UI rule

- Dùng button/chip selector, không dùng dropdown nếu số option ít.
- Option hết hàng phải disabled nhưng vẫn nhìn thấy.
- Option đang chọn phải rõ trạng thái selected.
- Khi chọn biến thể, cập nhật giá, ảnh, SKU, tồn kho.
- Nếu chưa chọn biến thể bắt buộc, bấm CTA phải báo lỗi cạnh selector.

### Validation message

```text
Vui lòng chọn màu sắc
Vui lòng chọn dung lượng
```

### Accessibility

- Selector phải hỗ trợ keyboard.
- Selected state phải có aria.
- Không dùng màu làm tín hiệu duy nhất.

---

## 6.8. Quantity Selector

### Mục đích

Cho khách chọn số lượng mua.

### UI

```text
[-] [1] [+]
```

### Rule

- Min = 1.
- Max = tồn kho khả dụng hoặc giới hạn mua.
- Nếu sản phẩm hết hàng, disable toàn bộ selector.
- Nếu số lượng vượt tồn kho, hiển thị lỗi.

### Error message

```text
Chỉ còn 3 sản phẩm trong kho
Số lượng tối đa cho mỗi đơn là 5
```

---

## 6.9. Stock Status

### Mục đích

Cho khách biết có mua được ngay không.

### Trạng thái

```text
in_stock
low_stock
out_of_stock
pre_order
coming_soon
contact_for_stock
```

### UI text

```text
Còn hàng
Sắp hết hàng
Hết hàng
Đặt trước
Sắp mở bán
Liên hệ kiểm tra hàng
```

### Rule màu

```text
Còn hàng: success
Sắp hết hàng: warning
Hết hàng: danger/neutral
Đặt trước: primary/info
```

### Behavior

- `in_stock`: Add to cart và Buy now enabled.
- `low_stock`: enabled, kèm warning.
- `out_of_stock`: disabled, có thể hiện nút “Thông báo khi có hàng”.
- `pre_order`: CTA đổi thành “Đặt trước”.
- `coming_soon`: CTA disabled hoặc “Nhận thông báo”.

---

## 6.10. CTA Area

### Mục đích

Đưa khách vào hành động mua.

### CTA chính

```text
Thêm vào giỏ
Mua ngay
```

### CTA phụ

```text
So sánh
Yêu thích
Liên hệ tư vấn
Thông báo khi có hàng
```

### Desktop layout

```text
Row 1: Quantity selector
Row 2: [Thêm vào giỏ] [Mua ngay]
Row 3: So sánh / Yêu thích / Hotline
```

### Mobile layout

Sticky bottom:

```text
[Thêm giỏ] [Mua ngay]
```

Nếu cần hotline:

```text
Icon phone trong sticky hoặc nằm trên CTA area
```

### Button priority

```text
Mua ngay: primary high-emphasis
Thêm vào giỏ: secondary hoặc primary-outline
So sánh: tertiary
Wishlist: icon button
```

### Rule

- Không được để CTA dưới quá sâu trên mobile.
- Nếu disabled phải có lý do rõ.
- CTA loading phải có spinner hoặc text loading.
- Không double submit khi bấm liên tục.

### Loading state

```text
Đang thêm...
Đang xử lý...
```

### Success behavior

Sau khi thêm giỏ:

Có 2 lựa chọn:

```text
Hiển thị mini cart drawer
Hoặc hiển thị toast + cập nhật cart icon
```

Recommended:

```text
Desktop: mini cart drawer
Mobile: toast + option xem giỏ
```

---

## 6.11. Trust & Service Box

### Mục đích

Tăng niềm tin và giảm lo lắng.

### Nội dung

```text
Bảo hành chính hãng
Đổi trả trong 7 ngày
Giao hàng nhanh
Thanh toán an toàn
Hỗ trợ kỹ thuật
Xuất hóa đơn VAT optional
```

### Layout

Desktop:

```text
Grid 2 cột hoặc vertical list trong purchase panel
```

Mobile:

```text
Cards nhỏ hoặc accordion
```

### Example

```text
Bảo hành 24 tháng chính hãng
Đổi trả 7 ngày nếu lỗi nhà sản xuất
Giao hàng nội thành trong 2 giờ
Hỗ trợ cài đặt phần mềm cơ bản
```

---

## 6.12. Delivery Estimator

### Mục đích

Cho khách biết phí ship và thời gian giao hàng dự kiến.

### Input

```text
City/Province
District
Ward optional
```

### Output

```text
Dự kiến giao: 1-2 ngày
Phí giao hàng: 30.000đ
Miễn phí từ: 5.000.000đ
```

### Behavior

- Nếu user đã đăng nhập và có địa chỉ mặc định, auto fill.
- Nếu chưa có địa chỉ, hiển thị chọn khu vực.
- Nếu API vận chuyển lỗi, hiển thị fallback.

### Fallback text

```text
Chưa thể tính phí giao hàng. Phí chính xác sẽ được xác nhận ở bước thanh toán.
```

---

## 6.13. Quick Specs Section

### Mục đích

Cho khách xem cấu hình chính ngay, không cần kéo xuống bảng thông số dài.

### Layout

Desktop:

```text
Card grid 4-6 item
```

Mobile:

```text
Grid 2 cột hoặc horizontal scroll
```

### Quick spec laptop example

```text
CPU: Intel Core i5-1235U
RAM: 16GB
SSD: 512GB
Màn hình: 15.6" FHD
GPU: Intel Iris Xe
Bảo hành: 24 tháng
```

### Quick spec phone example

```text
Chip: A17 Pro
RAM: 8GB
Bộ nhớ: 256GB
Màn hình: 6.7"
Camera: 48MP
Pin: 4422mAh
```

### Rule

- Chỉ hiển thị 4-8 thông số quan trọng.
- Không hiển thị tất cả specs ở đây.
- Thứ tự specs theo template category.
- Nếu thiếu spec, bỏ qua chứ không hiển thị `null`.

---

## 6.14. Product Description

### Mục đích

Giải thích giá trị sản phẩm bằng nội dung marketing có cấu trúc.

### Nội dung

```text
Tổng quan
Điểm nổi bật
Tình huống sử dụng
Hình ảnh/video minh họa
Nội dung dài SEO
```

### Rule

- Không nhồi chữ quá dài trước specs.
- Nội dung dài có thể collapse “Xem thêm”.
- Hình trong mô tả phải responsive.
- Không cho HTML nguy hiểm nếu lấy từ CMS.

### Recommended structure

```text
H2: Tổng quan sản phẩm
H2: Hiệu năng
H2: Màn hình
H2: Thiết kế
H2: Pin và kết nối
H2: Sản phẩm phù hợp với ai?
```

---

## 6.15. Technical Specifications

### Mục đích

Bảng thông số kỹ thuật là phần bắt buộc với đồ điện tử.

### Component

```text
TechnicalSpecsTable
SpecsGroup
SpecsRow
```

### Data

```ts
interface TechnicalSpecGroup {
  groupName: string;
  items: Array<{
    label: string;
    value: string;
    unit?: string;
    sortOrder: number;
  }>;
}
```

### Layout

Desktop:

```text
Table hoặc definition list
Group theo category
```

Mobile:

```text
Card list hoặc table 2 cột đơn giản
Không overflow ngang
```

### Group laptop example

```text
Bộ xử lý
- CPU
- Số nhân
- Tốc độ tối đa

Bộ nhớ
- RAM
- Loại RAM
- Số khe RAM

Lưu trữ
- Ổ cứng
- Khả năng nâng cấp

Màn hình
- Kích thước
- Độ phân giải
- Tần số quét

Đồ họa
- GPU

Kết nối
- Wi-Fi
- Bluetooth
- Cổng kết nối

Pin & sạc
- Dung lượng pin
- Công suất sạc

Thông tin khác
- Trọng lượng
- Hệ điều hành
- Bảo hành
```

### Rule

- Specs phải lấy từ attribute template của category.
- Không hard-code riêng laptop trong component generic.
- Nếu spec thiếu, không hiển thị dòng đó.
- Các thông số quan trọng nên có tooltip giải thích nếu cần.
- Có nút “Xem cấu hình đầy đủ” nếu bảng bị collapse.

### Collapse behavior

MVP:

```text
Hiển thị 10 dòng đầu
Button: Xem thêm thông số
```

Full:

```text
Tabs hoặc accordion theo group
```

---

## 6.16. Compare Entry Section

### Mục đích

Đồ điện tử cần so sánh model. Product detail phải cho khách đưa sản phẩm vào danh sách so sánh.

### CTA

```text
+ Thêm vào so sánh
Đã thêm vào so sánh
Xem so sánh
```

### Rule

- Tối đa số sản phẩm so sánh: 3 hoặc 4.
- Chỉ so sánh trong cùng category nếu specs khác nhau quá nhiều.
- Nếu thêm sản phẩm khác category, hiển thị cảnh báo.

### Message

```text
Bạn chỉ có thể so sánh các sản phẩm cùng danh mục.
Danh sách so sánh đã đủ 4 sản phẩm.
Đã thêm sản phẩm vào danh sách so sánh.
```

### Data storage

Có thể lưu compare list trong:

```text
Local storage cho guest
Database cho logged-in user
```

---

## 6.17. Reviews & Ratings

### Mục đích

Tăng niềm tin bằng đánh giá thực tế.

### Nội dung

```text
Average rating
Rating distribution
Review filters
Review list
Review form
Verified purchase badge
Images from customers optional
```

### Layout

Desktop:

```text
Left: rating summary
Right: distribution + filters
Below: review list
```

Mobile:

```text
Rating summary trên cùng
Filters dạng chips
Review list 1 cột
```

### Review item

```text
User name
Rating
Date
Verified purchase
Variant purchased
Content
Images optional
Helpful action optional
```

### Rule

- Chỉ user đã mua mới có “Verified purchase”.
- Review pending moderation không hiển thị public.
- Không được làm layout vỡ khi content dài.
- Content dài có “Xem thêm”.

### Empty state

```text
Chưa có đánh giá nào.
Hãy là người đầu tiên đánh giá sản phẩm này.
```

### Review filter

```text
Tất cả
5 sao
4 sao
Có hình ảnh
Đã mua hàng
```

---

## 6.18. Q&A Section

### Mục đích

Cho khách hỏi về sản phẩm, bảo hành, tương thích, giao hàng.

### Nội dung

```text
Question input
Question list
Admin/staff answer
Search within Q&A optional
```

### Rule

- Guest có thể hỏi bằng tên/email/phone hoặc yêu cầu đăng nhập.
- Câu hỏi cần moderation nếu public.
- Trả lời của shop phải có badge “Shop trả lời”.

### Empty state

```text
Chưa có câu hỏi nào cho sản phẩm này.
```

---

## 6.19. Related Products

### Mục đích

Gợi ý sản phẩm liên quan để tăng khả năng chuyển đổi.

### Nhóm gợi ý

```text
Sản phẩm tương tự
Sản phẩm cùng thương hiệu
Phụ kiện mua kèm
Sản phẩm thường được mua cùng
Sản phẩm đã xem gần đây
```

### Rule

- Product card dùng cùng chuẩn từ Product List.
- Quick specs vẫn cần hiển thị.
- Không gợi ý sản phẩm đã bị ẩn hoặc hết hàng nếu không có rule rõ.
- Nếu thiếu data recommendation, fallback sang cùng category.

---

## 6.20. Sticky Anchor Navigation

### Mục đích

Giúp khách nhảy nhanh đến phần quan trọng.

### Items

```text
Tổng quan
Thông số
Đánh giá
Hỏi đáp
Sản phẩm liên quan
```

### Behavior

- Desktop có thể sticky dưới header khi scroll qua main product area.
- Mobile có thể dùng horizontal tabs.
- Anchor scroll phải trừ chiều cao sticky header.

---

## 7. Data contract tổng quan

```ts
interface ProductDetailPageData {
  product: ProductDetail;
  variants: ProductVariant[];
  media: ProductMedia[];
  promotions: ProductPromotion[];
  quickSpecs: ProductSpecItem[];
  technicalSpecs: TechnicalSpecGroup[];
  stock: StockInfo;
  warranty: WarrantyInfo;
  deliveryOptions: DeliveryOption[];
  reviewsSummary: ReviewsSummary;
  reviews: ReviewItem[];
  relatedProducts: ProductCardData[];
  breadcrumbs: BreadcrumbItem[];
}
```

### ProductDetail

```ts
interface ProductDetail {
  id: string;
  slug: string;
  sku: string;
  modelCode?: string;
  brand: string;
  name: string;
  shortDescription?: string;
  descriptionHtml?: string;
  categoryId: string;
  categoryName: string;
  regularPrice: number;
  salePrice?: number;
  currency: string;
  status: 'active' | 'inactive' | 'discontinued';
  isNew?: boolean;
  isBestSeller?: boolean;
  isOfficialWarranty?: boolean;
}
```

### ProductVariant

```ts
interface ProductVariant {
  id: string;
  sku: string;
  attributes: Record<string, string>;
  regularPrice: number;
  salePrice?: number;
  stockStatus: StockStatus;
  quantityAvailable: number;
  mediaIds?: string[];
}
```

### StockInfo

```ts
type StockStatus =
  | 'in_stock'
  | 'low_stock'
  | 'out_of_stock'
  | 'pre_order'
  | 'coming_soon'
  | 'contact_for_stock';

interface StockInfo {
  status: StockStatus;
  quantityAvailable?: number;
  warehouseText?: string;
  expectedRestockDate?: string;
}
```

### WarrantyInfo

```ts
interface WarrantyInfo {
  type: 'official' | 'store' | 'none';
  durationMonths?: number;
  description: string;
  policyUrl?: string;
}
```

---

## 8. State design

## 8.1. Loading state

### Initial page loading

Hiển thị skeleton cho:

```text
Breadcrumb
Gallery
Product name
Price
Variant selector
CTA
Quick specs
Specs section
Related products
```

Không dùng spinner toàn trang nếu có thể skeleton.

### Section loading

Một số section có thể load sau:

```text
Reviews
Related products
Delivery estimator
Recently viewed
```

Mỗi section có skeleton riêng.

---

## 8.2. Empty state

### Product not found

```text
Không tìm thấy sản phẩm
Sản phẩm có thể đã ngừng kinh doanh hoặc đường dẫn không đúng.
Button: Quay lại danh mục
Button: Về trang chủ
```

### No media

```text
Chưa có hình ảnh
```

### No reviews

```text
Chưa có đánh giá nào
```

### No related products

Ẩn section hoặc hiển thị:

```text
Chưa có sản phẩm liên quan
```

Recommended: ẩn nếu không có data.

---

## 8.3. Error state

### API product error

```text
Không thể tải thông tin sản phẩm
Vui lòng thử lại sau.
Button: Tải lại
```

### Add to cart error

```text
Không thể thêm sản phẩm vào giỏ hàng.
Vui lòng kiểm tra lại số lượng hoặc thử lại.
```

### Variant out of stock after select

```text
Phiên bản này hiện đã hết hàng.
Vui lòng chọn phiên bản khác.
```

### Delivery estimator error

```text
Chưa thể tính phí giao hàng.
Bạn có thể kiểm tra lại ở bước thanh toán.
```

---

## 8.4. Success state

### Add to cart success

```text
Đã thêm sản phẩm vào giỏ hàng
Button: Xem giỏ hàng
Button: Tiếp tục mua sắm
```

### Compare success

```text
Đã thêm vào danh sách so sánh
Button: Xem so sánh
```

### Wishlist success

```text
Đã lưu vào danh sách yêu thích
```

---

## 9. Interaction rules

### 9.1. Add to cart

Flow:

```text
User chọn variant nếu bắt buộc
User chọn quantity
Click Thêm vào giỏ
Validate variant + quantity + stock
Call API
Update cart count
Show mini cart/toast
```

Validation order:

```text
Product status active?
Variant selected?
Variant in stock?
Quantity valid?
API success?
```

### 9.2. Buy now

Flow:

```text
Validate giống Add to cart
Create temporary checkout item hoặc add to cart
Redirect /checkout
```

Rule:

- Không bỏ qua validation.
- Nếu guest được checkout, không yêu cầu login ngay.
- Nếu bắt buộc login, lưu intended action sau login.

### 9.3. Select variant

Khi chọn variant:

```text
Update price
Update SKU
Update stock
Update gallery optional
Update URL query optional
Clear validation error
```

Không reload toàn trang nếu không cần.

### 9.4. Scroll to section

Click rating:

```text
Scroll to reviews
```

Click “Xem thông số đầy đủ”:

```text
Scroll to Technical Specifications
```

Click warranty detail:

```text
Open policy modal hoặc policy page
```

---

## 10. Responsive rules chi tiết

### 10.1. Desktop `>= 1200px`

```text
Container max width 1200/1280
Gallery và info 2 cột
CTA trong product info panel
Specs table rộng
Related product carousel hoặc grid 4 cột
```

### 10.2. Laptop `1024px - 1199px`

```text
Vẫn 2 cột
Giảm gap
Quick specs 3 cột
Product card related 3-4 cột
```

### 10.3. Tablet `768px - 1023px`

```text
Có thể chuyển 1 cột nếu info panel chật
Gallery full width
Info dưới gallery
Quick specs 2-3 cột
CTA không bị đẩy quá sâu
```

### 10.4. Mobile `< 768px`

```text
1 cột
Gallery full width
Sticky bottom CTA
Accordion cho specs/reviews/description nếu nội dung dài
Quick specs 2 cột
No horizontal overflow
```

### 10.5. Small mobile `< 390px`

```text
Tên sản phẩm không gây overflow
Button text có thể rút gọn
CTA sticky chia 2 cột
Price không vỡ dòng kỳ lạ
```

---

## 11. SEO rules

Product detail là trang SEO quan trọng.

### 11.1. Meta

```text
Title: Product name + Brand + Giá tốt
Description: Tóm tắt sản phẩm, thông số chính, bảo hành, ưu đãi
Canonical URL
Open Graph image
```

### 11.2. Structured data

Nên hỗ trợ schema:

```text
Product
Offer
AggregateRating
Review
BreadcrumbList
```

### 11.3. Heading structure

```text
H1: Product name
H2: Tổng quan
H2: Thông số kỹ thuật
H2: Đánh giá
H2: Hỏi đáp
H2: Sản phẩm liên quan
```

Không dùng nhiều H1.

### 11.4. URL rule

```text
/products/{slug}
```

Không để URL phụ thuộc query filter từ list page.

---

## 12. Accessibility rules

### 12.1. Keyboard

Phải thao tác được bằng keyboard:

```text
Gallery thumbnails
Variant selector
Quantity selector
CTA buttons
Accordion
Review filters
Modal image viewer
```

### 12.2. Screen reader

- Product name là H1.
- Price có aria-label rõ.
- Discount badge không chỉ dựa vào màu.
- Stock status có text.
- Image alt phải mô tả sản phẩm.
- Modal có focus trap.

### 12.3. Color contrast

Tuân thủ contrast từ design language gốc.

Không dùng text xám quá nhạt cho specs.

### 12.4. Form/error

Validation error phải liên kết với field liên quan.

Ví dụ variant selector:

```text
aria-describedby="storage-error"
```

---

## 13. Performance rules

### 13.1. Image optimization

- Product image dùng responsive image.
- Lazy load ảnh không nằm trong viewport.
- Ảnh chính nên preload hoặc priority load.
- Thumbnail dung lượng nhỏ.

### 13.2. Section lazy loading

Có thể lazy load:

```text
Reviews
Related products
Recently viewed
Q&A
```

Không lazy load phần giá/CTA/specs nhanh vì ảnh hưởng quyết định mua.

### 13.3. Avoid heavy description

Nếu mô tả sản phẩm chứa nhiều ảnh, phải lazy load và giới hạn kích thước.

### 13.4. Client state

Không để chọn variant gây rerender toàn bộ trang nếu không cần.

---

## 14. Security rules

### 14.1. Description HTML

Nếu `descriptionHtml` đến từ CMS/admin:

```text
Sanitize HTML
Không cho script
Không cho inline event handler
Không cho iframe lạ nếu chưa whitelist
```

### 14.2. Price trust

Giá hiển thị từ frontend chỉ để hiển thị. Backend phải tính lại giá khi checkout.

### 14.3. Add to cart

Không tin quantity, price, variant từ client. Backend phải validate lại.

---

## 15. Component list

Trang này nên tách thành component:

```text
ProductDetailPage
ProductBreadcrumb
ProductGallery
ProductSummary
PriceBlock
PromotionBox
VariantSelector
QuantitySelector
StockStatusBadge
ProductCTAGroup
TrustServiceBox
DeliveryEstimator
QuickSpecsGrid
ProductDescription
TechnicalSpecsTable
CompareAction
ReviewSummary
ReviewList
ReviewItem
QuestionAnswerSection
RelatedProductsSection
RecentlyViewedSection
StickyAnchorNav
MobileStickyCTA
```

### Component ownership

Generic component có thể tái dùng:

```text
Breadcrumb
PriceText
Badge
Button
Card
Accordion
Modal
Toast
Skeleton
RatingStars
ProductCard
```

Electronics-specific component:

```text
QuickSpecsGrid
TechnicalSpecsTable
CompareAction
WarrantyBox
DeliveryEstimator
```

---

## 16. Admin dependency

Product Detail phụ thuộc data từ admin.

Admin cần quản lý được:

```text
Product basic info
Product media
Product variant
Product price
Product promotion
Product stock
Product warranty
Product specs
Product description
SEO metadata
Related product manual override
```

Nếu thiếu admin cấu hình, frontend phải fallback an toàn.

---

## 17. Agent implementation rules

Khi agent implement trang này:

```text
1. Đọc design language gốc.
2. Đọc electronics theme.
3. Đọc product list spec để dùng chung product card/compare rule.
4. Không hard-code riêng laptop nếu component phải dùng cho mọi electronics category.
5. Tách component nhỏ, không viết toàn bộ vào một file page khổng lồ.
6. Không dùng CSS global bừa bãi.
7. Không để mobile overflow ngang.
8. Không bỏ qua loading, empty, error state.
9. Không fake price calculation ở checkout.
10. Không xóa validation để test pass.
```

### Required implementation order

```text
1. Data types / interfaces
2. Product detail layout shell
3. Gallery
4. Product summary + price
5. Variant + stock + CTA
6. Quick specs
7. Technical specs
8. Reviews/Q&A placeholder
9. Related products
10. Responsive + sticky CTA
11. Tests
```

---

## 18. Playwright test specification

## 18.1. Basic render

```text
Given product exists
When user opens /products/{slug}
Then product name is visible
And product image is visible
And price is visible
And CTA buttons are visible
```

## 18.2. Product not found

```text
Given product does not exist
When user opens invalid product URL
Then not found state is visible
And user can go back to category or home
```

## 18.3. Gallery interaction

```text
Given product has multiple images
When user clicks second thumbnail
Then main image changes
When user opens image modal
Then modal is visible
When user presses Escape
Then modal closes
```

## 18.4. Variant selection

```text
Given product has storage/color variants
When user selects a variant
Then price/SKU/stock updates if variant data differs
And selected state is visible
```

## 18.5. Add to cart validation

```text
Given product requires variant
When user clicks Add to cart without selecting variant
Then validation message is visible
And cart count does not change
```

## 18.6. Add to cart success

```text
Given product is in stock
And user selected required variant
When user clicks Add to cart
Then success toast or mini cart is visible
And cart count increases
```

## 18.7. Out of stock

```text
Given product is out of stock
When user opens product detail
Then stock status shows Hết hàng
And Add to cart is disabled
And Buy now is disabled
```

## 18.8. Quantity validation

```text
Given stock quantity is 3
When user sets quantity to 4
Then quantity validation error is visible
And Add to cart is blocked
```

## 18.9. Scroll anchors

```text
When user clicks rating summary
Then page scrolls to review section
When user clicks View full specs
Then page scrolls to specs section
```

## 18.10. Compare action

```text
When user clicks Add to compare
Then product is added to compare list
And compare status is visible
```

## 18.11. Mobile sticky CTA

Viewport:

```text
375x812
```

Test:

```text
When user opens product detail on mobile
Then sticky CTA is visible
And it does not cover important content unexpectedly
And page has no horizontal overflow
```

## 18.12. Responsive visual checks

Run screenshots for:

```text
1440px desktop
1024px laptop
768px tablet
375px mobile
```

Check:

```text
No broken layout
No overlapping CTA
No horizontal overflow
Gallery ratio correct
Specs readable
Price block readable
```

---

## 19. Visual regression checklist

Capture screenshots:

```text
Product detail normal
Product detail sale
Product detail out of stock
Product detail with many variants
Product detail mobile
Product detail image modal
Product detail specs expanded
Product detail no reviews
```

Important visual assertions:

```text
Price is prominent
CTA is visible
Gallery is not distorted
Specs table does not overflow
Promotion box does not dominate page
Sticky mobile CTA works
```

---

## 20. Manual QA checklist

### Desktop

```text
Breadcrumb đúng
Gallery đổi ảnh đúng
Zoom/modal hoạt động
Tên/giá/SKU đúng
Khuyến mãi rõ
Variant selected rõ
Quantity valid
Add to cart hoạt động
Buy now hoạt động
Specs đầy đủ
Review section đúng
Related products đúng
```

### Mobile

```text
Không overflow ngang
Gallery swipe được
CTA sticky không che nội dung
Variant selector dễ bấm
Specs dễ đọc
Accordion hoạt động
Font không quá nhỏ
```

### Edge cases

```text
Tên sản phẩm rất dài
Nhiều ảnh
Không có ảnh
Không có sale
Nhiều promotion
Hết hàng
Pre-order
Nhiều biến thể
Thiếu specs
Không có reviews
API reviews lỗi
```

---

## 21. Definition of Done

Trang Product Detail được xem là hoàn thành khi:

```text
Product data render đúng
Gallery hoạt động trên desktop/mobile
Price/sale/discount hiển thị đúng
Variant selector validate đúng
Stock status điều khiển CTA đúng
Add to cart thành công/fail có feedback
Buy now hoạt động đúng flow
Quick specs hiển thị rõ
Technical specs không overflow mobile
Reviews/Q&A có state rõ
Related products hiển thị đúng
SEO metadata có đủ
Accessibility cơ bản đạt
Playwright tests chính pass
Visual regression không có diff nghiêm trọng
No console error nghiêm trọng
No horizontal overflow ở mobile
```

---

## 22. MVP scope

Nếu làm MVP đầu tiên, bắt buộc có:

```text
Breadcrumb
Gallery cơ bản
Product name
Brand/SKU
Price block
Promotion box đơn giản
Variant selector
Quantity selector
Stock status
Add to cart
Buy now
Trust/warranty box
Quick specs
Technical specs
Related products
Mobile sticky CTA
Loading/error/not found state
```

Có thể để sau:

```text
Image zoom nâng cao
Video gallery
Q&A
Review form
Recommendation AI
Delivery estimator realtime
Installment calculator nâng cao
Product comparison nâng cao
Recently viewed nâng cao
```

---

## 23. Ghi chú tái sử dụng source

Trang này phải dùng được cho nhiều loại đồ điện tử:

```text
Laptop
Điện thoại
Tablet
Tai nghe
Màn hình
Linh kiện PC
Máy ảnh
Thiết bị mạng
Đồ gia dụng điện tử
```

Vì vậy:

```text
Không hard-code spec CPU/RAM/SSD vào component gốc.
Không hard-code variant storage/color cho mọi category.
Không hard-code warranty text.
Không hard-code related section chỉ cho laptop.
Dùng category attribute template để render specs.
Dùng data-driven UI cho variant, specs, promotion, warranty.
```

Component phải đọc dữ liệu dạng generic rồi hiển thị theo template cấu hình.

---

## 24. File tiếp theo nên tạo

Sau file này, nên tạo tiếp:

```text
05-storefront-cart-page.md
```

Vì sau Product Detail, flow tự nhiên là:

```text
Product Detail
-> Add to Cart
-> Cart Page
-> Checkout Page
```
