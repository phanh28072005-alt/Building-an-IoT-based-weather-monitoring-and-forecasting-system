# Trạm Vật Lý Thu Nhỏ Tích Hợp IoT & Dự Báo Khí Tượng Thông Minh

## 📝 Giới thiệu dự án
Dự án tập trung thiết kế và chế tạo một **Trạm vật lý thu nhỏ tích hợp công nghệ IoT** nhằm tự động hóa quy trình quan trắc khí tượng. Hệ thống kết hợp khả năng xử lý phần cứng tối ưu ở lớp dưới, truyền thông không dây linh hoạt và phân tích thông minh dựa trên mô hình toán học (Hồi quy Ridge) để đưa ra các kịch bản dự báo thời tiết ngắn hạn.

---

## 📐 Kiến trúc chức năng & Luồng xử lý

### 1. Sơ đồ khối chức năng hệ thống
Dưới đây là sơ đồ mô tả mối quan hệ, các liên kết phần cứng và luồng giao tiếp dữ liệu giữa khối xử lý trung tâm ATmega328P, khối truyền thông ESP32 và các module cảm biến môi trường.

![Sơ đồ khối chức năng](so_do_khoi_chuc_nang.png)

### 2. Lưu đồ thuật toán vận hành
Lưu đồ thể hiện chu trình hoạt động của trạm, từ giai đoạn khởi tạo cấu hình thanh ghi, quét ngắt, thu thập dữ liệu từ cảm biến, đóng gói truyền nhận dữ liệu đến phân tích hồi quy và cập nhật kết quả hiển thị.

![Lưu đồ thuật toán](luudothuattoan.png)

---

## 🎯 Mục tiêu cốt lõi
* **Tự động hóa quan trắc:** Thu thập chính xác các thông số môi trường theo thời gian thực.
* **Tối ưu hóa phần cứng:** Lập trình nhúng tối ưu bộ nhớ, tăng tốc độ xử lý ngắt và tiết kiệm điện năng.
* **Truyền thông không dây:** Đồng bộ hóa luồng dữ liệu liên tục lên máy chủ trung tâm.
* **Dự báo thông minh:** Áp dụng thuật toán học máy để phân tích xu hướng và dự báo thời tiết ngắn hạn trực quan.

---

## 🛠 Phân tích chi tiết thành phần mã nguồn (Firmware)

Dựa trên cấu trúc thư mục phát triển hệ thống, các tệp tin chức năng được phân bổ và chịu trách nhiệm các vai trò cụ thể như sau:

### 1. Chương trình chính & Cấu hình dự án (CodeVisionAVR)
* `TramThoiTiet.c`: Mã nguồn C chính điều khiển toàn bộ luồng xử lý luân phiên và xử lý ngắt của vi điều khiển ATmega328P.
* `TramThoiTiet.prj` / `TramThoiTiet.atsln`: Các file cấu hình project quản lý trình biên dịch trong môi trường CodeVisionAVR / Atmel Studio.
* Các file bổ trợ biên dịch: `TramThoiTiet.cfi`, `TramThoiTiet.cci`, `TramThoiTiet.cfg`, `TramThoiTiet.cof`, `TramThoiTiet.fct` (quản lý ngăn xếp, thông tin debug và sơ đồ phân bổ bộ nhớ ngắt).

### 2. Hệ thống thư viện & Trình điều khiển cảm biến lớp dưới (`.h`)
Hệ thống giao tiếp trực tiếp với phần cứng thông qua các thanh ghi tối ưu:
* **Giao tiếp ngoại vi nền tảng:**
  * `TWI_Lib2.h`: Thư viện cấu hình bộ mã hóa giao tiếp I2C/TWI (Two-wire Serial Interface) phần cứng để quét dữ liệu tốc độ cao.
  * `UART_Lib2.h`: Quản lý truyền nhận nối tiếp đồng bộ/bất đồng bộ để đóng gói dữ liệu đẩy sang module ESP32.
  * `ADC_Lib.h`: Trình cấu hình bộ chuyển đổi tương tự - số tích hợp, đọc giá trị điện áp từ các cảm biến analog.
* **Trình điều khiển cảm biến khí tượng:**
  * `dht11_lite.h`: Trình điều khiển tối ưu dung lượng (lite) để đọc dữ liệu nhiệt độ và độ ẩm môi trường.
  * `rain_sensor.h`: Đo lường và xác định trạng thái/mức độ lượng mưa tích tụ.
  * `hall_speed.h`: Xử lý tín hiệu ngắt từ cảm biến Hall để tính toán vận tốc quay của anemometer (đo tốc độ gió).
  * `wind_dir.h`: Xác định hướng gió thông qua các bộ chuyển đổi logic hoặc phân bậc điện áp ADC.
* **Hiển thị & Giao diện trạm:**
  * `lcd2004_i2c.h`: Điều khiển xuất màn hình LCD 20x4 thông qua chip chuyển đổi giao tiếp I2C để tiết kiệm chân IO cho vi điều khiển.

### 3. Khối nhúng toán học & Dự báo
* `model_weights.h`: Lưu trữ mảng hằng số chứa **trọng số tối ưu (weights & bias)** sau khi đã huấn luyện bằng mô hình Hồi quy Ridge trên Python. Vi điều khiển sẽ trực tiếp sử dụng các tham số này để thực hiện phép tính ma trận tuyến tính gọn nhẹ ngay trên chip, đưa ra kết quả dự báo thời tiết tại chỗ mà không cần tốn tài nguyên kết nối mạng liên tục.

### 4. Phân hệ truyền thông Wi-Fi & Gateway
* `esp32_test_mode_AP/`: Thư mục chứa mã nguồn cấu hình ESP32 chạy chế độ Access Point (AP) để thử nghiệm thu nhận gói dữ liệu cục bộ.
* `web_ui.h`: Lưu trữ mã nguồn HTML/CSS/JS tĩnh dưới dạng chuỗi nhúng, phục vụ việc khởi tạo một Web Server cục bộ ngay trên ESP32 giúp người dùng cấu hình Wi-Fi và xem thông số qua điện thoại/máy tính.

---

## 📍 Phạm vi nghiên cứu & Thực nghiệm

* **Không gian thực nghiệm:** Tập trung thu thập và đối sánh luồng dữ liệu thời tiết thực tế tại khu vực **Hà Nội**. Dữ liệu từ trạm IoT tự chế sẽ được so sánh trực tiếp với các trạm đo lường chính thống cùng khu vực để đánh giá, hiệu chuẩn sai số.
* **Giới hạn phần cứng:** Chỉ ứng dụng nhóm linh kiện bán dẫn dân dụng đặc thù (`ATmega328P`, `ESP32`, các module cảm biến và IC chuyển đổi mức logic bổ trợ).
* **Giới hạn thuật toán:** Tập trung tối ưu thuật toán Hồi quy Ridge (L2 Regularization), giúp giải quyết hiện tượng đa cộng tuyến của dữ liệu khí tượng bề mặt, không mở rộng sang các kiến trúc học sâu phức tạp khác nhằm bảo toàn tài nguyên bộ nhớ chip.
