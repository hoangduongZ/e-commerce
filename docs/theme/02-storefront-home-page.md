# 02 - Storefront Home Page Specification

> Theme: Electronics Store  
> Depends on: `ecommerce_design_language.md`, `01-electronics-store-theme.md`  
> Page type: Public storefront page  
> Primary users: Guest customer, logged-in customer  
> Goal: Giúp khách nhanh chóng hiểu cửa hàng bán gì, tìm sản phẩm, xem khuyến mãi, khám phá danh mục, và đi vào luồng mua hàng.

---

## 1. Mục đích của trang

Trang chủ là “mặt tiền” của website bán đồ điện tử.

Trang này không chỉ để đẹp. Nó phải làm được 5 việc chính:

1. Cho khách biết đây là website bán đồ điện tử đáng tin.
2. Giúp khách tìm sản phẩm thật nhanh.
3. Đẩy các nhóm hàng quan trọng như laptop, điện thoại, tablet, tai nghe, linh kiện.
4. Làm nổi bật khuyến mãi, sản phẩm bán chạy, sản phẩm mới.
5. Dẫn khách vào các luồng mua hàng chính: xem danh mục, xem chi tiết, thêm giỏ, so sánh.

Trang chủ không nên chứa quá nhiều logic phức tạp. Nó nên là một trang tổng hợp các section có thể bật/tắt từ admin.

---

## 2. Vai trò của trang trong toàn bộ storefront

Trang chủ liên kết tới các màn hình chính:

```text
Home Page
  -> Product Search Page
  -> Category Page
  -> Product Detail Page
  -> Cart Page
  -> Login/Register Page
  -> Promotion Landing Page
  -> Compare Page
  -> Support/Warranty Page
```

Trang chủ phải hoạt động tốt với cả khách chưa đăng nhập và khách đã đăng nhập.

Với khách chưa đăng nhập, trang ưu tiên:

- tìm kiếm sản phẩm
- xem danh mục
- xem khuyến mãi
- thêm vào giỏ như guest

Với khách đã đăng nhập, trang có thể bổ sung:

- sản phẩm đã xem gần đây
- gợi ý dựa trên lịch sử
- nhắc đơn hàng đang xử lý
- hiển thị tên tài khoản trong header

---

## 3. Nguyên tắc thiết kế riêng cho trang chủ đồ điện tử

### 3.1. Cảm giác giao diện

Trang chủ phải tạo cảm giác:

```text
Hiện đại
Tin cậy
Công nghệ
Rõ ràng
Nhiều thông tin nhưng không rối
Dễ tìm sản phẩm
Dễ so sánh giá và thông số
```

Không dùng phong cách quá “cute”, quá nhiều màu pastel, hoặc layout quá nghệ thuật làm giảm khả năng đọc thông tin.

### 3.2. Thứ tự ưu tiên thị giác

Người dùng đồ điện tử thường quan tâm theo thứ tự:

```text
Sản phẩm có đúng nhu cầu không?
Giá bao nhiêu?
Thông số chính là gì?
Có khuyến mãi không?
Còn hàng không?
Bảo hành thế nào?
Có đáng tin không?
```

Vì vậy UI phải ưu tiên:

- search bar rõ
- danh mục rõ
- giá rõ
- badge khuyến mãi rõ
- thông số nhanh dễ scan
- nút mua/so sánh dễ thấy

### 3.3. Không biến trang chủ thành catalog quá nặng

Trang chủ chỉ nên hiển thị các nhóm sản phẩm đại diện.

Danh sách dài, filter phức tạp, phân trang chi tiết phải nằm ở Product Listing Page hoặc Category Page.

---

## 4. Cấu trúc tổng thể của trang

Thứ tự section chuẩn:

```text
1. Top Announcement Bar
2. Main Header
3. Navigation / Mega Menu
4. Hero Section
5. Quick Category Grid
6. Trust Benefits Strip
7. Flash Sale Section
8. Featured Categories Section
9. Best Seller Section
10. New Arrivals Section
11. Laptop Buying Guide / Editorial Section
12. Brand Showcase Section
13. Recently Viewed Section
14. Newsletter / Support CTA
15. Footer
```

Một số section có thể tắt nếu chưa có dữ liệu:

```text
Flash Sale Section
Recently Viewed Section
Newsletter Section
Editorial Section
Brand Showcase Section
```

Các section bắt buộc nên có cho MVP:

```text
Main Header
Navigation / Category Menu
Hero Section
Quick Category Grid
Best Seller Section
Footer
```

---

## 5. Layout desktop

Áp dụng cho màn hình từ `1024px` trở lên.

### 5.1. Page width

- Max content width: `1200px` hoặc `1280px`.
- Content căn giữa.
- Section full-width có thể dùng background riêng nhưng nội dung vẫn nằm trong container.

```text
| full viewport background |
|   centered container    |
```

### 5.2. Desktop layout tổng quát

```text
+------------------------------------------------------+
| Announcement Bar                                     |
+------------------------------------------------------+
| Header: Logo | Search | Hotline | Account | Cart     |
+------------------------------------------------------+
| Nav: Categories | Laptop | Phone | Deals | Support   |
+------------------------------------------------------+
| Hero: Left promo banner | Right small promo cards     |
+------------------------------------------------------+
| Quick Categories                                     |
+------------------------------------------------------+
| Trust Benefits                                       |
+------------------------------------------------------+
| Flash Sale                                           |
+------------------------------------------------------+
| Featured Category: Laptop                            |
+------------------------------------------------------+
| Featured Category: Phone                             |
+------------------------------------------------------+
| Best Sellers                                         |
+------------------------------------------------------+
| New Arrivals                                         |
+------------------------------------------------------+
| Guide / Brands / Support CTA                         |
+------------------------------------------------------+
| Footer                                               |
+------------------------------------------------------+
```

### 5.3. Desktop grid rules

Product grid on desktop:

```text
>= 1280px: 5 columns
1024px - 1279px: 4 columns
```

Category grid on desktop:

```text
>= 1280px: 8 columns
1024px - 1279px: 6 columns
```

Hero layout on desktop:

```text
Left: 2/3 width
Right: 1/3 width
```

---

## 6. Layout tablet

Áp dụng cho màn hình từ `768px` đến `1023px`.

Product grid:

```text
3 columns
```

Category grid:

```text
4 columns
```

Hero layout:

```text
Hero main banner full width
Small promo cards below, 2 columns
```

Header:

```text
Logo left
Search center
Cart/account right
Category menu hidden behind button
```

---

## 7. Layout mobile

Áp dụng cho màn hình dưới `768px`.

### 7.1. Mobile page rules

Mobile là màn hình ưu tiên. Không được chỉ “co desktop lại”.

Bắt buộc:

- Không overflow ngang ở `375px`.
- Search bar phải dễ bấm.
- Cart icon luôn thấy trong header.
- Hero không quá cao.
- Product card không quá nhiều text.
- CTA phải rõ.

### 7.2. Mobile layout tổng quát

```text
+-----------------------------+
| Announcement Bar            |
+-----------------------------+
| Header: Menu | Logo | Cart  |
+-----------------------------+
| Search Bar                  |
+-----------------------------+
| Hero Banner                 |
+-----------------------------+
| Quick Categories            |
+-----------------------------+
| Trust Benefits Carousel     |
+-----------------------------+
| Flash Sale                  |
+-----------------------------+
| Product Sections            |
+-----------------------------+
| Support CTA                 |
+-----------------------------+
| Footer Accordion            |
+-----------------------------+
```

### 7.3. Mobile grid rules

Product grid:

```text
< 480px: 2 columns
480px - 767px: 2 columns
```

Category grid:

```text
< 480px: 4 columns
480px - 767px: 4 columns
```

Hero:

```text
1 main banner only
Secondary promo cards can become horizontal scroll
```

Footer:

```text
Use accordion groups
```

---

## 8. Section specification

## 8.1. Top Announcement Bar

### Mục đích

Thông báo nhanh các thông tin quan trọng như miễn phí giao hàng, chương trình sale, hỗ trợ trả góp, hotline.

### Nội dung gợi ý

```text
Miễn phí giao hàng cho đơn từ 2.000.000đ
Trả góp 0% qua thẻ tín dụng
Bảo hành chính hãng
Hotline: 1900 xxxx
```

### Layout desktop

```text
Left: promo message
Right: links: Store locator | Warranty | Support
```

### Layout mobile

- Chỉ hiển thị một message ngắn.
- Có thể auto-slide nhiều message.
- Không hiển thị quá 1 dòng.

### UI rules

- Nền dùng màu `tech-navy` hoặc `primary-dark`.
- Text trắng hoặc neutral rất sáng.
- Chiều cao khoảng `32px - 40px`.
- Font nhỏ nhưng dễ đọc.

### State

- Nếu không có announcement: ẩn toàn bộ section.
- Nếu message dài: truncate hoặc chuyển thành marquee/slider nhẹ.

---

## 8.2. Main Header

### Mục đích

Cung cấp nhận diện thương hiệu, tìm kiếm sản phẩm, tài khoản, giỏ hàng.

### Thành phần

```text
Logo
Search bar
Hotline/support link
Account menu
Cart icon
Mobile menu button
```

### Desktop layout

```text
Logo | Search Bar | Hotline | Account | Cart
```

Search bar chiếm không gian lớn nhất.

### Mobile layout

Dòng 1:

```text
Menu Button | Logo | Account/Cart
```

Dòng 2:

```text
Search Bar full width
```

### Search bar rules

Search bar phải hỗ trợ:

- nhập từ khóa
- nút search
- placeholder rõ
- autocomplete sau này
- focus state rõ

Placeholder gợi ý:

```text
Tìm laptop, điện thoại, tai nghe...
```

### Header behavior

- Desktop: header có thể sticky khi scroll.
- Mobile: header nên sticky để cart/search dễ truy cập.
- Khi sticky, giảm chiều cao header để không che nội dung.

### Accessibility

- Logo là link về `/`.
- Search input có label ẩn hoặc aria-label.
- Cart button có aria-label.
- Account menu mở được bằng keyboard.

---

## 8.3. Navigation / Mega Menu

### Mục đích

Giúp khách truy cập nhanh vào danh mục chính.

### Danh mục gợi ý

```text
Laptop
Điện thoại
Tablet
Màn hình
Tai nghe
Phụ kiện
Linh kiện PC
Thiết bị mạng
Khuyến mãi
Hỗ trợ
```

### Desktop behavior

- Menu ngang bên dưới header.
- Mục “Danh mục sản phẩm” mở mega menu.
- Mega menu chia theo nhóm sản phẩm.

Ví dụ mega menu:

```text
Laptop
  - Laptop văn phòng
  - Laptop gaming
  - Laptop đồ họa
  - Laptop mỏng nhẹ

Điện thoại
  - iPhone
  - Android
  - Điện thoại gaming

Phụ kiện
  - Chuột
  - Bàn phím
  - Sạc
  - Cáp
```

### Mobile behavior

- Dùng drawer từ trái.
- Menu có thể expand/collapse theo danh mục.
- Có search trong menu nếu danh mục nhiều.

### UI rules

- Active item dùng primary color.
- Hover item có background nhẹ.
- Không dùng quá nhiều animation.

---

## 8.4. Hero Section

### Mục đích

Đẩy chiến dịch bán hàng chính.

Ví dụ:

```text
Back to School Sale
Laptop Gaming giảm đến 20%
iPhone chính hãng trả góp 0%
Phụ kiện mua kèm giảm 50%
```

### Desktop layout

```text
Main Hero Banner: 2/3 width
Side Promo Cards: 1/3 width, stacked 2 cards
```

### Mobile layout

- Chỉ hiển thị main hero banner.
- Side promo cards chuyển xuống dưới dạng carousel hoặc hidden nếu quá chật.

### Hero content

Hero cần có:

```text
Eyebrow text
Headline
Short description
Primary CTA
Secondary CTA optional
Product/brand visual
```

Ví dụ:

```text
SUMMER TECH DEAL
Laptop gaming giảm đến 20%
Hiệu năng cao, bảo hành chính hãng 24 tháng
[Mua ngay] [Xem laptop gaming]
```

### CTA rules

- Primary CTA: nổi bật.
- Secondary CTA: ít nổi hơn.
- CTA phải link tới category, collection hoặc promotion page.

### Image rules

- Ảnh sắc nét.
- Không chứa quá nhiều chữ trong ảnh.
- Text quan trọng phải là HTML text, không nhúng hết vào ảnh.
- Có alt text.

### State

- Nếu không có hero data: dùng fallback hero mặc định.
- Nếu ảnh lỗi: dùng background gradient và text vẫn hiển thị.

---

## 8.5. Quick Category Grid

### Mục đích

Cho khách đi nhanh vào nhóm sản phẩm.

### Category item data

Mỗi item cần:

```text
category_id
name
slug
icon_url hoặc image_url
product_count optional
highlight optional
```

### UI

Category item gồm:

```text
Icon/Image
Category name
Optional product count
```

Ví dụ:

```text
Laptop
Điện thoại
Tai nghe
Màn hình
Bàn phím
Chuột
Ổ cứng
Router
```

### Desktop

- Grid 6-8 item mỗi hàng.
- Card vuông hoặc gần vuông.

### Mobile

- 4 item mỗi hàng.
- Text tối đa 2 dòng.
- Icon phải rõ.

### Rules

- Không dùng ảnh quá chi tiết gây rối.
- Tên danh mục phải ngắn.
- Nếu danh mục nhiều hơn 8, hiển thị “Xem thêm”.

---

## 8.6. Trust Benefits Strip

### Mục đích

Tăng niềm tin trước khi khách mua hàng.

### Benefit item gợi ý

```text
Bảo hành chính hãng
Giao hàng toàn quốc
Trả góp 0%
Đổi trả trong 7 ngày
Hỗ trợ kỹ thuật
Xuất hóa đơn VAT
```

### UI

Mỗi benefit gồm:

```text
Icon
Title
Short description
```

Ví dụ:

```text
Bảo hành chính hãng
Cam kết sản phẩm mới 100%
```

### Desktop

- Hiển thị 4-6 benefits trên một hàng.

### Mobile

- Chuyển thành horizontal scroll hoặc carousel.

### Rules

- Text rất ngắn.
- Icon cùng style.
- Không dùng quá nhiều màu.

---

## 8.7. Flash Sale Section

### Mục đích

Tạo cảm giác khẩn cấp và đẩy doanh số.

### Data cần có

```text
sale_id
sale_name
start_time
end_time
products[]
```

Mỗi product cần:

```text
id
name
slug
image
price
sale_price
discount_percent
stock_available
sold_count
rating
quick_specs
```

### UI structure

```text
Section header
Countdown timer
View all link
Product carousel/grid
```

### Section header example

```text
Flash Sale hôm nay
Kết thúc sau: 02:15:30
[Xem tất cả]
```

### Product card in flash sale

Flash sale card có thể nhấn mạnh:

- giá sale
- phần trăm giảm
- progress đã bán
- thời gian còn lại

### Rules

- Countdown phải dựa trên server time nếu có thể.
- Nếu sale hết hạn: ẩn section hoặc chuyển sang sale khác.
- Nếu sản phẩm hết hàng: hiển thị “Tạm hết hàng”.
- Không fake countdown nếu không có campaign thật.

### Mobile

- Dùng horizontal product carousel.
- Product card compact.
- Countdown không được chiếm quá nhiều height.

---

## 8.8. Featured Category Section

### Mục đích

Hiển thị nhóm sản phẩm quan trọng theo danh mục.

Ví dụ:

```text
Laptop nổi bật
Điện thoại bán chạy
Tai nghe khuyến mãi
Màn hình cho dân văn phòng
```

### Layout

```text
Section title
Category tabs optional
Product grid
View all link
```

### Product grid

Desktop:

```text
4-5 columns
```

Mobile:

```text
2 columns
```

### Product card data

Mỗi card cần:

```text
id
name
slug
image
price
original_price optional
sale_price optional
badge optional
rating optional
review_count optional
quick_specs
stock_status
```

### Rules

- Mỗi section chỉ nên hiển thị 8-10 sản phẩm.
- Không hiển thị quá nhiều section gây dài trang.
- Nếu category không có sản phẩm: ẩn section.

---

## 8.9. Best Seller Section

### Mục đích

Tạo niềm tin bằng sản phẩm đã được nhiều người mua.

### Data source

Có thể lấy theo:

```text
sold_count
order_count
revenue
rating
manual curated list
```

### UI rules

- Có badge “Bán chạy”.
- Có thể hiển thị số lượt bán.
- Product card giống chuẩn toàn site.

### Sorting

Mặc định sort theo:

```text
sold_count desc
rating desc
stock_available desc
```

---

## 8.10. New Arrivals Section

### Mục đích

Hiển thị sản phẩm mới nhập.

### Rules

- Dựa trên `created_at`, `published_at`, hoặc flag `is_new`.
- Badge “Mới” dùng màu nhẹ hơn sale badge.
- Không nên lẫn với flash sale nếu không cần.

---

## 8.11. Buying Guide / Editorial Section

### Mục đích

Giúp khách chưa biết chọn sản phẩm nào.

Ví dụ bài viết:

```text
Cách chọn laptop cho sinh viên
Nên mua laptop gaming RAM bao nhiêu?
So sánh SSD 512GB và 1TB
Top tai nghe chống ồn đáng mua
```

### UI

```text
Section title
3-4 article cards
View all link
```

### Article card

```text
Thumbnail
Title
Short summary
Category tag
Read time optional
```

### Rules

- Tiêu đề phải thực dụng.
- Không dùng bài viết quá chung chung.
- Link tới blog hoặc guide page.

---

## 8.12. Brand Showcase Section

### Mục đích

Cho khách duyệt theo thương hiệu.

### Brand gợi ý

```text
Apple
Dell
HP
Lenovo
Asus
Acer
Samsung
Xiaomi
Sony
Logitech
```

### UI

```text
Brand logo grid
Brand name fallback nếu logo lỗi
```

### Rules

- Logo cùng kích thước vùng hiển thị.
- Không bóp méo logo.
- Nếu brand không có logo, dùng text card.

---

## 8.13. Recently Viewed Section

### Mục đích

Giúp khách quay lại sản phẩm từng xem.

### Data source

- Local storage cho guest.
- User history cho logged-in user.

### Rules

- Chỉ hiển thị nếu có dữ liệu.
- Không cần section này trong MVP đầu tiên.
- Tối đa 10 sản phẩm.

---

## 8.14. Newsletter / Support CTA

### Mục đích

Thu lead hoặc điều hướng khách cần tư vấn.

### Biến thể CTA

```text
Đăng ký nhận khuyến mãi
Cần tư vấn chọn laptop?
Liên hệ hỗ trợ kỹ thuật
Tra cứu bảo hành
```

### UI

```text
Title
Short description
Input email hoặc phone optional
CTA button
```

### Rules

- Không ép khách nhập email nếu chưa cần.
- Với đồ điện tử, CTA tư vấn có thể hiệu quả hơn newsletter.

---

## 8.15. Footer

### Mục đích

Cung cấp thông tin shop, chính sách, liên hệ, pháp lý.

### Footer groups

```text
Về cửa hàng
Hỗ trợ khách hàng
Chính sách
Danh mục nổi bật
Kết nối với chúng tôi
Phương thức thanh toán
```

### Nội dung cần có

```text
Giới thiệu
Liên hệ
Hệ thống cửa hàng
Chính sách bảo hành
Chính sách đổi trả
Chính sách giao hàng
Chính sách bảo mật
Hướng dẫn mua hàng
Tra cứu đơn hàng
Tra cứu bảo hành
```

### Desktop

- Chia 4-5 columns.

### Mobile

- Dùng accordion.
- Chỉ mở 1-2 group mặc định nếu cần.

---

## 9. Component specification

## 9.1. HomeProductCard

### Mục đích

Hiển thị sản phẩm trên trang chủ.

### Props / Data

```ts
interface HomeProductCardData {
  id: string | number
  name: string
  slug: string
  imageUrl: string
  imageAlt: string
  price: number
  originalPrice?: number
  salePrice?: number
  discountPercent?: number
  currency: string
  rating?: number
  reviewCount?: number
  soldCount?: number
  quickSpecs?: string[]
  stockStatus: 'in_stock' | 'low_stock' | 'out_of_stock' | 'preorder'
  badges?: string[]
  isCompared?: boolean
}
```

### Visual hierarchy

Thứ tự hiển thị:

```text
Image
Badge
Product name
Quick specs
Rating/review
Price
Stock status
CTA row
```

### Product name rule

- Desktop: tối đa 2 dòng.
- Mobile: tối đa 2 dòng.
- Nếu quá dài: ellipsis.

### Quick specs rule

- Tối đa 3 dòng ngắn.
- Ví dụ laptop: `i5 / 16GB / SSD 512GB`.
- Ví dụ phone: `128GB / 6.7 inch / 5G`.

### Price rule

- `salePrice` hoặc `price` là giá nổi bật nhất.
- `originalPrice` hiển thị nhỏ hơn và gạch ngang.
- `discountPercent` dùng badge.

### Stock status rule

```text
in_stock: Còn hàng
low_stock: Sắp hết hàng
out_of_stock: Tạm hết hàng
preorder: Đặt trước
```

### CTA rule

Desktop:

```text
[Thêm vào giỏ] [So sánh]
```

Mobile:

```text
[Thêm]
Compare icon optional
```

### State

```text
normal
hover
loading
image_error
out_of_stock
compared
```

---

## 9.2. HomeSectionHeader

### Mục đích

Header dùng chung cho các section.

### Props

```ts
interface HomeSectionHeaderData {
  title: string
  subtitle?: string
  actionText?: string
  actionUrl?: string
  countdown?: {
    endTime: string
  }
}
```

### Rules

- Title ngắn.
- Subtitle không quá 1 dòng trên desktop.
- Mobile subtitle có thể 2 dòng.
- Action link nằm bên phải trên desktop.
- Action link xuống dưới title trên mobile nếu chật.

---

## 9.3. CategoryShortcutCard

### Props

```ts
interface CategoryShortcutCardData {
  id: string | number
  name: string
  slug: string
  iconUrl?: string
  imageUrl?: string
  productCount?: number
  isHighlighted?: boolean
}
```

### Rules

- Name tối đa 2 dòng.
- Icon có fallback.
- Card click tới category page.

---

## 9.4. PromoBannerCard

### Props

```ts
interface PromoBannerCardData {
  id: string | number
  title: string
  subtitle?: string
  imageUrl?: string
  backgroundColorToken?: string
  ctaText: string
  ctaUrl: string
  priority?: number
}
```

### Rules

- Text chính phải nằm trong HTML.
- Không phụ thuộc 100% vào text trong ảnh.
- CTA rõ ràng.

---

## 10. Data contract for home page

Backend hoặc CMS có thể trả về một object tổng cho home page.

Ví dụ:

```json
{
  "announcement": {
    "message": "Miễn phí giao hàng cho đơn từ 2.000.000đ",
    "url": "/promotions/free-shipping"
  },
  "hero": {
    "title": "Laptop Gaming giảm đến 20%",
    "subtitle": "Hiệu năng cao, bảo hành chính hãng",
    "imageUrl": "/images/hero-gaming-laptop.png",
    "primaryCta": {
      "text": "Mua ngay",
      "url": "/c/laptop-gaming"
    },
    "secondaryCta": {
      "text": "Xem khuyến mãi",
      "url": "/promotions"
    }
  },
  "categories": [],
  "flashSale": {
    "name": "Flash Sale hôm nay",
    "startTime": "2026-06-22T00:00:00+07:00",
    "endTime": "2026-06-22T23:59:59+07:00",
    "products": []
  },
  "featuredSections": [
    {
      "title": "Laptop nổi bật",
      "categorySlug": "laptop",
      "products": []
    }
  ],
  "bestSellers": [],
  "newArrivals": [],
  "brands": [],
  "editorials": []
}
```

### Rules

- Frontend không được crash nếu thiếu một section.
- Section nào không có dữ liệu thì ẩn hoặc hiển thị fallback hợp lý.
- Product card phải dùng cùng một data mapper.
- Không map dữ liệu trực tiếp lung tung trong từng component.

---

## 11. Loading, empty, error states

## 11.1. Page loading

Khi trang đang tải dữ liệu:

- Header vẫn hiển thị nếu có thể.
- Main content hiển thị skeleton.
- Product card skeleton phải giống kích thước card thật.
- Không dùng spinner toàn trang quá lâu.

## 11.2. Section loading

Nếu từng section load riêng:

```text
Hero skeleton
Category skeleton
Product card skeleton
```

## 11.3. Empty state

Nếu một section không có dữ liệu:

- Section optional: ẩn.
- Section bắt buộc: hiển thị empty state ngắn.

Ví dụ:

```text
Chưa có sản phẩm nổi bật.
```

## 11.4. Error state

Nếu API home page lỗi:

- Hiển thị message thân thiện.
- Có nút retry.
- Không hiển thị stack trace.

Ví dụ:

```text
Không thể tải trang chủ. Vui lòng thử lại.
[Thử lại]
```

## 11.5. Image error

Nếu ảnh sản phẩm lỗi:

- Dùng placeholder.
- Alt text vẫn có.
- Không để layout bị nhảy.

---

## 12. Responsive acceptance rules

Bắt buộc kiểm tra các viewport:

```text
1440 x 900
1280 x 800
1024 x 768
768 x 1024
390 x 844
375 x 667
360 x 740
```

### Desktop acceptance

- Header không vỡ.
- Search bar đủ rộng.
- Product grid đúng số cột.
- Hero không bị crop text.
- Footer columns không chồng nhau.

### Tablet acceptance

- Product grid 3 cột.
- Hero side cards xuống dưới.
- Navigation không bị tràn.

### Mobile acceptance

- Không overflow ngang.
- Header không quá cao.
- Search bar dễ bấm.
- Product card text không tràn.
- Footer accordion hoạt động.
- CTA không bị che bởi browser chrome.

---

## 13. SEO rules

Trang chủ cần hỗ trợ SEO cơ bản.

### HTML structure

- Chỉ có một `h1`.
- Section title dùng `h2`.
- Product name trong card có thể dùng text/link, không nhất thiết là heading.

### Meta data

Cần có:

```text
Title
Meta description
Canonical URL
Open Graph title
Open Graph description
Open Graph image
```

Ví dụ title:

```text
TechStore - Laptop, điện thoại, phụ kiện chính hãng
```

Ví dụ description:

```text
Mua laptop, điện thoại, tai nghe và phụ kiện chính hãng với giá tốt, bảo hành rõ ràng, giao hàng toàn quốc.
```

### Image SEO

- Product image phải có `alt`.
- Hero image phải có alt nếu truyền tải thông tin.
- Decorative image có thể dùng alt rỗng.

---

## 14. Accessibility rules

Trang chủ phải tuân thủ các rule sau:

- Tất cả button có accessible name.
- Link phải mô tả được mục tiêu.
- Search input có label hoặc aria-label.
- Menu drawer dùng được bằng keyboard.
- Focus state rõ.
- Contrast text đủ đọc.
- Không dùng màu là tín hiệu duy nhất.
- Carousel không auto chạy quá nhanh.
- Người dùng có thể pause carousel nếu auto-play.

---

## 15. Performance rules

Trang chủ thường nặng vì nhiều ảnh. Bắt buộc:

- Lazy load ảnh dưới fold.
- Preload hero image nếu cần.
- Dùng kích thước ảnh phù hợp viewport.
- Không load toàn bộ sản phẩm của nhiều section nếu không cần.
- Product section chỉ load 8-10 sản phẩm.
- Dùng CDN cho ảnh.
- Skeleton phải ổn định kích thước để giảm layout shift.

### Performance budget đề xuất

```text
Hero image <= 300KB sau tối ưu
Product thumbnail <= 80KB
Initial JS <= tùy framework, nhưng phải chia chunk
Home API response <= 200KB cho MVP
```

---

## 16. Storefront home page routes

Route chính:

```text
/
```

Route liên quan:

```text
/search?q=
/c/:categorySlug
/products/:productSlug
/cart
/compare
/promotions
/support
/warranty
```

---

## 17. Events and analytics

Trang chủ nên emit các event phân tích.

### Event list

```text
home_viewed
hero_clicked
search_submitted
category_clicked
product_card_clicked
add_to_cart_clicked
compare_clicked
flash_sale_viewed
promotion_clicked
brand_clicked
```

### Event payload example

```json
{
  "event": "product_card_clicked",
  "page": "home",
  "section": "best_seller",
  "product_id": "123",
  "product_name": "Laptop Dell Inspiron 15",
  "position": 3
}
```

### Rules

- Analytics không được làm chậm UI.
- Nếu analytics fail, UI vẫn hoạt động.
- Không gửi dữ liệu cá nhân nhạy cảm nếu không cần.

---

## 18. Admin configuration for home page

Trang chủ nên được cấu hình từ admin để tái sử dụng cho nhiều cửa hàng.

### Admin có thể cấu hình

```text
Announcement messages
Hero banners
Side promo banners
Quick categories
Flash sale campaign
Featured product sections
Brand showcase
Editorial articles
Support CTA
Footer links
```

### Home section config model

```ts
interface HomeSectionConfig {
  id: string
  type: 'hero' | 'category_grid' | 'product_section' | 'flash_sale' | 'brand_grid' | 'editorial' | 'support_cta'
  title?: string
  subtitle?: string
  isEnabled: boolean
  sortOrder: number
  dataSource: 'manual' | 'category' | 'promotion' | 'best_seller' | 'new_arrival'
  config: Record<string, unknown>
}
```

### Rules

- Admin có thể bật/tắt section.
- Admin có thể đổi thứ tự section.
- Không cho cấu hình làm vỡ layout.
- Nếu config sai, frontend dùng fallback an toàn.

---

## 19. Agent implementation rules

Khi agent implement trang này, bắt buộc:

1. Đọc `ecommerce_design_language.md` trước.
2. Đọc `01-electronics-store-theme.md` trước.
3. Đọc toàn bộ file này trước khi code.
4. Không tự ý đổi design token nếu không có yêu cầu.
5. Không hard-code category theo laptop/phone nếu source cần clone.
6. Dùng component tái sử dụng cho product card, section header, category card.
7. Không map API data trực tiếp trong UI component sâu.
8. Phải có loading state, empty state, error state.
9. Phải kiểm tra mobile 375px.
10. Nếu sửa UI, phải chụp screenshot hoặc chạy visual test nếu project có setup.

### Không được làm

- Không tạo CSS global tùy tiện.
- Không dùng magic number spacing ngoài token.
- Không để text dài làm vỡ card.
- Không để ảnh lỗi làm layout vỡ.
- Không bỏ qua trạng thái hết hàng.
- Không dùng selector test quá fragile.

---

## 20. Suggested file structure

Không phụ thuộc framework, nhưng source nên tách tương tự:

```text
src/
  pages/
    home/
      HomePage.*
      HomePage.mapper.*
      HomePage.types.*
      HomePage.api.*
      HomePage.test.*

  components/
    layout/
      AnnouncementBar.*
      MainHeader.*
      MegaMenu.*
      Footer.*

    home/
      HeroSection.*
      QuickCategoryGrid.*
      TrustBenefitsStrip.*
      FlashSaleSection.*
      FeaturedProductSection.*
      BrandShowcase.*
      EditorialSection.*
      SupportCTA.*

    product/
      HomeProductCard.*
      ProductPrice.*
      ProductBadge.*
      ProductRating.*
      StockStatusBadge.*
```

---

## 21. Playwright test specification

## 21.1. Basic render test

Test name:

```text
home page renders core sections
```

Expected:

- Header visible.
- Search input visible.
- Hero visible.
- Quick categories visible.
- At least one product section visible.
- Footer visible.

## 21.2. Search flow

Test name:

```text
customer can search from home page
```

Steps:

```text
Open home page
Fill search input with "laptop"
Click search button
Expect URL contains search query
Expect product listing or search result page visible
```

## 21.3. Category navigation

Test name:

```text
customer can open category from quick category grid
```

Steps:

```text
Open home page
Click Laptop category
Expect category page URL
Expect category title visible
```

## 21.4. Product card navigation

Test name:

```text
customer can open product detail from home product card
```

Steps:

```text
Open home page
Click first product card name or image
Expect product detail page visible
```

## 21.5. Add to cart from home

Test name:

```text
customer can add in-stock product to cart from home
```

Steps:

```text
Open home page
Find an in-stock product card
Click Add to cart
Expect cart count increased
Expect success toast visible
```

## 21.6. Out of stock rule

Test name:

```text
out of stock product cannot be added to cart
```

Expected:

- Add button disabled or hidden.
- Stock badge visible.

## 21.7. Mobile visual check

Test name:

```text
home page mobile layout has no horizontal overflow
```

Viewport:

```text
375 x 667
```

Expected:

- No horizontal scroll.
- Header visible.
- Search visible.
- Product grid has 2 columns.
- Footer accordion visible.

## 21.8. Visual regression screenshots

Suggested screenshots:

```text
home-desktop-1440.png
home-tablet-768.png
home-mobile-375.png
```

---

## 22. Definition of Done

Trang chủ được coi là hoàn thành khi:

1. Render được đầy đủ core sections.
2. Header, search, category, hero, product section, footer hoạt động.
3. Product card dùng đúng electronics theme.
4. Có loading, empty, error state.
5. Responsive đúng desktop/tablet/mobile.
6. Không overflow ngang ở 375px.
7. Add to cart hoạt động với sản phẩm còn hàng.
8. Out of stock không thể add to cart.
9. Search điều hướng đúng.
10. Category click điều hướng đúng.
11. SEO metadata cơ bản có đủ.
12. Accessibility cơ bản đạt yêu cầu.
13. Playwright tests chính pass.
14. Không hard-code logic làm mất khả năng clone sang ngành khác.

---

## 23. MVP scope

Nếu triển khai MVP, chỉ cần làm trước:

```text
Announcement Bar optional
Main Header
Navigation basic
Hero Section
Quick Category Grid
Trust Benefits Strip
Best Seller Section
New Arrivals Section
Footer
```

Chưa cần làm ngay:

```text
Recently Viewed
Buying Guide
Brand Showcase
Admin dynamic home builder
Advanced analytics
AI recommendation
Personalized content
```

---

## 24. Future extension

Sau MVP có thể mở rộng:

- Dynamic section builder từ admin.
- Personalized home page theo user.
- Recommendation engine.
- A/B testing hero banners.
- Live promotion countdown.
- Store availability by location.
- Compare widget sticky.
- Recently viewed sync between devices.

---

## 25. Summary for agent

Trang chủ storefront đồ điện tử phải là trang nhanh, rõ, tin cậy, dễ mua.

Không được chỉ làm một landing page đẹp. Nó phải là entry point của toàn bộ e-commerce flow.

Agent cần ưu tiên:

```text
Search rõ
Category rõ
Hero rõ
Product card giàu thông tin
Giá nổi bật
Thông số dễ scan
Mobile tốt
Không overflow
Có test
```

