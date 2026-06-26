# Giải thích file CSV import Jira cho backlog Ecommerce Backend

> File tham chiếu: `jira_backlog_fixed_import.csv`  
> Mục tiêu: giúp bạn hiểu **file CSV này đang làm gì**, **map vào Jira như thế nào**, và **Jira dùng để quản lý dự án ra sao** theo tầng kiến thức từ thấp lên cao.

---

## 1. Tóm tắt nhanh file CSV này

File CSV này là backlog của dự án backend ecommerce, đã được chỉnh theo hướng dễ import vào Jira Cloud hơn.

Tổng quan dữ liệu:

| Chỉ số | Giá trị |
|---|---:|
| Tổng work item | 146 |
| Tổng Story Point | 405 |
| Epic | 22 |
| Story | 62 |
| Task | 62 |
| Phase | 3 |
| Module | 19 |

Phân bổ theo work type:

| Work type | Số lượng |
|---|---:|
| Task | 62 |
| Story | 62 |
| Epic | 22 |

Phân bổ theo phase:

| Phase | Số lượng |
|---|---:|
| Phase 1 | 95 |
| Phase 2 | 38 |
| Phase 0 | 13 |

Phân bổ theo priority:

| Priority | Số lượng |
|---|---:|
| High | 85 |
| Medium | 40 |
| Low | 11 |
| Highest | 10 |

Phân bổ theo module:

| Module | Số lượng |
|---|---:|
| Catalog | 15 |
| Foundation | 12 |
| Payment | 11 |
| IAM | 10 |
| Search | 10 |
| Attribute | 9 |
| Order | 9 |
| Promotion | 7 |
| Cart | 7 |
| Inventory | 7 |
| QA | 7 |
| Admin | 6 |
| Notification | 6 |
| Security | 6 |
| Platform | 6 |
| Review | 5 |
| Personalization | 5 |
| Analytics | 4 |
| Shipping | 4 |

---

## 2. File CSV này sinh ra để làm gì?

Nói đơn giản:

> File CSV này là **bản danh sách công việc có cấu trúc** để import vào Jira, biến backlog trên file thành các work item thật trong Jira.

Trước khi import, backlog chỉ là bảng dữ liệu:

```txt
EPIC-00: Foundation & DevOps
ECM-001: Khởi tạo project Spring Boot
ECM-002: Cấu hình profile
...
```

Sau khi import vào Jira, mỗi dòng trở thành một work item có Jira key riêng:

```txt
ECOMMERCE-1: Foundation & DevOps
ECOMMERCE-24: Khởi tạo project Spring Boot
ECOMMERCE-25: Cấu hình profile
...
```

Điểm quan trọng:

```txt
ECOMMERCE-24 = mã Jira tự sinh
ECM-001      = mã task gốc của bạn, lưu trong custom field External Task ID
```

Hai mã này không giống nhau, nhưng đều có vai trò riêng.

---

## 3. Cấu trúc cột hiện tại của file CSV

Header file CSV đã fix:

```csv
Work item ID,Parent,External Task ID,Work type,Summary,Description,Acceptance Criteria,Priority,Story point estimate,Module,Dependency Codes,Responsible Role,Phase,Planned Sprint,Status
```

Ý nghĩa nhanh:

| Cột CSV | Ý nghĩa |
|---|---|
| `Work item ID` | ID kỹ thuật dạng số để Jira dùng nối quan hệ cha-con khi import. |
| `Parent` | Trỏ tới `Work item ID` của work item cha. |
| `External Task ID` | Mã gốc của bạn, ví dụ `EPIC-00`, `ECM-001`. |
| `Work type` | Loại công việc: `Epic`, `Story`, `Task`. |
| `Summary` | Tên ngắn của work item. Đây là tiêu đề chính trong Jira. |
| `Description` | Mô tả việc cần làm. |
| `Acceptance Criteria` | Tiêu chí nghiệm thu: thế nào là làm xong và đúng. |
| `Priority` | Độ ưu tiên: `Highest`, `High`, `Medium`, `Low`. |
| `Story point estimate` | Ước lượng độ lớn/độ khó/công sức theo Story Point. |
| `Module` | Module kỹ thuật/domain: Foundation, IAM, Catalog, Order... |
| `Dependency Codes` | Mã task phụ thuộc, ví dụ `ECM-044;ECM-057`. |
| `Responsible Role` | Vai trò phụ trách: Lead, BE, BE-Sr, DevOps, QA. |
| `Phase` | Giai đoạn: Phase 0, Phase 1, Phase 2. |
| `Planned Sprint` | Sprint dự kiến theo kế hoạch ban đầu. |
| `Status` | Trạng thái ban đầu, hiện là `Backlog`. |

---

## 4. Vì sao phải có `Work item ID` và `Parent`?

Đây là phần quan trọng nhất để hiểu import Jira.

Nếu file chỉ có:

```csv
External Task ID,Work type,Summary
EPIC-00,Epic,Foundation & DevOps
ECM-001,Task,Khởi tạo project Spring Boot
```

Jira sẽ tạo được item, nhưng không chắc biết `ECM-001` thuộc Epic nào.

Vì vậy file cần thêm ID kỹ thuật:

```csv
Work item ID,Parent,External Task ID,Work type,Summary
1,,EPIC-00,Epic,Foundation & DevOps
23,1,ECM-001,Task,Khởi tạo project Spring Boot
```

Đọc như sau:

```txt
Work item ID = 1
=> EPIC-00 là cha

Work item ID = 23, Parent = 1
=> ECM-001 là con của EPIC-00
```

Trong Jira sau import, kết quả mong muốn:

```txt
Foundation & DevOps
  ├── Khởi tạo project Spring Boot
  ├── Cấu hình profile
  ├── Docker compose local
  └── ...
```

---

## 5. Phân biệt 3 loại ID rất dễ nhầm

### 5.1. `Work item ID`

Đây là ID kỹ thuật chỉ phục vụ lúc import CSV.

Ví dụ:

```txt
1
2
3
23
24
25
```

Vai trò:

```txt
Jira dùng nó để biết item nào là cha, item nào là con.
```

Sau khi import xong, bạn thường không cần dùng nó nữa.

---

### 5.2. Jira Issue Key

Đây là mã Jira tự sinh sau khi import.

Ví dụ:

```txt
ECOMMERCE-1
ECOMMERCE-24
ECOMMERCE-25
```

Vai trò:

```txt
Dùng trong Jira, commit message, PR, link task, báo cáo, workflow.
```

Ví dụ commit:

```bash
git commit -m "ECOMMERCE-24 setup Spring Boot project"
```

---

### 5.3. `External Task ID`

Đây là mã task gốc của bạn.

Ví dụ:

```txt
EPIC-00
ECM-001
ECM-002
```

Vai trò:

```txt
Giữ mã backlog ban đầu.
Dùng để nói chuyện với AI agent.
Dùng để tra dependency trong file.
Dùng để đối chiếu giữa tài liệu, code, Jira.
```

Ví dụ khi nói với agent:

```txt
Hãy thực hiện task ECM-044: Reserve tồn kho atomic chống oversell.
```

Jira có thể đang gọi task đó là `ECOMMERCE-66`, nhưng bạn vẫn giữ được mã gốc `ECM-044`.

---

## 6. Mapping chuẩn khi import vào Jira

Khi import file CSV, map như sau:

| Cột CSV | Jira field |
|---|---|
| `Work item ID` | `Work item ID` hoặc `Issue ID` |
| `Parent` | `Parent` |
| `External Task ID` | Custom field: `External Task ID` |
| `Work type` | `Work type` hoặc `Issue Type` |
| `Summary` | `Summary` |
| `Description` | `Description` |
| `Acceptance Criteria` | Custom field: `Acceptance Criteria` |
| `Priority` | `Priority` |
| `Story point estimate` | `Story point estimate` hoặc `Story Points` |
| `Module` | Custom field: `Module` |
| `Dependency Codes` | Custom field: `Dependency Codes` |
| `Responsible Role` | Custom field: `Responsible Role` |
| `Phase` | Custom field: `Phase` |
| `Planned Sprint` | Custom field: `Planned Sprint` |
| `Status` | `Status` |

---

## 7. Custom field cần tạo trong Jira

Trước khi import, nên tạo các custom field sau:

| Custom field | Type nên dùng | Vì sao |
|---|---|---|
| `External Task ID` | Short text | Lưu mã `EPIC-00`, `ECM-001`. |
| `Acceptance Criteria` | Paragraph / multi-line text | AC thường dài, cần nhiều dòng. |
| `Module` | Single select | Module có danh sách cố định, tránh sai chính tả. |
| `Dependency Codes` | Paragraph / multi-line text | Lưu dependency dạng raw code. |
| `Responsible Role` | Single select | Role cố định: Lead, BE, BE-Sr, DevOps, QA. |
| `Phase` | Single select | Phase cố định: Phase 0, Phase 1, Phase 2. |
| `Planned Sprint` | Short text | Lưu sprint dự kiến, chưa phải sprint thật của Jira. |

Không nên tạo custom field cho:

```txt
Work type
Summary
Description
Priority
Status
Parent
```

Vì đây là field chuẩn của Jira.

---

## 8. Giá trị options nên tạo cho custom field

### 8.1. `Module`

```txt
Foundation
IAM
Catalog
Attribute
Inventory
Cart
Promotion
Order
Payment
Admin
Search
Security
Platform
QA
Notification
Shipping
Analytics
Personalization
Review
```

### 8.2. `Responsible Role`

```txt
Lead
BE
BE-Sr
DevOps
QA
```

### 8.3. `Phase`

```txt
Phase 0
Phase 1
Phase 2
```

### 8.4. `Priority`

Jira thường có sẵn:

```txt
Highest
High
Medium
Low
```

Nếu project không có cùng tên, khi import phải map value lại.

### 8.5. `Status`

File đang dùng:

```txt
Backlog
```

Nếu workflow Jira có status `Backlog`, map:

```txt
Backlog → Backlog
```

Nếu không có status `Backlog`, map tạm:

```txt
Backlog → To Do
```

---

## 9. Vì sao `Dependency Codes` chưa nên map thành issue link?

Trong CSV, dependency đang có dạng:

```txt
ECM-044;ECM-057;ECM-048
```

Nghĩa là task hiện tại phụ thuộc các task kia.

Ví dụ:

```txt
ECM-061: Tạo đơn từ giỏ hàng
Dependencies: ECM-044;ECM-057;ECM-048
```

Đọc là:

```txt
ECM-061 phụ thuộc:
- ECM-044: reserve tồn kho
- ECM-057: pricing engine
- ECM-048: cart model
```

Không nên map dependency thành issue link ngay trong lần import đầu vì lúc import, Jira key thật chưa ổn định.

Cách tốt hơn:

```txt
Lần 1: Import toàn bộ backlog.
Lần 2: Kiểm tra Jira key thật.
Lần 3: Convert Dependency Codes thành issue links: blocks / is blocked by.
```

Ví dụ sau import:

```txt
ECM-061 = ECOMMERCE-83
ECM-044 = ECOMMERCE-66
```

Khi đó mới tạo link:

```txt
ECOMMERCE-83 is blocked by ECOMMERCE-66
```

---

# 10. Overview Jira từ thấp nhất tăng dần

Phần này giúp bạn hiểu Jira từ gốc, không học theo kiểu nhớ vẹt.

---

## Cấp 1 — Field: thông tin nhỏ nhất

**Field** là một ô dữ liệu của work item.

Ví dụ:

```txt
Summary
Description
Priority
Status
Assignee
Story point estimate
Module
Phase
```

Tưởng tượng mỗi task là một dòng trong Excel, thì field chính là từng cột.

Ví dụ một dòng task:

| Field | Giá trị |
|---|---|
| Summary | Khởi tạo project Spring Boot |
| Priority | Highest |
| Status | Backlog |
| Story point estimate | 3 |
| Module | Foundation |

Field trả lời câu hỏi:

```txt
Task này tên gì?
Độ ưu tiên bao nhiêu?
Ai làm?
Đang ở trạng thái nào?
Thuộc module nào?
```

---

## Cấp 2 — Custom Field: trường riêng của team bạn

Jira có field mặc định, nhưng dự án nào cũng có thông tin riêng. Khi đó dùng custom field.

Ví dụ file này cần:

```txt
External Task ID
Acceptance Criteria
Module
Dependency Codes
Responsible Role
Phase
Planned Sprint
```

Custom field giúp Jira hiểu những thông tin riêng của backlog ecommerce.

Không có custom field thì các thông tin này có thể bị nhét hết vào Description, rất khó lọc, khó báo cáo.

---

## Cấp 3 — Work item: một đơn vị công việc

**Work item** là một việc cần quản lý.

Ví dụ:

```txt
Foundation & DevOps
Khởi tạo project Spring Boot
Docker compose môi trường local
API đăng nhập + phát hành JWT
Reserve tồn kho atomic chống oversell
```

Trong Jira, mỗi work item sẽ có một mã riêng:

```txt
ECOMMERCE-1
ECOMMERCE-24
ECOMMERCE-66
```

Work item là đơn vị bạn sẽ:

```txt
giao việc
ước lượng
kéo trạng thái
comment
gắn PR
test
đóng task
```

---

## Cấp 4 — Work Type / Issue Type: loại công việc

Không phải công việc nào cũng giống nhau.

Trong file này có 3 loại:

```txt
Epic
Story
Task
```

Hiểu đơn giản:

### Epic

Epic là nhóm việc lớn.

Ví dụ:

```txt
EPIC-01: Auth & User Management
```

Epic thường bao gồm nhiều story/task con:

```txt
- API đăng ký
- API đăng nhập
- JWT filter
- Refresh token
- User profile
```

### Story

Story là việc có giá trị theo góc nhìn người dùng hoặc nghiệp vụ.

Ví dụ:

```txt
API đăng nhập + phát hành JWT
```

Story thường trả lời:

```txt
Người dùng/actor làm được gì sau khi hoàn thành?
```

### Task

Task là việc kỹ thuật cụ thể.

Ví dụ:

```txt
SecurityConfig + JWT filter + RBAC
Docker compose môi trường local
BaseEntity + JPA Auditing
```

Task thường trả lời:

```txt
Dev cần làm phần kỹ thuật nào?
```

---

## Cấp 5 — Hierarchy: quan hệ cha-con

Hierarchy nghĩa là phân cấp công việc.

Trong file này:

```txt
Epic
  └── Story / Task
```

Ví dụ:

```txt
EPIC-00 Foundation & DevOps
  ├── ECM-001 Khởi tạo project Spring Boot
  ├── ECM-002 Cấu hình profile
  ├── ECM-003 Docker compose local
  └── ECM-011 Actuator health + base metrics
```

Trong CSV, hierarchy được tạo bằng:

```txt
Work item ID
Parent
Work type
```

Nếu thiếu một trong các cột này, import có thể mất quan hệ cha-con.

---

## Cấp 6 — Status: trạng thái hiện tại của công việc

Status trả lời:

```txt
Việc này đang ở đâu?
```

Ví dụ:

```txt
Backlog
To Do
In Progress
In Review
Testing
Done
```

File này đang set ban đầu:

```txt
Status = Backlog
```

Nghĩa là tất cả công việc mới nằm trong kho việc, chưa bắt đầu làm.

---

## Cấp 7 — Workflow: luật chuyển trạng thái

Workflow là đường đi của status.

Ví dụ workflow đơn giản:

```txt
Backlog → To Do → In Progress → Code Review → Testing → Done
```

Workflow trả lời:

```txt
Một task được phép đi qua những trạng thái nào?
Ai được chuyển trạng thái?
Khi nào coi là Done?
```

Ví dụ rule:

```txt
Chỉ QA được chuyển task từ Testing sang Done.
Không được Done nếu chưa có PR merged.
```

---

## Cấp 8 — Backlog: kho việc chưa làm

Backlog là nơi chứa tất cả việc cần làm nhưng chưa đưa vào sprint.

File CSV này sau import sẽ tạo một backlog lớn gồm:

```txt
Epic
Story
Task
```

Backlog dùng để:

```txt
ưu tiên
lọc theo module
chia sprint
ước lượng
lên kế hoạch release
```

---

## Cấp 9 — Sprint: khoảng thời gian làm việc

Sprint là một chu kỳ làm việc, thường 1 hoặc 2 tuần.

Ví dụ:

```txt
Sprint 0: Foundation
Sprint 1: IAM + Catalog basic
Sprint 2: Product + Variant
Sprint 3: Attribute + Inventory
```

File này có cột:

```txt
Planned Sprint
```

Nó là sprint dự kiến theo kế hoạch, chưa nhất thiết là sprint thật trong Jira.

Lý do không map thẳng vào Jira Sprint field ngay:

```txt
Jira Sprint field thường yêu cầu sprint thật đã tồn tại trong board.
Nếu chưa tạo sprint thật, import dễ lỗi.
```

Cách làm tốt:

```txt
Import vào Planned Sprint trước.
Sau đó vào Jira kéo task vào sprint thật.
```

---

## Cấp 10 — Board: màn hình quản lý dòng chảy công việc

Board là nơi bạn nhìn task theo cột trạng thái.

Ví dụ Kanban board:

```txt
Backlog | To Do | In Progress | Review | Done
```

Board giúp bạn thấy:

```txt
Task nào đang làm?
Task nào bị kẹt?
Ai đang có quá nhiều việc?
Sprint đang tiến triển thế nào?
```

---

## Cấp 11 — Project / Space: không gian quản lý toàn bộ dự án

Project/Space là container lớn chứa tất cả work item.

Ví dụ project của bạn:

```txt
ECOMMERCE
```

Jira sinh issue key theo project key:

```txt
ECOMMERCE-1
ECOMMERCE-2
ECOMMERCE-3
```

Project chứa:

```txt
work item
workflow
field
board
sprint
version
component
permission
automation
report
```

---

## Cấp 12 — Component / Module: nhóm kỹ thuật

Component trong Jira thường dùng để nhóm theo phần hệ thống.

Ví dụ:

```txt
IAM
Catalog
Order
Payment
Inventory
Search
```

Trong file này mình dùng custom field `Module` thay vì ép vào Component, vì module là domain kỹ thuật riêng của dự án.

Nếu sau này muốn Jira-native hơn, bạn có thể chuyển `Module` thành `Components`.

---

## Cấp 13 — Version / Release / Milestone

Jira có thể quản lý phiên bản release.

Ví dụ:

```txt
M1 - MVP Backend
M2 - Advanced Backend
```

Trong file này đang dùng `Phase`:

```txt
Phase 0
Phase 1
Phase 2
```

Nếu muốn quản lý release rõ hơn, bạn có thể tạo thêm `Fix versions`:

```txt
M1
M2
```

Cách phân biệt:

```txt
Phase = giai đoạn phát triển
Fix version = mốc release/bản phát hành
```

---

## Cấp 14 — JQL: ngôn ngữ tìm kiếm trong Jira

JQL là cách query task trong Jira.

Ví dụ tìm task Phase 1:

```jql
"Phase" = "Phase 1"
```

Tìm task module Order:

```jql
"Module" = Order
```

Tìm task priority Highest chưa done:

```jql
priority = Highest AND status != Done
```

Tìm task backend senior:

```jql
"Responsible Role" = "BE-Sr"
```

Tìm task có External Task ID:

```jql
"External Task ID" = "ECM-061"
```

---

## Cấp 15 — Report: báo cáo tiến độ

Khi dữ liệu trong Jira sạch, bạn có thể xem:

```txt
Sprint burndown
Velocity
Cumulative flow
Workload theo người
Task theo module
Task theo phase
Task theo priority
Story point còn lại
```

Điểm mấu chốt:

> Jira report chỉ đáng tin khi field, status, story point, sprint, assignee được nhập sạch.

Nếu data bẩn, report sẽ vô nghĩa.

---

# 11. Cách dùng Jira với AI Agent cho dự án này

Với dự án ecommerce backend, bạn nên dùng Jira như “bộ não quản lý việc”, còn AI agent là “dev hỗ trợ code”.

Flow khuyên dùng:

```txt
1. Chọn 1 task trong Jira
2. Copy External Task ID + Summary + Description + Acceptance Criteria
3. Gửi cho Agent lập plan
4. Review plan
5. Cho Agent code
6. Agent viết test
7. Chạy test
8. Gắn PR/commit với Jira key
9. Cập nhật status
10. Ghi note học được gì
```

Ví dụ prompt cho Agent:

```txt
Bạn là senior Java Spring Boot developer.
Hãy thực hiện Jira task:

Jira Key: ECOMMERCE-66
External Task ID: ECM-044
Summary: Reserve tồn kho atomic chống oversell
Description: Reserve atomic + optimistic lock + retry; chống bán âm
Acceptance Criteria:
- Test đồng thời N request mua 1 sản phẩm còn 1
- Đúng 1 request thành công
- Các request còn lại trả 409
- Không bán âm tồn kho

Yêu cầu:
- Giải thích plan trước khi code
- Không phá modular monolith boundary
- Viết integration test bằng Testcontainers
```

---

# 12. Cách kiểm tra import đã đúng chưa

Sau khi import vào Jira, kiểm tra 7 điểm:

## 12.1. Epic có task con chưa?

Mở Epic `Foundation & DevOps`, phải thấy các task con như:

```txt
Actuator health + base metrics
Spotless + Checkstyle + SonarQube gate
BaseEntity + JPA Auditing
...
```

Nếu có, tức là `Work item ID` và `Parent` đã map đúng.

---

## 12.2. Jira key có khác External Task ID không?

Đúng là phải khác.

Ví dụ:

```txt
Jira key: ECOMMERCE-24
External Task ID: ECM-001
```

Nếu bạn chỉ thấy `ECOMMERCE-24`, đó là Jira key.

Muốn thấy `ECM-001`, thêm cột custom field:

```txt
External Task ID
```

---

## 12.3. Story point có vào đúng không?

Mở một task như:

```txt
Reserve tồn kho atomic chống oversell
```

Kiểm tra:

```txt
Story point estimate = 8
```

---

## 12.4. Acceptance Criteria có vào đúng không?

Mở task và xem field:

```txt
Acceptance Criteria
```

Nếu trống, nghĩa là lúc import chưa map đúng custom field.

---

## 12.5. Module có vào đúng option không?

Ví dụ task IAM phải có:

```txt
Module = IAM
```

Task Order phải có:

```txt
Module = Order
```

---

## 12.6. Phase có đúng không?

Ví dụ:

```txt
Foundation task → Phase 0
MVP task → Phase 1
Advanced task → Phase 2
```

---

## 12.7. Status có đúng không?

Ban đầu hầu hết task nên ở:

```txt
Backlog
```

Nếu Jira workflow không có Backlog và bạn map sang To Do, thì toàn bộ sẽ nằm ở To Do.

---

# 13. Những lỗi thường gặp khi import CSV vào Jira

## Lỗi 1: Task không nằm dưới Epic

Nguyên nhân thường là:

```txt
Parent không trỏ đúng Work item ID của Epic cha.
Không map Work item ID.
Không map Work type.
Dùng import thường thay vì External System Import.
```

Cách fix:

```txt
Import lại bằng CSV có Work item ID, Parent, Work type.
```

---

## Lỗi 2: `ECM-001` không hiện ở cột link xanh

Không phải lỗi.

Cột link xanh là Jira issue key:

```txt
ECOMMERCE-24
```

Mã `ECM-001` nằm ở custom field:

```txt
External Task ID
```

---

## Lỗi 3: Acceptance Criteria trống

Nguyên nhân:

```txt
Chưa tạo custom field Acceptance Criteria.
Tạo rồi nhưng chưa add vào screen/work type.
Lúc import chưa map cột.
```

Cách fix:

```txt
Tạo field Paragraph.
Add vào Epic/Story/Task screen.
Import lại hoặc bulk update.
```

---

## Lỗi 4: Sprint không vào đúng

Nguyên nhân:

```txt
Bạn map vào Sprint thật nhưng sprint chưa tồn tại.
Jira cần sprint thuộc board cụ thể.
```

Cách nên làm:

```txt
Dùng custom field Planned Sprint.
Sau đó kéo task vào sprint thật trong Jira.
```

---

## Lỗi 5: Dependency chưa thành link

Không phải lỗi nếu bạn làm theo thiết kế này.

Cột:

```txt
Dependency Codes
```

chỉ lưu mã thô. Sau này mới convert thành issue link.

---

# 14. Cách hiểu file này theo góc nhìn quản lý dự án

Một dòng CSV không chỉ là một task. Nó chứa đầy đủ câu trả lời:

```txt
Việc gì?                    → Summary
Làm để đạt gì?              → Description
Thế nào là xong?            → Acceptance Criteria
Thuộc nhóm nào?             → Module
Quan trọng mức nào?         → Priority
Khó cỡ nào?                 → Story point estimate
Ai/role nào phù hợp làm?    → Responsible Role
Làm giai đoạn nào?          → Phase
Dự kiến sprint nào?         → Planned Sprint
Phụ thuộc việc nào?         → Dependency Codes
Đang ở trạng thái nào?      → Status
Thuộc Epic nào?             → Parent
Mã gốc là gì?               → External Task ID
```

Khi hiểu như vậy, Jira không còn là “tool tạo task”, mà là:

> Hệ thống quản lý dòng chảy công việc từ ý tưởng → backlog → sprint → implementation → review → testing → done.

---

# 15. Checklist vận hành Jira cho dự án này

## Trước sprint

```txt
1. Lọc Phase hiện tại.
2. Lọc Priority Highest/High.
3. Kiểm tra Dependencies.
4. Chọn task vào sprint.
5. Đảm bảo mỗi task có AC rõ.
6. Gán assignee thật.
7. Kiểm tra story point.
```

## Trong sprint

```txt
1. Dev kéo task từ To Do → In Progress.
2. Commit/PR gắn Jira key.
3. Khi code xong, kéo Review.
4. QA/test kéo Testing.
5. Đạt AC thì Done.
```

## Sau sprint

```txt
1. Xem task Done.
2. Xem task carry-over.
3. Xem velocity.
4. Cập nhật estimate nếu lệch.
5. Cập nhật backlog cho sprint sau.
```

---

# 16. Tư duy đúng khi dùng Jira

Không nên dùng Jira kiểu:

```txt
Tạo task cho có.
Ghi mô tả mơ hồ.
Không update status.
Không có AC.
Không có estimate.
Không liên kết PR.
```

Nên dùng Jira kiểu:

```txt
Mỗi task có mục tiêu rõ.
Có Acceptance Criteria.
Có owner.
Có estimate.
Có dependency.
Có trạng thái thật.
Có bằng chứng hoàn thành: PR, test, demo.
```

Jira chỉ có giá trị khi nó phản ánh đúng thực tế dự án.

---

# 17. Nguồn tham khảo chính thức

Các khái niệm trong tài liệu này dựa trên cách Jira Cloud hiện đang hướng dẫn về CSV import, hierarchy, custom fields và field mapping:

- Atlassian — Import data from a CSV file  
  https://support.atlassian.com/jira-cloud-administration/docs/import-data-from-a-csv-file/

- Atlassian — Import data to a software project using a CSV file  
  https://support.atlassian.com/jira-software-cloud/docs/import-data-to-a-software-project-using-a-csv-file/

- Atlassian — Keep work parent-child relationships during CSV import to Jira Cloud  
  https://support.atlassian.com/jira/kb/keep-issue-parent-child-mapping-during-csv-import-to-jira-cloud/

- Atlassian — Available custom fields for team-managed projects  
  https://support.atlassian.com/jira-software-cloud/docs/available-custom-fields-for-team-managed-projects/

- Atlassian — Create a custom field  
  https://support.atlassian.com/jira-cloud-administration/docs/create-a-custom-field/

---

# 18. Kết luận

File CSV này đã được thiết kế theo hướng:

```txt
1. Jira hiểu được Epic → Story/Task.
2. Bạn vẫn giữ mã gốc EPIC-xx / ECM-xxx.
3. Có đủ dữ liệu để quản lý sprint, phase, module, role, story point.
4. Có custom field để không nhét mọi thứ vào Description.
5. Có nền tảng để sau này tự động hóa với AI agent.
```

Điểm cần nhớ nhất:

```txt
Work item ID      = ID kỹ thuật để import hierarchy
Parent            = trỏ tới Work item ID của cha
External Task ID  = mã task gốc của bạn
Jira key          = mã Jira tự sinh sau import
```

Khi đã hiểu được 4 thứ này, bạn sẽ không bị nhầm giữa `ECOMMERCE-24` và `ECM-001` nữa.
