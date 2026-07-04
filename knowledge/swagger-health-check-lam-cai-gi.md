## Mục đích các URL local

| URL | Mục đích |
|---|---|
| `http://localhost:8080/swagger-ui.html` | Mở giao diện **Swagger UI** để xem, test và document toàn bộ REST API của backend. FE/dev/QA có thể dùng để biết endpoint nào có, request/response ra sao. |
| `http://localhost:8080/actuator/health` | Kiểm tra **trạng thái sống/chết của ứng dụng**. Dùng để biết app có đang chạy ổn không, DB/Redis có kết nối được không. Có thể kèm `/liveness` và `/readiness` để phục vụ deploy/monitoring. |
| `http://localhost:8080/actuator/prometheus` | Xuất **metrics dạng Prometheus** để hệ thống monitoring thu thập, ví dụ số request, thời gian phản hồi, lỗi, memory, JVM, thread, database pool. Dùng cho local/dev/staging để quan sát hiệu năng hệ thống. |

## Hiểu ngắn gọn

- `swagger-ui.html` → dành cho **developer/test API**.
- `actuator/health` → dành cho **health check / deployment / uptime check**.
- `actuator/prometheus` → dành cho **monitoring / metrics / performance tracking**.

## Uptime check là gì?

**Uptime check** là việc hệ thống tự động kiểm tra xem ứng dụng/server/API có đang **sống và truy cập được** hay không.

Nói đơn giản:

> Uptime check = kiểm tra hệ thống còn chạy không.

## Ví dụ trong dự án backend

Backend của anh có URL:

```txt
http://localhost:8080/actuator/health
```

## Phân biệt
| Khái niệm    | Ý nghĩa                                             |
| ------------ | --------------------------------------------------- |
| Uptime       | Thời gian hệ thống hoạt động bình thường            |
| Downtime     | Thời gian hệ thống bị lỗi/ngừng hoạt động           |
| Uptime check | Hành động kiểm tra hệ thống có đang hoạt động không |
| Health check | Endpoint/API dùng để báo trạng thái sức khỏe app    |

```
Uptime check giống như có người cứ vài giây hỏi backend:

"Ê mày còn sống không?"

Nếu backend trả lời "UP" → ổn.
Nếu không trả lời hoặc trả lời lỗi → báo động.
```