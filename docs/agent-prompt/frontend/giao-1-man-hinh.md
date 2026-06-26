Tôi giao cho bạn chuyển một mockup HTML thành màn hình Nuxt.js thật.

Thông tin task:

* Jira/Task ID: {{TASK_ID}}
* Tên màn hình: {{SCREEN_NAME}}
* File mockup HTML: {{HTML_FILE_PATH_OR_CONTENT}}
* Route mong muốn: {{ROUTE}}
* Layout mong muốn: {{LAYOUT_DEFAULT_ADMIN_AUTH}}
* Nuxt version: {{NUXT_2_OR_NUXT_3}}
* UI library/style system đang dùng: {{TAILWIND_ANT_DESIGN_VUE_VUETIFY_OR_OTHER}}
* API backend liên quan nếu có:

  * {{API_LIST}}
* Ghi chú nghiệp vụ:

  * {{BUSINESS_NOTES}}

Yêu cầu trước khi code:

1. Đọc mockup HTML và phân tích màn hình.
2. Tóm tắt mục đích màn hình.
3. Chỉ ra các block UI chính trong mockup.
4. Đề xuất route Nuxt chính xác.
5. Đề xuất layout sử dụng.
6. Đề xuất danh sách component cần tạo hoặc tái sử dụng.
7. Chỉ ra phần nào trong HTML đang là dữ liệu tĩnh cần chuyển thành:

   * props
   * state
   * API response
   * store/composable
8. Xác định các state bắt buộc:

   * loading
   * empty
   * error
   * success
   * validation nếu có form
9. Xác định event/action:

   * click
   * submit
   * search
   * filter
   * pagination
   * upload
   * modal open/close
10. Lập implementation plan từng bước.

Chưa code cho đến khi tôi duyệt plan.

Sau khi tôi duyệt plan, hãy implement theo quy tắc:

* Giữ UI sát mockup.
* Không hard-code danh sách dữ liệu.
* Tách component hợp lý.
* Code theo convention hiện có của project.
* Không phá layout/component cũ.
* Không bịa API nếu chưa có thông tin.
* Nếu API chưa sẵn sàng, tạo mock data/service tạm và đánh dấu TODO rõ ràng.
* Có xử lý loading/empty/error.
* Có responsive cơ bản.
* Có validation nếu form.
* Có comment tiếng Việt cho phần logic khó.
* Sau khi xong, tự review code và báo cáo file đã sửa.
