# Hệ Thống Truyền Hình Video Thời Gian Thực Qua Vô Tuyến (SDR Broadcasting System)

**Người thực hiện:** Nguyễn Phú Cường
**Thiết bị:** 02 máy tính Linux (Ubuntu), phần cứng USRP
**Công cụ phần mềm:** FFmpeg, GNU Radio

---

## 1. Kiến Trúc Hệ Thống (System Architecture)

Hệ thống mô phỏng quy trình truyền hình số thực tế (End-to-End SDR), truyền tải một luồng dữ liệu đa phương tiện (Hình ảnh + Âm thanh) qua môi trường không khí. Luồng dữ liệu đi qua 4 giai đoạn cốt lõi:

* **Giai đoạn 1: Tiền xử lý (Pre-processing):** Dùng `ffmpeg` nén file video gốc thành chuẩn MPEG-TS (`.ts`) với bitrate được ép chặt để phù hợp với sức tải của đường truyền vô tuyến.
* **Giai đoạn 2: Trạm Phát (TX):** Khối `File Source` trên GNU Radio đọc file `.ts`. Dữ liệu được chia làm 2 nhánh:
    * Nhánh 1: Đẩy ra cổng UDP (Port 2000) để theo dõi tại chỗ (Monitor).
    * Nhánh 2: Đi qua cụm điều chế QPSK và đưa ra ăng-ten qua khối `USRP Sink`.
* **Giai đoạn 3: Kênh Truyền (Channel):** Tín hiệu vô tuyến truyền lan trong không gian, chịu suy hao và nhiễu (AWGN). Lớp vỏ MPEG-TS giúp tăng khả năng kháng nhiễu và phục hồi lỗi.
* **Giai đoạn 4: Trạm Thu (RX):** `USRP Source` bắt sóng $\rightarrow$ Giải điều chế QPSK $\rightarrow$ Đẩy dòng dữ liệu khôi phục được qua `UDP Sink` (Port 1234) $\rightarrow$ Trình phát `ffplay` hứng dữ liệu, đồng bộ hóa và phát ra màn hình.

---

## 2. Thông Số Kỹ Thuật & Cấu Hình Thực Nghiệm

Để phần cứng USRP hoạt động ổn định (không báo lỗi Underrun `U` hoặc Overrun `O` hay bị tràn buffer), băng thông video được thiết kế ở mức cận biên an toàn so với sức tải tối đa của hệ thống vô tuyến (khoảng 360Kbps đối với hệ thống QPSK 375Kbps).

* **Video:**
Độ phân giải (Resolution): 640x480 (Chuẩn SD/480p)

Tốc độ khung hình (FPS): 24 khung hình/giây

Chuẩn nén (Codec): H.264 (thông qua thư viện libx264)

Tốc độ Bit trung bình (Video Bitrate): 200 Kbps

Khoảng cách khung hình gốc (GOP Size): 24 (1 giây/I-frame)
* **Audio:**
Chuẩn nén (Codec): AAC (Advanced Audio Coding)

Tốc độ Bit (Audio Bitrate): 48 Kbps
* **Tổng băng thông lý thuyết:** 272.8 Kbps

### 2.1. Lệnh Đóng Gói (Chạy tại Máy Phát)
```bash
ffmpeg -i '/home/cuong/Desktop/New Folder/theboys2.mp4'  -vf scale=640:480 -r 24 -vcodec libx264 -b:v 200k -maxrate 200k -bufsize 400k -c:a aac -b:a 48k -g 24 -f mpegts '/home/cuong/Desktop/New Folder/video_ts.ts'
```
### 2.2. Khởi động máy thu

Mở terminal ở máy thu và chạy lệnh hứng luồng trước 
```bash
ffplay -f mpegts -infbuf -sync ext udp://127.0.0.1:1234
```
Chạy sơ đồ QPSK_RX.grc 
<img width="960" height="380" alt="image" src="https://github.com/user-attachments/assets/7f5a292e-0f32-4db8-b247-064adad86616" />

### 2.3. Khởi động máy phát
Chạy sơ đồ QPSK_TX_FAKE.grc
<img width="1969" height="823" alt="image" src="https://github.com/user-attachments/assets/2385e9d2-c895-4341-889b-a7c2a5c3837a" />

### 3. Phân tích hiện tượng kỹ thuật 
Hiện tượng: Tiến trình phát trên GNU Radio kết thúc nhưng video tại trạm thu vẫn tiếp tục phát mượt mà cho đến hết.

Giải thích:
Hệ thống đạt được trạng thái tối ưu hóa bộ đệm (Perfect Buffering) nhờ 2 yếu tố:

1.Dư thừa băng thông kênh truyền: Khối File Source hoạt động phi thời gian thực, liên tục đẩy dữ liệu vào đường truyền. Do băng thông vật lý của USRP (~360 Kbps) lớn hơn độ nặng của file video (~298 Kbps), toàn bộ file video được truyền qua không khí với tốc độ nhanh hơn tốc độ xem thực tế. (Ví dụ: Video 60s chỉ mất 45s để truyền xong).

2.Cơ chế bảo vệ của thiết bị đầu cuối (ffplay): * Cờ -infbuf (Infinite Buffer) ép phần mềm gom toàn bộ luồng dữ liệu ồ ạt này nhét thẳng vào bộ nhớ RAM thay vì vứt bỏ.

-Cờ -sync ext (External Clock Sync) đóng vai trò làm van điều áp, ép phần mềm chỉ được lấy dữ liệu từ RAM ra và chiếu lên màn hình đúng nhịp 24 hình/giây.

-Nhờ sự kết hợp này, khi sóng vô tuyến ngắt, máy thu đã kịp trữ toàn bộ đoạn video còn lại trong RAM, giúp xem mượt mà đến giây cuối cùng bất chấp các biến động của môi trường vô tuyến.
