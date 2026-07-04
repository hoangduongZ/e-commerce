> ⚠️ **DEPRECATED — bản gốc, chỉ lưu trữ để so sánh. KHÔNG dùng để chỉ đạo agent.**
> Bản đã tối ưu để chạy: `install-knowledge-os-skill.md` (cài thành skill) hoặc `knowledge-os-oneshot.md` (one-shot). File này giữ lại để đối chiếu lịch sử tiến hóa của prompt.

# Prompt

Bạn là một Principal Software Engineer từng làm ở Microsoft/Google/Amazon/Meta, đồng thời là Software Architect, Tech Lead và AI Knowledge Engineer.

## Mục tiêu

Hãy tổng hợp và hệ thống hóa toàn bộ những tình huống mà một Senior Software Engineer thường gặp khi xây dựng **một ứng dụng hoàn toàn mới**, từ giai đoạn chưa có gì cho đến production, vận hành, mở rộng và bảo trì lâu dài.

Tôi **không cần checklist đơn giản**.

Điều tôi cần là **mô hình tư duy của Senior Engineer** ở từng tình huống để có thể chuyển hóa thành hệ thống tri thức `.md` cho AI Agent sử dụng về sau.

Mục tiêu cuối cùng là xây dựng một **Engineering Knowledge Operating System** dưới dạng Markdown Knowledge Base.

# Project Knowledge Location
Trước khi thực hiện, hãy dùng các placeholder sau để xác định vị trí lưu Knowledge
{{PROJECT_ROOT}} = <đường dẫn root của project> 
{{KNOWLEDGE_ROOT}} = <đường dẫn thư mục knowledge chính> 
{{AGENT_RULES_ROOT}} = <đường dẫn thư mục rule cho AI Agent, ví dụ .claude / .cursor / .github / docs/agent> 
{{DECISION_RECORDS_ROOT}} = <đường dẫn lưu ADR/decision records>
---

# Yêu cầu phân tích

Với mỗi case, hãy phân tích theo cấu trúc sau:

## 1. Case

Ví dụ:

* Khởi tạo project
* Chọn tech stack
* Thiết kế architecture
* Chia module
* Authentication
* Authorization
* Database design
* API design
* Queue
* Cache
* Search
* Logging
* Monitoring
* CI/CD
* Docker
* Kubernetes
* Cloud
* AI integration
* Payment
* Notification
* File storage
* Security
* Multi-tenant
* Internationalization
* Testing
* Refactoring
* Performance
* Scalability
* Production bug
* Incident response
* Cost optimization
* Legacy migration

Không giới hạn số lượng case.

---

## 2. Senior nghĩ gì khi gặp case này?

Hãy reverse engineer luồng suy nghĩ của Senior:

* Họ tự hỏi những câu nào?
* Họ ưu tiên điều gì?
* Họ lo sợ rủi ro gì?
* Họ cân nhắc trade-off nào?
* Điều gì cần quyết định ngay?
* Điều gì có thể trì hoãn?
* Điều gì tuyệt đối không nên làm?

---

## 3. Mục tiêu của case

Giải thích:

* Case này giải quyết vấn đề gì?
* Vì sao nó quan trọng?
* Nếu làm sai sẽ gây hậu quả gì?
* Nó ảnh hưởng đến product, engineering, business, security, cost, operation như thế nào?

---

## 4. Luồng tư duy Senior

Viết thành từng bước rõ ràng.

Không chỉ nói “làm gì”, mà phải giải thích **vì sao Senior nghĩ như vậy**.

Format gợi ý:

### Bước 1 — Xác định bản chất vấn đề

### Bước 2 — Xác định constraint

### Bước 3 — Đánh giá option

### Bước 4 — Chọn hướng phù hợp

### Bước 5 — Thiết kế để dễ thay đổi

### Bước 6 — Xác định rủi ro vận hành

---

## 5. Các lựa chọn phổ biến

Liệt kê các option thường gặp.

Ví dụ:

* REST vs GraphQL vs gRPC
* Monolith vs Modular Monolith vs Microservice
* SQL vs NoSQL
* Queue vs Sync API
* Cache local vs Redis vs CDN
* Serverless vs Container vs VM
* Manual deployment vs CI/CD
* Vertical scaling vs Horizontal scaling

Với mỗi option, hãy phân tích:

* Khi nào nên dùng
* Khi nào không nên dùng
* Ưu điểm
* Nhược điểm
* Trade-off
* Sai lầm thường gặp

---

## 6. Sai lầm Junior thường mắc

Ví dụ:

* Over-engineering
* Under-engineering
* Copy architecture từ công ty lớn mà không hiểu context
* Premature optimization
* Hardcode
* Tight coupling
* Không nghĩ đến failure case
* Không nghĩ đến vận hành production
* Không có logging/monitoring
* Không viết test cho phần quan trọng
* Thiết kế database khó migrate
* Không tính đến security

---

## 7. Dấu hiệu cần thay đổi

Phân tích:

* Khi nào thiết kế hiện tại không còn phù hợp?
* Những tín hiệu nào Senior sẽ nhận ra?
* Dấu hiệu từ performance
* Dấu hiệu từ team velocity
* Dấu hiệu từ bug production
* Dấu hiệu từ cost
* Dấu hiệu từ vận hành
* Dấu hiệu từ business requirement

---

## 8. Checklist quyết định

Liệt kê các câu hỏi Senior cần tự hỏi trước khi chốt phương án.

Checklist phải giúp AI Agent ra quyết định tốt hơn, không chỉ kiểm tra bề mặt.

---

## 9. Kiến thức cần học

Liệt kê các nhóm kiến thức cần học để hiểu sâu case này:

* Framework
* Design pattern
* Architecture pattern
* Distributed system
* Database
* Networking
* Cloud
* Security
* DevOps
* SRE
* Testing
* Product thinking
* Cost optimization

---

## 10. Tài liệu nên đọc

Gợi ý loại tài liệu nên đọc:

* Official docs
* RFC
* Engineering blog
* Books
* Papers
* Open source codebase
* Postmortem
* Case study

Ưu tiên tài liệu có giá trị thực chiến.

---

# Project Knowledge Location
Trước khi thực hiện, hãy dùng các placeholder sau để xác định vị trí lưu Knowledge, nếu chưa tồn tại có thì đó có thể là dự án chưa được triển khai tư duy này, đi xuống khối 'Yêu cầu tổ chức Knowledge Base' và thực hiện
{{PROJECT_ROOT}} = <đường dẫn root của project> 
{{KNOWLEDGE_ROOT}} = <đường dẫn thư mục knowledge chính> 
{{AGENT_RULES_ROOT}} = <đường dẫn thư mục rule cho AI Agent, ví dụ .claude / .cursor / .github / docs/agent> 
{{DECISION_RECORDS_ROOT}} = <đường dẫn lưu ADR/decision records>

# Yêu cầu tổ chức Knowledge Base

Sau khi phân tích các case, hãy đề xuất cấu trúc thư mục Markdown tối ưu để lưu tri thức.

Mục tiêu:

* Có thể mở rộng thành hàng trăm file `.md`
* AI Agent có thể đọc đúng file theo task
* Không cần nạp toàn bộ knowledge base
* Dễ index bằng RAG hoặc Context Engine
* Dễ maintain theo thời gian
* Dễ cập nhật sau mỗi task thực tế

Ví dụ cấu trúc:

```text
knowledge/
  00-index/
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
  19-decision-records/
  20-agent-context/
```

---

# Với mỗi file Markdown hãy đề xuất

* Tên file
* Đường dẫn
* Mục đích
* Nội dung chính
* Khi nào AI Agent cần đọc file này
* File liên quan
* Tags để index
* Độ ưu tiên khi nạp context

---

# Output mong muốn

Hãy output theo 3 phần:

## Phần 1 — Bản đồ tổng quan các nhóm case Senior thường gặp

## Phần 2 — Phân tích chi tiết từng case theo format trên

## Phần 3 — Đề xuất cấu trúc Knowledge Base Markdown cho AI Agent

---

# Nguyên tắc quan trọng

Không chỉ mô tả kiến thức.

Hãy cố gắng **reverse engineer tư duy của Senior Software Engineer**.

Mục tiêu là để AI Agent học được cách:

* Suy nghĩ
* Phân tích
* Đánh giá trade-off
* Nhận diện rủi ro
* Ra quyết định
* Biết khi nào cần hỏi lại
* Biết khi nào nên đơn giản hóa
* Biết khi nào cần thiết kế dài hạn
* Biết khi nào cần ghi lại quyết định vào Markdown

Ưu tiên nội dung thực tế, có thể áp dụng vào dự án thật, không viết chung chung.
