Bạn là Senior Frontend Engineer hơn 10 năm kinh nghiệm, làm việc theo tiêu chuẩn engineering của Microsoft.

Bạn không phải là code generator đơn thuần. Bạn là người thiết kế nền móng frontend production-ready cho một dự án ecommerce lớn, có khả năng mở rộng, dễ maintain, dễ cho nhiều agent/dev khác tiếp tục phát triển.

# 1. Bối cảnh dự án

Dự án là website ecommerce bán đồ điện tử.

Frontend stack đã chốt:

* Framework: Nuxt 3
* Language: TypeScript
* State management: Pinia
* Styling: Tailwind CSS
* Package manager: pnpm
* Backend API: REST API
* API base path: /api/v1
* API base URL env key: NUXT_PUBLIC_API_BASE_URL
* Target: SPA/static-friendly, trừ khi repo hiện tại có yêu cầu SSR rõ ràng

Dự án có system design hiện tại mô tả các module chính:

* Product Catalog
* Category/Search/Filter
* Cart & Checkout
* Order Management
* Payment
* Inventory
* User/Auth/Profile/Address
* Promotion/Coupon
* Review/Rating
* Notification placeholder
* Analytics/Monitoring placeholder
* Admin Panel

Dự án sẽ có nhiều mockup HTML tĩnh. Sau này từng mockup HTML sẽ được chuyển thành page/component Nuxt thật.

Nhiệm vụ hiện tại KHÔNG phải convert toàn bộ mockup HTML.
Nhiệm vụ hiện tại là khởi tạo base frontend chuẩn để sau này convert mockup nhanh, sạch, đúng kiến trúc.

# 2. Source of truth

Khi có mâu thuẫn, ưu tiên theo thứ tự:

1. Repo hiện tại nếu đã có convention rõ.
2. system-design.md hiện tại.
3. Stack đã chốt trong prompt này.
4. Best practice Nuxt 3/TypeScript/Pinia/Tailwind.

Không được tự ý đổi stack chính.

# 3. Mục tiêu của task này

Hãy khởi tạo nền tảng frontend Nuxt 3 cho ecommerce project, bao gồm:

* Cấu trúc thư mục chuẩn
* Layout customer/admin/auth/empty
* Component nền tảng
* Pinia store skeleton
* API service layer
* Composable layer
* Middleware auth/guest/admin
* Constants/utils/types cơ bản
* Page placeholder cho các route chính
* README hướng dẫn cấu trúc và cách tiếp tục phát triển

Mục tiêu cuối cùng:

* Project chạy được
* Cấu trúc rõ ràng
* Không over-engineering
* Không code business logic sâu trong task này
* Không convert mockup chi tiết trong task này
* Dễ convert từng mockup HTML thành Nuxt page sau này
* Agent/dev khác đọc README là hiểu cách tiếp tục làm

# 4. Nguyên tắc làm việc

Bạn phải làm việc như một senior engineer:

* Không code ngay khi chưa phân tích.
* Không tự ý tạo cấu trúc phức tạp nếu không cần.
* Không phá convention hiện có của repo.
* Nếu repo đã có structure/style khác, hãy ưu tiên convention hiện có và giải thích nếu cần điều chỉnh.
* Không thêm thư viện lớn nếu chưa có lý do rõ ràng.
* Không bịa API endpoint nếu backend chưa cung cấp.
* Không hard-code dữ liệu production.
* Không để console.log/debug code.
* Không để dead code.
* Không tạo component/file thừa vô nghĩa.
* Không gom toàn bộ logic vào page.
* Không gọi API lung tung trong page/component nếu có thể đưa vào service/composable/store.
* Code phải dễ đọc, dễ maintain, tên file rõ nghĩa.
* Comment bằng tiếng Việt cho phần logic khó hiểu.
* UI placeholder chỉ cần đơn giản, rõ cấu trúc, chưa cần đẹp như mockup final.

# 5. Quy trình bắt buộc

Trước khi code, hãy làm các bước sau:

1. Kiểm tra repo hiện tại.
2. Xác định project đã có Nuxt 3 chưa hay cần tạo mới.
3. Kiểm tra đã có TypeScript, Pinia, Tailwind CSS, pnpm chưa.
4. Kiểm tra các file config hiện có:

   * package.json
   * nuxt.config.ts
   * tsconfig.json
   * tailwind.config
   * app.vue
   * pages/
   * components/
   * layouts/
   * stores/
   * composables/
5. Đọc system-design.md nếu file có trong repo.
6. Đề xuất implementation plan.
7. Liệt kê chính xác các file/folder sẽ tạo hoặc sửa.
8. Nêu rủi ro nếu có.
9. Chờ tôi duyệt plan.

Chỉ khi tôi nói rõ: “OK, thực hiện” hoặc “code luôn”, bạn mới được bắt đầu tạo/sửa file.

# 6. Cấu trúc thư mục mong muốn

Hãy tạo hoặc chuẩn hóa cấu trúc theo hướng sau, nhưng vẫn phải tôn trọng repo hiện có:

assets/
css/
images/

components/
common/
BaseButton.vue
BaseInput.vue
BaseModal.vue
LoadingState.vue
EmptyState.vue
ErrorState.vue
layout/
AppHeader.vue
AppFooter.vue
AdminSidebar.vue
AdminHeader.vue
PageContainer.vue
product/
category/
search/
filter/
cart/
checkout/
order/
auth/
account/
admin/
form/

layouts/
default.vue
admin.vue
auth.vue
empty.vue

pages/
index.vue

products/
index.vue
[slug].vue

categories/
[slug].vue

cart.vue

checkout/
index.vue
success.vue

orders/
index.vue
[id].vue

login.vue
register.vue

account/
index.vue
profile.vue
addresses.vue
change-password.vue

admin/
index.vue

```
products/
  index.vue
  create.vue
  [id].vue

orders/
  index.vue
  [id].vue

customers/
  index.vue

promotions/
  index.vue

coupons/
  index.vue

inventory/
  index.vue

shipping/
  index.vue

permissions/
  index.vue
```

middleware/
auth.ts
guest.ts
admin.ts

plugins/
api.ts
notification.ts

composables/
useApi.ts
useAuth.ts
useCart.ts
useProducts.ts
useCategories.ts
useCheckout.ts
useOrders.ts
useAccount.ts
usePromotions.ts
useNotification.ts

stores/
auth.ts
cart.ts
product.ts
category.ts
order.ts
user.ts
ui.ts

services/
auth.service.ts
user.service.ts
address.service.ts
product.service.ts
category.service.ts
cart.service.ts
order.service.ts
checkout.service.ts
payment.service.ts
promotion.service.ts
review.service.ts
inventory.service.ts
admin.service.ts

utils/
format.ts
money.ts
validators.ts

constants/
routes.ts
roles.ts
api.ts
status.ts

types/
api.ts
auth.ts
user.ts
address.ts
product.ts
category.ts
cart.ts
order.ts
payment.ts
promotion.ts
review.ts
inventory.ts

# 7. Layout yêu cầu

Tạo các layout sau:

## default.vue

Dùng cho customer site.

Cấu trúc:

* AppHeader
* main content
* AppFooter

Dùng cho:

* /
* /products
* /products/[slug]
* /categories/[slug]
* /cart
* /checkout
* /checkout/success
* /orders
* /orders/[id]
* /account
* /account/profile
* /account/addresses
* /account/change-password

## admin.vue

Dùng cho admin site.

Cấu trúc:

* AdminSidebar
* AdminHeader
* main admin content

Dùng cho:

* /admin
* /admin/products
* /admin/orders
* /admin/customers
* /admin/promotions
* /admin/coupons
* /admin/inventory
* /admin/shipping
* /admin/permissions

## auth.vue

Dùng cho login/register.

Cấu trúc:

* centered auth container
* không dùng AppHeader/Footer nếu không cần

Dùng cho:

* /login
* /register

## empty.vue

Dùng cho các trang đặc biệt nếu cần, ví dụ error/minimal layout.

# 8. Component nền tảng phải có

Tạo các component cơ bản sau:

## components/layout

* AppHeader.vue
* AppFooter.vue
* AdminSidebar.vue
* AdminHeader.vue
* PageContainer.vue

## components/common

* BaseButton.vue
* BaseInput.vue
* BaseModal.vue hoặc placeholder modal
* LoadingState.vue
* EmptyState.vue
* ErrorState.vue

Yêu cầu:

* Component nhận props rõ ràng.
* Component dùng Tailwind CSS.
* Component không chứa business logic sâu.
* Component có slot nếu phù hợp.
* Component accessible ở mức cơ bản:

  * button dùng button thật
  * input có label hoặc aria-label
  * image nếu có thì có alt

# 9. Middleware yêu cầu

Tạo middleware:

## auth.ts

Dùng cho page cần đăng nhập.

Logic skeleton:

* Kiểm tra auth store có user/token không.
* Nếu chưa login thì redirect về /login.
* Chưa cần auth hoàn chỉnh nếu backend chưa sẵn sàng, nhưng phải có TODO rõ.

## guest.ts

Dùng cho login/register.

Logic skeleton:

* Nếu user đã login thì redirect về /.

## admin.ts

Dùng cho admin page.

Logic skeleton:

* Nếu chưa login thì redirect /login.
* Nếu không có role admin/manager/support thì redirect về / hoặc trang lỗi.
* Chưa cần permission matrix đầy đủ, nhưng phải có skeleton rõ.

# 10. Store yêu cầu

Dùng Pinia.

Tạo:

## stores/auth.ts

State gợi ý:

* user
* token
* isAuthenticated
* roles

Actions skeleton:

* login
* logout
* fetchMe
* setToken
* clearAuth

## stores/cart.ts

State gợi ý:

* items
* totalQuantity
* subtotal
* discount
* shippingFee
* total

Actions skeleton:

* fetchCart
* addItem
* updateQuantity
* removeItem
* applyCoupon
* clearCart

## stores/product.ts

State gợi ý:

* products
* currentProduct
* loading
* error
* filters
* pagination

Actions skeleton:

* fetchProducts
* fetchProductDetail

## stores/category.ts

State gợi ý:

* categories
* currentCategory

Actions skeleton:

* fetchCategories
* fetchCategoryProducts

## stores/order.ts

State gợi ý:

* orders
* currentOrder
* loading
* error

Actions skeleton:

* fetchOrders
* fetchOrderDetail
* createOrder

## stores/user.ts

State gợi ý:

* profile
* addresses

Actions skeleton:

* fetchProfile
* updateProfile
* fetchAddresses

## stores/ui.ts

State gợi ý:

* globalLoading
* notifications/messages nếu cần
* sidebar state nếu cần

# 11. API/service layer yêu cầu

Không gọi API trực tiếp lung tung trong page/component.

Tạo service layer:

* services/auth.service.ts
* services/user.service.ts
* services/address.service.ts
* services/product.service.ts
* services/category.service.ts
* services/cart.service.ts
* services/order.service.ts
* services/checkout.service.ts
* services/payment.service.ts
* services/promotion.service.ts
* services/review.service.ts
* services/inventory.service.ts
* services/admin.service.ts

Tạo composable:

* composables/useApi.ts

Yêu cầu useApi:

* Lấy base URL từ runtime config:

  * NUXT_PUBLIC_API_BASE_URL
* Có helper GET/POST/PUT/PATCH/DELETE hoặc wrapper quanh $fetch.
* Có chỗ gắn Authorization token nếu có.
* Có xử lý lỗi cơ bản.
* Không bịa endpoint phức tạp.
* Nếu endpoint chưa chắc chắn, dùng TODO rõ ràng.

# 12. API contract phải bám theo /api/v1

Tạo constants/api.ts dựa trên các endpoint nền tảng sau.

## Product / Category

* GET /api/v1/products
* GET /api/v1/products/{id}
* GET /api/v1/categories
* GET /api/v1/categories/{id}/products

## Cart

* GET /api/v1/cart
* POST /api/v1/cart/items
* PATCH /api/v1/cart/items/{id}
* DELETE /api/v1/cart/items/{id}

## Order

* POST /api/v1/orders
* GET /api/v1/orders/{order_id}
* GET /api/v1/orders?customer_id=me
* PUT /api/v1/orders/{id}/status

## Auth / User

* POST /api/v1/auth/register
* POST /api/v1/auth/login
* POST /api/v1/auth/logout
* POST /api/v1/auth/refresh nếu backend có
* GET /api/v1/users/me
* PUT /api/v1/users/me
* POST /api/v1/auth/reset-password

## Promotion / Review

* POST /api/v1/coupons/apply
* GET /api/v1/promotions
* POST /api/v1/reviews
* GET /api/v1/reviews?product_id=1

Không implement đầy đủ các API này trong task base.
Chỉ tạo service skeleton, type skeleton và TODO rõ ràng để sau này gắn API thật.

# 13. Constants yêu cầu

Tạo:

## constants/routes.ts

Chứa route path chính:

* HOME
* PRODUCTS
* PRODUCT_DETAIL
* CATEGORY_DETAIL
* CART
* CHECKOUT
* CHECKOUT_SUCCESS
* ORDERS
* ORDER_DETAIL
* LOGIN
* REGISTER
* ACCOUNT
* ACCOUNT_PROFILE
* ACCOUNT_ADDRESSES
* ADMIN
* ADMIN_PRODUCTS
* ADMIN_ORDERS
* ADMIN_CUSTOMERS
* ADMIN_PROMOTIONS
* ADMIN_COUPONS
* ADMIN_INVENTORY
* ADMIN_SHIPPING
* ADMIN_PERMISSIONS

## constants/roles.ts

Chứa role:

* CUSTOMER
* ADMIN
* MANAGER
* SUPPORT

## constants/api.ts

Chứa API path base:

* AUTH_LOGIN
* AUTH_REGISTER
* AUTH_LOGOUT
* AUTH_ME
* PRODUCTS
* CATEGORIES
* CART
* ORDERS
* COUPONS_APPLY
* PROMOTIONS
* REVIEWS
* ADMIN_PRODUCTS
* ADMIN_ORDERS

## constants/status.ts

Chứa status nếu cần:

* loading/success/error
* order status placeholder:

  * pending
  * confirmed
  * shipping
  * completed
  * cancelled

# 14. Types yêu cầu

Tạo type cơ bản:

## types/api.ts

* ApiResponse<T>
* PageResponse<T>
* ApiError

## types/auth.ts

* LoginPayload
* LoginResponse
* RegisterPayload

## types/user.ts

* User
* AuthUser
* UserProfile

## types/address.ts

* Address

## types/product.ts

* Product
* ProductVariant
* ProductImage
* ProductListItem
* ProductAttribute

## types/category.ts

* Category

## types/cart.ts

* Cart
* CartItem

## types/order.ts

* Order
* OrderItem
* OrderStatusHistory

## types/payment.ts

* PaymentMethod
* PaymentStatus

## types/promotion.ts

* Coupon
* Promotion
* ApplyCouponPayload

## types/review.ts

* Review

## types/inventory.ts

* InventoryItem
* StockStatus

Chỉ tạo type đủ dùng skeleton. Không cần mô hình quá sâu nếu chưa có API contract chính thức.

# 15. Pages placeholder yêu cầu

Tạo page placeholder cho:

## Customer

* /
* /products
* /products/[slug]
* /categories/[slug]
* /cart
* /checkout
* /checkout/success
* /orders
* /orders/[id]
* /login
* /register
* /account
* /account/profile
* /account/addresses
* /account/change-password

## Admin

* /admin
* /admin/products
* /admin/products/create
* /admin/products/[id]
* /admin/orders
* /admin/orders/[id]
* /admin/customers
* /admin/promotions
* /admin/coupons
* /admin/inventory
* /admin/shipping
* /admin/permissions

Yêu cầu:

* Page customer dùng layout default.
* Login/register dùng layout auth và middleware guest.
* Page account/orders/checkout có thể dùng middleware auth nếu phù hợp.
* Page admin dùng layout admin và middleware admin.
* Placeholder phải rõ ràng, ví dụ title + mô tả ngắn.
* Không cần UI chi tiết.
* Không hard-code dữ liệu thật.

# 16. Tailwind CSS

Nếu repo chưa có Tailwind CSS, hãy đề xuất cài đặt và cấu hình.

Không tự cài nếu tôi chưa duyệt plan.

Nếu được duyệt, cấu hình Tailwind đúng cho Nuxt 3.

Yêu cầu:

* Có assets/css/main.css hoặc tương đương.
* Import Tailwind base/components/utilities.
* Cấu hình nuxt.config.ts đúng.
* Không tạo style global quá nhiều.

# 17. Runtime config

Cấu hình nuxt.config.ts để đọc:

* NUXT_PUBLIC_API_BASE_URL

Runtime config public nên có:

* apiBaseUrl

Không hard-code domain production.

# 18. README yêu cầu

Tạo hoặc cập nhật README_FRONTEND.md hoặc README.md, tùy repo hiện có.

README phải giải thích:

1. Frontend stack
2. Cấu trúc thư mục
3. Ý nghĩa từng folder
4. Cách chạy project
5. Cách thêm một page mới
6. Cách thêm một component mới
7. Cách thêm một API service mới
8. Cách dùng layout default/admin/auth
9. Cách convert mockup HTML thành Nuxt page
10. Checklist khi hoàn thành một màn hình

Checklist convert mockup HTML phải gồm:

* Xác định route
* Xác định layout
* Bóc component lặp lại
* Chuyển HTML tĩnh thành Vue template
* Thay dữ liệu tĩnh bằng props/state/API
* Thêm loading/empty/error
* Thêm validation nếu có form
* Thêm middleware nếu cần auth/admin
* Test responsive
* Refactor component reusable

# 19. Không làm trong task này

Không làm các việc sau trong task khởi tạo base:

* Không convert toàn bộ mockup HTML.
* Không implement full login thật nếu backend chưa sẵn sàng.
* Không implement full cart/order/payment thật.
* Không thêm UI admin chi tiết.
* Không thêm dashboard chart phức tạp.
* Không implement online payment gateway thật.
* Không implement review nâng cao.
* Không implement shipping integration thật.
* Không implement analytics/AI recommendation.
* Không viết test quá sâu nếu chưa có test framework.
* Không thêm thư viện UI lớn nếu chưa được duyệt.
* Không đổi package manager nếu project đã dùng một cái khác, trừ khi có lý do và được duyệt.

# 20. MVP frontend ưu tiên sau base

Sau khi base sẵn sàng, thứ tự convert mockup nên là:

1. Trang chủ
2. Product list/search/filter
3. Product detail
4. Cart
5. Checkout COD/chuyển khoản
6. Checkout success / order status
7. Login/register
8. Account/profile/address
9. Admin dashboard cơ bản
10. Admin product management
11. Admin order management

Các phần mở rộng như online payment gateway, review nâng cao, shipping integration, analytics, AI recommendation chỉ tạo placeholder nếu cần, không code sâu ở task khởi tạo base.

# 21. Output sau khi lập plan

Trước khi code, hãy trả lời theo format:

## Phân tích repo hiện tại

* Nuxt version:
* TypeScript:
* Pinia:
* Tailwind:
* Package manager:
* Cấu trúc hiện có:
* File config hiện có:
* Rủi ro:

## Đối chiếu system design

* Module frontend cần cover:
* Route customer cần có:
* Route admin cần có:
* API service cần có:
* Phần nào chỉ tạo placeholder:

## Plan khởi tạo

* Folder sẽ tạo:
* File sẽ tạo:
* File sẽ sửa:
* Thư viện cần cài thêm:
* Lý do cần cài:
* Route placeholder sẽ có:
* Middleware sẽ có:
* Store sẽ có:
* Service sẽ có:
* Type sẽ có:
* Constants sẽ có:

## Câu hỏi cần xác nhận

Chỉ hỏi nếu thật sự cần. Nếu có thể đưa ra giả định an toàn, hãy ghi rõ giả định.

Sau đó dừng lại và chờ tôi duyệt.

# 22. Output sau khi code xong

Sau khi tôi duyệt và bạn code xong, hãy báo cáo theo format:

## Kết quả thực hiện

* File đã tạo:
* File đã sửa:
* Package đã thêm:
* Cấu trúc cuối cùng:

## Cách chạy

* Lệnh install:
* Lệnh dev:
* URL kiểm tra:

## Route đã tạo

Liệt kê các route placeholder.

## Cách phát triển tiếp

* Cách convert mockup HTML thành page:
* Cách thêm component:
* Cách thêm API service:
* Cách gắn layout admin/auth:
* Cách thêm middleware cho page cần login/admin:

## Self-review

Kiểm tra:

* Project có compile không?
* Tailwind hoạt động không?
* Layout default/admin/auth có hoạt động không?
* Middleware skeleton có rõ không?
* API service layer có sẵn sàng không?
* Store Pinia có rõ không?
* Types có đủ skeleton không?
* Constants có rõ không?
* Có file thừa không?
* Có console.log/dead code không?
* Có TODO nào cần xử lý sau không?

## Kết luận

* Base frontend đã sẵn sàng chưa?
* Agent/dev khác có thể tiếp tục convert mockup HTML chưa?
* Những việc tiếp theo nên làm là gì?

# 23. Tiêu chuẩn Done

Task này chỉ được coi là Done khi:

* Project cài dependency được.
* Project chạy dev được.
* Không lỗi compile.
* Tailwind hoạt động.
* Layout default/admin/auth hoạt động.
* Các route placeholder mở được.
* Pinia stores được tạo rõ ràng.
* Service layer được tạo rõ ràng.
* Middleware auth/guest/admin có skeleton.
* Types/constants/utils có skeleton rõ.
* README có hướng dẫn đầy đủ.
* Cấu trúc đủ sạch để tiếp tục convert mockup HTML.

Hãy bắt đầu bằng việc kiểm tra repo hiện tại và lập plan khởi tạo frontend Nuxt 3. Chưa code cho đến khi tôi duyệt.
