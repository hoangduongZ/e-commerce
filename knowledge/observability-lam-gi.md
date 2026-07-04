# Observability làm gì?

**Observability** là khả năng giúp mình **nhìn thấy bên trong hệ thống đang chạy ở production** để trả lời các câu hỏi:

- Hệ thống còn sống không?
- API nào đang chậm?
- Lỗi xảy ra ở đâu?
- Request của user đi qua những bước nào?
- Service nào đang quá tải?
- Database / Redis / external API có vấn đề không?
- Vì sao đơn hàng / thanh toán / giỏ hàng bị lỗi?

Nói ngắn gọn:

> Observability giúp developer/debugger/operator hiểu hệ thống đang hoạt động như thế nào mà không cần SSH vào server đoán mò.

---

## 3 trụ cột chính của Observability

1. Logs — Ghi lại chuyện gì đã xảy ra

2. Metrics — Đo số liệu hệ thống
- Ví dụ metrics:
```
http_server_requests_seconds_count
http_server_requests_seconds_avg
jvm_memory_used_bytes
db_connection_pool_active
redis_commands_total
orders_created_total
payment_failed_total
```

- Metrics giúp trả lời:
```
API có chậm không?
CPU/RAM có cao không?
Database connection có hết không?
Redis có bị lỗi không?
Số đơn tạo mỗi phút là bao nhiêu?
Tỷ lệ payment fail có tăng không?
```

## Tóm tắt ngắn
```
Observability = khả năng quan sát, đo lường và điều tra hệ thống production.
```
- Actuator = mở endpoint health/metrics.
- Micrometer = sinh metrics.
- Prometheus = thu thập metrics.
- Structured logging = log có cấu trúc để dễ search.
- [Correlation ID](#minh-hoạ-correlation-id) = nối toàn bộ log của cùng một request.

## `Scrape định kỳ` là sao?

**Scrape định kỳ** nghĩa là:

> Prometheus cứ sau một khoảng thời gian cố định sẽ chủ động gọi vào app để lấy metrics mới nhất.

Ví dụ app Spring Boot của bạn expose metrics tại:

```http
GET http://localhost:8080/actuator/prometheus
```
Prometheus sẽ tự gọi endpoint đó theo chu kỳ, ví dụ mỗi 15 giây.

```
App không tự đẩy metrics sang Prometheus.

Prometheus chủ động đi lấy metrics từ app.
```
Flow là:
```
Spring Boot App
    exposes metrics tại /actuator/prometheus
        ↑
        │ Prometheus scrape mỗi 15s
        │
Prometheus lưu dữ liệu time-series
        ↓
Grafana vẽ dashboard
```

## Minh hoạ Correlation ID

User gọi API:

```text
POST /api/v1/orders
Correlation-ID: req-abc-123
```
Toàn bộ log trong request đó đều mang cùng ID:
```
req-abc-123 | Auth success
req-abc-123 | Load cart from Redis
req-abc-123 | Check inventory
req-abc-123 | Create order failed: OUT_OF_STOCK
req-abc-123 | Return 409 to client
```
> Chỉ cần search req-abc-123, ta thấy toàn bộ hành trình của request đó.

