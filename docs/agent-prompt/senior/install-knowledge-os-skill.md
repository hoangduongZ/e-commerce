# install-knowledge-os-skill — Cài đặt Engineering Knowledge OS vào `.claude` dưới dạng skill

> **Prompt này dành cho AI Agent thực thi.** Nhiệm vụ chỉ là **cài đặt** skill vào `.claude` — KHÔNG sinh knowledge base. Việc sinh knowledge do người dùng gọi `/knowledge-os` sau khi cài xong.

## Nhiệm vụ

Cài đặt prompt "Engineering Knowledge OS" (nội dung tương đương bản one-shot `knowledge-os-oneshot.md` cùng thư mục; file installer này là nguồn sự thật, bản one-shot là bản trích) thành một **skill của Claude Code** theo cấu trúc:

```text
{{PROJECT_ROOT}}/
  .claude/
    skills/
      knowledge-os/
        SKILL.md            # quy trình 3 giai đoạn — nạp khi gọi /knowledge-os
        case-template.md    # template 8 mục — chỉ đọc khi vào Giai đoạn 2
        kb-structure.md     # cấu trúc thư mục + spec index/routing
  CLAUDE.md                 # chỉ thêm vài dòng pointer, không copy nội dung
  docs/
    knowledge/              # output tri thức sẽ nằm ở đây (chưa tạo ở bước này)
```

Lý do thiết kế (để hiểu, không cần output lại): `CLAUDE.md` nạp vào context mọi session nên chỉ chứa pointer; skill nạp theo nhu cầu khi được gọi; knowledge sinh ra là tài sản của repo nên nằm ở `docs/knowledge/`, không nằm trong `.claude`.

## Các bước thực hiện

### Bước 0 — Kiểm tra trước

1. `{{PROJECT_ROOT}}` = root của project hiện tại (thư mục chứa `.git` gần nhất).
2. Nếu `.claude/skills/knowledge-os/` **đã tồn tại** → dừng lại, báo người dùng và hỏi có ghi đè không. Không tự ghi đè.

### Bước 1 — Tạo `.claude/skills/knowledge-os/SKILL.md`

Ghi đúng nội dung sau:

````markdown
---
name: knowledge-os
description: Build and maintain the Engineering Knowledge OS — a markdown knowledge base capturing senior-engineer decision thinking per case (architecture, database, API design, security, ...) for AI agents to read per task. Use when the user asks to build or update the knowledge base, analyze an engineering case, or set up agent knowledge routing.
---

# Engineering Knowledge OS

Bạn là Principal Software Engineer kiêm AI Knowledge Engineer. Nhiệm vụ: **reverse-engineer tư duy của Senior Engineer** khi xây dựng một ứng dụng hoàn toàn mới — từ con số 0 đến production, vận hành và bảo trì lâu dài — rồi chuyển hóa thành Knowledge Base Markdown mà AI Agent đọc được theo từng task.

Đây không phải tài liệu lý thuyết và không phải checklist bề mặt. Mỗi file sinh ra phải trả lời được: *đứng trước quyết định này, Senior nghĩ gì, chọn gì, và vì sao*.

## Đường dẫn

| Placeholder | Ý nghĩa | Mặc định |
|---|---|---|
| `{{KNOWLEDGE_ROOT}}` | Thư mục knowledge chính | `docs/knowledge` (tính từ root project) |
| `{{DECISION_RECORDS_ROOT}}` | Nơi lưu ADR / decision records | `{{KNOWLEDGE_ROOT}}/19-decision-records` |

- `{{KNOWLEDGE_ROOT}}` **chưa tồn tại** → dự án chưa triển khai, bắt đầu từ Giai đoạn 1.
- **Đã tồn tại** → đọc `00-index/` trước, chỉ bổ sung case còn thiếu. Không ghi đè file đã có.

## Quy trình — 3 giai đoạn, không gộp làm một

### Giai đoạn 1 — Bản đồ case

Đọc `kb-structure.md` (cùng thư mục skill) để nắm cấu trúc thư mục. Liệt kê các case một Senior gặp khi xây app mới, nhóm theo lifecycle, gắn tier ưu tiên:

- **Tier 1 (làm trước — sai thì rất đắt để sửa):** khởi tạo project, chọn tech stack, architecture, chia module, database design, API design, authentication/authorization, security baseline, CI/CD, testing strategy.
- **Tier 2:** cache, queue, search, logging, monitoring, file storage, notification, payment, performance, Docker.
- **Tier 3:** scalability, multi-tenant, i18n, Kubernetes, cloud, cost optimization, legacy migration, incident response, refactoring, AI integration.

Danh sách trên là gợi ý — thêm/bớt theo context project thực tế. Output của giai đoạn này **chỉ là bản đồ**: tên case, 1 dòng mô tả, tier, đường dẫn file dự kiến. Chưa phân tích chi tiết.

### Giai đoạn 2 — Phân tích từng case, mỗi case một file

Đọc `case-template.md` (cùng thư mục skill) trước khi viết file đầu tiên. Mỗi lượt chỉ xử lý **1–3 case**, theo thứ tự tier. Với mỗi case: ghi một file `.md` vào đúng thư mục, đúng template. Xong lượt nào cập nhật index lượt đó rồi mới sang case tiếp theo — không dồn tất cả vào một lần output.

### Giai đoạn 3 — Index & routing cho Agent

Khi đã đủ case Tier 1, tạo các file theo spec trong `kb-structure.md`:

- `00-index/README.md` — bảng toàn bộ file.
- **Routing** — quy tắc canonical: nếu `.claude/context-routing.md` **đã tồn tại** (learning-loop đã cài), KHÔNG tạo `20-agent-context/routing.md` song song — chỉ đề xuất diff cập nhật `.claude/context-routing.md` (đổi các mục `(chưa có)` thành đường dẫn file knowledge vừa sinh), chờ người dùng duyệt theo memory-update-policy. Chỉ khi routing canonical CHƯA tồn tại mới tạo `20-agent-context/routing.md`.
- Đề xuất (≤ 10 dòng) bổ sung vào `CLAUDE.md` để Agent biết tra index trước khi làm task — không copy cả knowledge base vào đó.
````

### Bước 2 — Tạo `.claude/skills/knowledge-os/case-template.md`

Ghi đúng nội dung sau:

````markdown
# Template cho mỗi file case

## Frontmatter bắt buộc — để Agent/RAG lọc được mà không cần đọc thân file

```markdown
---
id: <slug>
title: <tên case>
tier: 1 | 2 | 3
tags: [architecture, database, ...]
when_to_read: <Agent cần đọc file này khi làm loại task gì>
related: [<id các file liên quan>]
updated: <YYYY-MM-DD>
---
```

## Thân file — đủ 8 mục, mỗi mục viết để ra quyết định được, không viết cho đủ chữ

1. **Bối cảnh & vấn đề** — case này giải quyết gì; làm sai thì hậu quả gì đến product, engineering, business, security, cost, operation.
2. **Senior nghĩ gì khi gặp case này** — họ tự hỏi câu gì, ưu tiên gì, sợ rủi ro gì; cái gì phải quyết ngay, cái gì hoãn được, cái gì tuyệt đối không làm.
3. **Luồng tư duy từng bước** — xác định bản chất vấn đề → constraint → đánh giá option → chọn hướng → thiết kế để dễ thay đổi → rủi ro vận hành. Mỗi bước phải giải thích *vì sao Senior nghĩ vậy*, không chỉ *làm gì*.
4. **Các option & trade-off** — với mỗi option: khi nào dùng, khi nào không, ưu/nhược, trade-off, sai lầm thường gặp. (Ví dụ: REST vs GraphQL vs gRPC; SQL vs NoSQL; monolith vs modular monolith vs microservice; queue vs sync API; serverless vs container vs VM.)
5. **Sai lầm Junior hay mắc** — cụ thể theo case này kèm hậu quả thật: over/under-engineering, copy architecture big-tech sai context, premature optimization, không nghĩ đến failure case, không nghĩ đến vận hành production, database khó migrate, bỏ qua security.
6. **Dấu hiệu thiết kế hiện tại không còn phù hợp** — tín hiệu từ performance, team velocity, bug production, cost, vận hành, business requirement.
7. **Checklist quyết định** — các câu hỏi Senior phải trả lời được trước khi chốt phương án; viết dạng câu hỏi có/không, kèm ngưỡng cụ thể khi có thể.
8. **Học sâu hơn** — 3–7 tài liệu đáng đọc nhất (official docs, RFC, engineering blog, book, postmortem, open source codebase), kèm một dòng lý do đọc. Không liệt kê tràn lan.

## Tiêu chí chất lượng — áp dụng cho mọi file

- Ưu tiên **quyết định** hơn mô tả: đoạn nào không giúp Agent chọn được hướng đi thì cắt.
- Cụ thể hơn tổng quát: nêu con số, ngưỡng, failure mode thật — không viết "cần cân nhắc kỹ lưỡng".
- Luôn nêu trade-off. Không có lựa chọn nào là "best practice" vô điều kiện.
- Ghi rõ khi nào nên **đơn giản hóa**, khi nào nên **thiết kế dài hạn**, và khi nào Agent nên **dừng lại hỏi người dùng** thay vì tự quyết.
- Quyết định kiến trúc lớn → ghi ADR vào `{{DECISION_RECORDS_ROOT}}`.
- File dài quá ~200 dòng → tách nhỏ, link chéo qua `related`.
````

### Bước 3 — Tạo `.claude/skills/knowledge-os/kb-structure.md`

Ghi đúng nội dung sau:

````markdown
# Cấu trúc Knowledge Base

```text
{{KNOWLEDGE_ROOT}}/
  00-index/            # bảng routing — Agent đọc file này đầu tiên
  01-product-thinking/
  02-architecture/
  03-backend/
  04-frontend/
  05-database/
  06-infrastructure/
  07-security/
  08-testing/
  09-observability/
  10-performance/
  11-scalability/
  12-devops/
  13-incident-response/
  14-ai-engineering/
  15-cost-optimization/
  16-refactoring/
  17-patterns/
  18-anti-patterns/
  19-decision-records/ # ADR
  20-agent-context/    # rule & routing riêng cho Agent
```

Yêu cầu thiết kế: Agent đọc đúng file theo task mà không cần nạp toàn bộ knowledge base; mở rộng được tới hàng trăm file; dễ index bằng RAG hoặc Context Engine; dễ cập nhật sau mỗi task thực tế.

## Spec file index — `00-index/README.md`

Bảng toàn bộ file trong knowledge base, mỗi dòng gồm: đường dẫn, mục đích (1 câu), khi nào Agent cần đọc, tags, độ ưu tiên khi nạp context (P1/P2/P3). Cập nhật ngay mỗi khi thêm/sửa file — index sai còn tệ hơn không có index.

## Spec file routing — `20-agent-context/routing.md`

Mapping *loại task → danh sách file cần đọc*, sắp theo độ ưu tiên. Ví dụ:

| Loại task | File cần đọc |
|---|---|
| Thêm API mới | `api-design`, `security-baseline` |
| Thay đổi schema DB | `database-design`, `19-decision-records/` liên quan |
| Bug production | `incident-response`, `logging`, `monitoring` |
| Chọn công nghệ mới | `tech-stack`, `architecture`, ADR liên quan |

Chỉ liệt kê task type đã có file tương ứng — không tạo mapping trỏ đến file chưa tồn tại.
````

### Bước 4 — Thêm pointer vào `CLAUDE.md` của project

Nếu `{{PROJECT_ROOT}}/CLAUDE.md` chưa tồn tại thì tạo mới; nếu đã có thì **append**, không sửa nội dung cũ:

```markdown
## Knowledge Base

Trước khi làm task thiết kế/kiến trúc (thêm module, chọn công nghệ, thay đổi schema...), tra `docs/knowledge/00-index/README.md` và đọc đúng file theo routing canonical `.claude/context-routing.md` (nếu chưa cài learning-loop thì theo `docs/knowledge/20-agent-context/routing.md`). Để xây hoặc bổ sung knowledge base, gọi skill `/knowledge-os`.
```

> Lưu ý routing canonical: nếu repo đã có `.claude/context-routing.md`, KHÔNG trỏ pointer sang `20-agent-context/routing.md`. Một khái niệm chỉ có một chỗ — trỏ hai nơi là mầm mống drift.

### Bước 5 — Kiểm tra và báo cáo

Verify là hành động **chạy được**, không phải lời hứa — chạy các lệnh sau và dán kết quả vào báo cáo:

1. `ls .claude/skills/knowledge-os/` → xác nhận đủ 3 file `SKILL.md`, `case-template.md`, `kb-structure.md`.
2. Parse frontmatter `SKILL.md` (vd `python3 -c "import re,sys; t=open('.claude/skills/knowledge-os/SKILL.md').read(); m=re.match(r'^---\n(.*?)\n---',t,re.S); assert m and 'name:' in m.group(1) and 'description:' in m.group(1)"`) → phải không lỗi.
3. `ls docs/knowledge 2>/dev/null` → xác nhận **không** tạo knowledge content ở bước cài (chỉ skill nằm trong `.claude/`).
4. Nếu có sửa `CLAUDE.md`: `wc -l CLAUDE.md` → xác nhận vẫn trong budget (≤ ~30 dòng).
5. Báo cáo cho người dùng: danh sách file đã tạo, thay đổi trong `CLAUDE.md`, và hướng dẫn 1 dòng: *"Mở session mới (hoặc reload) rồi gọi `/knowledge-os` để bắt đầu Giai đoạn 1."*

## Cập nhật bản đã cài (khi bộ prompt ra bản mới)

Bước 0 chặn ghi đè để bảo vệ tùy biến của người dùng — nhưng "chỉ bổ sung" nghĩa là bản cải tiến không tự vào lại repo đã cài. Khi cần nâng cấp: so nội dung 3 file skill hiện có với bản mới, **trình diff cho người dùng duyệt** (skill thuộc `.claude/` → nhóm cần approval), không tự ghi đè. Nếu muốn tự động phát hiện lệch, thêm dòng `template_version: <n>` vào đầu mỗi file và so version.

## Ràng buộc

- Không ghi đè bất kỳ file nào đã tồn tại mà không hỏi trước.
- Không tạo `docs/knowledge/` ở bước cài đặt — thư mục đó do skill tạo khi chạy thật.
- Không sửa nội dung khác trong `CLAUDE.md` ngoài phần append ở Bước 4.
