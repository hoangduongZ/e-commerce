# 00 - Ngôn ngữ thiết kế chung cho website bán hàng và trang admin

> Tài liệu này là **Design System gốc** cho một source website bán hàng có thể tái sử dụng cho nhiều ngành hàng khác nhau: thời trang, mỹ phẩm, đồ gia dụng, điện tử, thực phẩm, nội thất, khóa học, sản phẩm số...
>
> Mục tiêu: khi đưa tài liệu này cho coding agent/frontend agent, agent có thể code giao diện thống nhất từ đầu đến cuối mà không phải tự đoán style, layout, trạng thái, responsive, spacing, component behavior.

---

## 1. Mục tiêu của ngôn ngữ thiết kế

Website bán hàng có 2 khu vực lớn:

1. **Storefront**: trang khách hàng nhìn thấy.
2. **Admin**: trang người bán/quản trị dùng để vận hành shop.

Hai khu vực này có mục tiêu khác nhau, nhưng phải dùng chung một hệ thống thiết kế để source dễ bảo trì.

### 1.1. Mục tiêu Storefront

Storefront cần giúp khách hàng:

- Nhìn vào là hiểu shop bán gì.
- Tìm sản phẩm nhanh.
- Xem thông tin sản phẩm rõ ràng.
- Tin tưởng shop.
- Thêm vào giỏ dễ dàng.
- Thanh toán ít bước.
- Không bị rối trên mobile.

Storefront ưu tiên:

```text
Đẹp → Rõ → Tin cậy → Dễ mua → Nhanh
```

### 1.2. Mục tiêu Admin

Admin cần giúp người bán:

- Quản lý sản phẩm nhanh.
- Xem đơn hàng rõ.
- Cập nhật trạng thái chính xác.
- Lọc, tìm kiếm, xử lý dữ liệu nhiều dòng.
- Hạn chế thao tác nhầm.
- Nhìn dashboard biết tình hình kinh doanh.

Admin ưu tiên:

```text
Rõ → Nhanh → Chính xác → An toàn → Có thể xử lý nhiều dữ liệu
```

### 1.3. Một source, nhiều ngành hàng

Design System phải không bị gắn chết vào một ngành cụ thể.

Không hard-code phong cách kiểu:

```text
Chỉ hợp thời trang nữ
Chỉ hợp mỹ phẩm
Chỉ hợp điện tử 
Chỉ hợp luxury
```

Thay vào đó, hệ thống dùng **theme token** để đổi màu, font, ảnh, tone thương hiệu.

Ví dụ:

```text
Fashion theme      → mềm, nhiều ảnh, spacing thoáng
Electronics theme  → rõ thông số, grid chặt, badge kỹ thuật
Beauty theme       → nhẹ, sáng, hình ảnh lớn
Grocery theme      → rõ giá, rõ khuyến mãi, mua nhanh
Luxury theme       → ít màu, typography sang, nhiều khoảng trắng
B2B theme          → bảng dữ liệu, thông số, báo giá
```

---

## 2. Triết lý thiết kế

### 2.1. Rõ trước, đẹp sau

Giao diện bán hàng không chỉ để đẹp. Nó phải trả lời nhanh các câu hỏi của khách:

```text
Sản phẩm này là gì?
Giá bao nhiêu?
Có còn hàng không?
Có ưu đãi không?
Có phù hợp với tôi không?
Mua thế nào?
Shop có đáng tin không?
```

Nếu một UI đẹp nhưng khách không hiểu cách mua, UI đó thất bại.

### 2.2. Dữ liệu thật luôn xấu hơn dữ liệu mock

Khi thiết kế phải giả định dữ liệu sẽ có vấn đề:

- Tên sản phẩm rất dài.
- Ảnh sản phẩm sai tỷ lệ.
- Sản phẩm không có ảnh.
- Giá rất lớn.
- Mô tả rất dài.
- Có sản phẩm hết hàng.
- Có sản phẩm có nhiều biến thể.
- Có sản phẩm không có biến thể.
- Có danh mục nhiều cấp.
- Có đơn hàng rất nhiều sản phẩm.

Vì vậy mọi component phải có rule xử lý dữ liệu xấu.

### 2.3. Mobile là mặc định

Nhiều khách mua hàng bằng điện thoại, nên design phải mobile-first.

Tư duy:

```text
Mobile trước
Desktop sau
```

Desktop có thể rộng hơn, nhiều cột hơn. Nhưng mọi chức năng mua hàng cốt lõi phải hoạt động tốt ở mobile.

### 2.4. Trạng thái UI là một phần của thiết kế

Không chỉ thiết kế màn hình “đẹp lúc có data”. Mỗi màn hình phải có đủ trạng thái:

```text
Default
Loading
Empty
Error
Disabled
Success
Warning
Hover
Focus
Active
Selected
Skeleton
Permission denied
Offline
```

Agent không được tự đoán trạng thái.

### 2.5. Admin phải ít màu hơn Storefront

Storefront có thể dùng màu thương hiệu để tạo cảm xúc mua hàng.

Admin nên dùng màu tiết chế hơn để người vận hành tập trung vào dữ liệu.

Rule:

```text
Storefront: cảm xúc + chuyển đổi
Admin: dữ liệu + thao tác
```

---

## 3. Design tokens

Design token là biến thiết kế dùng chung cho toàn hệ thống. Khi clone source cho ngành khác, ưu tiên đổi token thay vì sửa từng component.

### 3.1. Nhóm token bắt buộc

```text
Color tokens
Typography tokens
Spacing tokens
Radius tokens
Shadow tokens
Border tokens
Breakpoint tokens
Z-index tokens
Motion tokens
Icon tokens
Layout tokens
```

---

## 4. Màu sắc

### 4.1. Nguyên tắc màu

Hệ thống dùng 4 nhóm màu chính:

1. **Brand colors**: màu thương hiệu.
2. **Neutral colors**: màu nền, text, border.
3. **Semantic colors**: màu trạng thái như success, warning, error.
4. **Commerce colors**: màu dùng riêng cho giá, sale, tồn kho, đơn hàng.

Không dùng màu trực tiếp trong component kiểu:

```css
color: #ff0000;
```

Phải dùng token:

```css
color: var(--color-error-600);
```

### 4.2. Brand colors

| Token | Giá trị mặc định | Mục đích |
|---|---:|---|
| brand-50 | #eff6ff | nền nhẹ |
| brand-100 | #dbeafe | hover nhẹ |
| brand-200 | #bfdbfe | border nhẹ |
| brand-300 | #93c5fd | phụ |
| brand-400 | #60a5fa | phụ |
| brand-500 | #3b82f6 | chính |
| brand-600 | #2563eb | primary |
| brand-700 | #1d4ed8 | hover |
| brand-800 | #1e40af | active |
| brand-900 | #1e3a8a | text đậm |

Mặc định dùng xanh dương vì trung tính, dễ áp dụng đa ngành. Khi clone cho ngành khác có thể đổi brand scale.

Ví dụ theme mỹ phẩm:

```text
brand-600 = hồng đất
brand-700 = hồng nâu
```

Ví dụ theme luxury:

```text
brand-600 = đen hoặc vàng đồng
brand-700 = đen đậm
```

### 4.3. Neutral colors

| Token | Giá trị | Mục đích |
|---|---:|---|
| neutral-0 | #ffffff | nền trắng |
| neutral-50 | #f9fafb | nền phụ |
| neutral-100 | #f3f4f6 | nền block |
| neutral-200 | #e5e7eb | border |
| neutral-300 | #d1d5db | border mạnh |
| neutral-400 | #9ca3af | placeholder |
| neutral-500 | #6b7280 | text phụ |
| neutral-600 | #4b5563 | text thường |
| neutral-700 | #374151 | text đậm |
| neutral-800 | #1f2937 | heading |
| neutral-900 | #111827 | text chính |

### 4.4. Semantic colors

| Nhóm | 50 | 600 | 700 |
|---|---:|---:|---:|
| success | #ecfdf5 | #059669 | #047857 |
| warning | #fffbeb | #d97706 | #b45309 |
| error | #fef2f2 | #dc2626 | #b91c1c |
| info | #eff6ff | #2563eb | #1d4ed8 |

### 4.5. Commerce colors

Commerce colors dùng cho các trạng thái bán hàng.

| Token | Giá trị | Dùng cho |
|---|---:|---|
| price-current | error-600 | giá bán |
| price-original | neutral-400 | giá gốc |
| sale-badge-bg | error-600 | badge sale |
| sale-badge-text | neutral-0 | chữ sale |
| stock-in | success-600 | còn hàng |
| stock-low | warning-600 | sắp hết |
| stock-out | neutral-500 | hết hàng |
| order-pending | warning-600 | chờ xử lý |
| order-paid | success-600 | đã thanh toán |
| order-cancelled | error-600 | đã hủy |
| order-shipping | info-600 | đang giao |

### 4.6. Rule dùng màu trong Storefront

Storefront dùng màu để hướng khách tới hành động.

Màu nổi bật chỉ dùng cho:

```text
Nút mua hàng
Giá khuyến mãi
Badge sale
Thông báo quan trọng
CTA chính
```

Không dùng quá nhiều màu cùng lúc trên card sản phẩm.

Một ProductCard chỉ nên có tối đa:

```text
1 màu CTA
1 màu sale
1 màu text chính
1 màu text phụ
```

### 4.7. Rule dùng màu trong Admin

Admin dùng màu để phân loại trạng thái, không dùng để trang trí quá nhiều.

Ví dụ:

```text
Đơn chờ xác nhận       → warning
Đơn đã xác nhận        → info
Đơn đang giao          → brand/info
Đơn hoàn thành         → success
Đơn đã hủy             → error/neutral
Thanh toán thất bại    → error
Tồn kho thấp           → warning
Hết hàng               → error hoặc neutral
```

Không dùng màu đỏ cho text thông thường, vì đỏ trong admin phải đại diện cho rủi ro/lỗi/xóa/hủy.

---

## 5. Typography

### 5.1. Nguyên tắc chữ

Typography phải dễ đọc trên mobile và rõ ràng trong bảng admin.

Không dùng quá nhiều font.

Khuyến nghị:

```text
Font chính: Inter, Roboto, system-ui, sans-serif
Font fallback: Arial, sans-serif
```

Nếu web bán hàng cho thị trường Nhật/Hàn/Trung, cần font hỗ trợ tốt CJK.

Nếu chỉ thị trường Việt Nam, dùng font Latin rõ dấu tiếng Việt.

### 5.2. Font scale

| Token | Size | Line height | Dùng cho |
|---|---:|---:|---|
| text-xs | 12px | 16px | caption |
| text-sm | 14px | 20px | text phụ |
| text-base | 16px | 24px | body |
| text-lg | 18px | 28px | subheading |
| text-xl | 20px | 28px | heading nhỏ |
| text-2xl | 24px | 32px | page title |
| text-3xl | 30px | 38px | hero title |
| text-4xl | 36px | 44px | landing title |

### 5.3. Font weight

| Token | Weight | Dùng cho |
|---|---:|---|
| font-regular | 400 | body |
| font-medium | 500 | label |
| font-semibold | 600 | heading/card title |
| font-bold | 700 | hero/price |

### 5.4. Storefront typography

Storefront cần rõ giá và tên sản phẩm.

Rule:

```text
Tên sản phẩm trong card: text-sm hoặc text-base, font-medium
Tên sản phẩm chi tiết: text-2xl desktop, text-xl mobile
Giá hiện tại: text-lg hoặc text-xl, font-bold
Giá gốc: text-sm, line-through, neutral-400
Mô tả ngắn: text-sm, neutral-600
```

Không để tên sản phẩm trong card vượt quá 2 dòng.

Nếu dài hơn:

```css
line-clamp: 2;
```

### 5.5. Admin typography

Admin ưu tiên mật độ dữ liệu.

Rule:

```text
Page title: text-2xl, font-semibold
Section title: text-lg, font-semibold
Table header: text-xs hoặc text-sm, font-semibold
Table cell: text-sm
Form label: text-sm, font-medium
Helper text: text-xs, neutral-500
Error text: text-xs, error-600
```

Không dùng font quá lớn trong bảng admin.

---

## 6. Spacing

### 6.1. Spacing scale

| Token | Giá trị |
|---|---:|
| space-0 | 0 |
| space-1 | 4px |
| space-2 | 8px |
| space-3 | 12px |
| space-4 | 16px |
| space-5 | 20px |
| space-6 | 24px |
| space-8 | 32px |
| space-10 | 40px |
| space-12 | 48px |
| space-16 | 64px |
| space-20 | 80px |

### 6.2. Rule spacing chung

Dùng spacing theo hệ 4px. Không dùng số lẻ tùy tiện như 13px, 17px, 23px.

Quy tắc dễ nhớ:

```text
Khoảng cách nhỏ trong component      → 4px / 8px
Khoảng cách giữa label và input      → 6px / 8px
Khoảng cách giữa các field           → 16px
Khoảng cách giữa các section         → 24px / 32px
Khoảng cách giữa các block lớn       → 48px / 64px
```

### 6.3. Storefront spacing

Storefront cần thoáng hơn để sản phẩm dễ nhìn.

```text
Page horizontal padding mobile: 16px
Page horizontal padding tablet: 24px
Page horizontal padding desktop: 32px hoặc max-width container
Product grid gap mobile: 12px
Product grid gap desktop: 24px
Section spacing mobile: 32px
Section spacing desktop: 48px hoặc 64px
```

### 6.4. Admin spacing

Admin cần gọn hơn.

```text
Page padding desktop: 24px
Page padding mobile: 16px
Card padding: 16px hoặc 24px
Table cell padding: 12px 16px
Form field gap: 16px
Toolbar gap: 12px
```

---

## 7. Border radius

### 7.1. Radius scale

| Token | Giá trị | Dùng cho |
|---|---:|---|
| radius-none | 0 | bảng/cạnh sắc |
| radius-sm | 4px | badge nhỏ |
| radius-md | 8px | input/button |
| radius-lg | 12px | card |
| radius-xl | 16px | modal/card lớn |
| radius-2xl | 24px | hero/card nổi bật |
| radius-full | 9999px | pill/avatar |

### 7.2. Rule radius

Storefront có thể dùng radius mềm hơn.

Admin nên dùng radius vừa phải.

```text
Button: radius-md
Input: radius-md
Product card: radius-lg
Admin table card: radius-lg
Modal: radius-xl
Badge: radius-full hoặc radius-sm
```

Không dùng nhiều loại radius khác nhau trong cùng một màn hình.

---

## 8. Shadow và elevation

### 8.1. Shadow scale

| Token | Giá trị |
|---|---|
| shadow-none | none |
| shadow-sm | nhẹ |
| shadow-md | vừa |
| shadow-lg | nổi |
| shadow-xl | modal |

### 8.2. Rule shadow

Không dùng shadow quá đậm.

Storefront:

```text
Product card hover: shadow-sm hoặc shadow-md
Dropdown: shadow-lg
Modal: shadow-xl
Sticky bar: shadow-md
```

Admin:

```text
Card thường: border thay vì shadow
Dropdown: shadow-lg
Modal: shadow-xl
Sidebar: border-right, không cần shadow mạnh
```

---

## 9. Border

### 9.1. Border tokens

```text
border-color-default = neutral-200
border-color-strong = neutral-300
border-color-focus = brand-500
border-color-error = error-600
border-width-default = 1px
```

### 9.2. Rule border

- Input bình thường dùng `neutral-300`.
- Input focus dùng `brand-500`.
- Input lỗi dùng `error-600`.
- Card admin ưu tiên border nhẹ thay vì shadow.
- Product card có thể không border nếu ảnh và spacing đã rõ.

---

## 10. Layout system

### 10.1. Breakpoints

| Token | Width |
|---|---:|
| xs | 0px |
| sm | 640px |
| md | 768px |
| lg | 1024px |
| xl | 1280px |
| 2xl | 1536px |

### 10.2. Container

Storefront dùng container để nội dung không bị quá rộng.

```text
Mobile: width 100%, padding 16px
Tablet: padding 24px
Desktop: max-width 1200px hoặc 1280px
Large desktop: max-width 1440px nếu cần grid lớn
```

Admin thường dùng full-width layout, nhưng content vẫn cần giới hạn spacing.

```text
Admin main content: full width
Page padding: 24px
Card max-width: tùy loại form
Form detail max-width: 960px
Dashboard: responsive grid
```

### 10.3. Grid Storefront

Product grid:

```text
Mobile: 2 cột
Small mobile rất hẹp: 1 hoặc 2 cột tùy sản phẩm
Tablet: 3 cột
Desktop: 4 cột
Large desktop: 5 cột nếu sản phẩm card nhỏ
```

Rule:

- Sản phẩm cần ảnh đẹp: ít cột hơn, ảnh lớn hơn.
- Sản phẩm cần so sánh thông số: nhiều cột vừa phải, card rõ specs.
- Không để card quá hẹp làm giá và tên bị vỡ.

### 10.4. Grid Admin

Admin dashboard:

```text
Mobile: 1 cột
Tablet: 2 cột
Desktop: 4 cột cho stat cards
```

Admin form:

```text
Mobile: 1 cột
Desktop: 2 cột nếu field ngắn
Field dài như mô tả, địa chỉ: full width
```

Admin table:

```text
Desktop: table đầy đủ
Tablet: ẩn bớt column phụ
Mobile: card list hoặc horizontal scroll có kiểm soát
```

---

## 11. Z-index

### 11.1. Z-index scale

| Token | Giá trị | Dùng cho |
|---|---:|---|
| z-base | 0 | nền |
| z-sticky | 100 | header sticky |
| z-dropdown | 200 | menu/dropdown |
| z-overlay | 300 | overlay |
| z-drawer | 400 | drawer |
| z-modal | 500 | modal |
| z-toast | 600 | toast |
| z-tooltip | 700 | tooltip |

### 11.2. Rule z-index

Không tự ý đặt z-index rất lớn như `999999`.

Nếu element bị che, phải kiểm tra stacking context thay vì tăng z-index bừa.

---

## 12. Motion

### 12.1. Motion tokens

| Token | Giá trị |
|---|---:|
| duration-fast | 120ms |
| duration-normal | 200ms |
| duration-slow | 300ms |
| easing-standard | ease-out |
| easing-emphasized | cubic-bezier(0.2, 0, 0, 1) |

### 12.2. Rule motion

Motion chỉ dùng để giúp người dùng hiểu trạng thái, không dùng để trang trí quá mức.

Dùng motion cho:

```text
Hover product card
Dropdown open/close
Drawer slide
Modal fade/scale
Toast appear
Skeleton loading
Button loading
```

Không dùng motion dài trong admin vì làm chậm thao tác.

Tôn trọng `prefers-reduced-motion`.

---

## 13. Icon system

### 13.1. Style icon

Icon nên thống nhất một style:

```text
Outline hoặc filled, không trộn bừa
Stroke 1.5px hoặc 2px
Kích thước chuẩn: 16px, 20px, 24px
```

### 13.2. Icon size

| Token | Size | Dùng cho |
|---|---:|---|
| icon-xs | 12px | badge nhỏ |
| icon-sm | 16px | input/helper |
| icon-md | 20px | button/menu |
| icon-lg | 24px | nav/action |
| icon-xl | 32px | empty state |

### 13.3. Icon rule

- Icon trong button phải có khoảng cách 8px với text.
- Icon không được thay thế text nếu action quan trọng.
- Action nguy hiểm như xóa cần icon + confirm, không chỉ icon thùng rác.

---

## 14. Image system

### 14.1. Nguyên tắc ảnh sản phẩm

Ảnh sản phẩm là yếu tố bán hàng rất quan trọng.

Rule:

```text
Ảnh chính nên tỷ lệ 1:1 hoặc 4:5
Không làm méo ảnh
Dùng object-fit: cover hoặc contain tùy ngành hàng
Có placeholder khi ảnh lỗi
Có alt text
Lazy load ảnh ngoài viewport
```

### 14.2. Tỷ lệ ảnh theo ngành

| Ngành | Tỷ lệ nên dùng |
|---|---:|
| Thời trang | 4:5 |
| Mỹ phẩm | 1:1 |
| Điện tử | 1:1 hoặc 4:3 |
| Nội thất | 4:3 |
| Thực phẩm | 1:1 |
| Luxury | 4:5 hoặc 16:9 |

### 14.3. Storefront image rules

Product card:

```text
Ảnh chiếm phần lớn card
Không để text đè lên ảnh trừ badge nhỏ
Badge sale đặt góc trên trái hoặc trên phải
Wishlist icon đặt góc đối diện badge nếu có
```

Product detail:

```text
Gallery ảnh rõ
Thumbnail có trạng thái selected
Zoom ảnh nếu cần
Mobile dùng carousel/swipe
```

### 14.4. Admin image rules

Admin không cần ảnh quá lớn trong list.

```text
Product table image: 48px hoặc 64px
Product edit image preview: 120px đến 160px
Upload area: rõ kéo thả, rõ file type, rõ dung lượng tối đa
```

---

## 15. Component foundation

Các component foundation dùng chung cho Storefront và Admin.

---

## 16. Button

### 16.1. Button variants

| Variant | Mục đích |
|---|---|
| primary | CTA chính |
| secondary | hành động phụ |
| outline | phụ ít nổi |
| ghost | nav/action nhẹ |
| danger | xóa/hủy |
| link | giống link |

### 16.2. Button sizes

| Size | Height | Padding |
|---|---:|---:|
| sm | 32px | 12px |
| md | 40px | 16px |
| lg | 48px | 20px |
| xl | 56px | 24px |

### 16.3. Button states

Mỗi button phải có đủ state:

```text
Default
Hover
Active
Focus visible
Disabled
Loading
```

### 16.4. Button rule Storefront

CTA mua hàng dùng primary.

Ví dụ:

```text
Thêm vào giỏ
Mua ngay
Thanh toán
Đặt hàng
```

Rule:

- Trên mobile, CTA chính trong checkout nên full-width.
- Product detail mobile có thể dùng sticky CTA bar ở cuối màn hình.
- Không có 2 primary button cạnh nhau nếu cùng mức ưu tiên.

### 16.5. Button rule Admin

Admin dùng primary cho hành động chính của trang.

Ví dụ:

```text
Tạo sản phẩm
Lưu thay đổi
Xác nhận đơn
Cập nhật trạng thái
```

Action nguy hiểm dùng danger:

```text
Xóa sản phẩm
Hủy đơn
Khóa tài khoản
```

Action nguy hiểm phải có confirm modal.

---

## 17. Input, Select, Textarea

### 17.1. Input anatomy

Một field chuẩn gồm:

```text
Label
Required mark nếu bắt buộc
Input control
Helper text nếu cần
Error message nếu lỗi
```

### 17.2. Input states

```text
Default
Hover
Focus
Filled
Disabled
Read-only
Error
Success
Loading
```

### 17.3. Field height

| Size | Height |
|---|---:|
| sm | 32px |
| md | 40px |
| lg | 48px |

### 17.4. Rule validation

- Không chỉ đổi border đỏ. Phải có error text.
- Error text đặt dưới input.
- Nếu form dài, khi submit lỗi phải scroll tới field lỗi đầu tiên.
- Field bắt buộc có dấu `*` hoặc text rõ.
- Placeholder không thay thế label.

### 17.5. Storefront form

Checkout form cần rõ ràng, ít field.

Field tối thiểu:

```text
Họ tên
Số điện thoại
Tỉnh/thành
Quận/huyện
Phường/xã
Địa chỉ chi tiết
Ghi chú đơn hàng
Phương thức thanh toán
```

Mobile:

```text
1 field mỗi dòng
Button đặt hàng full width
Tóm tắt đơn hàng có thể collapsible
```

### 17.6. Admin form

Admin form có nhiều field hơn, cần group theo section.

Product form nên chia:

```text
Thông tin cơ bản
Giá bán
Tồn kho
Biến thể
Ảnh sản phẩm
SEO
Trạng thái hiển thị
```

Không đặt toàn bộ field thành một cột rất dài nếu có thể chia section.

---

## 18. Checkbox, Radio, Switch

### 18.1. Checkbox

Dùng cho lựa chọn nhiều mục.

Ví dụ:

```text
Lọc nhiều thương hiệu
Chọn nhiều quyền admin
Chọn nhiều sản phẩm áp dụng coupon
```

### 18.2. Radio

Dùng khi chỉ chọn một.

Ví dụ:

```text
Phương thức thanh toán
Phương thức vận chuyển
Trạng thái hiển thị
```

### 18.3. Switch

Dùng cho bật/tắt trạng thái.

Ví dụ:

```text
Hiển thị sản phẩm
Cho phép bán khi hết hàng
Kích hoạt coupon
```

Không dùng switch cho hành động gây hậu quả lớn nếu không có confirm.

---

## 19. Badge và Tag

### 19.1. Badge type

```text
Sale badge
Stock badge
Order status badge
Payment status badge
User role badge
Category tag
Attribute tag
```

### 19.2. Badge size

| Size | Height |
|---|---:|
| sm | 20px |
| md | 24px |
| lg | 28px |

### 19.3. Badge rule Storefront

Product card có thể dùng:

```text
-20%
Sale
New
Best seller
Hết hàng
```

Không hiển thị quá 2 badge trên một product card.

### 19.4. Badge rule Admin

Admin status badge phải nhất quán.

Order status:

```text
pending       → Chờ xác nhận
confirmed     → Đã xác nhận
packing       → Đang đóng gói
shipping      → Đang giao
delivered     → Đã giao
completed     → Hoàn thành
cancelled     → Đã hủy
returned      → Hoàn hàng
```

Payment status:

```text
unpaid        → Chưa thanh toán
paid          → Đã thanh toán
failed        → Thất bại
refunded      → Đã hoàn tiền
cod_pending   → COD chờ thu
```

---

## 20. Card

### 20.1. Card anatomy

```text
Container
Header optional
Media optional
Body
Footer optional
Actions optional
```

### 20.2. ProductCard

ProductCard là component cực kỳ quan trọng.

Bắt buộc có:

```text
Product image
Product name
Current price
Original price nếu có
Sale badge nếu có
Stock state nếu hết hàng
CTA hoặc quick action
```

Tùy chọn:

```text
Rating
Sold count
Wishlist button
Brand
Short specs
Variant color preview
```

### 20.3. ProductCard states

```text
normal
hover
loading
image_error
out_of_stock
sale
selected
```

### 20.4. ProductCard rule

- Ảnh không được méo.
- Tên sản phẩm tối đa 2 dòng.
- Giá luôn dễ thấy hơn mô tả.
- Nếu hết hàng, CTA disabled và có badge.
- Nếu sale, giá sale nổi bật và giá gốc gạch ngang.
- Không để card thay đổi chiều cao quá nhiều giữa các sản phẩm cùng grid.

### 20.5. Admin cards

Admin card dùng cho:

```text
Dashboard stats
Order summary
Customer summary
Product edit section
Report block
```

Admin card nên có border nhẹ, padding rõ, ít shadow.

---

## 21. Table

### 21.1. Table dùng cho Admin

Admin cần table chuẩn để quản lý dữ liệu.

Table gồm:

```text
Toolbar
Search
Filters
Column headers
Rows
Row actions
Pagination
Bulk actions
Empty state
Loading state
```

### 21.2. Table density

| Density | Dùng cho |
|---|---|
| comfortable | dashboard/list thường |
| compact | dữ liệu nhiều |
| spacious | list ít dòng |

### 21.3. Table states

```text
loading
empty
error
filtered_empty
selected_rows
bulk_action_active
```

### 21.4. Table rule

- Header cố định nếu bảng dài.
- Action chính của row đặt cuối dòng.
- Không để quá nhiều icon action không có label.
- Với action nguy hiểm, cần confirm.
- Cột tiền căn phải.
- Cột ngày dùng format nhất quán.
- Cột status dùng badge.
- Mobile cần có strategy: ẩn cột phụ hoặc chuyển sang card list.

### 21.5. Product admin table

Cột nên có:

```text
Ảnh
Tên sản phẩm
SKU
Danh mục
Giá
Tồn kho
Trạng thái
Ngày cập nhật
Hành động
```

### 21.6. Order admin table

Cột nên có:

```text
Mã đơn
Khách hàng
Số điện thoại
Tổng tiền
Thanh toán
Trạng thái đơn
Ngày đặt
Hành động
```

---

## 22. Modal

### 22.1. Modal dùng khi nào

Dùng modal cho:

```text
Confirm xóa
Confirm hủy đơn
Quick view sản phẩm
Chỉnh sửa nhanh
Thông báo quan trọng
```

Không dùng modal cho form quá dài. Form dài nên dùng page riêng hoặc drawer.

### 22.2. Modal anatomy

```text
Overlay
Dialog container
Title
Description optional
Body
Footer actions
Close button
```

### 22.3. Modal rule

- ESC đóng modal nếu không có dữ liệu chưa lưu.
- Click overlay đóng modal nếu không nguy hiểm.
- Confirm nguy hiểm không đóng khi click overlay nếu dễ gây nhầm.
- Focus trap trong modal.
- Sau khi đóng, focus quay về element mở modal.

---

## 23. Drawer

### 23.1. Drawer dùng khi nào

Drawer dùng tốt cho:

```text
Mobile filter
Cart mini drawer
Admin detail preview
Quick edit
Notification panel
```

### 23.2. Drawer placement

```text
Storefront mobile filter: bottom hoặc left
Mini cart: right
Admin detail: right
Mobile menu: left
```

### 23.3. Drawer rule

- Drawer mobile không quá cao nếu dạng bottom.
- Có close button rõ.
- Overlay đủ tối nhưng không che mất context hoàn toàn.
- Action chính đặt cuối drawer nếu cần.

---

## 24. Toast và Alert

### 24.1. Toast dùng cho feedback ngắn

Ví dụ:

```text
Đã thêm vào giỏ hàng
Đã lưu sản phẩm
Cập nhật trạng thái đơn thành công
Không thể kết nối server
```

### 24.2. Alert dùng cho thông tin cần người dùng đọc

Ví dụ:

```text
Sản phẩm này hiện đã hết hàng
Đơn hàng chưa được thanh toán
Bạn có thay đổi chưa lưu
```

### 24.3. Rule

- Toast tự đóng sau 3-5 giây.
- Error nghiêm trọng không nên chỉ dùng toast. Cần hiển thị tại vị trí liên quan.
- Toast success không quá dài.
- Admin action quan trọng nên có toast + cập nhật UI ngay.

---

## 25. Loading, Empty, Error

### 25.1. Loading

Dùng skeleton cho UI có cấu trúc.

Ví dụ:

```text
ProductCard skeleton
ProductDetail skeleton
Table skeleton
OrderDetail skeleton
```

Dùng spinner cho action ngắn.

Ví dụ:

```text
Button đang lưu
Button đang đặt hàng
```

### 25.2. Empty state

Empty state phải nói rõ:

```text
Hiện tại chưa có gì
Vì sao chưa có
Người dùng nên làm gì tiếp theo
```

Ví dụ:

```text
Giỏ hàng của bạn đang trống
Hãy thêm sản phẩm để bắt đầu mua hàng
[Tiếp tục mua sắm]
```

Admin empty state:

```text
Chưa có sản phẩm nào
Tạo sản phẩm đầu tiên để bắt đầu bán hàng
[Tạo sản phẩm]
```

### 25.3. Error state

Error state phải có:

```text
Thông báo dễ hiểu
Hành động retry nếu có
Không lộ lỗi kỹ thuật cho khách
Có log kỹ thuật cho dev/admin nếu cần
```

Không hiển thị cho khách:

```text
Cannot read properties of undefined
SQLSTATE 23000
500 Internal Server Error raw
```

Nên hiển thị:

```text
Không thể tải sản phẩm. Vui lòng thử lại.
```

---

## 26. Navigation

### 26.1. Storefront header

Header desktop nên có:

```text
Logo
Menu danh mục
Search bar
Account link
Wishlist optional
Cart icon
Hotline optional
```

Header mobile nên có:

```text
Menu icon
Logo
Cart icon
Search bar hoặc search icon
```

### 26.2. Storefront nav rule

- Header có thể sticky, nhưng không được chiếm quá nhiều chiều cao mobile.
- Search phải dễ thấy nếu catalog lớn.
- Cart icon hiển thị số lượng item.
- Menu danh mục hỗ trợ nhiều cấp nếu cần.

### 26.3. Admin sidebar

Admin sidebar nên có:

```text
Dashboard
Đơn hàng
Sản phẩm
Danh mục
Tồn kho
Khách hàng
Khuyến mãi
Báo cáo
Cài đặt
```

### 26.4. Admin nav rule

- Sidebar desktop có thể collapse.
- Mobile dùng drawer.
- Active menu phải rõ.
- Không để menu admin quá sâu quá 2-3 cấp.

---

## 27. Search và Filter

### 27.1. Storefront search

Search cần hỗ trợ:

```text
Nhập từ khóa
Gợi ý sản phẩm
Gợi ý danh mục
Lịch sử tìm kiếm optional
Không có kết quả
```

### 27.2. Storefront filter

Filter phổ biến:

```text
Danh mục
Khoảng giá
Thương hiệu
Màu sắc
Kích thước
Đánh giá
Tình trạng còn hàng
Thuộc tính riêng theo ngành
```

Mobile filter dùng drawer.

Desktop filter có thể ở sidebar trái.

### 27.3. Admin search/filter

Admin filter cần mạnh hơn:

Order filter:

```text
Mã đơn
Tên khách
Số điện thoại
Trạng thái đơn
Trạng thái thanh toán
Ngày đặt
Khoảng tiền
```

Product filter:

```text
Tên/SKU
Danh mục
Trạng thái
Tồn kho
Giá
Ngày cập nhật
```

Rule:

- Có nút reset filter.
- Filter đang áp dụng phải nhìn thấy.
- Search nên debounce.
- Filter phức tạp có thể dùng advanced filter drawer.

---

## 28. Breadcrumb

Breadcrumb giúp người dùng biết vị trí.

Storefront dùng ở:

```text
Category page
Product detail
Blog/article optional
```

Admin dùng ở:

```text
Product edit
Order detail
Customer detail
Settings subpage
```

Rule:

```text
Trang chủ / Danh mục / Tên sản phẩm
Admin / Sản phẩm / Chỉnh sửa
```

Mobile có thể rút gọn.

---

## 29. Pagination

### 29.1. Storefront

Có thể dùng:

```text
Pagination truyền thống
Load more
Infinite scroll
```

Khuyến nghị:

- Web bán hàng SEO tốt hơn với pagination truyền thống hoặc load more có URL rõ.
- Infinite scroll cần cẩn thận vì khách khó quay lại vị trí.

### 29.2. Admin

Admin nên dùng pagination truyền thống.

Cần có:

```text
Page size
Current page
Total items
Next/prev
```

---

## 30. Storefront page language

Phần này định nghĩa ngôn ngữ thiết kế cho các trang khách hàng.

---

## 31. Home Page

### 31.1. Mục tiêu

Trang chủ cần trả lời:

```text
Shop này bán gì?
Có gì nổi bật?
Tôi nên bắt đầu từ đâu?
Có ưu đãi gì?
Shop có đáng tin không?
```

### 31.2. Section chuẩn

```text
Header
Hero banner
Category shortcuts
Featured products
Promotion section
Best sellers
New arrivals
Trust section
Review/testimonial optional
Blog/content optional
Footer
```

### 31.3. Hero banner

Hero nên có:

```text
Headline ngắn
Subheadline
CTA chính
Ảnh hoặc background
```

Rule:

- CTA chính dẫn tới danh mục hoặc sản phẩm ưu tiên.
- Không để text quá dài trên hero.
- Mobile phải đảm bảo text đọc được.

### 31.4. Trust section

Trust section có thể gồm:

```text
Giao hàng nhanh
Đổi trả dễ dàng
Thanh toán an toàn
Hỗ trợ khách hàng
Sản phẩm chính hãng
```

### 31.5. Home responsive

Mobile:

```text
Hero nhỏ hơn
Category dạng horizontal scroll
Product grid 2 cột
Section spacing gọn hơn
```

Desktop:

```text
Hero rộng
Category grid
Product grid 4-5 cột
```

---

## 32. Product Listing Page

### 32.1. Mục tiêu

Giúp khách tìm sản phẩm phù hợp nhanh.

### 32.2. Layout desktop

```text
Header
Breadcrumb
Page title
Filter sidebar left
Product grid right
Sort dropdown
Pagination
Footer
```

### 32.3. Layout mobile

```text
Header
Search/filter bar
Filter drawer
Sort sheet
Product grid 2 cột
Load more hoặc pagination
```

### 32.4. Required states

```text
Loading products
No products found
Filter applied
API error
Products loaded
```

### 32.5. Acceptance criteria

- Mobile 375px không overflow ngang.
- Product card đều chiều cao hợp lý.
- Filter áp dụng xong cập nhật URL query.
- Có thể reset filter.
- Sort hoạt động.
- Empty state có CTA xóa filter.

---

## 33. Product Detail Page

### 33.1. Mục tiêu

Giúp khách hiểu sản phẩm và quyết định mua.

### 33.2. Layout desktop

```text
Breadcrumb
Left: Product gallery
Right: Product info
Below: Description / Specs / Reviews
Related products
```

### 33.3. Layout mobile

```text
Gallery top
Product info below
Variant selector
Quantity selector
Sticky CTA bottom
Description accordion
Reviews
Related products
```

### 33.4. Required components

```text
ProductGallery
ProductInfo
PriceDisplay
SaleBadge
StockStatus
VariantSelector
QuantitySelector
AddToCartButton
BuyNowButton
ProductDescription
ProductSpecs
ReviewSummary
RelatedProducts
```

### 33.5. Required states

```text
Loading
Product not found
Image error
Out of stock
Variant required
Variant unavailable
Add to cart loading
Add to cart success
Add to cart error
```

### 33.6. Product detail rules

- Nếu có sale price, giá sale lớn hơn giá gốc.
- Nếu hết hàng, disable CTA.
- Nếu sản phẩm có biến thể bắt buộc, không cho thêm vào giỏ khi chưa chọn.
- Quantity không được nhỏ hơn 1.
- Quantity không được vượt tồn kho nếu hệ thống chặn tồn kho.
- Product description dài cần có format dễ đọc.

---

## 34. Cart Page

### 34.1. Mục tiêu

Giúp khách kiểm tra sản phẩm trước checkout.

### 34.2. Layout desktop

```text
Cart items left
Order summary right
Coupon input
Checkout CTA
```

### 34.3. Layout mobile

```text
Cart items full width
Order summary below
Checkout CTA sticky bottom optional
```

### 34.4. Cart item

Cart item có:

```text
Ảnh
Tên sản phẩm
Biến thể
Giá
Quantity control
Tổng dòng
Remove action
```

### 34.5. Required states

```text
Cart loading
Cart empty
Item updating
Item remove confirm optional
Coupon applied
Coupon invalid
Stock changed
Price changed
```

### 34.6. Cart rules

- Nếu giỏ rỗng, có CTA quay lại mua sắm.
- Nếu sản phẩm hết hàng sau khi thêm vào giỏ, hiển thị cảnh báo.
- Nếu giá thay đổi, hiển thị thông báo.
- Không cho checkout nếu có item không hợp lệ.

---

## 35. Checkout Page

### 35.1. Mục tiêu

Giúp khách đặt hàng nhanh, ít lỗi.

### 35.2. Layout desktop

```text
Left: Customer/shipping/payment form
Right: Order summary sticky
```

### 35.3. Layout mobile

```text
Form full width
Order summary collapsible
Place order button full width
```

### 35.4. Checkout sections

```text
Thông tin người nhận
Địa chỉ giao hàng
Phương thức vận chuyển
Phương thức thanh toán
Ghi chú
Tóm tắt đơn hàng
```

### 35.5. Required states

```text
Form validation error
Address loading
Shipping fee calculating
Payment method selected
Place order loading
Place order success
Place order error
Cart invalid
```

### 35.6. Checkout rules

- Không yêu cầu đăng ký tài khoản bắt buộc ở MVP.
- Số điện thoại phải validate rõ.
- Địa chỉ phải đủ thông tin để giao hàng.
- Button đặt hàng disabled khi đang submit.
- Nếu submit lỗi, scroll tới lỗi đầu tiên.
- Không để khách bấm đặt hàng 2 lần.

---

## 36. Order Success Page

### 36.1. Mục tiêu

Xác nhận khách đã đặt hàng và hướng dẫn bước tiếp theo.

### 36.2. Nội dung

```text
Icon success
Mã đơn hàng
Tổng tiền
Phương thức thanh toán
Thông tin nhận hàng
Trạng thái đơn
CTA xem đơn hàng
CTA tiếp tục mua sắm
Hướng dẫn chuyển khoản nếu có
```

### 36.3. Rules

- Mã đơn phải nổi bật.
- Nếu thanh toán chuyển khoản, hiển thị nội dung chuyển khoản rõ.
- Không để khách nghi ngờ đơn có tạo thành công hay không.

---

## 37. Customer Account Pages

### 37.1. Pages

```text
Login
Register
Forgot password
Profile
Address book
Order history
Order detail
Wishlist optional
```

### 37.2. Rules

- Form đăng nhập ngắn.
- Error login không tiết lộ email tồn tại hay không nếu cần bảo mật.
- Order history có filter trạng thái.
- Order detail có timeline.

---

## 38. Admin design language

Admin là khu vực vận hành. Phải tối ưu cho xử lý dữ liệu và giảm lỗi thao tác.

---

## 39. Admin Dashboard

### 39.1. Mục tiêu

Cho người bán nhìn nhanh tình hình shop.

### 39.2. Components

```text
Revenue stat card
Order count stat card
Pending orders stat card
Low stock stat card
Sales chart
Recent orders table
Top products list
Inventory alerts
```

### 39.3. Rules

- Số liệu chính rõ, không trang trí quá nhiều.
- Có thời gian lọc: hôm nay, 7 ngày, 30 ngày, custom.
- Nếu dữ liệu chưa có, hiển thị empty state thân thiện.
- Chart không dùng quá nhiều màu.

---

## 40. Admin Product Management

### 40.1. Product List

Cần có:

```text
Search by name/SKU
Filter by category/status/stock
Product table
Bulk actions
Create product button
Import/export optional
```

### 40.2. Product Create/Edit

Form chia section:

```text
Thông tin cơ bản
Ảnh sản phẩm
Danh mục
Giá
Tồn kho
Biến thể
Thuộc tính
SEO
Trạng thái
```

### 40.3. Product form rules

- Tên sản phẩm bắt buộc.
- SKU nên unique.
- Giá không được âm.
- Tồn kho không được âm.
- Ảnh chính phải chọn được.
- Biến thể phải kiểm tra trùng SKU.
- Trước khi rời trang khi có thay đổi chưa lưu, cần confirm.

---

## 41. Admin Order Management

### 41.1. Order list

Cần có:

```text
Search mã đơn/khách/sđt
Filter trạng thái đơn
Filter thanh toán
Filter ngày đặt
Table đơn hàng
Bulk export optional
```

### 41.2. Order detail

Cần có:

```text
Thông tin đơn
Thông tin khách
Địa chỉ giao hàng
Danh sách sản phẩm
Tổng tiền
Trạng thái thanh toán
Trạng thái vận chuyển
Timeline đơn hàng
Ghi chú nội bộ
Actions
```

### 41.3. Order action rules

Action có thể gồm:

```text
Xác nhận đơn
Đóng gói
Đang giao
Hoàn thành
Hủy đơn
Hoàn tiền
In đơn
```

Không phải trạng thái nào cũng chuyển được sang mọi trạng thái.

Ví dụ:

```text
pending → confirmed
confirmed → packing
packing → shipping
shipping → delivered
pending → cancelled
confirmed → cancelled
completed không nên quay lại pending
```

Action nguy hiểm như hủy đơn hoặc hoàn tiền phải có confirm modal.

---

## 42. Admin Customer Management

### 42.1. Customer list

Cần có:

```text
Tên khách
Email
Số điện thoại
Số đơn
Tổng chi tiêu
Ngày đăng ký
Trạng thái
```

### 42.2. Customer detail

Cần có:

```text
Profile
Address list
Order history
Customer notes
Support history optional
```

### 42.3. Rules

- Không hiển thị quá nhiều thông tin nhạy cảm nếu admin không có quyền.
- Có phân quyền xem/sửa khách hàng.

---

## 43. Admin Promotion Management

### 43.1. Coupon form

Cần có:

```text
Mã coupon
Loại giảm giá
Giá trị giảm
Điều kiện đơn tối thiểu
Giảm tối đa
Thời gian bắt đầu/kết thúc
Giới hạn lượt dùng
Áp dụng cho sản phẩm/danh mục
Trạng thái
```

### 43.2. Rules

- Coupon hết hạn phải có badge rõ.
- Coupon chưa bắt đầu không được áp dụng.
- Coupon disabled không được áp dụng.
- Nếu percentage discount, cần max discount để tránh lỗ nếu nghiệp vụ yêu cầu.

---

## 44. Admin Settings

Settings có thể gồm:

```text
Thông tin shop
Logo/Favicon
Theme color
Phương thức thanh toán
Vận chuyển
Thuế
Email templates
Quyền admin
SEO
Chính sách
```

Settings cần chia nhóm, không để một form quá dài.

---

## 45. Accessibility

### 45.1. Nguyên tắc

Giao diện phải dùng được bằng bàn phím, screen reader, và có contrast đủ.

### 45.2. Rule bắt buộc

```text
Button phải focus được
Input phải có label
Image quan trọng phải có alt
Icon-only button phải có aria-label
Modal phải trap focus
Dropdown phải điều hướng được bằng keyboard nếu custom
Text và background phải đủ contrast
Không chỉ dùng màu để truyền đạt trạng thái
```

### 45.3. Focus visible

Focus state phải rõ.

```text
focus ring: 2px brand-500
focus offset: 2px
```

Không xóa outline nếu không thay thế bằng focus style khác.

---

## 46. Responsive rules

### 46.1. Viewport bắt buộc test

```text
Mobile small: 320px
Mobile common: 375px
Mobile large: 430px
Tablet: 768px
Laptop: 1024px
Desktop: 1440px
Large desktop: 1536px
```

### 46.2. Storefront responsive checklist

```text
Header không vỡ
Search dùng được
Product grid không overflow
Product card không quá hẹp
Filter mobile mở được
Cart item không tràn
Checkout form dễ nhập
Sticky CTA không che content
Footer không quá dài/rối
```

### 46.3. Admin responsive checklist

```text
Sidebar chuyển drawer trên mobile
Table có strategy mobile
Form không quá chật
Toolbar wrap đúng
Modal không vượt màn hình
Action quan trọng vẫn bấm được
```

---

## 47. Content design

### 47.1. Tone Storefront

Storefront nên thân thiện, rõ, thúc đẩy hành động.

Ví dụ:

```text
Thêm vào giỏ
Mua ngay
Tiếp tục mua sắm
Đặt hàng thành công
Sản phẩm hiện đã hết hàng
```

Không dùng text kỹ thuật.

Không viết:

```text
Submit failed due to validation error
```

Nên viết:

```text
Vui lòng kiểm tra lại thông tin giao hàng.
```

### 47.2. Tone Admin

Admin nên ngắn, chính xác.

Ví dụ:

```text
Lưu thay đổi
Tạo sản phẩm
Xác nhận đơn
Hủy đơn hàng
Cập nhật tồn kho
```

Thông báo lỗi admin có thể chi tiết hơn khách hàng, nhưng vẫn phải dễ hiểu.

---

## 48. Data formatting

### 48.1. Money

Tiền Việt Nam:

```text
1.000đ
15.000đ
1.250.000đ
```

Rule:

- Căn phải trong table.
- Không hiển thị quá nhiều decimal nếu không cần.
- Nếu đa tiền tệ, dùng currency code rõ.

### 48.2. Date/time

Chuẩn đề xuất:

```text
22/06/2026
22/06/2026 16:30
```

Admin có thể hiển thị ngày giờ đầy đủ.

Storefront chỉ hiển thị ngày giờ khi cần.

### 48.3. Quantity

```text
Tồn kho: 120
Đã bán: 1.240
Số lượng: 2
```

### 48.4. Phone

Input phone cần format và validate theo thị trường.

Việt Nam:

```text
0900000000
+84900000000
```

---

## 49. Theme strategy

### 49.1. Theme layers

Hệ thống có 3 lớp theme:

```text
Core tokens
Semantic tokens
Component tokens
```

Core tokens là giá trị thô:

```text
brand-600
neutral-900
space-4
radius-md
```

Semantic tokens là ý nghĩa:

```text
color-primary
color-text
color-background
color-border
```

Component tokens là cho component:

```text
button-primary-bg
card-border
input-focus-border
```

### 49.2. Clone theme cho ngành khác

Khi clone source, ưu tiên sửa:

```text
Brand color
Font
Product image ratio
Border radius intensity
Homepage section order
CTA wording
```

Không sửa:

```text
Order flow cốt lõi
Cart behavior
Checkout validation
Admin table pattern
Status system
```

### 49.3. Theme preset ví dụ

#### Neutral Commerce

```text
brand: blue
radius: medium
image ratio: 1:1
style: đa ngành
```

#### Beauty

```text
brand: rose
radius: large
image ratio: 1:1 hoặc 4:5
style: sáng, mềm
```

#### Fashion

```text
brand: black/neutral
radius: medium
image ratio: 4:5
style: nhiều ảnh
```

#### Electronics

```text
brand: blue/indigo
radius: small-medium
image ratio: 1:1
style: rõ thông số
```

#### Grocery

```text
brand: green/orange
radius: medium
image ratio: 1:1
style: mua nhanh, giá rõ
```

---

## 50. Component naming convention

### 50.1. Storefront components

```text
StoreHeader
StoreFooter
HeroBanner
CategoryMenu
ProductCard
ProductGrid
ProductGallery
PriceDisplay
StockBadge
VariantSelector
QuantitySelector
CartItem
OrderSummary
CheckoutForm
PaymentMethodSelector
ShippingMethodSelector
OrderTimeline
```

### 50.2. Admin components

```text
AdminLayout
AdminSidebar
AdminTopbar
AdminPageHeader
DataTable
TableToolbar
StatusBadge
StatCard
FilterPanel
BulkActionBar
ConfirmDialog
FormSection
ImageUploader
OrderStatusTimeline
```

### 50.3. Shared components

```text
Button
Input
Select
Textarea
Checkbox
Radio
Switch
Badge
Card
Modal
Drawer
Toast
Tooltip
Tabs
Accordion
Pagination
Breadcrumb
Skeleton
EmptyState
ErrorState
Avatar
DropdownMenu
```

---

## 51. CSS architecture guideline

Không phụ thuộc công nghệ, nhưng cần rule để tránh CSS loạn.

### 51.1. Rule chung

- Không sửa global style tùy tiện.
- Component tự quản style của nó.
- Dùng token thay vì hard-code value.
- Không dùng `!important` trừ trường hợp bất khả kháng.
- Không dùng selector quá sâu.
- Không style dựa vào DOM structure phức tạp nếu dễ vỡ.

### 51.2. CSS variable ví dụ

```css
:root {
  --color-brand-600: #2563eb;
  --color-text-primary: #111827;
  --color-text-secondary: #4b5563;
  --color-border-default: #e5e7eb;
  --color-bg-page: #ffffff;
  --space-4: 16px;
  --radius-md: 8px;
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
}
```

### 51.3. Component token ví dụ

```css
.button-primary {
  background: var(--button-primary-bg, var(--color-brand-600));
  color: var(--button-primary-text, #ffffff);
  border-radius: var(--button-radius, var(--radius-md));
}
```

---

## 52. Design QA checklist

Trước khi coi một màn hình là xong, phải kiểm tra:

### 52.1. Visual

```text
Spacing đúng token
Typography đúng cấp
Màu đúng token
Không overflow ngang
Không bị lệch layout
Ảnh không méo
Button/input cùng style
```

### 52.2. State

```text
Loading có hiển thị
Empty có hiển thị
Error có hiển thị
Disabled đúng
Hover/focus rõ
Success feedback có
```

### 52.3. Responsive

```text
320px
375px
768px
1024px
1440px
```

### 52.4. Accessibility

```text
Tab được
Focus visible
Label đầy đủ
Alt image
Contrast ổn
Modal trap focus
```

### 52.5. Commerce

```text
Giá hiển thị đúng
Sale đúng
Hết hàng đúng
Không thêm giỏ sai
Checkout validate đúng
Order status đúng
```

---

## 53. Rule cho coding agent

Đây là rule bắt buộc khi agent code frontend theo design system này.

### 53.1. Không được tự ý

Agent không được tự ý:

```text
Đổi token màu
Đổi spacing scale
Đổi radius scale
Đổi layout breakpoint
Dùng màu hard-code
Bỏ qua loading/empty/error state
Bỏ qua mobile
Xóa test để pass
Dùng selector fragile trong test
```

### 53.2. Khi tạo component mới

Agent phải xác định:

```text
Component thuộc shared/storefront/admin?
Props là gì?
State là gì?
Responsive ra sao?
Có accessibility requirement gì?
Có test cần thêm không?
```

### 53.3. Khi sửa UI

Agent phải báo cáo:

```text
File đã sửa
Component ảnh hưởng
Token đã dùng
Viewport đã kiểm tra
Test đã chạy
Screenshot nếu có
```

### 53.4. Khi gặp thiếu spec

Nếu thiếu spec nhỏ, agent chọn theo design system này.

Nếu thiếu spec ảnh hưởng nghiệp vụ lớn, agent phải ghi rõ assumption trong file implementation note hoặc báo lại.

---

## 54. Playwright visual rule đề xuất

Các màn hình bắt buộc nên có visual snapshot:

```text
Home page
Product listing page
Product detail page
Cart page
Checkout page
Order success page
Admin dashboard
Admin product list
Admin product form
Admin order list
Admin order detail
```

Viewport bắt buộc:

```text
375px mobile
768px tablet
1440px desktop
```

Các lỗi cần fail test:

```text
Horizontal overflow
Main CTA invisible
Console error
Image broken nếu ảnh chính
Form submit lỗi không hiển thị message
Modal không đóng được
```

---

## 55. Definition of Done cho UI

Một UI task chỉ được coi là xong khi đạt:

```text
Đúng spec
Đúng token
Đúng responsive
Đủ state
Không lỗi console
Không overflow ngang
Có accessibility cơ bản
Có test hoặc screenshot chứng minh
Không phá component khác
```

---

## 56. Tóm tắt ngôn ngữ thiết kế

Design system này dùng triết lý:

```text
Storefront: rõ sản phẩm, dễ mua, tạo niềm tin.
Admin: rõ dữ liệu, thao tác nhanh, giảm lỗi vận hành.
```

Mọi UI phải dựa trên:

```text
Token
Component
State
Responsive
Accessibility
Testable acceptance criteria
```

Khi clone source cho ngành khác, không viết lại toàn bộ UI. Chỉ thay:

```text
Theme token
Ảnh
Nội dung
Thứ tự section
Một vài component ngành đặc thù
```

Logic giao diện nền tảng vẫn giữ nguyên:

```text
Header
Search
Product grid
Product detail
Cart
Checkout
Order
Admin product
Admin order
Admin settings
```

Đây là nền móng đầu tiên để xây tiếp:

```text
01-storefront-page-spec.md
02-admin-page-spec.md
03-component-spec.md
04-playwright-test-strategy.md
05-agent-coding-rules.md
```
