# Agent Prompt — Hướng dẫn sử dụng theo kịch bản

Bộ prompt này xây hai thứ cho một project: **tri thức tĩnh** (tư duy senior theo case — knowledge-os) và **vòng học động** (agent tích lũy tri thức sau mỗi task — learning loop). Chọn kịch bản bên dưới rồi đưa từng file cho agent theo đúng thứ tự.

## Các file và vai trò

| File | Vai trò | Dùng khi |
|---|---|---|
| [install-agent-learning-loop.md](install-agent-learning-loop.md) | Cài hạ tầng học vào `.claude`: context routing, memory policy, skill `/task-wrapup`, coding rules rút từ codebase | Mọi kịch bản — đây là nền tảng |
| [install-knowledge-os-skill.md](install-knowledge-os-skill.md) | Cài skill `/knowledge-os` vào `.claude` để sinh knowledge base tư duy senior theo case | Khi cần tri thức nền để ra quyết định kiến trúc |
| [knowledge-os-oneshot.md](knowledge-os-oneshot.md) | Prompt sinh knowledge base dùng **trực tiếp một lần**, không cài gì vào `.claude` | Muốn thử trước, hoặc dùng với tool khác (Cursor/Codex) |
| [knowledge-os-prompt-goc-deprecated.md](knowledge-os-prompt-goc-deprecated.md) | ⚠️ Bản gốc chưa tối ưu — lưu trữ để so sánh | Không dùng để chỉ đạo agent |
| [review-de-xuat-cai-thien.md](review-de-xuat-cai-thien.md) | Bản review bộ prompt + đề xuất cải thiện (theo P1/P2/P3) | Khi muốn hiểu vì sao bộ prompt thiết kế vậy / trước khi sửa prompt |

Hai file install đều có bước khảo sát tương thích ở đầu (Bước 0) nên chạy trước sau đều không phá nhau — nhưng thứ tự dưới đây cho kết quả tốt nhất.

> **Routing canonical:** khi cài **cả hai**, `.claude/context-routing.md` (do learning-loop tạo) là file routing **duy nhất**. knowledge-os khi chạy sẽ đề xuất diff cập nhật file này, KHÔNG tạo `20-agent-context/routing.md` song song. Chỉ khi cài knowledge-os **mà không** cài learning-loop thì routing mới nằm ở `20-agent-context/routing.md`.

## Kịch bản 1 — Bắt đầu dự án mới từ số 0

Thứ tự đưa cho agent:

1. **`install-agent-learning-loop.md`** — dựng khung trước khi có code: routing, approval policy, `/task-wrapup`. Lưu ý: bước rút `coding-rules.md` từ codebase sẽ mỏng vì chưa có code — chấp nhận bản tối thiểu, vài tuần sau yêu cầu agent đề xuất diff cập nhật.
2. **`install-knowledge-os-skill.md`** — cài skill `/knowledge-os`.
3. **Gọi `/knowledge-os`** (session mới) — chạy Giai đoạn 1 để có bản đồ case, rồi sinh knowledge **Tier 1** (tech stack, architecture, database design, API design, auth, security baseline, CI/CD, testing). Đây chính là tri thức bạn cần khi đang đứng trước các quyết định khởi đầu.
4. **Ra quyết định thật → ghi ADR ngay** — mỗi lựa chọn lớn (stack, kiến trúc, DB) yêu cầu agent tạo ADR `status: proposed` trong `19-decision-records/` để bạn duyệt. Dự án mới là lúc ADR rẻ nhất và giá trị nhất.
5. **Làm task bình thường** — sau task đáng nhớ gọi `/task-wrapup`. Tier 2/3 của knowledge-os chỉ sinh khi sắp đụng đến (sắp thêm queue thì mới sinh case queue), không sinh trước cho đủ.

Lý do thứ tự này: learning loop chạy trước để `.claude/context-routing.md` là file routing canonical duy nhất ngay từ đầu — nếu chạy knowledge-os trước, nó tạo routing riêng ở `20-agent-context/` và bạn phải duyệt thêm một bước di chuyển.

## Kịch bản 2 — Join vào dự án đang chạy

Dự án đã ra quyết định xong phần lớn — việc cần là **trích xuất cái đang có** trước, sinh tri thức tổng quát sau:

1. **`install-agent-learning-loop.md`** — quan trọng nhất là Bước 4: agent quét codebase thật và rút 5–10 convention của project (điều người mới sẽ làm sai nếu không được bảo). Đây chính xác là vị trí của bạn khi mới join.
2. **Giao task backfill đầu tiên**: yêu cầu agent ghi ADR ngược cho 3–5 quyết định kiến trúc lớn *đang tồn tại* trong code (vì sao stack này, vì sao cấu trúc module này) — `status: accepted`, đánh dấu là ADR hồi cứu. Không có bước này, mọi lesson sau đó thiếu gốc để bám.
3. **Làm task bình thường + `/task-wrapup`** — với dự án đang chạy, tri thức giá trị nhất đến từ task thật và bug thật, không phải từ knowledge sinh sẵn.
4. **`install-knowledge-os-skill.md` — tùy chọn, cài khi cần**: chỉ gọi `/knowledge-os` cho đúng case sắp phải quyết (ví dụ team sắp thêm search, sắp scale) thay vì sinh cả bộ. Knowledge tổng quát sinh hàng loạt trong dự án đã định hình phần lớn sẽ là context thừa.

## Đo giá trị & khi nào KHÔNG dùng bộ này

**Không cài cho project nhỏ, ngắn hạn hoặc thử nghiệm** — ngưỡng gợi ý: project sống dưới ~1 tháng hoặc dưới ~20 task thì bộ máy này là over-engineering (đúng lỗi mà chính knowledge-os cảnh báo); một `CLAUDE.md` vài dòng là đủ.

Với project dùng thật, mỗi ~4 tuần đánh giá bằng 3 câu hỏi: (1) **lỗi lặp lại có giảm không** — cùng loại bug xuất hiện lần hai mà không có lesson tương ứng nghĩa là vòng ghi không chạy; (2) **agent có hỏi lại ít hơn** về convention đã ghi không; (3) **vào task mới có nhanh hơn không**. Hai chu kỳ liền không cải thiện được câu nào → dừng nuôi hệ thống và tìm nguyên nhân — thường là `/task-wrapup` không được chạy hoặc routing trỏ sai.

## Quy tắc chung cho cả hai kịch bản

- Mỗi file install chạy trong **một session riêng**, xong thì mở session mới để skill được nạp.
- Agent sẽ dừng chờ bạn duyệt ở các điểm: ghi đè file tồn tại, `coding-rules.md`, chuyển ADR sang `accepted`, mọi thay đổi trong `.claude/`. Đó là thiết kế, không phải agent hỏi thừa.
- Hệ thống chỉ có giá trị khi vòng ghi được chạy thật: nếu 2 tuần không có file mới trong `21-task-history/` hoặc `22-lessons-learned/`, nhắc agent (hoặc cân nhắc thêm hook `Stop` nhắc `/task-wrapup`).
- Hệ thống phải biết trừ, không chỉ biết cộng: mỗi ~20 task hoặc mỗi tháng chạy `/task-wrapup audit` để dọn lesson sai/hết hạn, ADR bị code vượt qua — tri thức sai nằm lại lâu làm agent kém đi.
- `CLAUDE.md` giữ dưới ~30 dòng — mọi đề xuất thêm của agent phải kèm dòng nào bỏ đi.
