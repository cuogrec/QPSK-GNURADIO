### 1.1. Giai đoạn 1: Tiền xử lý (Xưởng đóng gói dữ liệu)
* **Công cụ sử dụng:** `ffmpeg`
* **Nhiệm vụ:** Nén dữ liệu video gốc (`.mp4`) để thích ứng với giới hạn băng thông vật lý khắt khe của kênh truyền vô tuyến. Chuyển đổi định dạng sang container `.ts` (MPEG Transport Stream) - chuẩn mã hóa kháng nhiễu, chuyên dụng cho truyền dẫn truyền hình số.

### 1.2. Giai đoạn 2: Trạm Phát (Máy phát TX)
* **Công cụ sử dụng:** GNU Radio (chạy nền Python Terminal), Cột sóng phần cứng USRP, và `ffplay` (Màn hình kiểm giám - Monitor).
* **Luồng hoạt động:** Khối `File Source` đọc file tĩnh `.ts` từ ổ cứng. Dữ liệu đầu ra được chia làm 2 nhánh:
    * **Nhánh Monitor tại chỗ:** Đẩy luồng dữ liệu qua cổng mạng nội bộ `UDP Sink (Port 8000)` để hiển thị lên màn hình `ffplay` giám sát của trạm phát.
    * **Nhánh Lên sóng:** Dữ liệu đi vào cụm điều chế vật lý (Gắn Header, định thời Packet, điều chế số **QPSK**) và truyền tới khối `USRP Sink` để phát xạ sóng điện từ ra không trung qua ăng-ten.

### 1.3. Giai đoạn 3: Môi trường vô tuyến (Kênh truyền vật lý)
* Tín hiệu truyền lan trong không gian giữa ăng-ten phát và ăng-ten thu. Đây là môi trường bất định, chịu tác động của suy hao đường truyền, nhiễu trắng (AWGN), và hiện tượng rớt gói tin (Packet Loss) do vật cản di động.

### 1.4. Giai đoạn 4: Trạm Thu (Máy thu RX)
* **Công cụ sử dụng:** Phần cứng USRP, GNU Radio, và `ffplay` (Thiết bị đầu cuối hiển thị).
* **Luồng hoạt động:** Khối `USRP Source` hứng sóng vô tuyến $\rightarrow$ Chuyển qua các khối đồng bộ tín hiệu và giải điều chế QPSK để khôi phục lại dòng Byte thô $\rightarrow$ Đẩy dòng dữ liệu sạch qua `UDP Sink (Port 1234)` $\rightarrow$ Trình phát `ffplay` hứng dữ liệu, bù đắp xung nhịp và hiển thị mượt mà cả hình ảnh lẫn âm thanh ra loa/màn hình.

---

## 2. Kịch Bản Thực Nghiệm & Câu Lệnh Triển Khai

Để đảm bảo hệ thống vận hành ổn định, không bị nghẽn băng thông dẫn đến hiện tượng treo luồng vật lý hoặc báo lỗi `U` (Underrun) trên USRP, các thông số đã được tối ưu hóa ở điểm ngọt (Sweet spot): **Độ phân giải 480p, Tốc độ khung hình 24 FPS, Tổng Bitrate ~298 Kbps** (An toàn dưới ngưỡng trần vật lý 360 Kbps của cấu hình QPSK 750k Sps).

### Bước 1: Nén và Đóng gói Video tại Trạm Phát
Sử dụng câu lệnh `ffmpeg` dưới đây để ép cân dữ liệu và tạo file nén tĩnh:
```bash
ffmpeg -i '/home/cuong/Desktop/New Folder/test.mp4' \\
  -vf scale=640:480 \\
  -r 24 \\
  -vcodec libx264 -b:v 250k -maxrate 250k -bufsize 500k \\
  -c:a aac -b:a 48k \\
  -g 24 \\
  -f mpegts '/home/cuong/Desktop/New Folder/video_project.ts'
