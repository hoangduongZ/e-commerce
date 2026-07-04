# Hướng dẫn Đọc hiểu Báo cáo Đo lường Kiểm thử (JaCoCo Report)

> **Dành cho người không chuyên DevOps/Kiểm thử:** Bạn có thể hình dung file `index.html` của JaCoCo giống như **"Bản đồ nhiệt chụp X-Quang"** cho ngôi nhà code của dự án. Khi các con Robot chạy kiểm thử (Test), JaCoCo sẽ đứng ngoài quan sát xem con Robot đó đã đi qua những căn phòng nào, bật công tắc nào, và góc khuất nào trong nhà chưa hề được ai bước chân tới.

---

## 1. Cách mở báo cáo
Bạn chỉ cần nhấp đúp chuột vào file `index.html` (ví dụ: `/Users/macbook/Downloads/jacoco-report/index.html`), file sẽ tự động mở lên trên trình duyệt web (Chrome, Safari, Edge...). Bạn không cần cài đặt thêm phần mềm nào hay cần kết nối mạng để xem.

---

## 2. Giải thích các Cột Chỉ số (Thuật ngữ trong Bảng)
Khi nhìn vào bảng tổng hợp, bạn sẽ thấy các thanh màu **Xanh lá (Đã được test kiểm tra)** và **Đỏ (Bị bỏ sót, chưa test)**. Dưới đây là ý nghĩa chi tiết của từng cột từ trái sang phải, kèm theo **số liệu THẬT của dự án ElectroStore hiện tại**:

| Cột chỉ số | Ý nghĩa là gì? | Ví dụ thực tế từ dự án ElectroStore |
| :--- | :--- | :--- |
| **Element** | Tên gói (package), tên lớp (class) hoặc tên hàm đang được chấm điểm. | Ví dụ: `com.electrostore.common.util` |
| **Missed Instructions / Cov.** | **Độ phủ lệnh máy (Độ chính xác cao nhất):** JaCoCo chia nhỏ dòng code thành các chỉ thị máy tính cực nhỏ (bytecode) để chấm. Cột Cov (Coverage) thể hiện tỷ lệ % lệnh đã được chạy qua. | Dự án đang đạt **85.8%** (đã test 546 / 636 lệnh máy). Đây là con số rất cao cho giai đoạn nền móng! |
| **Missed Branches / Cov.** | **Độ phủ ngã rẽ (Các câu lệnh `if / else`, `switch`):** Kiểm tra xem khi gặp một ngã ba đường (ví dụ: *Nếu đúng thì làm A, Sai thì làm B*), bài test đã thử đi hết cả 2 đường chưa hay chỉ đi 1 đường. | Dự án đạt **75%** (đã kiểm tra 12 / 16 ngã rẽ điều kiện). |
| **Missed / Cxty** | **Độ phức tạp (Cyclomatic Complexity):** Đánh giá xem đoạn code đó rắc rối, nhiều nhánh rẽ hay đơn giản. Số càng lớn thì càng cần nhiều bài test để bao phủ hết. | — |
| **Missed / Lines** | **Độ phủ số dòng code (Dễ hiểu nhất):** Một dòng code trong file `.java` có được bài test nào chạm tới hay không. | Dự án đạt **82.6%** (đã chạy qua 100 / 121 dòng code). |
| **Missed / Methods** | **Độ phủ Hàm / Phương thức:** Số lượng hàm đã được gọi thử nghiệm trong lúc chạy test. | Dự án đạt **76%** (đã test 38 / 50 hàm). |
| **Missed / Classes** | **Độ phủ Lớp:** Số lượng file Lớp (Class) đã được chạm tới. | Dự án đạt **94.7%** (18 / 19 lớp đã được test chạm qua). |

---

## 3. Ý nghĩa 3 Màu sắc trong Báo cáo (Đèn Giao thông)
Khi bạn bấm chuột vào tên một package (ví dụ: bấm vào `com.electrostore.common.util`), rồi bấm tiếp vào tên một file (ví dụ: `SlugUtil.java`), bạn sẽ nhìn thấy chính xác từng dòng code được tô màu giống như đèn giao thông:

### 🟢 Màu Xanh lá (Green - Hoàn hảo)
* **Ý nghĩa:** Dòng code này **đã được chạy qua ít nhất 1 lần** trong quá trình test.
* **Ví dụ:** Trong file `SlugUtil.java`, toàn bộ hàm tạo đường dẫn thân thiện (`toSlug`) đều có màu xanh lá vì chúng ta đã viết bài test `SlugUtilTest.java` kiểm tra rất kỹ.

### 🟡 Màu Vàng (Yellow - Nguy hiểm ngầm)
* **Ý nghĩa:** Dòng code có điều kiện rẽ nhánh (ví dụ: `if (tuoi >= 18 && co_cccd)`) nhưng bài test **mới chỉ thử một nửa kịch bản**. 
* **Ví dụ:** Bạn mới viết test cho trường hợp *người trên 18 tuổi có CCCD* (điều kiện đúng), nhưng chưa viết bài test thử trường hợp *dưới 18 tuổi* hoặc *không có CCCD* (điều kiện sai). Dòng `if` đó sẽ bị tô màu vàng để nhắc bạn bổ sung test case.

### 🔴 Màu Đỏ (Red - Báo động / Bỏ sót)
* **Ý nghĩa:** Dòng code này **hoàn toàn mồ côi**, chưa có bất kỳ bài test nào bước chân tới. Nếu đoạn code này có vi trùng hoặc lỗi ẩn, hệ thống sẽ không thể phát hiện ra cho tới khi khách hàng sử dụng thật.

---

## 4. Giải mã các vùng "Màu Đỏ" hiện tại của dự án (Tại sao không đạt 100%?)
Khi xem báo cáo của ElectroStore, bạn sẽ thấy tổng thể đạt ~83% chứ không phải 100%. Có một số vùng bị tô màu đỏ, lý do là:

1. **Hàm khởi chạy ứng dụng (`EcommerceApplication.main` - Màu đỏ):**
   Đây là hàm khởi động framework Spring Boot của toàn hệ thống. Trong kiểm thử tự động, người ta không gọi trực tiếp hàm `main(String[] args)` này, nên JaCoCo đếm là "chưa chạy qua". Đây là điều **hoàn toàn bình thường và chấp nhận được**.
2. **Các hàm khởi tạo rỗng / Exception (`NotFoundException`, `BaseEntity` - Màu đỏ/vàng):**
   Một số lớp chỉ dùng để chứa dữ liệu hoặc khai báo thông báo lỗi đơn giản. Chúng ta không cần tốn thời gian viết test cho những class rỗng này.

---

## 5. Lời khuyên Thực chiến cho Dự án Doanh nghiệp
Là người quản lý hoặc lập trình viên, bạn **không nên mù quáng theo đuổi chỉ số 100% Xanh toàn bộ**. Việc cố gắng test 100% sẽ tốn thời gian vô ích vào những dòng code tầm thường (như hàm `get/set` dữ liệu).

👉 **Tiêu chuẩn Vàng cho ElectroStore:**
* **Mức đạt chuẩn (Target):** Tổng thể độ phủ dòng code (**Lines**) đạt từ **70% - 85%** là cực kỳ xuất sắc.
* **Nơi cần 100% Xanh:** Tập trung viết test siêu kỹ cho các vùng **Logic Nghiệp vụ Cốt lõi** (nằm trong thư mục `app/` và `domain/`), ví dụ: *tính tiền giỏ hàng, áp dụng mã giảm giá, trừ tồn kho, thanh toán*.
* **Nơi được phép Đỏ:** Các file cấu hình (`config/`), các file khai báo bảng biểu DTO, hoặc hàm `main` khởi chạy ứng dụng.
