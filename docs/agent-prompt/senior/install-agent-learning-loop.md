# install-agent-learning-loop — Cơ chế giúp AI Agent thông minh lên theo thời gian

> **Prompt này dành cho AI Agent thực thi trong một project thật.** Nhiệm vụ là **cài đặt hệ thống file hoạt động được** — không phải viết tài liệu thiết kế lý thuyết. Nội dung các file đích đã được nhúng nguyên văn bên dưới.

## Vai trò & nguyên tắc cốt lõi

Bạn là Principal Software Engineer kiêm AI Agent Workflow Architect. Bạn thiết kế cơ chế để agent tích lũy tri thức qua từng task trong project này.

Agent thông minh lên **không phải nhờ nhồi nhiều context**, mà nhờ 4 điều:

1. **Đọc đúng** — phân loại task rồi đọc đúng vài file liên quan, không quét repo.
2. **Ghi có chọn lọc** — sau task chỉ ghi lại tri thức tái sử dụng được; đa số task không cần ghi gì.
3. **Promote có ngưỡng** — task note → lesson → rule theo tiêu chí đo được, không theo cảm tính.
4. **Tách bạch quyền hạn** — `.claude/` là *cách agent làm việc* (ổn định, cần approval); `knowledge/` là *cái agent học được* (agent tự ghi).

## Đường dẫn & kiểm tra tương thích

| Placeholder | Mặc định |
|---|---|
| `{{PROJECT_ROOT}}` | root project hiện tại (thư mục chứa `.git` gần nhất) |
| `{{KNOWLEDGE_ROOT}}` | `{{PROJECT_ROOT}}/docs/knowledge` |

Quy tắc tương thích — **một khái niệm chỉ có một chỗ, không bao giờ có hai cây knowledge song song**:

- Nếu `{{KNOWLEDGE_ROOT}}` đã tồn tại (hệ thống knowledge-os đã cài): **không** tạo cây mới — bổ sung các thư mục còn thiếu, nối tiếp số thứ tự hiện có (ví dụ đã có `00`–`20` thì thêm `21-task-history/`, `22-lessons-learned/`, `23-debugging/`; ADR dùng lại `19-decision-records/`).
- Nếu đã có `20-agent-context/routing.md`: gộp nội dung vào `.claude/context-routing.md` (file canonical duy nhất), thay file cũ bằng một dòng trỏ sang vị trí mới. Việc này đụng vào hành vi agent → trình diff cho người dùng duyệt trước.
- Nếu `.claude/` đã có nội dung: đọc trước, chỉ bổ sung, không ghi đè.

## Kiến trúc đích

```text
{{PROJECT_ROOT}}/
  CLAUDE.md                      # ≤ 30 dòng — chỉ pointer, có budget cứng
  .claude/
    rules/
      coding-rules.md            # convention THẬT của project — agent không tự sửa
    context-routing.md           # task type → file cần đọc (canonical, duy nhất)
    memory-update-policy.md      # ai được ghi gì, khi nào cần approval
    skills/
      task-wrapup/
        SKILL.md                 # quy trình ghi tri thức sau task
  docs/knowledge/
    00-index/                    # bảng routing nội dung — cập nhật cùng lúc với mọi file mới
    19-decision-records/         # ADR (số thư mục điều chỉnh theo cây hiện có)
    21-task-history/             # mỗi task đáng nhớ một file, YYYY-MM-DD-<slug>.md
    22-lessons-learned/          # bài học tái sử dụng được
    23-debugging/                # bug tốn công tìm + root cause + cách nhận ra sớm
```

## Vòng đời một task — cơ chế tự cải thiện

**Trước task:** phân loại task theo `.claude/context-routing.md` → đọc theo thứ tự: (1) `rules/coding-rules.md`, (2) các file khớp routing, (3) lesson cùng `area`, (4) tối đa 2–3 task history gần nhất cùng nhóm. Không đọc toàn bộ knowledge, không đọc history khác nhóm, không đọc lại file đã có trong context.

**Trong task:** ghi chú nháp những gì *gây ngạc nhiên* — giả định sai, convention ngầm không có trong docs, bug do thiếu tri thức project. Phân biệt: fix cục bộ chỉ đúng cho task này (không ghi) vs tri thức lần sau vẫn đúng (ghi).

**Sau task:** chạy quy trình `/task-wrapup` theo bảng quyết định trong skill (nhúng ở Bước 3 bên dưới). Mặc định là **không ghi gì** — chỉ ghi khi vượt ngưỡng.

## Các bước thực thi

### Bước 0 — Khảo sát và trình kế hoạch

1. Kiểm tra hiện trạng: `.claude/`, `{{KNOWLEDGE_ROOT}}`, `CLAUDE.md`, file routing cũ.
2. Báo cáo ngắn: cây thư mục sẽ tạo (đã điều chỉnh số thứ tự theo cây hiện có), file nào bị đụng, cần quyết định gì.
3. Nếu phải sửa/di chuyển file đã tồn tại → chờ xác nhận. Nếu chỉ tạo mới → đi tiếp.

### Bước 1 — Tạo `.claude/context-routing.md`

Ghi nội dung sau, **điều chỉnh đường dẫn theo cây knowledge thực tế**. Quy tắc cứng: chỉ trỏ đến file/thư mục đã tồn tại; nhóm chưa có knowledge thì ghi `(chưa có)` — routing trỏ đến file ma còn tệ hơn không có routing.

````markdown
# Context Routing — đọc gì trước khi làm task

Phân loại task theo bảng dưới (một task có thể khớp nhiều nhóm — đọc hợp của các nhóm, vẫn giới hạn tối đa ~5 file). Luôn đọc `rules/coding-rules.md` trước.

| Nhóm task | Tín hiệu nhận biết | Đọc (theo thứ tự) |
|---|---|---|
| Backend / API | endpoint, service, controller, business logic | `02-architecture/`, `03-backend/`, ADR `*api*` |
| Frontend | UI, component, page, state | `04-frontend/`, lesson `area: frontend` |
| Database | schema, migration, query, index | `05-database/`, ADR `*db*`, `*schema*` |
| Auth (authn/authz) | login, token, session, permission, role | `03-backend/auth*`, `07-security/`, ADR `*auth*`, history `*auth*` gần nhất |
| Payment | thanh toán, order, refund, webhook | `03-backend/payment*`, `07-security/`, ADR `*payment*` |
| File upload / storage | upload, S3, media | `03-backend/file*`, `07-security/` |
| Queue / async | job, worker, event, retry | `06-infrastructure/queue*`, lesson `area: queue` |
| Cache | cache, TTL, invalidation | `06-infrastructure/cache*`, lesson `area: cache` |
| Search | search, index, full-text | `06-infrastructure/search*` |
| Testing | test, coverage, mock | `08-testing/`, `rules/coding-rules.md` mục test |
| Bug fixing | fix, lỗi, không chạy | `23-debugging/` cùng khu vực lỗi, history cùng module |
| Refactoring | refactor, cleanup, tách module | `02-architecture/`, `16-refactoring/`, ADR liên quan module |
| Performance | chậm, N+1, tối ưu | `10-performance/`, `23-debugging/` liên quan perf |
| Security | lỗ hổng, CVE, hardening | `07-security/` (toàn bộ), ADR `*security*` |
| Deployment / CI | deploy, pipeline, release | `12-devops/`, lesson `area: deploy` |
| Incident | production down, rollback, hotfix | `13-incident-response/`, `23-debugging/`, history incident gần nhất |
| AI integration | LLM, prompt, embedding, agent | `14-ai-engineering/`, ADR `*ai*` |

Không khớp nhóm nào → đọc `00-index/README.md` để tự tìm, tối đa 3 file. Vẫn không có → làm với context của codebase và ghi nhận khoảng trống knowledge trong wrap-up.
````

### Bước 2 — Tạo `.claude/memory-update-policy.md`

Ghi đúng nội dung sau:

````markdown
# Memory Update Policy — quyền ghi và approval

## Ma trận quyền

| File / thư mục | Quyền của agent |
|---|---|
| `21-task-history/`, `22-lessons-learned/`, `23-debugging/` | Tự ghi, liệt kê trong báo cáo cuối task |
| `00-index/README.md` | Tự cập nhật, bắt buộc cùng lúc với mọi file mới |
| `19-decision-records/` (ADR) | Tự tạo với `status: proposed`; chuyển `accepted` phải được người dùng duyệt |
| Knowledge nội dung (`02`–`17`) | Sửa file có sẵn: tự làm nếu là bổ sung, trình diff nếu là thay đổi kết luận |
| `.claude/context-routing.md` | Chỉ đề xuất diff, chờ duyệt |
| `.claude/rules/`, `CLAUDE.md` | Bắt buộc hỏi trước — không bao giờ tự sửa |
| `.claude/memory-update-policy.md` | Không bao giờ tự sửa |

Commit knowledge tách khỏi commit code: `docs(knowledge): <nội dung>`. Team nhiều người: knowledge **thay đổi kết luận** (không phải chỉ bổ sung) đi qua PR review như code.

## An toàn dữ liệu (cứng — không có ngoại lệ)

Không bao giờ ghi secret, token, credential, PII hay dữ liệu production vào knowledge. Payload, log, stacktrace trong debugging notes phải ẩn danh hóa trước khi ghi — các file này được commit và chia sẻ cho cả team.

## Vệ sinh tri thức — hệ thống phải biết trừ, không chỉ biết cộng

- Mọi file knowledge phải trỏ đến ít nhất một đường dẫn code thật (frontmatter `refs` hoặc trong thân) — code đổi thì mới phát hiện được knowledge đã lệch.
- Đang làm task mà phát hiện lesson/knowledge sai → sửa hoặc xóa ngay trong wrap-up (agent tự làm được), nêu lý do trong commit message.
- Audit định kỳ bằng `/task-wrapup audit` (mỗi ~20 task hoặc mỗi tháng, tùy cái đến trước): lesson có `seen_count` không tăng qua 2 kỳ audit, ADR bị code hiện tại vượt qua, `refs` trỏ đến file không còn tồn tại → trình danh sách đề xuất xóa/cập nhật cho người dùng duyệt.

## Ngưỡng promote — Task note → Lesson → Rule → CLAUDE.md

| Cấp | Điều kiện lên cấp này | Ai quyết |
|---|---|---|
| Task note (trong task history) | Điều bất ngờ trong 1 task, chưa chắc lặp lại | Agent |
| Lesson learned | Gặp ≥ 2 lần, hoặc 1 lần nhưng gây mất > 30 phút; viết được thành "lần sau làm X thay vì Y" | Agent |
| Project rule (`.claude/rules/`) | Lesson đã áp dụng đúng ≥ 3 lần, hoặc vi phạm sẽ gây lỗi production; nén được ≤ 3 dòng | Người dùng duyệt |
| `CLAUDE.md` | Rule mà *mọi* task đều cần biết, không chỉ một nhóm task | Người dùng duyệt |

## Khi nào KHÔNG promote (chống nhiễu context)

- Chỉ đúng một lần, phụ thuộc context task cụ thể.
- Không nén được xuống ≤ 3 dòng mà vẫn đúng.
- Đã hết hạn (thư viện đã nâng cấp, module đã xóa) → xóa lesson thay vì giữ.
- `CLAUDE.md` có budget cứng ~30 dòng: muốn thêm 1 dòng phải chỉ ra dòng nào bỏ đi hoặc chứng minh còn chỗ.
````

### Bước 3 — Tạo `.claude/skills/task-wrapup/SKILL.md`

Ghi đúng nội dung sau:

````markdown
---
name: task-wrapup
description: Capture reusable knowledge after finishing a task — write task history, lessons learned, debugging notes, or ADR drafts per the memory update policy. Use after completing a significant task, fixing a hard bug, or making an architecture decision.
---

# Task Wrap-up — ghi tri thức sau task

**Điều kiện tiên quyết:** task đã được verify (test pass hoặc chạy thật và quan sát được kết quả). Task chưa verify thì tri thức rút ra chưa đáng tin — quay lại verify trước, không wrap-up.

Đọc `.claude/memory-update-policy.md` trước. Mặc định là **không ghi gì** — chỉ ghi khi khớp bảng dưới. Hai quy tắc khi ghi: (1) trước khi tạo lesson mới, tìm lesson cùng `area` đã có — trùng thì tăng `seen_count` thay vì tạo file mới; (2) mọi file ghi ra phải trỏ đến ít nhất một đường dẫn code thật. Ghi xong file nào phải cập nhật `00-index/README.md` ngay.

| Điều xảy ra trong task | Ghi vào |
|---|---|
| Task thay đổi nhiều file / có quyết định đáng nhớ / chạm module quan trọng | `21-task-history/YYYY-MM-DD-<slug>.md` |
| Chọn công nghệ, đổi kiến trúc, trade-off có hệ quả dài hạn | ADR trong `19-decision-records/` (`status: proposed`) |
| Bug tốn > 30 phút vì thiếu tri thức project | `23-debugging/<slug>.md` + cân nhắc lesson |
| Nhận ra pattern/convention gặp ≥ 2 lần | `22-lessons-learned/lesson-<slug>.md` |
| Lesson có sẵn được áp dụng lần thứ 3 | Tăng `seen_count`, đề xuất promote thành rule (chờ duyệt) |
| Task routine, không có gì mới | Không ghi — báo "không có tri thức mới" là kết quả hợp lệ |

## Template task history (≤ 30 dòng/file)

```markdown
---
id: <YYYY-MM-DD-slug>
type: task-history
area: [backend, auth]
refs: [<đường dẫn code chính đã chạm>]
related: [<id ADR/lesson liên quan>]
---
# <Tên task>
**Đã làm:** <2-3 câu>
**Quyết định & lý do:** <quyết định nào, vì sao chọn>
**Bất ngờ / giả định sai:** <điều không như dự đoán>
**Lần sau nên:** <hành động cụ thể>
```

## Template lesson learned

```markdown
---
id: lesson-<slug>
type: lesson
area: [<nhóm task>]
refs: [<đường dẫn code liên quan>]
seen_count: <số lần gặp>
promoted: false
---
**Bài học:** <1-2 câu, dạng "làm X thay vì Y">
**Bối cảnh phát hiện:** <task nào, chuyện gì xảy ra>
**Cách áp dụng:** <nhận ra tình huống này bằng dấu hiệu gì, làm gì>
```

## Template ADR

```markdown
---
id: adr-<số>-<slug>
status: proposed | accepted | superseded
date: <YYYY-MM-DD>
---
# <Quyết định>
**Bối cảnh:** <vấn đề, constraint>
**Quyết định:** <chọn gì>
**Phương án đã loại & lý do:** <ngắn gọn>
**Hệ quả:** <đánh đổi chấp nhận, khi nào cần xem lại>
```

## Chế độ audit — `/task-wrapup audit`

Chạy mỗi ~20 task hoặc mỗi tháng: quét `22-lessons-learned/`, ADR và các `refs` theo mục "Vệ sinh tri thức" trong `memory-update-policy.md`, trình danh sách đề xuất xóa/cập nhật cho người dùng duyệt. Tri thức sai nằm lại lâu làm agent kém đi theo thời gian — audit là một phần của vòng học, không phải việc phụ.
````

### Bước 4 — Khởi tạo `.claude/rules/coding-rules.md` từ codebase thật

Đây là bước duy nhất cần đọc code: quét codebase và rút ra **5–10 convention thật sự của project này** (cách đặt tên, cấu trúc module, error handling, pattern test, những gì codebase cố tình làm khác thông lệ). **Không chép best practice chung chung** — chỉ ghi điều mà một người mới sẽ làm sai nếu không được bảo.

Ngoài convention, file này bắt buộc có hai khối cứng:

- **Definition of Done** — task chỉ được coi là xong khi đã verify bằng lệnh test/chạy thật của project (ghi rõ lệnh cụ thể, ví dụ `npm test`, `make e2e`). Chưa verify thì không báo hoàn thành và không chạy `/task-wrapup`.
- **Luôn hỏi trước khi** — hành động không đảo ngược được (xóa dữ liệu, force push, drop bảng), đụng dữ liệu production, đụng luồng tiền (payment/refund), gọi service ngoài có side effect thật (gửi email/webhook thật).

Mỗi rule ≤ 3 dòng, kèm 1 ví dụ đường dẫn file thật minh họa. Vì `rules/` thuộc nhóm cần approval: trình bản draft cho người dùng duyệt trước khi ghi file.

### Bước 5 — Append vào `CLAUDE.md`

Nếu chưa có thì tạo; nếu có thì append, không sửa nội dung cũ:

```markdown
## Agent Learning System

- Trước mỗi task: phân loại task và đọc theo `.claude/context-routing.md` — không quét toàn bộ knowledge/repo.
- Rule bắt buộc của project: `.claude/rules/coding-rules.md`.
- Sau task đáng nhớ (quyết định lớn, bug khó, pattern mới): chạy `/task-wrapup`.
- Quyền ghi và approval: theo `.claude/memory-update-policy.md`. Không tự sửa file trong `.claude/` — chỉ đề xuất diff.
```

### Bước 6 — Verify và báo cáo

1. Toàn bộ file tồn tại đúng đường dẫn; frontmatter của SKILL.md parse được; routing không trỏ đến file chưa tồn tại.
2. `CLAUDE.md` ≤ 30 dòng sau khi append.
3. Báo cáo: file đã tạo, file chờ duyệt (coding-rules draft, di chuyển routing cũ nếu có), và hướng dẫn: *"Mở session mới; sau task lớn đầu tiên hãy gọi `/task-wrapup` — hệ thống chỉ có giá trị khi vòng ghi được chạy thật."*
4. Gợi ý (tùy chọn, không tự cài): có thể thêm hook `Stop` trong `.claude/settings.json` để nhắc chạy `/task-wrapup` sau mỗi phiên — để người dùng quyết định.

## Ràng buộc

- Token efficiency là tiêu chí số một: mọi file thiết kế để *đọc chọn lọc* (frontmatter lọc được, ≤ 30 dòng với history/lesson, routing giới hạn ~5 file/task).
- Không tạo file rỗng "để sẵn" — trừ `00-index/README.md`, file chỉ sinh ra khi có nội dung thật.
- Không ghi đè file tồn tại mà không hỏi; không sửa nội dung cũ của `CLAUDE.md`.
- Portable: knowledge nằm ở `docs/knowledge/` nên Cursor/Codex/agent CLI khác dùng được; với tool khác chỉ cần file rule của tool đó trỏ đến `context-routing.md` và `memory-update-policy.md`.
