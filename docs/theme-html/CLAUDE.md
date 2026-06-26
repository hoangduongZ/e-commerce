# Rules tạo HTML Mockup — Electronics Store

> File này là **luật bắt buộc** cho mọi agent khi tạo/sửa file `.html` trong thư mục `docs/theme-html/`.
> Mục tiêu: mọi mockup sinh ra phải **đồng nhất tuyệt đối** với nhau và bám sát design system, để khách/dev nhìn vào thấy như cùng một website.

---

## 0. Đọc gì trước khi code (BẮT BUỘC)

Trước khi tạo bất kỳ trang nào, đọc theo đúng thứ tự ưu tiên. Khi xung đột, file đứng sau override file đứng trước **chỉ về màu/layout/nội dung ngành**, còn rule core (accessibility, responsive, spacing, state) thì file gốc thắng.

```text
1. docs/main/ecommerce_design_language.md   ← design system gốc (token, component, state)
2. docs/theme/01-electronics-store-theme.md ← lớp theme điện tử (màu, density, product info)
3. docs/theme/NN-<tên-trang>.md             ← spec CỤ THỂ của trang đang làm
4. Một file .html đã có trong thư mục này   ← để copy header/nav/footer + style nền
```

**Không bao giờ** code trang mới mà chưa đọc spec markdown tương ứng trong `docs/theme/`. Spec là nguồn sự thật về section order, layout desktop/mobile, state bắt buộc, acceptance criteria.

Khi thiếu spec nhỏ → tự quyết theo design system. Khi thiếu spec ảnh hưởng lớn → ghi `<!-- ASSUMPTION: ... -->` ngay tại chỗ và báo lại trong phần tổng kết.

---

## 1. Tech stack — KHÓA CỨNG, không đổi

Mỗi mockup là **một file `.html` self-contained duy nhất** (không tách CSS/JS ra file riêng, không build step). Dùng đúng boilerplate sau cho `<head>`:

```html
<!DOCTYPE html>
<html lang="vi" data-theme="electronics">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title><!-- Title SEO của trang --></title>
<meta name="description" content="<!-- mô tả ngắn --></meta">
<script src="https://cdn.tailwindcss.com"></script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
<script>tailwind.config = { theme: { extend: { fontFamily: { sans: ['Inter','system-ui','sans-serif'] } } } }</script>
<style>
  body { font-family:'Inter',system-ui,sans-serif; background:#f8fafc; }
  .line-clamp-1 { display:-webkit-box;-webkit-line-clamp:1;-webkit-box-orient:vertical;overflow:hidden; }
  .line-clamp-2 { display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden; }

  /* Product card — chuẩn dùng chung mọi trang */
  .product-card { transition:all .18s ease; }
  .product-card:hover { box-shadow:0 8px 28px rgba(0,0,0,.13); border-color:#93c5fd !important; transform:translateY(-2px); }

  /* Màu giá — KHÔNG hard-code đỏ rải rác, dùng các class này */
  .price-s { color:#dc2626; }                              /* giá bán / giá sale */
  .price-o { color:#94a3b8; text-decoration:line-through; } /* giá gốc */

  /* Skeleton loading */
  .skeleton { background:linear-gradient(90deg,#f1f5f9 25%,#e2e8f0 50%,#f1f5f9 75%); background-size:200% 100%; animation:shimmer 1.4s infinite; }
  @keyframes shimmer { 0%{background-position:200% 0} 100%{background-position:-200% 0} }
</style>
</head>
<body class="text-slate-900 antialiased">
```

Quy tắc:
- **Bắt buộc** `lang="vi"` và `data-theme="electronics"` trên `<html>`.
- Tailwind chỉ qua CDN, font chỉ là **Inter**. Không thêm framework/CSS lib khác.
- `<style>` chỉ chứa cái Tailwind utility không làm được: line-clamp, hover phức tạp, skeleton, drawer transform, countdown... Thêm style riêng của trang vào cuối block này, mỗi nhóm có comment.
- JS vanilla (không jQuery/thư viện) đặt trong `<script>` cuối `<body>`, chỉ cho tương tác demo (countdown, mở drawer, toggle accordion). Mỗi IIFE phải `if (!el) return;` để không vỡ khi thiếu element.

---

## 2. Bảng màu — map Token → class Tailwind (PHẦN QUAN TRỌNG NHẤT)

Toàn bộ palette electronics map gần như khớp tuyệt đối với Tailwind. **Phải dùng đúng các họ màu này, không tự chọn họ khác** (đây là thứ quyết định "chuẩn xác"):

| Vai trò | Token design | Class Tailwind | Hex |
|---|---|---|---|
| Brand / CTA chính | brand-600 | `blue-600` | #2563eb |
| Brand hover | brand-700 | `blue-700` | #1d4ed8 |
| Brand border nhẹ (card hover) | brand-300 | `blue-300` | #93c5fd |
| Brand nền nhẹ | brand-50 | `blue-50` | #eff6ff |
| Nền trang | neutral-50 | `slate-50` / `#f8fafc` | #f8fafc |
| Nền block phụ | neutral-100 | `slate-100` | #f1f5f9 |
| Border mặc định | neutral-200 | `slate-200` | #e2e8f0 |
| Placeholder/giá gạch | neutral-400 | `slate-400` | #94a3b8 |
| Text phụ | neutral-500 | `slate-500` | #64748b |
| Text thường | neutral-700 | `slate-700` | #334155 |
| Text chính / heading | neutral-900 | `slate-900` | #0f172a |
| Header/Footer/Nav nền tối | navy-900 | `bg-[#0f172a]` | #0f172a |
| Nav phụ tối | navy-800 | `bg-[#1e293b]` | #1e293b |
| Giá bán / sale | sale-600 | `.price-s` / `red-600` | #dc2626 |
| Badge sale | sale-500/600 | `bg-red-500` | #ef4444 |
| Trả góp / promo | promo-500 | `orange-500` | #f97316 |
| Còn hàng | stock-in | `green-600` | #16a34a |
| Sắp hết hàng | stock-low | `orange-600`/`amber-600` | #d97706 |
| Hết hàng | stock-out | `slate-400`/`slate-500` | #94a3b8 |
| Lỗi / nguy hiểm | danger-500 | `red-600` | #dc2626 |

**Luật màu cứng:**
- Neutral **luôn dùng họ `slate`**, KHÔNG dùng `gray`/`zinc`/`neutral`.
- Brand/CTA **luôn dùng `blue`**. Một CTA mua hàng = `bg-blue-600 hover:bg-blue-700`.
- Giá bán/sale = đỏ (`.price-s` hoặc `red-600`); giá gốc = `.price-o`. Không tô đỏ text thường.
- Màu accent khác (`purple`, `emerald`, `cyan`, `indigo`, `rose`...) **chỉ** được dùng cho icon danh mục trang trí hoặc promo banner phụ — KHÔNG dùng cho nút mua, giá, hay text nội dung.
- Trong 1 product card tối đa: 1 màu CTA + 1 màu sale + 1 màu text chính + 1 màu text phụ.

---

## 3. Typography & spacing

- Font duy nhất: **Inter**. Weight dùng: 400/500/600/700/800.
- Thang chữ (theo theme điện tử):

| Element | Class gợi ý |
|---|---|
| Section title | `text-xl font-bold` (≈20px) |
| Tên SP trong card | `text-sm font-semibold` (mobile/card nhỏ) hoặc `text-base` |
| Giá trong card | `text-base font-extrabold .price-s` |
| Giá gốc | `text-xs .price-o` |
| Quick specs / caption | `text-[10px]`–`text-xs` |
| Body | `text-sm` |
| Badge | `text-[10px]`–`text-[11px] font-bold` |

- Spacing theo hệ 4px (dùng thang Tailwind `p-2 p-3 p-4 gap-4 ...`). KHÔNG dùng số lẻ (`p-[13px]`).
- Container chuẩn: `max-w-7xl mx-auto px-4`. Mọi section bọc nội dung trong container này; nền full-width thì cho màu nền ra ngoài, nội dung vẫn trong `max-w-7xl`.
- Section spacing: `py-5`–`py-8`. Giữa các block lớn dùng `border-b border-slate-100` để phân tách (đúng như 2 trang đã có).
- Radius: card `rounded-2xl`, button/input `rounded-xl`, badge `rounded-md`/`rounded-full`. Không trộn quá nhiều loại radius trong cùng màn.

---

## 4. Layout dùng chung — COPY nguyên văn, không vẽ lại

Header, Navigation/Mega menu, và Footer **phải giống hệt** giữa mọi trang. Khi làm trang mới:

1. Mở `01-home-page.html`, copy nguyên 3 block: `ANNOUNCEMENT BAR`, `MAIN HEADER`, `NAVIGATION / MEGA MENU`, và `FOOTER`.
2. Dán vào trang mới, chỉ đổi `active` state của menu cho đúng trang hiện tại.
3. Giữ nguyên class, SVG icon, cấu trúc, text. **Không** tự thiết kế header/footer mới.

Đặc điểm phải giữ:
- Header `sticky top-0 z-50`, nav `sticky top-[72px] z-40`.
- Mobile: hàng search riêng (`md:hidden`), nút hamburger, cart luôn hiển thị + badge số lượng.
- Footer nền `#0f172a`, 5 cột desktop / accordion-style 2 cột mobile, có dòng copyright + GPKD.
- Mỗi section lớn mở đầu bằng comment banner:

```html
<!-- ============================================================
     N. TÊN SECTION
     ============================================================ -->
```

---

## 5. Component chuẩn

### 5.1. Product Card (quan trọng nhất)
Cấu trúc cố định, theo đúng card trong `01-home-page.html`:

```text
<div class="product-card bg-white border border-slate-200 rounded-2xl overflow-hidden cursor-pointer">
  [Ảnh] aspect-square, nền slate-50, object-contain, có badge sale góc trái
  [Tên] text-sm font-semibold line-clamp-2          ← TỐI ĐA 2 DÒNG
  [Quick specs] 3 chip text-[10px] bg-slate-100      ← i5 / 16GB / SSD 512GB
  [Rating] ★ vàng + (số review)
  [Giá] .price-s nổi bật + .price-o gạch ngang + badge -%
  [Promo line] text-[10px] (trả góp / quà tặng) — tối đa 1 dòng
  [Stock line] ✓ Còn hàng (green) / ⚠ Sắp hết (orange) / — Hết hàng (slate)
  [Action] nút "Thêm vào giỏ" full-width bg-blue-600 + nút So sánh (icon)
</div>
```

Bắt buộc: ảnh `object-contain` (KHÔNG `cover` — đồ điện tử không được crop mất sản phẩm); tên `line-clamp-2`; card cùng grid phải đều chiều cao.

### 5.2. Các state bắt buộc minh hoạ trong mockup
Mockup phải **show được các trạng thái thật**, không chỉ trạng thái đẹp. Trong mỗi grid sản phẩm, đặt ít nhất 1 card mỗi loại khi phù hợp:
- **Hết hàng**: badge "Hết hàng" (slate-400), ảnh `opacity-50`, nút disabled `bg-slate-200 text-slate-400 cursor-not-allowed`, text giá xám.
- **Sale**: badge `-%` đỏ + giá gốc gạch ngang.
- **Sắp hết hàng**: dòng stock màu cam + (tuỳ trang) progress "đã bán X%".

Ngoài ra mỗi trang gọi data phải có khối minh hoạ (có thể để trong comment hoặc 1 section riêng cuối trang) cho:
- **Loading** (`.skeleton`), **Empty state** (icon + message + CTA), **Error state** (message thân thiện + nút "Thử lại"). Không hiển thị lỗi kỹ thuật cho khách.

### 5.3. Badge
- Tối đa 2 badge / card. Ưu tiên: Sale > Hết hàng > New/Hot > Trả góp.
- Text badge ngắn: `-16%`, `Trả góp 0%`, `Chính hãng`, `Mới`. Không viết câu dài.

---

## 6. Icon & ảnh

- **Icon**: dùng inline SVG kiểu Heroicons outline (`fill="none" stroke="currentColor" stroke-width="2"`), `class="h-X w-X"`. Copy từ file đã có để đồng bộ. Emoji chỉ dùng cho điểm nhấn nhỏ (ticker announcement, ✓/⚠ trước dòng stock, liên hệ footer) — KHÔNG thay icon chức năng bằng emoji.
- **Ảnh placeholder**: dùng `https://placehold.co/<W>x<H>/<bg>/<fg>?text=<Tên>`. Quy ước màu theo nhóm hàng cho dễ nhìn, ví dụ:
  - Laptop: `f1f5f9/1e40af` · iPhone: `f5f3ff/7c3aed` · Tai nghe: `fff7ed/ea580c` · Màn hình/Asus: `f0fdf4/16a34a`
  - Kích thước card: `220x220` (vuông 1:1).
- Mọi `<img>` phải có `alt` mô tả thật (tên sản phẩm), và `loading="lazy"` cho ảnh dưới fold.

---

## 7. Nội dung & định dạng dữ liệu

- **Toàn bộ text tiếng Việt**, có dấu, giọng văn thân thiện-tư vấn (không khoa trương).
- **Giá tiền VND**: định dạng dấu chấm + `đ`, ví dụ `15.990.000đ`. Không dùng `$`, không số thập phân.
- Dữ liệu mẫu phải **thực tế và đa dạng**: tên sản phẩm dài (để test line-clamp), có sản phẩm hết hàng, có sản phẩm sale, giá lớn. "Dữ liệu thật luôn xấu hơn data mock" — mockup phải chứng minh layout chịu được data xấu.
- CTA dùng từ rõ ràng: `Mua ngay`, `Thêm vào giỏ`, `So sánh`, `Xem cấu hình`, `Xem tất cả`.

---

## 8. Responsive (mobile-first)

Breakpoint Tailwind: `sm 640 · md 768 · lg 1024 · xl 1280`.

Grid sản phẩm chuẩn cho storefront:
```text
Mobile (<768):  grid-cols-2
Tablet (md):    md:grid-cols-3  hoặc md:grid-cols-4
Desktop (lg):   lg:grid-cols-4  (≥1280 có thể 5 cột)
```
Grid danh mục nhanh: `grid-cols-4 md:grid-cols-8`.

Luật bắt buộc:
- **375px KHÔNG được overflow ngang** — đây là tiêu chí fail/pass số 1.
- Card mobile: tên ≤2 dòng, quick specs ≤3 item, nút có thể rút gọn ("Thêm").
- Filter trên mobile → drawer/bottom sheet (xem cách làm trong `02-product-list-page.html`).
- Bảng admin (nếu có) → mobile chuyển card list hoặc horizontal scroll có kiểm soát.
- Sticky CTA bar (product detail mobile) không che nội dung, chừa safe-area.

---

## 9. Accessibility (mức cơ bản, bắt buộc)

- Mọi button chỉ-có-icon phải có `aria-label`.
- `<input>` search có `<label class="sr-only">` hoặc `aria-label`.
- Ảnh truyền tải thông tin có `alt`; ảnh trang trí `alt=""`.
- Logo là link về `/` với `aria-label`.
- Chỉ một `<h1>` mỗi trang; section title dùng `<h2>`.
- Không dùng **chỉ màu** để báo trạng thái — luôn kèm text/icon (ví dụ "Hết hàng" + màu xám).
- Focus nhìn thấy được (đừng xoá outline mà không thay bằng style khác).

---

## 10. Quy ước file

- Tên file: `NN-<ten-trang>.html`, số thứ tự khớp với spec trong `docs/theme/` (vd `03-product-detail-page.html` ↔ `04-storefront-product-detail-page.md`). Kiểm tra số đã có trong thư mục trước khi đặt.
- Một trang = một file. Không tạo file CSS/JS riêng.
- Encoding UTF-8 (đã có dấu tiếng Việt + emoji).

---

## 11. Checklist trước khi báo "xong" (Definition of Done)

Chỉ được coi là hoàn thành khi tất cả đúng:

```text
[ ] Đã đọc spec trang tương ứng trong docs/theme/
[ ] Header / Nav / Footer copy nguyên từ trang đã có, đồng nhất
[ ] Dùng đúng họ màu: slate (neutral) + blue (brand) + red (giá/sale)
[ ] Không hard-code hex màu ngoài bảng token ở mục 2
[ ] Section order đúng theo spec
[ ] Product card: ảnh object-contain, tên line-clamp-2, giá nổi bật
[ ] Có minh hoạ state: hết hàng + sale (+ loading/empty/error nếu trang gọi data)
[ ] Giá định dạng 15.990.000đ, text tiếng Việt có dấu
[ ] Mobile 375px KHÔNG overflow ngang
[ ] Grid đúng số cột mobile/tablet/desktop
[ ] Icon-only button có aria-label, input search có label, img có alt
[ ] Chỉ 1 thẻ h1, title + meta description có nội dung
[ ] File self-contained, mở trực tiếp bằng trình duyệt là chạy được
```

---

## 12. KHÔNG được làm

```text
✗ Tự thiết kế header/footer/nav mới khác các trang đã có
✗ Dùng họ màu gray/zinc thay cho slate, hoặc màu CTA khác blue
✗ Hard-code hex màu lung tung ngoài bảng token
✗ object-fit: cover cho ảnh sản phẩm chính (crop mất sản phẩm)
✗ Tên sản phẩm tràn quá 2 dòng làm vỡ card
✗ Bỏ qua state hết hàng / loading / empty / error
✗ Để layout overflow ngang ở 375px
✗ Thêm thư viện JS/CSS ngoài Tailwind CDN + Inter
✗ Dùng emoji thay cho icon chức năng quan trọng
✗ Hiển thị lỗi kỹ thuật (stack trace, 500 raw) cho khách
✗ Tách CSS/JS ra file riêng — mỗi mockup là 1 file .html duy nhất
```
