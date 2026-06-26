# Thiết Kế Chi Tiết Cho Trang Web Bán Hàng Chuẩn

## 1. Giới Thiệu Chung

Một website thương mại điện tử là hệ thống phức tạp bao gồm nhiều thành phần hoạt động kết hợp để người dùng có thể tìm kiếm sản phẩm, đặt hàng, thanh toán và theo dõi đơn hàng. Các thành phần chính gồm:

- **Product Catalog System** – lưu trữ thông tin sản phẩm (tên, mô tả, giá, hình ảnh, biến thể, danh mục, thương hiệu...) và hỗ trợ tìm kiếm, lọc.
- **Shopping Cart & Checkout System** – quản lý danh sách sản phẩm khách đã chọn, tính toán giá, thuế, phí vận chuyển và xử lý luồng thanh toán.
- **Payment Processing System** – tích hợp các phương thức thanh toán (COD, chuyển khoản, ví điện tử, thẻ quốc tế) và đảm bảo giao dịch an toàn.
- **Order Management System** – theo dõi vòng đời đơn hàng từ đặt hàng đến giao nhận, ghi nhận hóa đơn và lịch sử.
- **Inventory Management System** – cập nhật số lượng tồn kho, ngăn bán vượt quá số lượng.
- **User & Personalization System** – quản lý đăng ký, đăng nhập, hồ sơ người dùng, đề xuất sản phẩm, danh sách yêu thích.
- **Pricing & Promotions Engine** – quản lý chương trình giảm giá, mã coupon, giá động theo mùa.
- **Analytics & Monitoring** – theo dõi hành vi người dùng, thống kê bán hàng, hiệu năng để ra quyết định.

---

## 2. Yêu Cầu Chức Năng

### 2.1. Yêu Cầu Cốt Lõi

1. **Quản lý danh mục và tồn kho** – tạo, sửa, xóa danh mục; thêm thông tin sản phẩm; cập nhật tồn kho.
2. **Tìm kiếm và duyệt sản phẩm** – hỗ trợ tìm kiếm toàn văn, lọc theo danh mục, giá, thương hiệu, thuộc tính.
3. **Quản lý đơn hàng** – khách đặt hàng, hệ thống tạo đơn; admin theo dõi, cập nhật trạng thái đơn.
4. **Xử lý thanh toán** – hỗ trợ nhiều phương thức (COD, chuyển khoản, ví điện tử, thẻ quốc tế...); đảm bảo tính an toàn.
5. **Quản lý vận chuyển** – tính phí giao hàng, hỗ trợ nhiều đơn vị vận chuyển; cập nhật trạng thái giao.
6. **Quản lý khách hàng** – đăng ký/đăng nhập, sửa hồ sơ, xem lịch sử đơn hàng.

### 2.2. Yêu Cầu Mở Rộng

- **Analytics & Insights** – thu thập dữ liệu truy cập, phân tích hiệu suất bán hàng và hành vi khách hàng.
- **AI/Recommendation** – gợi ý sản phẩm dựa trên lịch sử mua, xu hướng.
- **Quản lý khuyến mãi** – tạo mã giảm giá, flash sale, combo.

### 2.3. Yêu Cầu Phi Chức Năng

- **Hiệu năng và khả năng mở rộng** – đáp ứng số lượng người dùng lớn, tối ưu thời gian phản hồi. Hỗ trợ lưu trữ và tìm kiếm dữ liệu lớn (ElasticSearch cho sản phẩm, cache Redis).
- **Bảo mật** – SSL/TLS, chuẩn PCI DSS cho thẻ tín dụng, JWT/OAuth2 cho xác thực; phân quyền rõ ràng (customer, admin).
- **Khả năng tái sử dụng** – các mô-đun độc lập, có thể tái dùng cho nhiều ngành hàng (thời trang, điện tử, nội thất...).

---

## 3. Kiến Trúc Tổng Quan

Hệ thống được tổ chức theo **three-tier architecture**:

1. **Presentation Layer** – ứng dụng web/mobile giao tiếp với người dùng. Trình bày sản phẩm, giỏ hàng, thanh toán.
2. **Application Layer** – xử lý nghiệp vụ: thao tác giỏ hàng, tạo đơn hàng, tính toán phí, xử lý thanh toán, quản lý người dùng.
3. **Data Layer** – DBMS lưu sản phẩm, đơn hàng, tài khoản; dịch vụ lưu trữ file (hình ảnh); công cụ tìm kiếm.

### 3.1. Mô-Đun Chính

#### 3.1.1. Product Catalog Service

**Chức năng:** CRUD sản phẩm, tìm kiếm, phân loại theo danh mục/thuộc tính, quản lý giá.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `Product` | id, sku, name, slug, description, short_description, price, price_promo, currency, weight, dimensions, brand, status, created_at, updated_at |
| `ProductImage` | id, product_id, url, alt_text, is_primary |
| `ProductVariant` | id, product_id, attribute_values (JSON), sku, price_override, quantity |
| `Category` | id, parent_id (nullable), name, slug, description, image_url |
| `Attribute` | id, name, type (select/text/number/color/date), values |
| `ProductAttributeValue` | id, product_id, attribute_id, value |

**API:**

```
GET    /products                  # danh sách (filter, pagination, sort)
GET    /products/{id|slug}        # chi tiết
POST   /products                  # tạo mới (admin)
PUT    /products/{id}             # chỉnh sửa (admin)
DELETE /products/{id}             # xóa/ẩn (admin)
GET    /categories                # danh mục (nested)
```

**Search:** ElasticSearch/Solr cho full-text search, autocomplete, lọc theo thuộc tính.

---

#### 3.1.2. Cart Service

**Chức năng:** quản lý giỏ hàng cho user đăng nhập và guest; cập nhật số lượng; tính tạm thời giá và thuế.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `Cart` | id, customer_id (nullable), session_id, created_at, updated_at |
| `CartItem` | id, cart_id, product_id, variant_id (nullable), quantity, price_at_add_time |

**API:**

```
GET    /cart                      # danh sách item, tổng tiền, thuế, phí ship
POST   /cart/items                # thêm item {product_id, variant_id, quantity}
PATCH  /cart/items/{id}           # cập nhật số lượng
DELETE /cart/items/{id}           # xóa item
```

> **Lưu ý:** Guest cart lưu Redis với expiry. Khi đăng nhập, sync về DB.

---

#### 3.1.3. Order Service

**Chức năng:** xử lý đặt hàng, lưu đơn, cập nhật trạng thái, thông báo, hủy/trả hàng.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `Order` | id, order_number, customer_id, total_amount, discount_amount, shipping_fee, tax_fee, payment_method, order_status, shipping_status, shipping_provider, shipping_tracking_code, shipping_address_id, billing_address_id, placed_at, updated_at |
| `OrderItem` | id, order_id, product_id, variant_id, product_name_snapshot, price_snapshot, quantity |
| `Address` | id, customer_id, full_name, phone, street, ward, district, city, country, postal_code, type (shipping/billing), is_default |
| `OrderStatusHistory` | id, order_id, status, changed_at, note |

**API:**

```
POST   /orders                    # tạo đơn từ giỏ hàng
GET    /orders/{id}               # chi tiết đơn
GET    /orders?customer_id=       # lịch sử đơn
PUT    /orders/{id}/status        # cập nhật trạng thái (admin)
```

**Luồng xử lý:**

1. Kiểm tra tồn kho → reserve hàng.
2. Tạo `Order` + `OrderItem`; snapshot giá tại thời điểm đặt.
3. Chuyển sang Payment Service.
4. Thanh toán thành công → cập nhật status; thất bại → release reserve.
5. Gửi email/SMS xác nhận → đẩy sang Fulfillment Service.

---

#### 3.1.4. Payment Service

**Chức năng:** tích hợp các cổng thanh toán (VNPay, MoMo, PayPal, Stripe, thẻ tín dụng...); hỗ trợ COD.

**Quy trình:**

1. Order Service gửi yêu cầu thanh toán (số tiền, mô tả, `return_url`, `cancel_url`).
2. Payment Service redirect user đến cổng thanh toán.
3. Nhận callback/webhook → cập nhật trạng thái thanh toán.
4. Trả kết quả về Order Service.

> **Bảo mật:** PCI DSS khi lưu thẻ; HTTPS; xác thực HMAC cho API cổng thanh toán.

---

#### 3.1.5. Inventory Service

**Chức năng:** quản lý tồn kho theo kho/chi nhánh; cập nhật khi nhập/xuất; đồng bộ với đơn hàng.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `InventoryItem` | id, product_id, variant_id, warehouse_id, quantity_available, quantity_reserved, last_updated |
| `StockMovement` | id, inventory_item_id, type (import/export/reserve/release), quantity, related_order_id, created_at |

**API:**

```
GET    /inventory/{product_id}    # tồn kho theo sản phẩm/biến thể
POST   /inventory/reserve         # giữ hàng (atomic, rollback nếu thất bại)
POST   /inventory/release         # trả hàng (đơn bị hủy)
POST   /inventory/adjust          # nhập/xuất kho (admin)
```

---

#### 3.1.6. User Service

**Chức năng:** quản lý user (customer, admin), đăng ký, đăng nhập, quên mật khẩu, phân quyền.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `User` | id, email, phone, password_hash, role (customer/admin), status (active/locked), created_at |
| `UserProfile` | id, user_id, full_name, gender, date_of_birth, avatar_url |
| `UserRolePermission` | mapping role → quyền (product.manage, order.view, order.update...) |

**API:**

```
POST   /auth/register             # đăng ký
POST   /auth/login                # đăng nhập (trả JWT/session id)
POST   /auth/refresh              # refresh token
GET    /users/me                  # thông tin cá nhân
PUT    /users/me                  # cập nhật hồ sơ
POST   /auth/reset-password       # đổi mật khẩu qua email/SMS
```

---

#### 3.1.7. Promotion & Pricing Service

**Chức năng:** quản lý giảm giá, voucher, flash sale; tính giá cuối cho đơn hàng.

**Data models:**

| Entity | Thuộc tính chính |
|---|---|
| `Coupon` | id, code, description, discount_type (percentage/fixed), discount_value, min_order_amount, max_discount_amount, start_date, end_date, usage_limit, usage_per_user, status |
| `Promotion` | id, name, description, applicable_products, discount_type, discount_value, start_date, end_date, priority |

**API:**

```
POST   /coupons/apply             # áp dụng mã giảm giá
GET    /promotions                # danh sách khuyến mãi hiện hành
POST   /promotions                # tạo chương trình khuyến mãi (admin)
```

---

#### 3.1.8. Review & Rating Service *(tùy chọn)*

Cho phép khách đánh giá sản phẩm, viết bình luận, cho điểm sao; kiểm duyệt bình luận.

**Data model:** `Review` (id, product_id, user_id, rating, title, content, status, created_at)

```
POST   /reviews
GET    /reviews?product_id=
```

---

#### 3.1.9. Notification Service

Gửi email/SMS/push notification khi có đơn hàng mới, xác nhận, giao hàng thành công, quên mật khẩu...

Tích hợp với dịch vụ bên ngoài (Twilio, SendGrid). Cấu trúc message template và log gửi.

---

#### 3.1.10. Analytics & Monitoring

Thu thập dữ liệu truy cập, lượt xem sản phẩm, tỷ lệ chuyển đổi. Dùng để vẽ báo cáo và dự báo tồn kho, doanh thu.

Triển khai qua Google Analytics, Matomo, hoặc hệ thống BI nội bộ.

---

### 3.2. Admin Panel

| Module | Chức năng |
|---|---|
| Dashboard | Thống kê doanh thu, đơn hàng, sản phẩm bán chạy, tồn kho thấp |
| Quản lý sản phẩm | CRUD sản phẩm, biến thể, giá, ảnh; sắp xếp danh mục |
| Quản lý đơn hàng | Xem/lọc danh sách; xem chi tiết; cập nhật trạng thái; in hóa đơn |
| Quản lý khách hàng | Thông tin, lịch sử mua, tổng chi tiêu, phân hạng |
| Khuyến mãi/Coupon | Tạo/chỉnh sửa mã giảm giá, chương trình khuyến mãi |
| Quản lý vận chuyển | Cài đặt đơn vị giao hàng, phí theo khu vực/khối lượng |
| Phân quyền | Thiết lập vai trò (admin, manager, support) và quyền truy cập |

---

## 4. Mô Hình Cơ Sở Dữ Liệu (ERD)

| Entity | Thuộc tính chính | Quan hệ |
|---|---|---|
| `User` | id (PK), email, phone, password_hash, role, status, created_at, updated_at | 1-n với Address, Order, Review |
| `UserProfile` | id (PK), user_id (FK), full_name, gender, date_of_birth, avatar_url | 1-1 với User |
| `Category` | id (PK), parent_id (FK Category), name, slug, description, image_url | 1-n với Product |
| `Product` | id (PK), category_id (FK), sku, name, slug, description, short_description, price, price_promo, currency, brand, weight, dimensions, status, created_at | 1-n với ProductImage, ProductVariant, Review; n-m với Attribute |
| `ProductImage` | id (PK), product_id (FK), url, alt_text, is_primary | n-1 với Product |
| `ProductVariant` | id (PK), product_id (FK), sku, price_override, quantity | n-1 với Product |
| `Attribute` | id (PK), name, type, options (JSON) | n-m với Product qua ProductAttributeValue |
| `ProductAttributeValue` | id (PK), product_id (FK), attribute_id (FK), value | n-1 với Product, Attribute |
| `Cart` | id (PK), customer_id (FK User), session_id, created_at, updated_at | 1-n với CartItem |
| `CartItem` | id (PK), cart_id (FK), product_id (FK), variant_id (FK), quantity, price_at_add_time | n-1 với Cart |
| `Order` | id (PK), order_number, customer_id (FK), total_amount, discount_amount, shipping_fee, tax_fee, payment_method, order_status, shipping_status, placed_at, updated_at | 1-n với OrderItem, OrderStatusHistory |
| `OrderItem` | id (PK), order_id (FK), product_id (FK), variant_id (FK), product_name_snapshot, price_snapshot, quantity | n-1 với Order |
| `Address` | id (PK), customer_id (FK), full_name, phone, street, ward, district, city, country, postal_code, type, is_default | n-1 với User; 1-n với Order |
| `InventoryItem` | id (PK), product_id (FK), variant_id (FK), warehouse_id, quantity_available, quantity_reserved, last_updated | |
| `StockMovement` | id (PK), inventory_item_id (FK), type, quantity, related_order_id, created_at | |
| `Coupon` | id (PK), code, description, discount_type, discount_value, min_order_amount, max_discount_amount, start_date, end_date, usage_limit, usage_per_user, status | |
| `Promotion` | id (PK), name, description, discount_type, discount_value, start_date, end_date, priority | |
| `Review` | id (PK), product_id (FK), user_id (FK), rating, title, content, status, created_at | n-1 với Product, User |

> **Ghi chú:** Có thể tách địa chỉ shipping/billing thành bảng riêng; tách bảng logs và lịch sử để phục vụ audit.

---

## 5. Giao Diện và Luồng Người Dùng (Frontend)

### 5.1. Các Trang Chính

| Trang | Nội dung |
|---|---|
| **Trang chủ** | Banner, danh mục nổi bật, sản phẩm mới/khuyến mãi, tin tức; thanh tìm kiếm, menu danh mục, giỏ hàng, đăng nhập; footer |
| **Danh mục / Tìm kiếm** | Danh sách sản phẩm theo danh mục/keyword; bộ lọc (giá, thương hiệu, màu sắc, size, đánh giá, thuộc tính); phân trang; sắp xếp (giá, bán chạy, mới nhất) |
| **Chi tiết sản phẩm** | Album ảnh; tên, giá gốc/khuyến mãi; mô tả; thông số kỹ thuật; biến thể (màu, size); đánh giá khách; sản phẩm liên quan; nút "Thêm vào giỏ" / "Mua ngay"; chính sách đổi trả |
| **Giỏ hàng** | Danh sách item; số lượng; giá từng dòng; tổng tạm tính; phí ship ước tính; áp dụng mã giảm giá |
| **Checkout** | Chọn/nhập địa chỉ giao hàng; chọn phương thức giao hàng và thanh toán; tóm tắt đơn (tạm tính, giảm giá, thuế, phí ship, tổng cộng); xác nhận đặt hàng |
| **Xác nhận / Trạng thái đơn** | Mã đơn, tổng tiền, trạng thái ban đầu, hướng dẫn theo dõi; lịch sử trạng thái (đóng gói, đang giao, đã giao...) |
| **Đăng ký / Đăng nhập** | Form đăng ký (email, mật khẩu, tên, SĐT); đăng nhập bằng email/phone + mật khẩu hoặc OTP |
| **Tài khoản** | Sửa thông tin; quản lý địa chỉ; lịch sử đơn hàng; đổi mật khẩu |
| **Admin Panel** | Dashboard; quản lý sản phẩm/danh mục (CRUD, import/export CSV, tồn kho); quản lý đơn hàng; quản lý khách hàng; khuyến mãi/coupon; phân quyền |

### 5.2. UX & Responsive

- **Responsive** – tối ưu mobile-first vì đa phần traffic từ điện thoại.
- **Performance** – Lazy Loading cho ảnh, CDN cache, hạn chế script nặng.
- **Usability** – nút mua rõ ràng, filter dễ dùng, hiển thị trạng thái giỏ hàng, hỗ trợ đa ngôn ngữ.
- **Transparency** – chính sách đổi trả, bảo mật, phương thức thanh toán hiển thị rõ.

---

## 6. API Reference

### 6.1. Product API

```http
GET  /api/v1/products?search=&page=1&page_size=20&category_id=&min_price=&max_price=&sort=
GET  /api/v1/products/{id}
POST /api/v1/products                        # admin
PUT  /api/v1/products/{id}                   # admin
DELETE /api/v1/products/{id}                 # admin
GET  /api/v1/categories
GET  /api/v1/categories/{id}/products
```

### 6.2. Cart & Order API

```http
# Giỏ hàng
GET    /api/v1/cart
POST   /api/v1/cart/items          # {"product_id":1, "variant_id":2, "quantity":3}
PATCH  /api/v1/cart/items/{id}     # {"quantity":5}
DELETE /api/v1/cart/items/{id}

# Đơn hàng
POST   /api/v1/orders              # {"shipping_address_id":1, "payment_method":"cod", "coupon_code":"SALE10"}
GET    /api/v1/orders/{order_id}
GET    /api/v1/orders?customer_id=me
PUT    /api/v1/orders/{id}/status  # {"status":"confirmed"}  admin
```

### 6.3. User & Auth API

```http
POST   /api/v1/auth/register       # {"email":"","password":"","phone":"","full_name":""}
POST   /api/v1/auth/login          # {"email":"","password":""}
POST   /api/v1/auth/logout
GET    /api/v1/users/me
PUT    /api/v1/users/me            # {"full_name":"","avatar_url":""}
POST   /api/v1/auth/reset-password # {"email":""}
```

### 6.4. Promotion & Review API

```http
POST   /api/v1/coupons/apply       # {"code":"SALE10"}
GET    /api/v1/promotions
POST   /api/v1/promotions          # admin

POST   /api/v1/reviews             # {"product_id":1, "rating":5, "title":"Good", "content":"..."}
GET    /api/v1/reviews?product_id=1
```

> Tất cả API trả JSON với HTTP status codes chuẩn (200, 201, 400, 401, 403, 404, 500...). Endpoint admin yêu cầu kiểm tra role (admin/manager).

---

## 7. Các Vấn Đề Cần Lưu Ý Khi Triển Khai

1. **Quản lý thuộc tính sản phẩm đa dạng** – Dùng bảng `Attribute` + `ProductAttributeValue` cho thuộc tính linh hoạt; hoặc JSONB (PostgreSQL) / document DB cho schemaless.
2. **Tính nhất quán dữ liệu** – Dùng transaction khi tạo đơn để đồng bộ đặt hàng, thanh toán, tồn kho. Với microservices, áp dụng Saga/Outbox pattern.
3. **Cache & query optimization** – Redis/Memcached cho sản phẩm thường xem, danh mục; cache giỏ hàng guest; CDN cho hình ảnh.
4. **Bảo mật & compliance** – bcrypt/argon2 cho mật khẩu; lưu token an toàn; rate limit đăng nhập; tuân thủ GDPR.
5. **Scalability** – Tách service theo domain; message queue cho async jobs (email, inventory update); load balancer + monitoring.
6. **Tái sử dụng** – API và module độc lập theo SOLID. Khi clone cho ngành hàng khác, chỉ cần cập nhật danh mục, thuộc tính, giao diện; logic đặt hàng/thanh toán giữ nguyên.

---

## 8. Lộ Trình Phát Triển

### 8.1. MVP

- [ ] CRUD danh mục và sản phẩm cơ bản
- [ ] Tìm kiếm và lọc cơ bản
- [ ] Giỏ hàng (thêm, sửa, xóa)
- [ ] Checkout: nhập thông tin nhận hàng, COD/chuyển khoản (không cần cổng thanh toán)
- [ ] Quản lý đơn hàng đơn giản (chờ xác nhận → đang giao → hoàn thành/hủy)
- [ ] Admin panel cơ bản: quản lý sản phẩm và đơn hàng

### 8.2. Mở Rộng

- [ ] Thanh toán online: VNPay, MoMo, ZaloPay, Stripe, PayPal
- [ ] Tài khoản khách hàng: đăng ký/đăng nhập, lịch sử mua, wishlist, đánh giá
- [ ] Khuyến mãi nâng cao: flash sale, voucher theo phân khúc, điểm thưởng
- [ ] Vận chuyển tự động: tích hợp API hãng vận chuyển (tính phí, tạo vận đơn)
- [ ] Quản lý kho đa kênh: sync tồn kho từ nhiều kho/chi nhánh
- [ ] Analytics & AI: dự đoán nhu cầu, gợi ý sản phẩm, tối ưu tồn kho

---

## 9. Kết Luận

Bản thiết kế trên cung cấp khung kiến trúc, mô hình dữ liệu và giao diện cho một website bán hàng chuẩn, độc lập với công nghệ cụ thể. Các thành phần tách rõ theo mô-đun, dễ mở rộng và tái sử dụng cho nhiều ngành hàng.

Khi triển khai thực tế, cần đảm bảo:
- Quản lý sản phẩm, đơn hàng, thanh toán và bảo mật đúng chuẩn.
- Có lộ trình phát triển rõ ràng để thêm tính năng nâng cao theo giai đoạn.

---

*Tham khảo: [E-commerce Architecture - GeeksforGeeks](https://www.geeksforgeeks.org/system-design/e-commerce-architecture-system-design-for-e-commerce-website/) | [System Design for E-Commerce Platform - Medium](https://medium.com/@prasanta-paul/system-design-for-e-commerce-platform-3048047b5323)*