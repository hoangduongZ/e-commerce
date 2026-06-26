# 03 - Storefront Product List Page Specification

> Theme: Electronics Store  
> Depends on: `ecommerce_design_language.md`, `01-electronics-store-theme.md`, `02-storefront-home-page.md`  
> Page type: Public storefront page  
> Primary users: Guest customer, logged-in customer  
> Goal: Giúp khách duyệt, lọc, sắp xếp, so sánh và chọn sản phẩm điện tử nhanh, rõ thông số, không bị rối.

---

## 1. Mục đích của trang

Trang danh sách sản phẩm là nơi khách bắt đầu ra quyết định mua hàng.

Với web bán đồ điện tử, trang này quan trọng hơn nhiều ngành khác vì khách thường không mua chỉ vì ảnh đẹp. Họ cần so sánh:

```text
Giá
Thương hiệu
CPU / chip
RAM
Dung lượng
Màn hình
Pin
Bảo hành
Tình trạng còn hàng
Khuyến mãi
Đánh giá
```

Trang này phải giúp khách trả lời nhanh 4 câu hỏi:

1. Có sản phẩm tôi cần không?
2. Sản phẩm nào phù hợp ngân sách?
3. Sản phẩm nào có cấu hình tốt hơn?
4. Sản phẩm nào đáng mua nhất lúc này?

---

## 2. Vai trò trong storefront

```text
Home Page
  -> Category Product List Page
  -> Search Result Page
  -> Promotion Product List Page
  -> Brand Product List Page
  -> Product Detail Page
  -> Compare Page
  -> Cart Page
```

Trang này có thể xuất hiện dưới nhiều biến thể URL:

```text
/products
/search?q=laptop
/categories/laptop
/categories/phones
/brands/dell
/promotions/flash-sale
```

Tất cả các biến thể nên dùng chung một layout nền, chỉ khác dữ liệu đầu vào và tiêu đề trang.

---

## 3. Nguyên tắc thiết kế chính

### 3.1. Ưu tiên scan nhanh

Khách không đọc từng dòng. Họ scan nhanh theo pattern:

```text
Ảnh sản phẩm
Tên sản phẩm
Thông số nhanh
Giá
Khuyến mãi
Tình trạng hàng
Đánh giá
Nút hành động
```

Vì vậy product card phải có hierarchy rõ.

### 3.2. Filter phải mạnh nhưng không gây sợ

Đồ điện tử có nhiều thuộc tính. Nếu show toàn bộ filter ngay từ đầu, giao diện sẽ nặng.

Quy tắc:

```text
Desktop: sidebar filter cố định bên trái.
Tablet: filter có thể thu gọn.
Mobile: filter nằm trong drawer/bottom sheet.
```

Chỉ hiển thị các filter quan trọng đầu tiên. Các nhóm ít dùng có thể collapse.

### 3.3. Không hard-code theo ngành nhỏ

Trang này phải dùng được cho:

```text
Laptop
Điện thoại
Tablet
Tai nghe
Màn hình
Linh kiện PC
Đồ gia dụng điện tử
```

Vì vậy filter phải dựa trên `Category Attribute Template`, không viết cứng field như `cpu`, `ram`, `storage` vào UI.

### 3.4. Dữ liệu quyết định layout

Nếu category là laptop, quick specs có thể là:

```text
CPU / RAM / SSD / GPU / Screen
```

Nếu category là điện thoại, quick specs có thể là:

```text
Chip / RAM / Storage / Camera / Battery
```

Nếu category là tai nghe, quick specs có thể là:

```text
Type / Bluetooth / Battery / Noise Canceling
```

UI component giữ nguyên. Dữ liệu cấu hình quyết định nội dung.

---

## 4. Loại trang và context đầu vào

### 4.1. Category page

Ví dụ:

```text
/categories/laptop
/categories/smartphone
/categories/headphones
```

Context:

```json
{
  "pageType": "category",
  "categorySlug": "laptop"
}
```

### 4.2. Search page

Ví dụ:

```text
/search?q=laptop+dell+i5
```

Context:

```json
{
  "pageType": "search",
  "query": "laptop dell i5"
}
```

### 4.3. Brand page

Ví dụ:

```text
/brands/dell
/brands/apple
```

Context:

```json
{
  "pageType": "brand",
  "brandSlug": "dell"
}
```

### 4.4. Promotion page

Ví dụ:

```text
/promotions/back-to-school
/promotions/flash-sale
```

Context:

```json
{
  "pageType": "promotion",
  "promotionSlug": "back-to-school"
}
```

---

## 5. Layout tổng thể

### 5.1. Desktop layout

Breakpoint tham khảo:

```text
>= 1200px
```

Layout:

```text
Global Header
Breadcrumb
Page Title + Result Summary
Featured Category Banner / Search Summary
Quick Filter Chips
Main Content
  Left: Filter Sidebar
  Right: Product Area
    Toolbar
    Product Grid
    Pagination / Load More
SEO Content Block
Recently Viewed
Footer
```

Grid:

```text
Container max-width: theo design system gốc
Filter sidebar width: 260px - 320px
Gap giữa filter và product grid: 24px
Product grid: 3 hoặc 4 cột tùy width
```

### 5.2. Laptop/tablet layout

Breakpoint tham khảo:

```text
768px - 1199px
```

Layout:

```text
Header
Breadcrumb
Title
Filter toolbar row
Product grid 2 - 3 cột
Pagination
Footer
```

Filter có thể là:

```text
Button "Bộ lọc"
Sort dropdown
Compare shortcut
Applied filter chips
```

### 5.3. Mobile layout

Breakpoint tham khảo:

```text
<= 767px
```

Layout:

```text
Mobile Header
Search Bar
Breadcrumb rút gọn hoặc ẩn
Title + result count
Sticky control bar
  Button Filter
  Button Sort
  Button Compare
Applied filter chips horizontal scroll
Product grid 2 cột hoặc list compact
Pagination / Load More
Footer compact
```

Mobile filter mở bằng drawer/bottom sheet.

Không để sidebar filter xuất hiện trực tiếp trên mobile.

---

## 6. Page header

### 6.1. Breadcrumb

Ví dụ category page:

```text
Trang chủ > Laptop
```

Ví dụ brand page:

```text
Trang chủ > Thương hiệu > Dell
```

Ví dụ search page:

```text
Trang chủ > Tìm kiếm
```

Rule:

```text
Desktop: hiển thị đầy đủ.
Mobile: có thể rút gọn, ví dụ "Trang chủ > Laptop".
Không dùng breadcrumb quá dài.
```

### 6.2. Page title

Theo page type:

```text
Category: Laptop
Search: Kết quả tìm kiếm cho "laptop dell i5"
Brand: Dell
Promotion: Flash Sale Laptop
```

Rule hiển thị:

```text
Title rõ, không quá dài.
Có result count ngay gần title.
Nếu query không có kết quả, vẫn hiển thị query để khách biết mình đang tìm gì.
```

Ví dụ:

```text
Laptop
128 sản phẩm
```

### 6.3. Category description

Chỉ hiển thị đoạn ngắn ở đầu trang.

Ví dụ:

```text
Laptop chính hãng, bảo hành rõ ràng, nhiều lựa chọn cho học tập, văn phòng, gaming và đồ họa.
```

Rule:

```text
Desktop: hiển thị tối đa 2 dòng.
Mobile: có thể collapse với nút "Xem thêm".
SEO content dài đặt cuối trang, không nhồi vào đầu.
```

---

## 7. Featured category banner

### 7.1. Mục đích

Banner giúp page có cảm giác chuyên ngành và đẩy campaign.

Ví dụ:

```text
Back to School Laptop Sale
Giảm đến 20%, tặng balo và chuột không dây
```

### 7.2. Khi nào hiển thị

Hiển thị nếu có config từ admin:

```text
Category campaign active
Brand campaign active
Promotion landing page
```

Không bắt buộc ở search page.

### 7.3. Nội dung

```text
Title
Subtitle
Background image hoặc gradient
CTA optional
Campaign badge optional
```

### 7.4. Rule UI

```text
Banner không được cao quá 240px desktop.
Mobile dùng ratio thấp hơn, tránh đẩy product xuống quá xa.
CTA không được là hành động chính của page nếu đã ở đúng category.
```

---

## 8. Quick filter chips

### 8.1. Mục đích

Quick filter giúp khách lọc nhanh theo nhu cầu phổ biến.

Ví dụ cho laptop:

```text
Văn phòng
Gaming
Đồ họa
Mỏng nhẹ
Sinh viên
Dưới 15 triệu
RAM 16GB
SSD 512GB
```

Ví dụ cho điện thoại:

```text
5G
Pin trâu
Camera đẹp
Chụp đêm
Dưới 10 triệu
256GB
Sạc nhanh
```

### 8.2. Rule

```text
Quick filter là shortcut của filter thật.
Không tạo logic riêng.
Khi click chip, URL query phải thay đổi.
Chip active phải hiển thị rõ.
Có thể bỏ chip bằng nút x trong applied filters.
```

### 8.3. Data contract

```json
{
  "quickFilters": [
    {
      "id": "gaming",
      "label": "Gaming",
      "query": {
        "attributes": {
          "usage": ["gaming"]
        }
      }
    },
    {
      "id": "ram-16gb",
      "label": "RAM 16GB",
      "query": {
        "attributes": {
          "ram": ["16gb"]
        }
      }
    }
  ]
}
```

---

## 9. Filter sidebar

### 9.1. Mục đích

Filter giúp khách thu hẹp danh sách sản phẩm theo điều kiện.

Với đồ điện tử, filter là phần sống còn.

### 9.2. Nhóm filter bắt buộc

```text
Category/Subcategory
Brand
Price range
Promotion
Availability
Rating
Dynamic attributes
Warranty
```

### 9.3. Nhóm filter động theo danh mục

Ví dụ laptop:

```text
CPU
RAM
Storage
GPU
Screen size
Screen refresh rate
Weight
Usage need
Operating system
```

Ví dụ điện thoại:

```text
Chip
RAM
Storage
Screen size
Camera
Battery
Charging
SIM
5G support
```

Ví dụ tai nghe:

```text
Connection type
Battery life
Noise cancellation
Microphone
Water resistance
Use case
```

### 9.4. Filter item types

| Type | Dùng cho |
|---|---|
| checkbox | Brand, RAM, storage |
| radio | Availability |
| range | Price, screen size |
| rating | Rating |
| toggle | On sale, in stock |
| search-in-filter | Brand dài |
| color swatch | Màu sắc |

Không đưa câu dài vào bảng. Mỗi type phải được document chi tiết ở component spec nếu cần.

### 9.5. Filter group behavior

```text
Mặc định mở các group quan trọng: Brand, Price, CPU/RAM hoặc Chip/Storage.
Các group ít dùng có thể collapse.
Khi group có filter active, group phải auto mở.
Mỗi group có thể có "Xem thêm" nếu nhiều hơn 6 option.
```

### 9.6. Price range

Yêu cầu:

```text
Có preset range.
Có input min/max.
Validate min <= max.
Format tiền rõ ràng.
Không apply filter ngay khi đang gõ nếu gây lag.
```

Ví dụ preset laptop:

```text
Dưới 10 triệu
10 - 15 triệu
15 - 20 triệu
20 - 30 triệu
Trên 30 triệu
```

### 9.7. Applied filters

Khi khách chọn filter, hiển thị chips:

```text
Dell x
RAM 16GB x
Dưới 15 triệu x
Còn hàng x
Xóa tất cả
```

Rule:

```text
Desktop: đặt trên product grid toolbar.
Mobile: horizontal scroll ngay dưới sticky control bar.
Chip phải có label dễ hiểu.
Không hiển thị raw id như ram_16gb.
```

### 9.8. URL synchronization

Mọi filter phải sync vào URL để share được.

Ví dụ:

```text
/categories/laptop?brand=dell,hp&ram=16gb&price_min=10000000&price_max=20000000&sort=price_asc&page=1
```

Rule:

```text
Khi filter thay đổi, reset page về 1.
URL không chứa param rỗng.
Param cần stable để SEO và share link.
Không lưu filter quan trọng chỉ trong local state.
```

### 9.9. Mobile filter drawer

Mobile filter mở bằng button:

```text
Bộ lọc
```

Drawer gồm:

```text
Header: Bộ lọc + nút đóng
Body: filter groups
Footer sticky: Xóa tất cả + Áp dụng
```

Rule:

```text
Không apply từng click ngay trên mobile nếu drawer chưa bấm Áp dụng.
Có count số filter đang chọn.
Drawer không được vượt màn hình.
Body scroll độc lập.
Footer luôn visible.
```

---

## 10. Product toolbar

### 10.1. Thành phần

Desktop toolbar gồm:

```text
Result count
Applied filter chips
View mode toggle
Sort dropdown
Compare status
```

Mobile toolbar gồm:

```text
Filter button
Sort button
Compare button
Result count compact
```

### 10.2. Result count

Ví dụ:

```text
128 sản phẩm
```

Với search:

```text
Tìm thấy 24 sản phẩm cho "laptop dell i5"
```

### 10.3. Sort options

| Key | Label |
|---|---|
| relevance | Phù hợp nhất |
| newest | Mới nhất |
| best_selling | Bán chạy |
| price_asc | Giá thấp đến cao |
| price_desc | Giá cao đến thấp |
| rating_desc | Đánh giá cao |
| discount_desc | Giảm giá nhiều |

Rule:

```text
Search page mặc định relevance.
Category page mặc định best_selling hoặc newest theo config.
Sort thay đổi phải update URL.
```

### 10.4. View mode

Desktop có thể hỗ trợ:

```text
Grid view
List view
Compact specs view
```

MVP có thể chỉ cần grid view.

Nếu có list view, list view phải ưu tiên thông số kỹ thuật và compare.

---

## 11. Product grid

### 11.1. Desktop grid

```text
3 hoặc 4 cột tùy width.
Gap theo design system.
Card cùng hàng nên có height ổn định.
Ảnh sản phẩm có ratio thống nhất.
```

### 11.2. Tablet grid

```text
2 hoặc 3 cột.
Filter có thể thu gọn.
Product card giảm bớt quick specs nếu thiếu không gian.
```

### 11.3. Mobile grid

Có 2 lựa chọn:

```text
2-column compact grid
1-column list card
```

Khuyến nghị cho đồ điện tử:

```text
Mobile category page: 2-column compact grid.
Mobile search result phức tạp: có thể dùng 1-column list nếu cần nhiều specs.
```

Rule mobile:

```text
Không overflow ngang.
Tên sản phẩm tối đa 2 dòng.
Quick specs tối đa 2-3 dòng.
Button không quá to làm vỡ card.
Compare có thể là icon nhỏ.
```

---

## 12. Product card specification

### 12.1. Mục tiêu

Product card phải giúp khách hiểu nhanh:

```text
Sản phẩm gì?
Cấu hình chính là gì?
Giá bao nhiêu?
Có sale không?
Còn hàng không?
Có đáng tin không?
Có thể so sánh không?
```

### 12.2. Thành phần card

```text
Product image
Badges
Product name
Quick specs
Rating summary
Price block
Promotion summary
Availability
Actions
Compare control
```

### 12.3. Card data contract

```json
{
  "id": "p_1001",
  "slug": "dell-inspiron-15-i5-16gb-512gb",
  "name": "Laptop Dell Inspiron 15 Intel Core i5 16GB 512GB",
  "brand": "Dell",
  "image": {
    "url": "/images/products/dell-inspiron-15.jpg",
    "alt": "Laptop Dell Inspiron 15 màu bạc"
  },
  "price": 15990000,
  "compareAtPrice": 18990000,
  "currency": "VND",
  "discountPercent": 16,
  "rating": 4.6,
  "reviewCount": 128,
  "soldCount": 530,
  "availability": "in_stock",
  "warrantyMonths": 24,
  "quickSpecs": [
    { "label": "CPU", "value": "Intel Core i5" },
    { "label": "RAM", "value": "16GB" },
    { "label": "SSD", "value": "512GB" },
    { "label": "Màn hình", "value": "15.6 inch FHD" }
  ],
  "badges": [
    { "type": "sale", "label": "-16%" },
    { "type": "official", "label": "Chính hãng" }
  ],
  "promotions": [
    "Tặng chuột không dây",
    "Hỗ trợ trả góp 0%"
  ]
}
```

### 12.4. Visual hierarchy

Thứ tự ưu tiên thị giác:

```text
Ảnh
Tên sản phẩm
Quick specs
Giá bán
Badge sale
Rating / review
Availability
Actions
```

Giá bán phải nổi bật hơn thông số.
Tên sản phẩm phải rõ hơn badge.
Badge không được làm nhiễu card.

### 12.5. Product image

Rule:

```text
Nền ảnh nên sáng, sạch.
Ảnh giữ ratio thống nhất.
Không crop mất sản phẩm.
Có placeholder khi ảnh lỗi.
Lazy load ảnh ngoài viewport.
Alt text phải có tên sản phẩm.
```

### 12.6. Product name

Rule:

```text
Desktop: tối đa 2 dòng trong grid view.
Mobile: tối đa 2 dòng.
List view: có thể 3 dòng.
Tên không được bị cắt làm mất model quan trọng nếu có đủ không gian.
```

Ví dụ tốt:

```text
Laptop Dell Inspiron 15 Intel Core i5 16GB 512GB
```

Ví dụ xấu:

```text
Sản phẩm tuyệt vời giá rẻ siêu hot...
```

### 12.7. Quick specs

Rule:

```text
Hiển thị 3-5 specs quan trọng.
Specs lấy từ category attribute template.
Không hard-code field.
Nếu thiếu dữ liệu, ẩn spec đó.
Không hiển thị dấu gạch hoặc null.
```

Format:

```text
Core i5 · 16GB · SSD 512GB · 15.6" FHD
```

Hoặc:

```text
CPU: Core i5
RAM: 16GB
SSD: 512GB
```

Tùy density của card.

### 12.8. Price block

Rule:

```text
Giá hiện tại nổi bật.
Giá gốc hiển thị nhỏ hơn và gạch ngang nếu có sale.
Discount percent dùng badge ngắn.
Không hiển thị giá gốc nếu bằng giá bán.
Nếu sản phẩm cần liên hệ giá, hiển thị "Liên hệ".
```

Ví dụ:

```text
15.990.000đ
18.990.000đ  -16%
```

### 12.9. Availability

Các trạng thái:

| Key | Label |
|---|---|
| in_stock | Còn hàng |
| low_stock | Sắp hết |
| out_of_stock | Hết hàng |
| pre_order | Đặt trước |
| coming_soon | Sắp về hàng |

Rule:

```text
Còn hàng dùng tone success.
Sắp hết dùng tone warning.
Hết hàng dùng tone disabled/danger nhẹ.
Hết hàng thì disable add to cart.
Đặt trước thì CTA đổi thành "Đặt trước".
```

### 12.10. Actions

Card action desktop:

```text
Primary: Thêm vào giỏ
Secondary: So sánh
Optional: Xem nhanh
```

Card action mobile:

```text
Icon cart hoặc nút nhỏ
Compare icon
```

Rule:

```text
Click vào card/name/image đi tới product detail.
Click button không được trigger navigation card.
Add to cart phải có loading state.
Nếu thiếu variant bắt buộc, mở quick variant modal hoặc vào detail.
```

---

## 13. Compare feature

### 13.1. Mục đích

Khách mua đồ điện tử thường cần so sánh cấu hình.

Compare là tính năng nên có trong theme electronics.

### 13.2. Basic behavior

```text
Khách chọn checkbox/icon "So sánh" trên product card.
Sản phẩm được thêm vào compare tray.
Compare tray hiển thị số lượng đã chọn.
Khách bấm "So sánh" để vào compare page.
```

### 13.3. Giới hạn

```text
Tối đa 4 sản phẩm.
Chỉ so sánh sản phẩm cùng category hoặc cùng compare group.
Nếu chọn khác category, hiển thị thông báo rõ.
```

Ví dụ thông báo:

```text
Chỉ có thể so sánh các sản phẩm cùng nhóm Laptop.
```

### 13.4. Compare tray

Desktop:

```text
Fixed bottom bar hoặc floating mini panel.
Hiển thị thumbnail + tên rút gọn.
Có nút Xóa từng sản phẩm.
Có nút So sánh.
```

Mobile:

```text
Floating button: So sánh (2)
Mở bottom sheet để xem sản phẩm đã chọn.
```

### 13.5. Data lưu trữ

Có thể lưu local storage cho guest.

```json
{
  "compare": {
    "categoryId": "laptop",
    "items": ["p_1001", "p_1002"]
  }
}
```

---

## 14. Pagination / Load more

### 14.1. Lựa chọn UX

Có 3 cách:

```text
Pagination truyền thống
Load more
Infinite scroll
```

Khuyến nghị:

```text
Desktop: pagination hoặc load more.
Mobile: load more thân thiện hơn.
SEO page: ưu tiên pagination có URL rõ.
```

### 14.2. Rule

```text
Khi đổi filter, reset về page 1.
Khi đổi sort, reset về page 1.
URL phải phản ánh page hiện tại.
Không dùng infinite scroll nếu làm SEO category nghiêm túc mà không có fallback.
```

### 14.3. Empty page guard

Nếu URL page vượt quá số trang:

```text
Hiển thị empty state hoặc redirect về page cuối hợp lệ.
Không crash.
```

---

## 15. Loading state

### 15.1. Initial loading

Khi mới vào trang:

```text
Skeleton breadcrumb/title optional
Skeleton filter sidebar
Skeleton product cards
```

Card skeleton gồm:

```text
Image block
Title lines
Specs lines
Price line
Button block
```

### 15.2. Filter loading

Khi thay đổi filter:

```text
Không làm page trắng hoàn toàn.
Giữ filter hiện tại.
Product area có loading overlay nhẹ hoặc skeleton.
Toolbar có trạng thái đang cập nhật.
```

### 15.3. Add to cart loading

Khi bấm thêm giỏ:

```text
Button loading riêng card đó.
Không block toàn bộ page.
Sau khi thành công, toast hiển thị.
```

---

## 16. Empty state

### 16.1. Không có sản phẩm sau filter

Hiển thị:

```text
Không tìm thấy sản phẩm phù hợp
Hãy thử bỏ bớt bộ lọc hoặc thay đổi khoảng giá.
[ Xóa tất cả bộ lọc ]
```

Gợi ý:

```text
Danh mục liên quan
Sản phẩm bán chạy
Từ khóa phổ biến
```

### 16.2. Không có kết quả search

Hiển thị:

```text
Không tìm thấy kết quả cho "..."
```

Gợi ý:

```text
Kiểm tra chính tả
Dùng từ khóa ngắn hơn
Thử tìm theo thương hiệu hoặc model
```

### 16.3. Category chưa có sản phẩm

Hiển thị:

```text
Danh mục này chưa có sản phẩm
```

Không hiển thị filter sidebar rỗng nếu không có dữ liệu.

---

## 17. Error state

### 17.1. API lỗi

Hiển thị trong product area:

```text
Không thể tải danh sách sản phẩm
[ Thử lại ]
```

Rule:

```text
Không làm mất filter đã chọn.
Retry gọi lại API với cùng query.
Ghi log lỗi cho monitoring.
```

### 17.2. Filter config lỗi

Nếu product API thành công nhưng filter config lỗi:

```text
Vẫn hiển thị product list.
Ẩn hoặc show fallback filter tối thiểu.
Không block toàn bộ page.
```

Fallback filter:

```text
Brand
Price
Availability
```

---

## 18. SEO rules

### 18.1. Title tag

Category page:

```text
Laptop chính hãng, giá tốt | Store Name
```

Search page:

```text
Tìm kiếm "laptop dell i5" | Store Name
```

Brand page:

```text
Laptop Dell chính hãng | Store Name
```

### 18.2. Meta description

Ngắn, rõ:

```text
Mua laptop chính hãng, nhiều cấu hình, bảo hành rõ ràng, giao hàng nhanh. Xem giá, khuyến mãi và so sánh sản phẩm tại Store Name.
```

### 18.3. Canonical

Rule:

```text
Category base page cần canonical rõ.
Các filter nhiều tổ hợp có thể noindex tùy chiến lược SEO.
Search result thường noindex nếu query tự do.
Promotion page có thể index nếu là landing page có nội dung riêng.
```

### 18.4. Structured data

Có thể dùng:

```text
ItemList
Product
BreadcrumbList
```

Rule:

```text
Structured data phải khớp dữ liệu hiển thị.
Không đưa rating nếu không có review thật.
Không fake availability.
```

### 18.5. SEO content block cuối trang

Category page có thể có block nội dung ở cuối:

```text
Giới thiệu danh mục
Cách chọn sản phẩm
Câu hỏi thường gặp
Link danh mục liên quan
```

Rule:

```text
Không đẩy product xuống quá sâu.
Không nhồi keyword.
Có thể collapse FAQ.
```

---

## 19. Accessibility rules

### 19.1. Filter

```text
Filter group phải dùng semantic heading hoặc aria-label.
Checkbox/radio có label rõ.
Drawer focus trap trên mobile.
Esc đóng drawer.
Tab order hợp lý.
```

### 19.2. Product card

```text
Tên sản phẩm là link rõ ràng.
Ảnh có alt text.
Button add to cart có accessible name.
Giá sale và giá gốc phải đọc được bởi screen reader.
Badge không chỉ dựa vào màu.
```

### 19.3. Sort

```text
Dropdown có label.
Khi sort thay đổi, screen reader biết danh sách đang cập nhật.
```

### 19.4. Loading

```text
Product area có aria-busy khi đang tải.
Không làm focus nhảy bất ngờ.
```

---

## 20. Performance rules

### 20.1. Product images

```text
Lazy load ảnh dưới fold.
Dùng responsive image sizes.
Ưu tiên format hiện đại nếu có.
Có placeholder nhẹ.
```

### 20.2. Filter interaction

```text
Debounce input search/range.
Không gọi API quá nhiều khi kéo slider.
Dùng cancel request nếu query mới tới.
```

### 20.3. Product query

```text
Phân trang.
Chỉ trả fields cần cho list page.
Không trả description dài.
Không trả toàn bộ spec nếu chỉ cần quick specs.
```

### 20.4. Caching

Có thể cache:

```text
Category tree
Filter config
Brand list
Product list query phổ biến
```

---

## 21. Data contract tổng quát

### 21.1. Request query

```json
{
  "pageType": "category",
  "categorySlug": "laptop",
  "search": "",
  "brand": ["dell", "hp"],
  "priceMin": 10000000,
  "priceMax": 20000000,
  "attributes": {
    "ram": ["16gb"],
    "storage": ["512gb"],
    "cpu_family": ["intel-core-i5"]
  },
  "availability": ["in_stock"],
  "ratingMin": 4,
  "sort": "price_asc",
  "page": 1,
  "pageSize": 24
}
```

### 21.2. Response shape

```json
{
  "meta": {
    "pageType": "category",
    "title": "Laptop",
    "description": "Laptop chính hãng, nhiều cấu hình.",
    "totalItems": 128,
    "page": 1,
    "pageSize": 24,
    "totalPages": 6
  },
  "filters": [
    {
      "id": "brand",
      "label": "Thương hiệu",
      "type": "checkbox",
      "options": [
        { "value": "dell", "label": "Dell", "count": 24 },
        { "value": "hp", "label": "HP", "count": 18 }
      ]
    },
    {
      "id": "ram",
      "label": "RAM",
      "type": "checkbox",
      "options": [
        { "value": "8gb", "label": "8GB", "count": 32 },
        { "value": "16gb", "label": "16GB", "count": 64 }
      ]
    }
  ],
  "sortOptions": [
    { "value": "best_selling", "label": "Bán chạy" },
    { "value": "price_asc", "label": "Giá thấp đến cao" }
  ],
  "products": []
}
```

### 21.3. Product list item shape

```json
{
  "id": "p_1001",
  "slug": "dell-inspiron-15-i5-16gb-512gb",
  "name": "Laptop Dell Inspiron 15 Intel Core i5 16GB 512GB",
  "categoryId": "laptop",
  "brand": "Dell",
  "image": {
    "url": "/images/products/dell-inspiron-15.jpg",
    "alt": "Laptop Dell Inspiron 15"
  },
  "price": 15990000,
  "compareAtPrice": 18990000,
  "currency": "VND",
  "rating": 4.6,
  "reviewCount": 128,
  "availability": "in_stock",
  "quickSpecs": [],
  "badges": [],
  "promotions": []
}
```

---

## 22. Admin configuration needed

Trang này phụ thuộc vào admin config sau:

```text
Category tree
Category attribute template
Filter visibility per category
Quick filter per category
Sort default per category
Banner per category/brand/promotion
Product card quick specs per category
SEO metadata
Noindex/index rule per filter pattern
```

Admin nên có màn hình cấu hình:

```text
Admin > Catalog > Categories > Product List Settings
```

Fields đề xuất:

```text
Default sort
Page size
Quick filters
Visible filter groups
Collapsed filter groups
Quick specs fields
Category banner
SEO title
SEO description
SEO content
Indexing rule
```

---

## 23. Component breakdown

### 23.1. Page-level components

```text
ProductListPage
ProductListHeader
CategoryBanner
QuickFilterChips
ProductFilterSidebar
MobileFilterDrawer
AppliedFilterChips
ProductToolbar
ProductGrid
ProductCard
CompareTray
Pagination
ProductListSeoContent
RecentlyViewedSection
```

### 23.2. Filter components

```text
FilterGroup
CheckboxFilter
RadioFilter
RangeFilter
RatingFilter
ToggleFilter
FilterSearchBox
ColorSwatchFilter
```

### 23.3. Shared components

```text
Breadcrumb
Badge
PriceDisplay
RatingStars
AvailabilityBadge
SkeletonCard
EmptyState
ErrorState
Toast
Drawer
Modal
```

### 23.4. Component ownership

```text
ProductCard dùng lại ở Home, Product List, Related Products.
PriceDisplay dùng lại ở Product Card, Detail, Cart.
AvailabilityBadge dùng lại ở Card, Detail, Admin.
Filter components chỉ dùng ở listing/search.
CompareTray dùng ở listing và compare entry points.
```

---

## 24. URL examples

### 24.1. Laptop category

```text
/categories/laptop
/categories/laptop?brand=dell&ram=16gb&sort=price_asc
/categories/laptop?usage=gaming&gpu=rtx-3050
```

### 24.2. Phone category

```text
/categories/smartphone?brand=apple,samsung&storage=256gb
/categories/smartphone?price_max=15000000&support_5g=true
```

### 24.3. Search

```text
/search?q=laptop+dell+i5
/search?q=iphone+15&sort=price_asc
```

### 24.4. Brand

```text
/brands/dell
/brands/apple?category=smartphone
```

---

## 25. Interaction rules

### 25.1. Filter apply desktop

Desktop có thể apply ngay khi click.

Flow:

```text
User check RAM 16GB
URL update
Product area loading
Result update
Applied chip appears
```

### 25.2. Filter apply mobile

Mobile nên apply khi bấm nút.

Flow:

```text
User mở drawer
User chọn filter
Footer count update
User bấm Áp dụng
Drawer đóng
URL update
Product list update
```

### 25.3. Clear filters

```text
Clear one chip: bỏ đúng filter đó.
Clear all: giữ category/search context, bỏ filter/sort/page.
```

Ví dụ:

```text
/categories/laptop?brand=dell&ram=16gb
Clear all -> /categories/laptop
```

### 25.4. Sort interaction

```text
User chọn sort
URL update sort
Page reset về 1
Product list update
```

### 25.5. Product navigation

```text
Click ảnh/tên/card area -> Product Detail.
Middle click/open new tab phải hoạt động nếu dùng anchor.
Button action không trigger navigation.
```

---

## 26. Business rules

### 26.1. Giá

```text
Không hiển thị giá âm.
Nếu price null và allow_contact_price true, hiển thị "Liên hệ".
Nếu compareAtPrice <= price, không hiển thị discount.
Format tiền theo locale.
```

### 26.2. Tồn kho

```text
out_of_stock disable add to cart.
low_stock hiển thị warning nhẹ.
pre_order CTA là Đặt trước.
coming_soon không cho mua nếu chưa mở preorder.
```

### 26.3. Promotion

```text
Chỉ hiển thị promotion còn hiệu lực.
Không hiển thị quá 2 dòng promotion trong card.
Promotion dài đưa vào detail page.
```

### 26.4. Rating

```text
Chỉ hiển thị rating nếu reviewCount > 0.
Không fake rating.
Nếu chưa có review, có thể hiển thị "Chưa có đánh giá".
```

### 26.5. Compare

```text
Compare chỉ cho sản phẩm có compareGroup giống nhau.
Compare item hết hàng vẫn so sánh được.
Compare không bắt buộc đăng nhập.
```

---

## 27. Responsive detail

### 27.1. Desktop

```text
Filter sidebar luôn visible.
Product grid đủ khoảng thở.
Toolbar không sticky mặc định.
Compare tray có thể fixed bottom.
```

### 27.2. Tablet

```text
Filter sidebar có thể collapse.
Product grid 2-3 cột.
Quick specs rút gọn.
```

### 27.3. Mobile

```text
Không sidebar.
Control bar sticky top dưới header.
Filter drawer bottom sheet.
Applied chips horizontal scroll.
Product card compact.
Compare floating button.
```

### 27.4. Mobile anti-break rules

```text
Không overflow ngang ở 320px, 360px, 375px, 414px.
Không dùng fixed width lớn.
Button không chồng lên text.
Drawer footer không che option cuối.
Product image không méo.
```

---

## 28. MVP scope

MVP nên có:

```text
Category page
Search page
Product grid
Filter brand/price/availability/dynamic attributes
Sort
Applied chips
Pagination hoặc load more
Product card electronics
Mobile filter drawer
Add to cart basic
Compare basic optional
Loading/empty/error state
```

MVP có thể chưa cần:

```text
Advanced recommendation
AI search
Infinite scroll
Personalized sort
A/B testing
Complex SEO noindex matrix
Multi-warehouse availability per area
```

---

## 29. Agent implementation rules

Agent/code bắt buộc tuân thủ:

```text
Không hard-code filter theo laptop/phone.
Filter phải đọc từ config/API.
Không bỏ loading/empty/error state.
Không dùng CSS width cố định gây overflow mobile.
Không xóa accessibility label.
Không sửa ProductCard làm hỏng Home Page.
Không đổi token màu ngoài electronics theme nếu chưa có yêu cầu.
Không tự invent trạng thái availability ngoài danh sách chuẩn.
```

Khi implement trang này, agent phải:

```text
Đọc design system gốc.
Đọc electronics theme.
Đọc spec trang này.
Xác định component dùng lại.
Viết hoặc cập nhật test.
Code.
Chạy test.
Chụp screenshot desktop/mobile nếu có tool.
Báo cáo file sửa và test đã chạy.
```

---

## 30. Playwright test specification

### 30.1. Basic render

Test:

```text
Vào /categories/laptop
Page title hiển thị
Result count hiển thị
Product cards hiển thị
Filter sidebar hiển thị trên desktop
```

### 30.2. Search result

Test:

```text
Vào /search?q=laptop
Title có query
Product list hiển thị hoặc empty state đúng
```

### 30.3. Filter desktop

Test:

```text
Click filter Brand Dell
URL có brand=dell
Applied chip Dell hiển thị
Product list reload
Page reset về 1
```

### 30.4. Clear filter

Test:

```text
Chọn nhiều filter
Click x ở chip RAM 16GB
Chỉ RAM bị bỏ
Brand vẫn giữ
Click Xóa tất cả
URL về category base
```

### 30.5. Sort

Test:

```text
Chọn Giá thấp đến cao
URL có sort=price_asc
Product list reload
```

### 30.6. Mobile filter drawer

Viewport:

```text
375x812
```

Test:

```text
Filter sidebar không hiển thị trực tiếp
Click Bộ lọc
Drawer mở
Chọn RAM 16GB
Click Áp dụng
Drawer đóng
Chip RAM 16GB hiển thị
Không overflow ngang
```

### 30.7. Product card actions

Test:

```text
Click tên sản phẩm -> đi tới detail page
Click Thêm vào giỏ -> cart count tăng hoặc toast hiển thị
Click So sánh -> compare tray count tăng
```

### 30.8. Empty state

Mock API trả totalItems = 0.

Test:

```text
Empty message hiển thị
Button Xóa tất cả bộ lọc hiển thị nếu có filter
Không crash
```

### 30.9. Error state

Mock API lỗi.

Test:

```text
Error message hiển thị
Button Thử lại hiển thị
Click Thử lại gọi API lại
```

### 30.10. Visual tests

Viewport cần chụp:

```text
1440 desktop
1024 laptop/tablet
768 tablet
375 mobile
```

Screens:

```text
Default category page
Filtered category page
Search no result
Mobile filter drawer open
Compare tray active
```

---

## 31. Definition of Done

Trang được coi là hoàn thành khi đạt đủ:

```text
Render đúng desktop/tablet/mobile.
Filter hoạt động và sync URL.
Sort hoạt động và sync URL.
Applied filter chips hoạt động.
Product card hiển thị đúng ảnh, tên, specs, giá, rating, tồn kho.
Mobile filter drawer hoạt động.
Loading/empty/error state đầy đủ.
Không overflow ngang mobile.
Accessibility cơ bản đạt.
SEO title/meta/canonical có strategy rõ.
Playwright test chính pass.
Visual test quan trọng pass hoặc có diff được chấp nhận.
```

---

## 32. Gợi ý thứ tự implement

Nên implement theo thứ tự:

```text
1. Static layout
2. Product card
3. Product grid
4. Toolbar + sort
5. Filter sidebar desktop
6. URL sync
7. Mobile filter drawer
8. Loading/empty/error state
9. Add to cart action
10. Compare tray
11. SEO block
12. Playwright tests
```

Không nên implement compare trước filter.
Không nên làm visual polish trước khi data contract ổn.

---

## 33. Ghi chú tái sử dụng source

Trang này phải được thiết kế để clone sang ngành khác.

Các phần giữ nguyên:

```text
ProductListPage structure
Filter engine
Sort engine
URL sync
Pagination
Product grid
Loading/empty/error state
```

Các phần thay đổi theo ngành:

```text
Theme token
Product card quick specs
Filter attribute template
Category banner
SEO content
Promotion copy
```

Nếu làm đúng, cùng source có thể dùng cho:

```text
Đồ điện tử
Thời trang
Mỹ phẩm
Đồ gia dụng
Sách
Nội thất
```

Chỉ cần đổi theme và attribute template.
