# 18 - Chống giao diện "AI hoá" / generic

> **⚠️ Chuẩn đồng bộ — đọc trước:** kế thừa [`../main/ecommerce_design_language.md`](../main/ecommerce_design_language.md) + [`01-electronics-store-theme.md`](01-electronics-store-theme.md) + [`../theme-html/CLAUDE.md`](../theme-html/CLAUDE.md).
> Tài liệu này là **lớp tinh chỉnh thị giác** áp lên design system, để giao diện trông như sản phẩm thật của một hãng bán đồ điện tử, **không** giống template AI sinh ra.

---

## 1. Vì sao UI trông "AI hoá"?

Các "tell" khiến giao diện trông generic/AI (theo thứ tự nặng → nhẹ):

1. **Bo góc to khắp nơi** — `rounded-2xl`/`rounded-xl` (12–16px) cho mọi card, input, button, ảnh. Tất cả mềm như nhau → không có cá tính.
2. **Pill cho mọi badge** — `border-radius:999px` cho cả status/data tag. Status đơn hàng mà bo tròn viên thuốc → trông đồ chơi.
3. **Shadow mềm ở trạng thái nghỉ** — `shadow-lg/xl` nổi bồng bềnh trên mọi thẻ. Hover nhấc mạnh `translateY(-2px)` + blur lớn.
4. **Đồng nhất tuyệt đối** — mọi thứ cùng radius, cùng shadow, cùng spacing → thiếu nhịp, thiếu phân cấp.
5. **Gradient tím/indigo, gradient hero ở mọi block, emoji làm icon** — tín hiệu "demo".
6. **Card lồng card lồng card** — mỗi cụm là một thẻ nổi trên nền xám.

> Nguyên tắc gốc: **flat + hairline + dense + có chủ đích** thắng **rounded + shadow + airy + đồng đều**.

---

## 2. Thang bo góc theo VAI TRÒ (không dùng một radius cho tất cả)

Bo góc phải phản ánh chức năng. Đồ điện tử = cảm giác "kỹ thuật, chính xác".

| Bề mặt | Radius | Tailwind |
|---|---|---|
| Bảng dữ liệu, hàng bảng, tab, ô nhập trong admin | **0–4px** (gần vuông) | `rounded` / `rounded-none` |
| Button, input, select, chip | **6px** | `rounded-md` |
| Card / container / ảnh sản phẩm | **8px** (tối đa) | `rounded-lg` |
| Modal, drawer, popover | **8–12px** | `rounded-lg` |
| Badge / status tag | **4–5px** (chữ nhật bo nhẹ) | `rounded` |
| Avatar, status dot, số đếm, nút icon tròn | **full** | `rounded-full` |

**Quy tắc:**
- **Cấm** `rounded-2xl`, `rounded-3xl` làm mặc định. Cap card ở `rounded-lg`.
- **Storefront** được mềm hơn admin một bậc (card `rounded-lg`); **Admin** crisp hơn (card/control `rounded-md`, bảng gần vuông) vì là công cụ vận hành dày dữ liệu.
- **Không** bo tròn pill cho status/data — chỉ pill cho avatar/dot/đếm/nút icon.
- **Không** lồng nhiều lớp bo góc: phần tử con trong card đã bo thì con cháu nên vuông.

---

## 3. Elevation (đổ bóng) có kỷ luật

- **Bề mặt nghỉ = phẳng + viền hairline** (`border border-slate-200`), **không shadow**. Phân tách bằng viền/đường kẻ/nền-band, không bằng bóng.
- **Shadow chỉ dùng cho lớp nổi thật**: dropdown, popover, modal, toast, thanh sticky. Dùng `shadow-md`/`shadow-lg`, không `shadow-2xl` tràn lan.
- **Hover** tinh tế: đổi màu viền + bóng nhẹ (`0 2px 10px rgba(15,23,42,.08)`), nhấc tối đa `translateY(-1px)` (hoặc không nhấc). Không hấc mạnh.

---

## 4. Mật độ & nhịp (electronics = nhiều thông tin)

- Vùng dữ liệu (bảng giá, spec, đơn hàng) đi **dày** hơn; thoáng chỉ dành cho marketing/hero.
- Dùng **tabular numbers** cho giá/thông số/số lượng (`tabular-nums`) để cột số thẳng hàng.
- Phân cấp bằng **typography** (đậm/nhạt, size) thay vì bằng bóng đổ.
- Gom nội dung liên quan vào **một container có viền**, thay vì nhiều card nổi rời rạc.

---

## 5. Màu & cá tính

- Giữ hệ slate / blue (brand) / green (success) / red (sale). **Giới hạn gradient ≤ 1 vùng/màn** (hero hoặc 1 promo banner). Không gradient ở card thường.
- Không tím/indigo "AI gradient". Accent (emerald/cyan/...) chỉ cho icon danh mục trang trí.
- Một ProductCard tối đa: 1 màu CTA + 1 màu sale + text chính + text phụ (theo design language §4).

---

## 6. Icon & chi tiết "thủ công"

- Icon dùng **SVG stroke đồng bộ** (Heroicons), **không** emoji làm icon chức năng (✓ glyph trong badge chấp nhận được; 💳🎁🔥🚚 thì không).
- Căn lề chuẩn, đơn vị rõ (đ, GB, ", Hz), dữ liệu mẫu "thật" → tạo cảm giác sản phẩm thật.

---

## 7. Áp dụng vào mockup HTML (đã thực hiện)

Bulk pass trên `docs/theme-html/*`:
- `rounded-2xl`/`rounded-xl` → storefront `rounded-lg`, admin `rounded-md`; admin `rounded-lg` → `rounded-md`.
- `.badge` (admin) `border-radius:999px` → `5px` (status tag chữ nhật).
- Elevation hạ một bậc: `shadow-2xl→lg`, `shadow-xl→md`, `shadow-lg→sm`; hover product-card bóng nhẹ + `translateY(-1px)`.
- Giữ: avatar/dot/đếm `rounded-full`; 1 promo banner trang trí; hệ màu slate/green.

> `CLAUDE.md` đã được cập nhật thang radius/elevation tương ứng để mockup mới không tái tạo "AI look".
