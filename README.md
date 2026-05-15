# Adaptive Edge IoT Transmission with GRU

Hệ thống truyền dữ liệu cảm biến thích ứng trên ESP32 và Raspberry Pi, sử dụng mạng GRU (Gated Recurrent Unit) để dự đoán điều kiện mạng và tự động chọn chiến lược truyền (gửi thô, nén, hoặc chờ) theo thời gian thực.

## Tổng quan

Dự án này triển khai một pipeline truyền thông vòng kín (closed‑loop) giữa ESP32 (node cảm biến) và Raspberry Pi 4B (bộ điều khiển biên).  
ESP32 thu thập dữ liệu cảm biến (giả lập hoặc thực tế) trong các **cửa sổ 1 giây**, gửi gói tin qua Wi‑Fi. Raspberry Pi nhận các gói, đo lường ba thông số mạng: **tỉ lệ mất gói, độ trễ trung bình, thông lượng (throughput)**. Một sliding window 10 mẫu (10 giây) được đưa vào mô hình GRU huấn luyện sẵn để suy luận chiến lược truyền tối ưu cho cửa sổ tiếp theo:  
- **SEND** – gửi dữ liệu gốc.  
- **COMPRESS** – nén dữ liệu (zlib/LZ4) rồi gửi.  
- **WAIT** – đệm dữ liệu, không gửi trong cửa sổ này.  

Quyết định được gửi lại ESP32 thông qua gói ACK, khép kín vòng điều khiển với độ trễ dưới 20 ms.

## Kiến trúc hệ thống
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/4249b9fd-0315-4474-933e-205a8c549f90" />

## Yêu cầu phần cứng

- **ESP32** (bất kỳ board nào hỗ trợ Wi‑Fi, ví dụ: ESP32‑DevKitC).
- **Raspberry Pi 4B** (hoặc Raspberry Pi 3B+, Pi 5) chạy Raspberry Pi OS.
- Mạng Wi‑Fi cục bộ (router).

## Yêu cầu phần mềm

### ESP32
- Arduino IDE (hoặc PlatformIO) với board ESP32.
- Thư viện: `WiFi.h`, `WiFiUdp.h` (có sẵn trong ESP32 core).

### Raspberry Pi
- Python ≥ 3.7
- Các gói Python: `torch`, `numpy`  
  ```bash
  pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cpu
  pip3 install numpy
### Cài đặt và thiết lập
1. Huấn luyện mô hình (trên máy tính)
Chạy script train_model_pytorch.py để sinh dữ liệu giả, huấn luyện GRU và lưu mô hình cùng bộ chuẩn hóa vào file gru_model.pt:
  ```bash
  pip install torch numpy
  ```
Sau khi hoàn tất, bạn sẽ có file gru_model.pt. Copy file này sang Raspberry Pi (thư mục chứa script điều khiển).

2. Nạp firmware cho ESP32
Mở file esp32_adaptive_sender.ino (hoặc code đã cung cấp ở trên) trong Arduino IDE.

Sửa địa chỉ IP của Raspberry Pi (rpiIP) và thông tin Wi‑Fi (ssid, password).

Chọn board ESP32, cổng COM và nạp.

Mở Serial Monitor (115200 baud) để quan sát log.

3. Cấu hình và chạy Raspberry Pi Controller
Đảm bảo file gru_model.pth đã nằm cùng thư mục với script rpi_controller_pytorch.py.

Sửa biến ESP32_IP trong script thành địa chỉ IP thực tế của ESP32 (có thể thấy trên Serial Monitor sau khi ESP32 kết nối Wi‑Fi).

Chạy script:
```bash
python3 rpi_controller_pytorch.py
```
Controller sẽ bắt đầu lắng nghe dữ liệu từ ESP32 và gửi ACK.
