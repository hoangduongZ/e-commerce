# Review bộ prompt `agent-prompt/senior` — đề xuất cải thiện

> Reviewer: đóng vai Senior/Tech Lead. Bối cảnh đặc biệt: bản review này viết **ngay sau khi thực thi** `install-agent-learning-loop.md` + `install-knowledge-os-skill.md` lên repo `e-commerce-backend`. Các mục có nhãn **[gặp thật]** là lỗi/ma sát va phải khi chạy, không phải suy đoán trên giấy.
>
> Phạm vi: 5 file — `README.md`, `install-agent-learning-loop.md`, `install-knowledge-os-skill.md`, `knowledge-os-oneshot.md` (trước là `senior_02.md`), `knowledge-os-prompt-goc-deprecated.md` (trước là `senior_01.md`).

> **Trạng thái (cập nhật 2026-07-04):** toàn bộ **P1.1–P1.5 đã áp dụng** vào bộ prompt, và `senior_01/02.md` đã đổi tên có ngữ nghĩa. Các mục P1 dưới đây giữ lại làm changelog (đánh dấu ✅ ĐÃ SỬA). P2/P3 vẫn mở, tùy nhu cầu.

## TL;DR

Bộ prompt **tốt về tư duy, chắc về nguyên tắc** (ngưỡng promote, "mặc định không ghi gì", tách quyền `.claude` vs `knowledge`, biết trừ chứ không chỉ cộng, dám nói "đừng cài cho project nhỏ"). Nhưng khi *chạy thật* lộ ra một số **mâu thuẫn giữa hai file install** và **điểm chưa quyết dứt khoát** khiến agent phải tự phán đoán — mà tự phán đoán thì mỗi lần một khác. Ưu tiên sửa nhóm P1 (mâu thuẫn routing + verify không cưỡng chế + nguồn sự thật trùng lặp) vì chúng ảnh hưởng đúng đắn, không chỉ thẩm mỹ.

---

## Điểm mạnh (giữ nguyên, đừng "cải thiện" mất)

- **Mô hình ngưỡng promote** (task note → lesson → rule → CLAUDE.md) có tiêu chí đo được, không cảm tính. Đây là phần giá trị nhất.
- **"Mặc định không ghi gì"** — chống đúng bệnh knowledge-base phình to thành rác.
- **Budget cứng CLAUDE.md ~30 dòng + "thêm 1 dòng phải bỏ 1 dòng"** — hiếm prompt nào tự áp kỷ luật context như vậy.
- **Phần "khi nào KHÔNG dùng bộ này"** (README §Đo giá trị) — trung thực, tránh over-engineering chính nó.
- **Embed nội dung verbatim vào installer** — installer tự chứa, không phụ thuộc file ngoài lúc chạy.

---

## P1 — Nên sửa trước (mâu thuẫn / ảnh hưởng đúng đắn)

### P1.1 — Hai installer bất đồng về "routing sống ở đâu" **[gặp thật]** — ✅ ĐÃ SỬA

- **Triệu chứng:** `install-knowledge-os-skill.md` Bước 4 (dòng ~183) chèn vào `CLAUDE.md` câu: *"đọc đúng file theo `docs/knowledge/20-agent-context/routing.md`"*. Và SKILL.md của knowledge-os (Giai đoạn 3) vẫn dặn *tạo* `20-agent-context/routing.md`. Nhưng `install-agent-learning-loop.md` lập `.claude/context-routing.md` làm **routing canonical duy nhất**, và README (dòng ~26) cảnh báo đúng nguy cơ "knowledge-os tạo routing riêng ở 20-agent-context/".
- **Hệ quả:** Chạy theo đúng thứ tự README khuyến nghị (learning-loop trước) thì knowledge-os sẽ tạo **routing thứ hai song song** — đúng cái README bảo tránh. Khi thực thi tôi phải tự tay sửa câu pointer và thêm ghi chú tương thích vào `kb-structure.md`. Người khác chạy sẽ không biết mà sửa.
- **Gốc rễ:** cảnh báo nằm trong **README** (prose), nhưng agent chạy `/knowledge-os` ở session mới **không đọc lại README** — nó chỉ đọc SKILL.md. Guardrail đặt sai chỗ.
- **Đề xuất:**
  1. Sửa pointer ở knowledge-os Bước 4 trỏ về `.claude/context-routing.md` (không phải `20-agent-context/routing.md`).
  2. Đưa quy tắc "nếu đã có `.claude/context-routing.md` thì KHÔNG tạo routing song song, chỉ đề xuất diff" **vào trong SKILL.md của knowledge-os** (Giai đoạn 3), không để riêng ở README.
  3. Chốt một câu ở README: "routing canonical = `.claude/context-routing.md`; `20-agent-context/` chỉ dùng khi cài knowledge-os **mà không** cài learning-loop."

### P1.2 — Có nên tạo sẵn thư mục knowledge rỗng hay không: không ai quyết **[gặp thật]** — ✅ ĐÃ SỬA

- **Triệu chứng:** `install-agent-learning-loop.md` "Kiến trúc đích" (dòng ~44–48) vẽ `21-task-history/`, `22-lessons-learned/`, `23-debugging/` như thư mục đích; nhưng Ràng buộc (dòng ~258) lại nói *"Không tạo file rỗng để sẵn — trừ `00-index/README.md`"*. Các Bước 1–6 **không nói rõ** tạo hay không tạo các thư mục này.
- **Hệ quả:** Khi chạy tôi tạo cả 4 thư mục rồi phải tự xóa 3 cái rỗng để tôn trọng ràng buộc — mỗi agent sẽ xử lý một kiểu (người tạo `.gitkeep`, người để rỗng, người bỏ hẳn), routing thì lại trỏ tới chúng.
- **Đề xuất:** thêm một câu dứt khoát ở Bước 1 hoặc Ràng buộc: *"Chỉ tạo `docs/knowledge/00-index/README.md`. Các thư mục `19/21/22/23` do `/task-wrapup` tạo khi ghi file thật đầu tiên (mkdir on-demand) — routing được phép trỏ tới đường dẫn chưa tồn tại vì đó là đích tích lũy, khác với knowledge content phải có thật."* (Đồng thời làm rõ mâu thuẫn với quy tắc "routing chỉ trỏ file đã tồn tại" — phân biệt "thư mục tích lũy" vs "file knowledge".)

### P1.3 — Bước verify chỉ *khẳng định*, không *cưỡng chế* **[gặp thật]** — ✅ ĐÃ SỬA

- **Triệu chứng:** learning-loop Bước 6 yêu cầu "frontmatter SKILL.md parse được", "routing không trỏ file chưa tồn tại", "CLAUDE.md ≤ 30 dòng" — nhưng không đưa **lệnh kiểm tra**. knowledge-os Bước 5 tương tự.
- **Hệ quả:** agent "lười" sẽ chỉ *tuyên bố* đã kiểm tra. Tôi có chạy `python` parse YAML + `wc -l` nhưng đó là do tôi chủ động, prompt không bắt.
- **Đề xuất:** ghi thẳng các lệnh kiểm tra tối thiểu vào Bước verify, ví dụ: `wc -l CLAUDE.md` (khẳng định < 30), một snippet parse frontmatter, `grep` các đường dẫn trong routing đối chiếu file tồn tại. "Verify" phải là hành động chạy được, không phải lời hứa.

### P1.4 — Trùng lặp nguồn sự thật: `knowledge-os-oneshot.md` ⇄ nội dung embed trong installer — ✅ ĐÃ SỬA

- **Triệu chứng:** `knowledge-os-oneshot.md` (bản standalone) và phần embed trong `install-knowledge-os-skill.md` (SKILL.md + case-template.md + kb-structure.md) là **gần như trùng nội dung**. Template 8 mục, cấu trúc thư mục, tiêu chí chất lượng đều xuất hiện ở cả hai.
- **Hệ quả:** cải thiện template ở một chỗ mà quên chỗ kia → hai bản drift. README đã phải giải thích cùng một hành vi lần thứ ba.
- **Đề xuất:** chọn **một** làm nguồn sự thật. Gợi ý: giữ installer là nguồn (vì nó là cái được thực thi), biến `knowledge-os-oneshot.md` thành *"bản trích để dùng one-shot với Cursor/Codex — nội dung sinh ra từ installer, đừng sửa trực tiếp"*, hoặc ngược lại. Kèm một dòng ở đầu file phụ chỉ về file chính.
- **Đã làm:** installer là nguồn sự thật; `knowledge-os-oneshot.md` có banner đầu file chỉ về installer; dòng "(nguồn: ...)" trong installer đã sửa lại cho đúng chiều phụ thuộc.

### P1.5 — Không có đường "update/nâng cấp" sau lần cài đầu

- **Triệu chứng:** cả hai installer chỉ xử lý *first-install* ("nếu `.claude/` đã có nội dung: đọc trước, chỉ bổ sung, không ghi đè"). "Chỉ bổ sung" nghĩa là khi bộ prompt ra bản mới (sửa template routing, policy...), **bản cải tiến không bao giờ vào được** repo đã cài.
- **Đề xuất:** thêm một mục "Cập nhật bản đã cài": so version/diff từng file `.claude/`, trình diff cho người dùng duyệt (vì `.claude/` là nhóm cần approval). Có thể gắn version marker (frontmatter `template_version`) để phát hiện lệch.

---

## P2 — Nên sửa (thiếu robust, dễ hiểu sai)

### P2.1 — Phát hiện knowledge hiện có quá cứng nhắc về đường dẫn **[gặp thật]**

Compat check giả định `{{KNOWLEDGE_ROOT}}` = `docs/knowledge`. Nhưng workspace này có repo `e-commerce-docs` với thư mục `knowledge/` ở **root** (không có tiền tố `docs/`, không đánh số) — check sẽ **không nhận ra** đó là knowledge và có thể tạo cây thứ hai. Thực tế phổ biến hơn: **docs repo tách khỏi code repo**. Đề xuất: (1) bổ sung hướng dẫn cho trường hợp knowledge nằm ở repo docs riêng; (2) check nên tìm theo *dấu hiệu* (có thư mục `00-index/`, có file numbered) chứ không chỉ khớp đường dẫn cố định.

### P2.2 — Audit theo `seen_count` xóa nhầm tri thức hiếm-nhưng-đắt

Policy: "lesson có `seen_count` không tăng qua 2 kỳ audit → đề xuất xóa". Nhưng lesson kiểu *"đừng sửa migration đã merge"* hay footgun payment có thể **nhiều tháng không tái diễn** mà vẫn cực giá trị. Heuristic đang đánh đồng "lâu không gặp" với "vô dụng". Đề xuất: thêm chiều **severity** — lesson gắn `severity: high` (vi phạm gây lỗi production/mất tiền) thì **miễn** khỏi luật xóa theo tần suất, chỉ xóa khi thực sự hết hạn (module bị bỏ, lib thay).

### P2.3 — Hai cơ chế routing chồng nhau, không có luật ưu tiên

Case file mang frontmatter `when_to_read`, đồng thời có bảng routing tập trung. Agent có thể route bằng bảng, hoặc quét `when_to_read` — hai nguồn có thể mâu thuẫn. Đề xuất: chốt "bảng routing là primary; `when_to_read` chỉ để RAG/tìm kiếm khi task không khớp bảng", ghi rõ thứ tự ưu tiên.

### P2.4 — `senior_01.md` bị "giáng cấp" nhưng không tự dán nhãn — ✅ ĐÃ SỬA

README bảo *"không dùng để chỉ đạo agent"*, nhưng bản thân file không có banner cảnh báo. Ai trỏ agent vào thẳng file (như chính task này trỏ vào thư mục) dễ nhặt nhầm. **Đã làm:** đổi tên `senior_01.md` → `knowledge-os-prompt-goc-deprecated.md` và thêm banner ⚠️ DEPRECATED ở đầu file.

### P2.5 — Budget CLAUDE.md không có lối thoát khi file gốc đã lớn

Cả hai installer đều *append* vào CLAUDE.md và giả định còn chỗ dưới 30 dòng. Repo thật thường đã có CLAUDE.md dài; append mù sẽ vỡ budget mà installer không có chỉ dẫn "nếu append làm vượt budget thì trình phương án nén phần cũ cho người dùng duyệt". Đề xuất thêm nhánh xử lý này.

---

## P3 — Nice-to-have (đánh bóng, không gấp)

- **Ref-integrity tự động:** policy dựa vào audit thủ công để phát hiện `refs` trỏ file đã moved/deleted. Có thể gợi ý một check nhẹ (grep các `refs:` đối chiếu file tồn tại) chạy trong CI hoặc hook — biến "code đổi thì phát hiện knowledge lệch" thành tự động thay vì trông chờ audit.
- **Lý do `description` phải tiếng Anh chưa ghi:** SKILL frontmatter `description` để tiếng Anh (đúng — skill router match tốt hơn) nhưng prompt không nói vì sao; người bảo trì "Việt hóa cho đồng bộ" sẽ vô tình phá matching. Ghi 1 dòng rationale.
- **Không có baseline cho §Đo giá trị:** README hỏi "lỗi lặp có giảm không / vào task nhanh hơn không" mỗi 4 tuần, nhưng install không chụp **baseline t=0** → "giảm so với cái gì?" vô định. Gợi ý ghi 1 dòng baseline lúc cài.
- **Redaction guidance quá trừu tượng cho domain payment:** "ẩn danh hóa trước khi ghi" là đúng nhưng mỏng cho ecommerce/payment. Nên kèm checklist redaction cụ thể (mask token/card/PII, ví dụ pattern) vì `23-debugging/` chính là nơi người ta hay dán stacktrace có token.
- **Thiếu ví dụ một vòng đời hoàn chỉnh:** bộ prompt mạnh về "luật", yếu về "một ví dụ chạy từ đầu đến cuối" (một task → wrap-up → sinh lesson → lần sau đọc lại lesson đó). Một ví dụ end-to-end ngắn sẽ giúp người mới tin và làm đúng.

---

## Thứ tự sửa đề xuất

1. **P1.1 + P1.2** (mâu thuẫn routing + thư mục rỗng) — sửa cùng lúc vì cùng đụng phần "routing & cấu trúc", và đây là 2 lỗi va phải ngay lần chạy đầu.
2. **P1.3** (verify cưỡng chế) — rẻ, tăng độ tin cậy mọi lần cài về sau.
3. **P1.4 + P1.5** (nguồn sự thật + đường update) — vấn đề bảo trì dài hạn, sửa trước khi bộ prompt được tái sử dụng nhiều nơi.
4. P2 theo nhu cầu; P3 khi rảnh.

> Ghi chú phạm vi: bản review này soi **tính đúng/nhất quán/vận hành được** của bộ prompt, không phán xét triết lý (triết lý tốt). Mọi số dòng là tương đối, đối chiếu bản đọc ngày 2026-07-04.
