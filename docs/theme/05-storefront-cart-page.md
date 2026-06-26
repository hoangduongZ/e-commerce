# 05 - Storefront Cart Page Specification

> File này đặc tả chi tiết trang **Giỏ hàng** cho website bán đồ điện tử.  
> Tài liệu này là một phần của bộ thiết kế storefront dựa trên:
>
> - `ecommerce_design_language.md`
> - `01-electronics-store-theme.md`
> - `02-storefront-home-page.md`
> - `03-storefront-product-list-page.md`
> - `04-storefront-product-detail-page.md`

---

## 1. Mục tiêu của trang

Trang giỏ hàng là bước trung gian giữa **trang chi tiết sản phẩm** và **checkout**.

Mục tiêu chính:

1. Cho khách kiểm tra lại sản phẩm đã chọn.
2. Cho khách thay đổi số lượng.
3. Cho khách xóa sản phẩm khỏi giỏ.
4. Cho khách nhìn rõ tổng tiền tạm tính.
5. Cho khách áp dụng mã giảm giá nếu có.
6. Cho khách ước tính phí vận chuyển.
7. Cảnh báo các vấn đề trước khi checkout.
8. Dẫn khách sang trang thanh toán nhanh, rõ ràng.

Trang này phải tạo cảm giác:

- Rõ ràng.
- Tin cậy.
- Không rối.
- Dễ kiểm tra thông tin.
- Dễ chuyển sang thanh toán.

Đối với web bán đồ điện tử, trang giỏ hàng rất quan trọng vì sản phẩm thường có giá trị cao. Khách cần thấy rõ:

- Model đã chọn.
- Biến thể đã chọn.
- Chính sách bảo hành.
- Giá khuyến mãi.
- Tình trạng tồn kho.
- Tổng tiền phải trả.

---

## 2. Vai trò trong user journey

Luồng chính:

```text
Product Detail
→ Add to Cart
→ Cart Page
→ Checkout Page
→ Order Success Page
```

Luồng phụ:

```text
Product List
→ Add to Cart
→ Mini Cart
→ Cart Page
```

Luồng tiếp tục mua:

```text
Cart Page
→ Continue Shopping
→ Product List / Previous Category
```

Luồng xử lý lỗi:

```text
Cart Page
→ Item out of stock
→ User remove item / choose similar product
```

```text
Cart Page
→ Item price changed
→ User confirm updated price
→ Checkout
```

---

## 3. Người dùng mục tiêu

### 3.1. Guest user

Guest user là khách chưa đăng nhập.

Họ vẫn được phép:

- Thêm sản phẩm vào giỏ.
- Xem giỏ hàng.
- Cập nhật số lượng.
- Xóa sản phẩm.
- Nhập coupon.
- Sang checkout.

Dữ liệu giỏ hàng có thể lưu bằng:

- Cookie.
- Local storage.
- Session id.
- Server-side anonymous cart.

Khi guest đăng nhập, hệ thống cần merge giỏ hàng hiện tại với giỏ hàng của tài khoản.

### 3.2. Logged-in customer

Logged-in customer là khách đã đăng nhập.

Họ có thêm lợi ích:

- Giỏ hàng được lưu giữa nhiều thiết bị.
- Có thể thấy địa chỉ mặc định.
- Có thể dùng coupon cá nhân.
- Có thể thấy điểm tích lũy.
- Có thể thấy ưu đãi thành viên.

### 3.3. Returning customer

Returning customer là khách đã mua trước đó.

Trang cart có thể gợi ý thêm:

- Sản phẩm liên quan.
- Phụ kiện đi kèm.
- Bảo hành mở rộng.
- Sản phẩm đã xem gần đây.

---

## 4. Page identity

| Thuộc tính | Giá trị |
|---|---|
| Page name | Cart Page |
| Route | `/cart` |
| Page type | Storefront |
| Auth required | No |
| Primary CTA | Checkout |
| Secondary CTA | Continue shopping |
| Main entity | Cart |
| Child entity | CartItem |

---

## 5. Layout tổng quan

### 5.1. Desktop layout

Desktop dùng layout 2 cột.

```text
┌────────────────────────────────────────────────────────────┐
│ Header                                                     │
├────────────────────────────────────────────────────────────┤
│ Breadcrumb                                                 │
├────────────────────────────────────────────────────────────┤
│ Page Title: Giỏ hàng                                      │
├──────────────────────────────────────┬─────────────────────┤
│ Cart Items                            │ Order Summary       │
│ - Item 1                              │ - Subtotal          │
│ - Item 2                              │ - Discount          │
│ - Item 3                              │ - Shipping estimate │
│                                      │ - Total             │
│ Recommended accessories               │ - Checkout CTA      │
├──────────────────────────────────────┴─────────────────────┤
│ Recently Viewed / Related Products                         │
├────────────────────────────────────────────────────────────┤
│ Footer                                                     │
└────────────────────────────────────────────────────────────┘
```

Desktop width rule:

- Main content max-width: theo design language gốc.
- Cart items column: khoảng 65% - 70%.
- Order summary column: khoảng 30% - 35%.
- Order summary nên sticky khi scroll nếu chiều cao viewport đủ.

### 5.2. Tablet layout

Tablet có thể dùng 1 cột hoặc 2 cột tùy width.

Khuyến nghị:

- Từ 768px đến 1023px: vẫn có thể dùng 2 cột nếu đủ rộng.
- Nếu layout bị chật, chuyển order summary xuống dưới cart items.

### 5.3. Mobile layout

Mobile dùng layout 1 cột.

```text
┌──────────────────────┐
│ Header               │
├──────────────────────┤
│ Page Title           │
├──────────────────────┤
│ Cart Item 1          │
├──────────────────────┤
│ Cart Item 2          │
├──────────────────────┤
│ Coupon               │
├──────────────────────┤
│ Shipping Estimate    │
├──────────────────────┤
│ Order Summary        │
├──────────────────────┤
│ Sticky Checkout Bar  │
└──────────────────────┘
```

Mobile cần có sticky checkout bar ở cuối màn hình.

Sticky bar hiển thị:

- Tổng tiền.
- Nút thanh toán.

Không nên để khách phải scroll quá xa mới thấy nút checkout.

---

## 6. Thành phần chính trên trang

Trang cart gồm các block sau:

1. Header.
2. Breadcrumb.
3. Page title.
4. Cart item list.
5. Cart item card.
6. Quantity selector.
7. Item warning.
8. Coupon box.
9. Shipping estimator.
10. Order summary.
11. Trust policy box.
12. Recommended accessories.
13. Recently viewed products.
14. Empty cart state.
15. Sticky mobile checkout bar.

---

## 7. Header rule

Header dùng chung với storefront.

Trên cart page, header cần hiển thị:

- Logo.
- Search bar.
- Category menu.
- Account entry.
- Cart icon.

Cart icon nên hiển thị số lượng item hiện tại.

Nếu đang ở trang cart, cart icon vẫn active nhưng không cần mở mini cart khi click.

---

## 8. Breadcrumb

Breadcrumb giúp khách quay lại mua tiếp.

Ví dụ:

```text
Home / Cart
```

Tiếng Việt:

```text
Trang chủ / Giỏ hàng
```

Breadcrumb rule:

- Desktop: hiển thị đầy đủ.
- Mobile: có thể rút gọn hoặc ẩn nếu không cần.
- Link `Trang chủ` dẫn về `/`.

---

## 9. Page title

Page title:

```text
Giỏ hàng
```

Có thể kèm số lượng item:

```text
Giỏ hàng của bạn (3 sản phẩm)
```

Nếu tổng quantity là 5 nhưng có 3 dòng sản phẩm, text nên dùng rõ:

```text
Giỏ hàng của bạn (5 sản phẩm)
```

Không nên dùng:

```text
Giỏ hàng của bạn (3 item)
```

Vì khách thường hiểu theo số lượng thực tế.

---

## 10. Cart item list

### 10.1. Mục đích

Cart item list hiển thị toàn bộ sản phẩm khách đã thêm vào giỏ.

Mỗi item phải đủ thông tin để khách kiểm tra:

- Ảnh.
- Tên sản phẩm.
- SKU/model.
- Biến thể.
- Giá.
- Số lượng.
- Tổng tiền dòng.
- Tồn kho.
- Bảo hành.
- Hành động xóa.

### 10.2. Desktop item layout

```text
┌─────────────────────────────────────────────────────────────┐
│ [Image]  Product info                  Price   Qty   Total  │
│          Variant / SKU / Warranty                           │
│          Stock status / Warning                              │
│          Remove / Save for later / Compare                   │
└─────────────────────────────────────────────────────────────┘
```

### 10.3. Mobile item layout

```text
┌──────────────────────┐
│ [Image] Product name │
│ Variant / SKU        │
│ Price                │
│ Qty selector         │
│ Line total           │
│ Warning              │
│ Remove               │
└──────────────────────┘
```

Mobile không nên ép nhiều cột. Tất cả thông tin quan trọng cần xuống dòng rõ ràng.

---

## 11. Cart item card specification

### 11.1. Required fields

| Field | Required |
|---|---|
| product image | Yes |
| product name | Yes |
| product link | Yes |
| unit price | Yes |
| quantity | Yes |
| line total | Yes |
| remove action | Yes |
| stock status | Yes |
| variant | Conditional |
| warranty | Recommended |
| promotion | Conditional |

### 11.2. Product image

Rule:

- Aspect ratio: vuông hoặc gần vuông.
- Không bị méo ảnh.
- Có background trung tính.
- Nếu ảnh lỗi, dùng placeholder.
- Alt text phải mô tả tên sản phẩm.

Kích thước gợi ý:

- Desktop: 96px - 120px.
- Mobile: 80px - 96px.

### 11.3. Product name

Product name là link về trang chi tiết sản phẩm.

Rule:

- Desktop: tối đa 2 dòng.
- Mobile: tối đa 2 hoặc 3 dòng.
- Không cắt mất model quan trọng nếu có thể.
- Nếu tên dài, dùng line clamp.

Ví dụ tốt:

```text
Laptop Dell Inspiron 15 3520 i5 1235U / 16GB / 512GB SSD
```

Ví dụ không tốt:

```text
Laptop Dell Inspiron...
```

Vì mất thông tin model.

### 11.4. Variant display

Đồ điện tử thường có biến thể:

- Màu.
- Dung lượng.
- RAM.
- SSD.
- Phiên bản bảo hành.
- Phiên bản chip.

Hiển thị biến thể dưới tên sản phẩm.

Ví dụ:

```text
Màu: Silver · RAM: 16GB · SSD: 512GB
```

Nếu không có biến thể, không hiển thị dòng này.

### 11.5. SKU/model display

SKU hoặc model giúp khách kiểm tra đúng sản phẩm.

Ví dụ:

```text
SKU: LAP-DELL-INS-3520-I5-16-512
```

Trên mobile có thể rút gọn:

```text
SKU: INS-3520-I5
```

### 11.6. Warranty display

Với đồ điện tử, bảo hành là thông tin quan trọng.

Ví dụ:

```text
Bảo hành chính hãng 24 tháng
```

Nếu sản phẩm có bảo hành mở rộng, hiển thị option upsell:

```text
Thêm gói bảo hành 12 tháng
```

Phần upsell không được làm phân tâm khỏi checkout.

### 11.7. Unit price

Unit price là giá một sản phẩm.

Rule:

- Hiển thị rõ giá bán hiện tại.
- Nếu có giá gốc, hiển thị nhỏ hơn và gạch ngang.
- Nếu có discount, hiển thị badge nhỏ.

Ví dụ:

```text
15.990.000đ
18.990.000đ
-16%
```

### 11.8. Quantity selector

Quantity selector gồm:

- Button giảm.
- Input số lượng.
- Button tăng.

Rule:

- Min quantity: 1.
- Max quantity: theo tồn kho hoặc rule bán hàng.
- Không cho nhập số âm.
- Không cho nhập ký tự không phải số.
- Nếu nhập vượt tồn kho, hiển thị lỗi và tự điều chỉnh hoặc chặn update.

Ví dụ:

```text
[-] [2] [+]
```

### 11.9. Line total

Line total = unit price hiện tại × quantity.

Nếu có discount item-level, cần tính rõ.

Ví dụ:

```text
31.980.000đ
```

Line total nên nổi bật hơn unit price.

### 11.10. Remove action

Action xóa sản phẩm phải rõ nhưng không quá nổi.

Text:

```text
Xóa
```

Hoặc:

```text
Remove
```

Khi click xóa, nên có confirm nhẹ hoặc undo toast.

Khuyến nghị dùng undo toast:

```text
Đã xóa sản phẩm khỏi giỏ hàng. Hoàn tác
```

Undo tốt hơn confirm modal vì ít làm gián đoạn flow.

---

## 12. Cart item states

### 12.1. Normal state

Item hợp lệ, còn hàng, giá không đổi.

Hiển thị:

- Ảnh.
- Tên.
- Biến thể.
- Giá.
- Số lượng.
- Line total.
- CTA remove.

### 12.2. Out of stock state

Sản phẩm đã hết hàng sau khi thêm vào giỏ.

UI rule:

- Hiển thị badge `Hết hàng`.
- Disable quantity selector.
- Không cho checkout nếu vẫn còn item hết hàng.
- Hiển thị action `Xóa khỏi giỏ`.
- Có thể gợi ý sản phẩm tương tự.

Message:

```text
Sản phẩm này hiện đã hết hàng. Vui lòng xóa khỏi giỏ để tiếp tục thanh toán.
```

### 12.3. Low stock state

Sản phẩm còn ít hàng.

UI rule:

- Hiển thị warning nhẹ.
- Không chặn checkout.

Message:

```text
Chỉ còn 2 sản phẩm trong kho.
```

### 12.4. Price changed state

Giá sản phẩm thay đổi sau khi thêm vào giỏ.

UI rule:

- Hiển thị giá cũ nhỏ hơn.
- Hiển thị giá mới rõ ràng.
- Hiển thị warning.
- Cần user nhận biết trước khi checkout.

Message:

```text
Giá sản phẩm đã thay đổi từ 15.490.000đ sang 15.990.000đ.
```

Nếu muốn an toàn hơn, yêu cầu khách xác nhận:

```text
Tôi đồng ý với giá mới
```

MVP có thể chỉ hiển thị warning và dùng giá mới.

### 12.5. Variant unavailable state

Biến thể đã chọn không còn bán.

Ví dụ màu Silver hết nhưng màu Black còn.

UI rule:

- Hiển thị warning.
- Cho khách đổi biến thể nếu có.
- Không cho checkout với biến thể không còn bán.

Message:

```text
Phiên bản Silver / 512GB hiện không còn bán. Vui lòng chọn phiên bản khác.
```

### 12.6. Invalid quantity state

Số lượng vượt tồn kho hoặc vượt giới hạn mua.

Message:

```text
Bạn chỉ có thể mua tối đa 3 sản phẩm này.
```

### 12.7. Loading update state

Khi khách tăng/giảm số lượng, item có trạng thái loading nhẹ.

Rule:

- Disable button tăng/giảm trong lúc request.
- Không làm toàn trang loading.
- Nếu request fail, rollback quantity về giá trị cũ.
- Hiển thị toast lỗi.

### 12.8. Removed state with undo

Sau khi xóa item:

- Item biến mất khỏi list.
- Toast xuất hiện.
- Cho undo trong vài giây.

Message:

```text
Đã xóa Laptop Dell Inspiron khỏi giỏ hàng. Hoàn tác
```

---

## 13. Coupon box

### 13.1. Mục đích

Coupon box cho khách nhập mã giảm giá.

Đối với web đồ điện tử, coupon có thể là:

- Giảm tiền trực tiếp.
- Giảm phần trăm.
- Miễn phí vận chuyển.
- Giảm khi mua kèm phụ kiện.
- Giảm theo phương thức thanh toán.

### 13.2. Layout

Desktop:

```text
┌────────────────────────────┐
│ Mã giảm giá                │
│ [Nhập mã] [Áp dụng]        │
│ Gợi ý: TECH500             │
└────────────────────────────┘
```

Mobile:

```text
Mã giảm giá
[Nhập mã]
[Áp dụng]
```

### 13.3. Coupon states

| State | UI |
|---|---|
| empty | input trống |
| applying | button loading |
| applied | success message |
| invalid | error message |
| expired | error message |
| not eligible | warning |

### 13.4. Applied coupon display

Khi áp dụng thành công:

```text
TECH500 đã được áp dụng
-500.000đ
[Xóa mã]
```

### 13.5. Error message examples

Invalid:

```text
Mã giảm giá không hợp lệ.
```

Expired:

```text
Mã giảm giá đã hết hạn.
```

Not eligible:

```text
Đơn hàng cần tối thiểu 20.000.000đ để dùng mã này.
```

Already used:

```text
Bạn đã sử dụng mã này trước đó.
```

---

## 14. Shipping estimator

### 14.1. Mục đích

Shipping estimator giúp khách ước tính phí giao hàng trước khi checkout.

Với đồ điện tử, phí ship có thể phụ thuộc vào:

- Khu vực giao hàng.
- Khối lượng.
- Kích thước.
- Giá trị đơn hàng.
- Phương thức giao.
- Chính sách miễn phí ship.

### 14.2. MVP version

MVP có thể đơn giản:

- Chọn tỉnh/thành.
- Chọn quận/huyện nếu cần.
- Hiển thị phí tạm tính.

Ví dụ:

```text
Giao đến: Hà Nội
Phí vận chuyển dự kiến: 30.000đ
```

### 14.3. Full version

Full version:

- Chọn tỉnh/thành.
- Chọn quận/huyện.
- Chọn phường/xã.
- Nhập địa chỉ.
- Chọn phương thức giao.

Options:

```text
Giao tiêu chuẩn: 30.000đ - 2 đến 3 ngày
Giao nhanh: 50.000đ - trong ngày
Nhận tại cửa hàng: miễn phí
```

### 14.4. Shipping states

| State | UI |
|---|---|
| not selected | ask location |
| estimating | loading |
| available | show options |
| unavailable | warning |
| free shipping | success badge |

### 14.5. Free shipping message

```text
Bạn được miễn phí vận chuyển cho đơn hàng này.
```

Hoặc:

```text
Mua thêm 1.000.000đ để được miễn phí vận chuyển.
```

---

## 15. Order summary

### 15.1. Mục đích

Order summary là block quan trọng nhất trong cart page.

Nó cho khách biết:

- Tổng tiền hàng.
- Giảm giá.
- Phí ship ước tính.
- Tổng thanh toán dự kiến.
- Nút thanh toán.

### 15.2. Desktop layout

```text
┌────────────────────────────┐
│ Tóm tắt đơn hàng           │
├────────────────────────────┤
│ Tạm tính        31.980.000đ│
│ Giảm giá          -500.000đ│
│ Phí vận chuyển      30.000đ│
├────────────────────────────┤
│ Tổng cộng        31.510.000đ│
├────────────────────────────┤
│ [Tiến hành thanh toán]     │
│ [Tiếp tục mua sắm]         │
└────────────────────────────┘
```

### 15.3. Mobile layout

Trên mobile, summary vẫn hiển thị trong flow, nhưng sticky checkout bar phải có:

```text
Tổng cộng: 31.510.000đ  [Thanh toán]
```

### 15.4. Fields

| Field | Required |
|---|---|
| subtotal | Yes |
| discount | Conditional |
| shipping_fee | Conditional |
| tax | Conditional |
| total | Yes |
| checkout CTA | Yes |
| continue shopping | Recommended |

### 15.5. Total amount rule

Total amount phải là số nổi bật nhất trong order summary.

Rule:

- Font lớn hơn các dòng khác.
- Dùng primary/dark text.
- Không dùng quá nhiều màu.
- Nếu có giảm giá, discount dùng màu sale/accent.

### 15.6. Checkout CTA

Primary CTA text:

```text
Tiến hành thanh toán
```

Hoặc ngắn hơn:

```text
Thanh toán
```

Rule:

- Full width trong order summary.
- Disabled nếu giỏ hàng không hợp lệ.
- Khi disabled, cần có message giải thích.

Disabled reasons:

- Cart empty.
- Có item hết hàng.
- Có item quantity invalid.
- Có item variant unavailable.
- Giá thay đổi cần xác nhận.

Message example:

```text
Vui lòng xử lý sản phẩm hết hàng trước khi thanh toán.
```

---

## 16. Trust policy box

### 16.1. Mục đích

Trust policy box tăng niềm tin trước khi khách checkout.

Đồ điện tử giá trị cao nên cần rõ:

- Bảo hành.
- Đổi trả.
- Giao hàng.
- Hỗ trợ kỹ thuật.
- Thanh toán an toàn.

### 16.2. Content examples

```text
Bảo hành chính hãng
Đổi trả trong 7 ngày
Giao hàng toàn quốc
Hỗ trợ kỹ thuật sau mua
Thanh toán an toàn
```

### 16.3. UI rule

- Dùng icon nhỏ.
- Text ngắn.
- Không chiếm quá nhiều diện tích.
- Desktop đặt dưới order summary.
- Mobile đặt gần summary hoặc dưới checkout button.

---

## 17. Recommended accessories

### 17.1. Mục đích

Gợi ý phụ kiện giúp tăng giá trị đơn hàng.

Ví dụ với laptop:

- Chuột.
- Balo laptop.
- Bàn phím.
- Tai nghe.
- Đế tản nhiệt.
- Gói Microsoft Office.
- Gói bảo hành mở rộng.

Ví dụ với điện thoại:

- Ốp lưng.
- Kính cường lực.
- Củ sạc.
- Tai nghe.
- Sạc dự phòng.

### 17.2. Placement

Khuyến nghị đặt dưới cart item list, trước recently viewed.

Không nên đặt trên order summary vì có thể làm phân tâm.

### 17.3. UI style

Dạng horizontal carousel hoặc grid nhỏ.

Card phụ kiện nên nhỏ hơn product card chính.

Fields:

- Image.
- Name.
- Price.
- Add button.

### 17.4. Add accessory action

Khi click thêm:

- Thêm item vào cart.
- Update order summary.
- Hiển thị toast.

Toast:

```text
Đã thêm Chuột Logitech M331 vào giỏ hàng.
```

---

## 18. Recently viewed products

### 18.1. Mục đích

Giúp khách quay lại sản phẩm đã xem nếu muốn thay đổi lựa chọn.

### 18.2. Rule

- Chỉ hiển thị nếu có dữ liệu.
- Không lặp lại sản phẩm đã có trong giỏ nếu không cần.
- Có thể đặt cuối trang.

---

## 19. Empty cart state

### 19.1. Khi nào hiển thị

Hiển thị khi giỏ hàng không có sản phẩm.

### 19.2. UI

```text
[Empty cart illustration]
Giỏ hàng của bạn đang trống
Hãy khám phá các sản phẩm công nghệ mới nhất.
[Tiếp tục mua sắm]
```

### 19.3. CTA

Primary CTA:

```text
Tiếp tục mua sắm
```

Link về:

- Home page.
- Category page.
- Product list page.

Khuyến nghị:

```text
/category/laptop
```

nếu hệ thống đang theo electronics/laptop MVP.

### 19.4. Optional sections

Có thể hiển thị:

- Sản phẩm bán chạy.
- Danh mục nổi bật.
- Recently viewed.

Nhưng không được làm empty state quá nặng.

---

## 20. Sticky mobile checkout bar

### 20.1. Mục đích

Trên mobile, khách không phải scroll xuống cuối để thanh toán.

### 20.2. Content

Sticky bar gồm:

- Tổng tiền.
- Checkout button.

Ví dụ:

```text
Tổng: 31.510.000đ  [Thanh toán]
```

### 20.3. Rule

- Chỉ hiển thị khi cart không rỗng.
- Không che nội dung cuối trang.
- Cần có bottom padding cho page content.
- Nếu checkout disabled, button disabled và tap vào có thể scroll đến warning.
- Không che browser safe area trên iOS.

### 20.4. Safe area

CSS cần tính đến:

```css
padding-bottom: env(safe-area-inset-bottom);
```

---

## 21. Data contract

### 21.1. Cart response

Ví dụ JSON tham khảo:

```json
{
  "id": "cart_123",
  "customer_id": "user_456",
  "session_id": "session_abc",
  "currency": "VND",
  "items": [
    {
      "id": "cart_item_1",
      "product_id": "prod_1",
      "variant_id": "var_1",
      "sku": "LAP-DELL-INS-3520-I5-16-512",
      "name": "Laptop Dell Inspiron 15 3520 i5 1235U / 16GB / 512GB SSD",
      "slug": "laptop-dell-inspiron-15-3520-i5-16gb-512gb",
      "image_url": "/images/products/dell-inspiron-3520.jpg",
      "unit_price": 15990000,
      "original_price": 18990000,
      "quantity": 2,
      "line_total": 31980000,
      "stock_status": "in_stock",
      "available_quantity": 5,
      "warranty": "Bảo hành chính hãng 24 tháng",
      "variant_attributes": [
        { "name": "RAM", "value": "16GB" },
        { "name": "SSD", "value": "512GB" },
        { "name": "Màu", "value": "Silver" }
      ],
      "warnings": []
    }
  ],
  "coupon": {
    "code": "TECH500",
    "discount_amount": 500000
  },
  "subtotal": 31980000,
  "discount_amount": 500000,
  "shipping_fee": 30000,
  "tax_amount": 0,
  "total_amount": 31510000,
  "is_checkout_allowed": true,
  "checkout_blockers": [],
  "updated_at": "2026-06-22T10:00:00+07:00"
}
```

### 21.2. Cart item fields

| Field | Type |
|---|---|
| id | string |
| product_id | string |
| variant_id | string/null |
| sku | string |
| name | string |
| slug | string |
| image_url | string |
| unit_price | number |
| original_price | number/null |
| quantity | number |
| line_total | number |
| stock_status | string |
| available_quantity | number |
| warranty | string/null |
| variant_attributes | array |
| warnings | array |

### 21.3. Checkout blockers

`checkout_blockers` là danh sách lý do không cho checkout.

Ví dụ:

```json
[
  {
    "type": "out_of_stock",
    "cart_item_id": "cart_item_2",
    "message": "Sản phẩm Laptop Asus Vivobook hiện đã hết hàng."
  }
]
```

Blocker types:

| Type | Meaning |
|---|---|
| out_of_stock | hết hàng |
| invalid_quantity | số lượng không hợp lệ |
| variant_unavailable | biến thể không còn bán |
| price_changed | giá thay đổi |
| coupon_invalid | coupon lỗi |

---

## 22. API behavior

### 22.1. Get cart

```http
GET /api/v1/cart
```

Returns current cart by:

- Auth token.
- Session id.
- Guest cart id.

### 22.2. Update quantity

```http
PATCH /api/v1/cart/items/{cart_item_id}
Content-Type: application/json

{
  "quantity": 3
}
```

Expected behavior:

- Validate quantity.
- Check stock.
- Update line total.
- Return updated cart.

### 22.3. Remove item

```http
DELETE /api/v1/cart/items/{cart_item_id}
```

Expected behavior:

- Remove item.
- Return updated cart.
- Frontend shows undo toast if supported.

### 22.4. Apply coupon

```http
POST /api/v1/cart/coupon
Content-Type: application/json

{
  "code": "TECH500"
}
```

Expected behavior:

- Validate coupon.
- Apply discount.
- Return updated cart.

### 22.5. Remove coupon

```http
DELETE /api/v1/cart/coupon
```

### 22.6. Estimate shipping

```http
POST /api/v1/cart/shipping-estimate
Content-Type: application/json

{
  "city": "Hà Nội",
  "district": "Hà Đông",
  "ward": "Mộ Lao"
}
```

### 22.7. Merge guest cart

```http
POST /api/v1/cart/merge
```

Called after login.

---

## 23. Formatting rules

### 23.1. Currency

Currency default:

```text
VND
```

Format:

```text
15.990.000đ
```

Do not show:

```text
15990000 VND
```

### 23.2. Quantity

Quantity display:

```text
1
2
3
```

No decimal.

### 23.3. Discount

Discount display:

```text
-500.000đ
```

Percentage badge:

```text
-16%
```

---

## 24. Loading states

### 24.1. Initial loading

Khi tải cart lần đầu:

- Hiển thị skeleton cho item list.
- Hiển thị skeleton cho order summary.
- Không hiển thị empty state khi chưa biết cart rỗng hay chưa.

### 24.2. Quantity updating

Khi update quantity:

- Chỉ loading item đang update.
- Disable quantity control của item đó.
- Summary có thể hiển thị shimmer nhẹ hoặc cập nhật sau khi response về.

### 24.3. Coupon applying

Khi apply coupon:

- Button `Áp dụng` chuyển loading.
- Input disabled.
- Không block toàn page.

### 24.4. Shipping estimating

Khi estimate ship:

- Field location disabled.
- Hiển thị loading nhỏ.

---

## 25. Error states

### 25.1. Cart load error

Khi không tải được cart:

```text
Không thể tải giỏ hàng. Vui lòng thử lại.
[Thử lại]
```

### 25.2. Update quantity error

```text
Không thể cập nhật số lượng. Vui lòng thử lại.
```

Frontend cần rollback quantity về giá trị cũ nếu update fail.

### 25.3. Remove item error

```text
Không thể xóa sản phẩm. Vui lòng thử lại.
```

### 25.4. Coupon error

Hiển thị gần input coupon, không chỉ toast.

### 25.5. Shipping estimate error

```text
Không thể ước tính phí vận chuyển. Vui lòng nhập lại địa chỉ.
```

---

## 26. Empty and partial data handling

### 26.1. Missing image

Dùng placeholder product image.

### 26.2. Missing warranty

Không hiển thị dòng warranty.

Không hiển thị:

```text
Bảo hành: null
```

### 26.3. Missing original price

Không hiển thị giá gốc.

### 26.4. Missing variant attributes

Không hiển thị variant row.

### 26.5. Missing shipping fee

Hiển thị:

```text
Phí vận chuyển: Sẽ tính ở bước thanh toán
```

---

## 27. Accessibility rules

### 27.1. Keyboard navigation

Người dùng phải dùng được bằng bàn phím:

- Tab đến quantity button.
- Tab đến remove button.
- Tab đến coupon input.
- Tab đến checkout button.

### 27.2. Button labels

Icon button phải có accessible label.

Ví dụ:

```text
Tăng số lượng Laptop Dell Inspiron
Giảm số lượng Laptop Dell Inspiron
Xóa Laptop Dell Inspiron khỏi giỏ hàng
```

### 27.3. Error announcement

Lỗi form hoặc cart blocker nên được screen reader đọc được.

Dùng `aria-live` cho vùng tổng hợp lỗi.

### 27.4. Focus management

Sau khi xóa item:

- Focus không được mất.
- Focus nên chuyển đến item tiếp theo hoặc toast undo.

Sau khi checkout button disabled do lỗi:

- Click/tap vào button có thể focus/scroll đến lỗi đầu tiên.

---

## 28. Responsive rules

### 28.1. Breakpoints

Dùng breakpoint theo design language gốc.

Gợi ý:

| Device | Width |
|---|---|
| mobile | 375px |
| tablet | 768px |
| laptop | 1024px |
| desktop | 1440px |

### 28.2. Mobile rules

Mobile bắt buộc:

- Không overflow ngang.
- Cart item không bị vỡ.
- Product name không che price.
- Quantity selector dễ bấm.
- Checkout sticky bar không che footer.
- Coupon input không bị quá hẹp.

### 28.3. Desktop rules

Desktop bắt buộc:

- Order summary nằm bên phải.
- Khoảng trắng hợp lý.
- Item list dễ scan.
- Table-like layout không bị lệch khi product name dài.

---

## 29. Performance rules

### 29.1. Image optimization

- Dùng ảnh kích thước phù hợp.
- Lazy load recommended products.
- Cart item image nên ưu tiên tải nhanh vì nằm above the fold.

### 29.2. API optimization

- `GET /cart` nên trả về đủ summary để tránh frontend gọi nhiều API.
- Update quantity nên trả về updated cart.
- Apply coupon nên trả về updated cart.

### 29.3. Avoid unnecessary reload

Không reload toàn page khi:

- Update quantity.
- Remove item.
- Apply coupon.
- Estimate shipping.

---

## 30. SEO rules

Cart page thường không cần index.

Rule:

```html
<meta name="robots" content="noindex,nofollow" />
```

Cart là trang cá nhân hóa, không nên để search engine index.

---

## 31. Security rules

### 31.1. Price trust

Frontend không được tự quyết định giá cuối cùng.

Frontend chỉ hiển thị dữ liệu server trả về.

Backend phải là source of truth cho:

- Unit price.
- Discount.
- Shipping fee.
- Tax.
- Total amount.

### 31.2. Quantity validation

Frontend validate để UX tốt.

Backend vẫn phải validate lại.

### 31.3. Coupon validation

Không trust coupon tính ở client.

### 31.4. Cart ownership

User chỉ được truy cập cart của chính họ hoặc cart theo session hợp lệ.

---

## 32. Analytics events

Nên bắn event cho các hành động quan trọng.

| Event | Trigger |
|---|---|
| view_cart | page loaded |
| update_cart_quantity | qty changed |
| remove_from_cart | item removed |
| apply_coupon | coupon applied |
| coupon_failed | coupon failed |
| estimate_shipping | shipping estimated |
| begin_checkout | checkout clicked |
| add_accessory_from_cart | accessory added |

Event data nên gồm:

- cart_id.
- product_id.
- variant_id.
- quantity.
- price.
- coupon_code.
- total_amount.

Không gửi dữ liệu nhạy cảm như phone/email ở event tracking nếu không cần.

---

## 33. Admin dependency

Cart page phụ thuộc vào admin/data config:

- Product active status.
- Variant active status.
- Stock quantity.
- Pricing.
- Promotion.
- Coupon.
- Shipping rules.
- Warranty info.
- Product images.

Nếu admin thay đổi giá/tồn kho, cart page phải phản ánh cập nhật mới.

---

## 34. Component list

Frontend nên tách thành các component:

```text
CartPage
CartHeader
CartItemList
CartItemCard
CartItemImage
CartItemInfo
CartItemPrice
CartQuantitySelector
CartItemWarning
CartCouponBox
CartShippingEstimator
CartOrderSummary
CartTrustPolicyBox
RecommendedAccessories
RecentlyViewedProducts
EmptyCartState
MobileCheckoutBar
```

### 34.1. Reusable components

Có thể tái sử dụng từ design system:

```text
Button
Input
Select
Badge
Toast
Card
Skeleton
PriceText
ProductImage
ProductCardCompact
Alert
```

---

## 35. Component contracts

### 35.1. CartQuantitySelector props

```ts
interface CartQuantitySelectorProps {
  value: number;
  min: number;
  max: number;
  disabled?: boolean;
  loading?: boolean;
  productName?: string;
  onChange: (nextValue: number) => void;
}
```

Rules:

- `value` không được nhỏ hơn `min`.
- `value` không được lớn hơn `max`.
- Khi disabled, tất cả control disabled.
- Khi loading, không nhận thêm input.

### 35.2. CartItemCard props

```ts
interface CartItemCardProps {
  item: CartItem;
  onQuantityChange: (itemId: string, quantity: number) => void;
  onRemove: (itemId: string) => void;
  onMoveToWishlist?: (itemId: string) => void;
}
```

### 35.3. CartOrderSummary props

```ts
interface CartOrderSummaryProps {
  subtotal: number;
  discountAmount: number;
  shippingFee?: number | null;
  taxAmount?: number;
  totalAmount: number;
  currency: string;
  checkoutAllowed: boolean;
  checkoutBlockers: CheckoutBlocker[];
  onCheckout: () => void;
}
```

---

## 36. Interaction rules

### 36.1. Click checkout

Nếu checkout allowed:

```text
/cart → /checkout
```

Nếu checkout blocked:

- Không chuyển trang.
- Hiển thị lỗi.
- Scroll đến blocker đầu tiên.

### 36.2. Click continue shopping

Ưu tiên redirect về nơi khách vừa đến:

1. Previous category page.
2. Recently viewed category.
3. Home page.

Nếu không có history, dùng `/` hoặc `/products`.

### 36.3. Quantity debounce

Nếu user nhập số lượng nhanh, frontend nên debounce hoặc chỉ submit khi blur/enter.

Với button tăng/giảm, có thể gọi API ngay nhưng phải tránh race condition.

### 36.4. Race condition handling

Nếu user click `+` liên tục:

- Disable trong lúc request.
- Hoặc dùng request queue.
- Response cũ không được override response mới.

---

## 37. Edge cases

### 37.1. Cart expired

Nếu guest cart hết hạn:

```text
Giỏ hàng của bạn đã hết hạn. Vui lòng thêm sản phẩm lại.
```

### 37.2. Product deleted

Nếu sản phẩm bị admin xóa/ẩn:

```text
Sản phẩm này hiện không còn bán.
```

### 37.3. Coupon removed by admin

Nếu coupon đang áp dụng bị admin tắt:

```text
Mã giảm giá không còn khả dụng và đã được xóa khỏi đơn hàng.
```

### 37.4. Multiple warehouses

Nếu hàng nằm ở nhiều kho, cart page không cần hiển thị chi tiết kho trong MVP.

Full version có thể hiển thị:

```text
Giao từ kho Hà Nội
```

### 37.5. Store pickup

Nếu hỗ trợ nhận tại cửa hàng, shipping estimator có thể thêm option:

```text
Nhận tại cửa hàng - Miễn phí
```

---

## 38. Copywriting rules

Tone:

- Rõ ràng.
- Tin cậy.
- Không gây hoảng.
- Không dùng từ quá kỹ thuật.

### 38.1. Good copy

```text
Sản phẩm này hiện đã hết hàng. Vui lòng xóa khỏi giỏ để tiếp tục thanh toán.
```

### 38.2. Bad copy

```text
Item invalid. Checkout blocked.
```

### 38.3. Button labels

| Purpose | Vietnamese |
|---|---|
| Checkout | Thanh toán |
| Continue | Tiếp tục mua sắm |
| Apply coupon | Áp dụng |
| Remove item | Xóa |
| Undo | Hoàn tác |
| Retry | Thử lại |

---

## 39. Visual style rules

### 39.1. Cart item card

- Nền trắng hoặc surface token.
- Border nhẹ.
- Radius theo design system.
- Shadow rất nhẹ hoặc không shadow.
- Khoảng cách thoáng.

### 39.2. Warning

- Out of stock: danger tone.
- Low stock: warning tone.
- Price changed: warning tone.
- Coupon success: success tone.

### 39.3. Price emphasis

- Total amount: mạnh nhất.
- Line total: mạnh thứ hai.
- Unit price: bình thường.
- Original price: muted + strike-through.

---

## 40. Playwright test specification

### 40.1. Basic render

```text
Given cart has items
When user opens /cart
Then cart items are visible
And order summary is visible
And checkout button is visible
```

### 40.2. Empty cart

```text
Given cart is empty
When user opens /cart
Then empty cart message is visible
And continue shopping button is visible
And checkout button is not visible
```

### 40.3. Update quantity

```text
Given cart has product quantity 1
When user clicks increase quantity
Then quantity becomes 2
And line total updates
And order summary total updates
```

### 40.4. Decrease quantity minimum

```text
Given cart item quantity is 1
When user clicks decrease quantity
Then quantity remains 1
And no invalid quantity is sent
```

### 40.5. Remove item

```text
Given cart has an item
When user clicks remove
Then item disappears
And undo toast is visible
And order summary updates
```

### 40.6. Undo remove

```text
Given user removed an item
When user clicks undo
Then item returns to cart
And order summary updates
```

### 40.7. Apply valid coupon

```text
Given cart has eligible subtotal
When user enters valid coupon
And clicks apply
Then discount appears
And total decreases
```

### 40.8. Apply invalid coupon

```text
Given cart has items
When user enters invalid coupon
And clicks apply
Then error message is visible
And total does not change
```

### 40.9. Out of stock blocker

```text
Given cart has out of stock item
When user opens /cart
Then out of stock warning is visible
And checkout button is disabled
```

### 40.10. Price changed warning

```text
Given item price changed
When user opens /cart
Then price changed warning is visible
And new price is displayed
```

### 40.11. Shipping estimate

```text
Given cart has items
When user selects delivery location
Then shipping fee is displayed
And total updates
```

### 40.12. Checkout navigation

```text
Given cart is valid
When user clicks checkout
Then user is navigated to /checkout
```

### 40.13. Mobile sticky checkout

```text
Given viewport is 375px wide
When user opens /cart with items
Then mobile sticky checkout bar is visible
And no horizontal overflow exists
```

---

## 41. Visual regression checklist

Capture screenshots for:

1. Desktop cart with 3 items.
2. Desktop cart with out of stock item.
3. Desktop empty cart.
4. Mobile cart with 2 items.
5. Mobile sticky checkout bar.
6. Mobile empty cart.
7. Coupon applied state.
8. Long product name state.
9. Price changed state.
10. Loading skeleton state.

Viewport:

| Case | Width |
|---|---|
| mobile | 375px |
| tablet | 768px |
| desktop | 1440px |

---

## 42. Agent implementation rules

Agent khi implement cart page phải tuân thủ:

1. Đọc file design language gốc trước.
2. Đọc `01-electronics-store-theme.md` trước.
3. Không tự ý đổi màu/token ngoài theme.
4. Không hard-code dữ liệu sản phẩm trong component chính.
5. Không tính total cuối cùng chỉ ở frontend.
6. Không bỏ qua trạng thái empty/loading/error.
7. Không bỏ qua mobile sticky checkout.
8. Không dùng selector fragile cho test.
9. Không xóa test để làm pass.
10. Không kết luận xong nếu chưa chạy test liên quan.

Khi báo cáo kết quả, agent phải ghi:

```text
Files changed:
- ...

Tests run:
- ...

Result:
- ...

Screenshots/Reports:
- ...
```

---

## 43. Definition of Done

Cart page được coi là hoàn thành khi:

### 43.1. Functional done

- Hiển thị cart items đúng.
- Hiển thị empty cart đúng.
- Update quantity được.
- Remove item được.
- Undo remove hoạt động nếu được implement.
- Apply coupon hoạt động.
- Shipping estimate hoạt động nếu trong scope.
- Order summary tính đúng theo server response.
- Checkout button điều hướng đúng.
- Checkout bị chặn khi cart invalid.

### 43.2. UI done

- Desktop layout 2 cột rõ ràng.
- Mobile layout 1 cột không overflow.
- Sticky checkout bar hoạt động trên mobile.
- Long product name không phá layout.
- Product image không méo.
- Warning/error hiển thị rõ.
- Loading skeleton có mặt.

### 43.3. Accessibility done

- Dùng được bằng keyboard.
- Icon button có accessible label.
- Error có thể được screen reader nhận biết.
- Focus không bị mất sau remove/update.

### 43.4. Test done

- E2E basic cart pass.
- E2E empty cart pass.
- E2E update quantity pass.
- E2E remove item pass.
- E2E coupon pass.
- E2E checkout navigation pass.
- Visual regression cho desktop/mobile pass.

---

## 44. MVP scope

Nếu cần làm MVP nhanh, chỉ bắt buộc:

1. Cart item list.
2. Quantity update.
3. Remove item.
4. Order summary.
5. Checkout CTA.
6. Empty cart state.
7. Mobile responsive.
8. Loading/error state cơ bản.

Có thể để sau:

- Coupon.
- Shipping estimator.
- Recommended accessories.
- Recently viewed.
- Save for later.
- Warranty upsell.
- Store pickup.

---

## 45. Future extensions

Sau MVP, có thể mở rộng:

1. Save for later.
2. Wishlist integration.
3. Compare from cart.
4. Bundle discount.
5. Accessory bundle.
6. Warranty upgrade.
7. Installment payment preview.
8. Loyalty point preview.
9. Multi-shipping address.
10. Store pickup reservation.

---

## 46. Summary

Cart page không chỉ là nơi hiển thị sản phẩm đã chọn. Với web bán đồ điện tử, nó là bước kiểm tra quan trọng trước khi khách quyết định thanh toán.

Trang này phải ưu tiên:

- Thông tin sản phẩm chính xác.
- Giá và tổng tiền rõ ràng.
- Tình trạng tồn kho minh bạch.
- Cảnh báo sớm nếu có vấn đề.
- Mobile checkout nhanh.
- Không làm khách mất niềm tin.

Khi agent code trang này, hãy yêu cầu chạy Playwright test và visual regression để đảm bảo flow `Add to cart → Cart → Checkout` không bị hỏng.
