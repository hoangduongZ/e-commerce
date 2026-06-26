MCP sẽ biến Jira thành “bộ não vận hành”, còn agent là “dev phụ”.
1. Bạn chọn task trong Jira
2. Agent đọc task qua MCP
3. Agent đọc External Task ID, Summary, Description, Acceptance Criteria
4. Agent kiểm tra Dependency Codes
5. Agent lập plan
6. Bạn duyệt plan
7. Agent code trong repo
8. Agent chạy test
9. Agent comment kết quả vào Jira
10. Agent chuyển trạng thái: Backlog → In Progress → Code Review/Done

Rule:
Agent được đọc Jira.
Agent được comment Jira.
Agent được chuyển status.
Agent không được xóa issue.
Agent không được bulk update quá 5 issue nếu chưa hỏi.
Agent không được tự đổi Priority/Phase/Sprint nếu chưa hỏi.

Các hành động được phép:
Read issue
Search issue
Add comment
Update status
Update description/checklist khi bạn yêu cầu
Create sub-task khi bạn yêu cầu

Các hành động nên bắt xác nhận:
Delete issue
Bulk edit
Change parent
Change sprint hàng loạt
Close nhiều issue cùng lúc

MCP chính thức: Atlassian Rovo MCP Server chính thức
Server url chính thức: https://mcp.atlassian.com/v1/mcp