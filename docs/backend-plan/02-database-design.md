# 02 — Thiết Kế Cơ Sở Dữ Liệu & Migration

> Dựa trên ERD trong [system-design.md §4](../main/system-design.md), bổ sung phần đặc thù điện tử (Attribute Template, Brand, Warehouse) và xử lý đồng thời.

---

## 1. Nguyên tắc thiết kế

- **Khoá chính:** `BIGINT identity` (hoặc UUID cho bảng phân tán). MVP dùng `BIGINT`.
- **Audit:** mọi bảng nghiệp vụ có `created_at, updated_at, created_by, updated_by` (qua `BaseEntity` + JPA Auditing).
- **Soft delete:** sản phẩm/category dùng cột `status` thay vì xoá cứng (giữ lịch sử đơn).
- **Snapshot:** `OrderItem` lưu `product_name_snapshot`, `price_snapshot` để bất biến theo thời điểm mua.
- **Tiền tệ:** `NUMERIC(15,2)` + cột `currency` (mặc định `VND`). Không dùng `float`.
- **JSONB:** dùng cho `variant.attribute_values`, `attribute.options` (linh hoạt nhưng có kiểm soát).
- **Index:** index FK, cột lọc (status, category_id, brand_id, price), unique (sku, slug, coupon.code, user.email).

---

## 2. Nhóm bảng theo module

### 2.1. IAM (Identity & Access)
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `users` | id, email (uniq), phone, password_hash, role, status, created_at | role: CUSTOMER/ADMIN/MANAGER/SUPPORT |
| `user_profiles` | id, user_id (FK 1-1), full_name, gender, dob, avatar_url | |
| `addresses` | id, user_id (FK), full_name, phone, street, ward, district, city, country, postal_code, type, is_default | type: SHIPPING/BILLING |
| `refresh_tokens` | (lưu Redis) jti, user_id, expires_at | có thể dùng Redis thay bảng |

### 2.2. Catalog
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `categories` | id, parent_id (FK self), name, slug (uniq), description, image_url, status | nested set/adjacency list |
| `brands` | id, name, slug (uniq), logo_url, status | thêm so với spec gốc (điện tử cần brand) |
| `products` | id, category_id (FK), brand_id (FK), sku (uniq), name, slug (uniq), short_description, description, base_price, sale_price, currency, weight, status, created_at | |
| `product_images` | id, product_id (FK), url, alt_text, is_primary, sort_order | |
| `product_variants` | id, product_id (FK), sku (uniq), attribute_values (JSONB), price_override, status | tồn kho ở inventory |

### 2.3. Attribute / Spec (đặc thù điện tử)
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `attribute_groups` | id, name, sort_order | vd: Bộ xử lý, Bộ nhớ, Màn hình |
| `attribute_definitions` | id, group_id (FK), code, name, data_type, unit, is_filterable, is_comparable, is_quick_spec, sort_order | data_type: TEXT/SELECT/NUMBER/COLOR/BOOLEAN |
| `attribute_options` | id, attribute_id (FK), value, label | cho type SELECT |
| `attribute_templates` | id, name, status | vd: Laptop Template |
| `template_attributes` | id, template_id (FK), attribute_id (FK), sort_order | n-n template↔attribute |
| `category_attribute_template` | id, category_id (FK), template_id (FK) | gán template cho danh mục |
| `product_attribute_values` | id, product_id (FK), attribute_id (FK), value, value_number | n-1 product, attribute |

> `value_number` để lọc range (vd screen size 13–17"). `value` để hiển thị.

### 2.4. Inventory
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `warehouses` | id, name, address | MVP có thể 1 kho mặc định |
| `inventory_items` | id, product_id, variant_id, warehouse_id, quantity_available, quantity_reserved, low_stock_threshold, **version**, last_updated | `version` cho optimistic lock |
| `stock_movements` | id, inventory_item_id (FK), type, quantity, related_order_id, note, created_at | type: IMPORT/EXPORT/RESERVE/RELEASE/ADJUST |

### 2.5. Cart
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `carts` | id, customer_id (nullable), session_id, created_at, updated_at | user → DB; guest → Redis (TTL) |
| `cart_items` | id, cart_id (FK), product_id, variant_id, quantity, price_at_add_time | |

### 2.6. Promotion & Pricing
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `coupons` | id, code (uniq), description, discount_type, discount_value, min_order_amount, max_discount_amount, start_date, end_date, usage_limit, usage_per_user, status | |
| `coupon_redemptions` | id, coupon_id (FK), user_id, order_id, redeemed_at | enforce usage_per_user |
| `promotions` | id, name, description, discount_type, discount_value, start_date, end_date, priority, status | |
| `promotion_products` | id, promotion_id (FK), product_id (FK) | áp dụng cho sản phẩm |

### 2.7. Order
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `orders` | id, order_number (uniq), customer_id, subtotal, discount_amount, shipping_fee, tax_fee, total_amount, payment_method, payment_status, order_status, shipping_status, shipping_provider, shipping_tracking_code, shipping_address_id, billing_address_id, note, placed_at, updated_at | |
| `order_items` | id, order_id (FK), product_id, variant_id, product_name_snapshot, sku_snapshot, price_snapshot, quantity | |
| `order_status_history` | id, order_id (FK), status, changed_by, changed_at, note | audit trạng thái |

### 2.8. Payment
| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `payments` | id, order_id (FK), method, amount, status, transaction_ref, gateway_payload (JSONB), created_at, paid_at | method: COD/BANK_TRANSFER/VNPAY/MOMO |
| `outbox_events` | id, aggregate_type, aggregate_id, event_type, payload (JSONB), status, created_at, processed_at | Outbox pattern |

### 2.9. Review (Phase 2)
| Bảng | Cột chính |
|---|---|
| `reviews` | id, product_id, user_id, order_id, rating, title, content, status, created_at |

---

## 3. Chiến lược Migration (Flyway)

- Mọi thay đổi schema qua file `src/main/resources/db/migration/V<timestamp>__<mo_ta>.sql`.
- **Quy tắc bất biến:** migration đã merge vào `develop` **không được sửa** — chỉ thêm migration mới.
- Đặt tên: `V202606231000__create_users.sql`.
- Seed data tách riêng (`V...__seed_roles.sql`, `V...__seed_laptop_attribute_template.sql`) hoặc dùng `R__` (repeatable) cho seed reference.
- Mỗi PR đổi schema phải kèm migration + cập nhật entity + test.
- Thứ tự tạo bảng theo phụ thuộc FK (users → categories/brands → products → variants → inventory → cart → order → payment).

---

## 4. Xử lý đồng thời tồn kho (chống oversell) — trọng yếu

Luồng đặt hàng phải đảm bảo không bán vượt tồn. Áp dụng kết hợp:

1. **Reserve khi tạo đơn** (không trừ `quantity_available` ngay khi thêm giỏ):
   ```
   UPDATE inventory_items
   SET quantity_available = quantity_available - :qty,
       quantity_reserved  = quantity_reserved  + :qty,
       version = version + 1
   WHERE id = :id AND quantity_available >= :qty;
   ```
   Nếu `affected rows = 0` → ném `InsufficientStockException`.
2. **Optimistic locking** bằng `@Version` ở entity; xung đột → retry có giới hạn (vd 3 lần) hoặc fallback pessimistic lock (`SELECT ... FOR UPDATE`) cho sản phẩm hot.
3. **Ghi `StockMovement`** mỗi thao tác (RESERVE/RELEASE/EXPORT) — không bao giờ sửa tồn kho mà thiếu movement log.
4. **Release** khi: huỷ đơn, thanh toán thất bại, hết hạn giữ đơn (job quét).
5. Toàn bộ trong **một transaction** với tạo Order; thất bại bất kỳ bước nào → rollback toàn bộ.

> Test bắt buộc: mô phỏng N request đồng thời mua sản phẩm còn 1 cái → chỉ 1 đơn thành công ([ECM-044], [ECM-120]).

---

## 5. Index & hiệu năng gợi ý

| Bảng | Index |
|---|---|
| products | `(status, category_id)`, `(brand_id)`, `(slug)` uniq, GIN trên tsvector(name+desc) |
| product_attribute_values | `(attribute_id, value_number)`, `(product_id)` |
| inventory_items | `(product_id, variant_id, warehouse_id)` uniq |
| orders | `(customer_id, placed_at)`, `(order_status)`, `(order_number)` uniq |
| coupons | `(code)` uniq, `(status, end_date)` |

---

## 6. Sơ đồ quan hệ rút gọn

```
User 1─n Address
User 1─1 UserProfile
User 1─n Order ─n OrderItem ─→ Product / Variant (snapshot)
Category 1─n Product n─1 Brand
Product 1─n ProductImage
Product 1─n ProductVariant
Product n─n Attribute (qua ProductAttributeValue)
Category n─n AttributeTemplate (qua category_attribute_template)
AttributeTemplate n─n Attribute (qua template_attributes)
Product/Variant 1─n InventoryItem 1─n StockMovement
Cart 1─n CartItem
Order 1─n Payment
Order 1─n OrderStatusHistory
Coupon 1─n CouponRedemption
```
