## Envelope là gì?

**Envelope** là một lớp bọc chuẩn cho mọi response API.

Thay vì mỗi API trả dữ liệu một kiểu khác nhau, backend luôn trả cùng một format.

## Ví dụ không dùng envelope
```json
{
  "message": "Product not found"
}
```

```json
{
  "id": 1,
  "name": "iPhone 15"
}
```
## Ví dụ dùng envelope

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "iPhone 15"
  },
  "error": null,
  "meta": null
}
```
- Thất bại
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "Không tìm thấy sản phẩm"
  },
  "meta": null
}
```