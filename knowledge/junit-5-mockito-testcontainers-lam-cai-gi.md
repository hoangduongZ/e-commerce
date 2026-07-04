## Test Stack

### JUnit 5
Framework chính để viết và chạy test trong Java/Spring Boot.  
Dùng để test unit, integration test, lifecycle test và assertion.

### Mockito
Thư viện dùng để mock dependency khi viết unit test.  
Giúp test service/domain logic mà không cần gọi thật database, API ngoài hoặc service khác.

### Testcontainers
Thư viện tạo container Docker thật trong lúc chạy integration test.  
Dùng để test với PostgreSQL, Redis hoặc service thật gần giống môi trường production.