Hãy tự review phần code bạn vừa implement với vai trò Senior Frontend Engineer.

Kiểm tra theo checklist sau:

1. Đúng requirement chưa?

* Màn hình có đúng mockup không?
* Route đúng chưa?
* Layout đúng chưa?
* Các text/button/form/table/card có thiếu gì không?

2. Componentization

* Có copy-paste HTML quá nhiều không?
* Component nào nên tách thêm?
* Component nào bị tách quá nhỏ không cần thiết?
* Props và events đã rõ ràng chưa?

3. Data/state

* Dữ liệu tĩnh đã được chuyển thành props/state/API chưa?
* Có hard-code danh sách sản phẩm/order/user không?
* Có loading state chưa?
* Có empty state chưa?
* Có error state chưa?

4. API integration

* API call đặt đúng nơi chưa?
* Có xử lý lỗi API chưa?
* Có tránh gọi API lặp không cần thiết không?
* Endpoint nào đang TODO/mock?

5. Form/action

* Form có validation chưa?
* Submit có loading/disabled chưa?
* Error message có rõ ràng không?
* User action có feedback chưa?

6. Auth/permission

* Page cần login đã có middleware chưa?
* Page admin đã dùng layout/admin guard chưa?
* Có lộ dữ liệu admin ra public không?

7. Maintainability

* Tên file/component có rõ nghĩa không?
* Code có dễ đọc không?
* Có magic string/magic number không?
* Có console.log/dead code không?
* Có logic nào nên đưa vào service/composable/store không?

8. UI/UX

* Spacing, màu sắc, font, responsive có sát mockup không?
* Mobile có vỡ layout không?
* Button/input có accessible không?
* Image có alt không?

9. Kết luận
   Hãy trả lời theo format:

* Tổng quan chất lượng:
* File đã tạo/sửa:
* Điểm đã đạt:
* Vấn đề còn tồn tại:
* Đề xuất refactor:
* TODO còn lại:
* Lệnh cần chạy để kiểm tra:
* Có thể chuyển Jira sang Done chưa? Vì sao?
