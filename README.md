# Hướng dẫn Vận hành Hệ thống Thu/Phát QPSK bằng USRP trên GNU Radio
Tài liệu này cung cấp các bước chuẩn bị, cấu hình và khắc phục sự cố để vận hành ổn định hệ thống truyền thông QPSK qua phần cứng SDR (USRP N321) trên môi trường Ubuntu.
1. Yêu cầu Hệ thống (Prerequisites)
-Hệ điều hành: Ubuntu 24.04 Linux.
-Phần cứng: 2 thiết bị USRP, kết nối qua mạng Gigabit Ethernet.
-Phần mềm: GNU Radio Companion, gói driver UHD.
-Công cụ phụ trợ: python3, ffmpeg (xử lý video), vlc (phát video).
2. Cấu hình Tối ưu Hóa Hệ điều hành 
-Để tránh hiện tượng nghẽn mạng và tràn bộ đệm khi giao tiếp với USRP ở tốc độ cao, nên chạy các lệnh sau trong Terminal trước khi bật GNU Radio:
sudo sysctl -w net.core.wmem_max=2500000
sudo sysctl -w net.core.rmem_max=2500000
3. Chuẩn bị Dữ liệu Truyền 
-Hệ thống sử dụng kích thước gói (Packet Length) là 252 bytes. Mọi file trước khi đưa vào GNU Radio đều phải đi qua script đệm Padded_File_Source.py (sử dụng byte rỗng \x00 để đệm đuôi).
a.Truyền file text (văn bản)
-Có sẵn file .txt của bạn.
-Chạy lệnh: python3 Padded_File_Source.py <ten_file.txt>
<img width="1222" height="530" alt="image" src="https://github.com/user-attachments/assets/73e47af6-af18-416f-92ec-91690c841cfe" />

b.Truyền ảnh
tbc...
c.Truyền video (bắt buộc dùng định dạng mpeg-ts)
-Giảm độ phân giải, ép bitrate và bọc file thành chuẩn truyền hình kỹ thuật số .ts để chống vỡ file khi rớt gói do nhiễu không gian.
-Chạy lệnh FFmpeg: ffmpeg -i <video_goc.mp4> video_ts.ts
-Đệm file: python3 Padded_File_Source.py video_ts.ts
-Mở file: Đổi tên file đuôi txt thành ts rồi chạy lênh ffplay <output.ts>
<img width="1732" height="874" alt="image" src="https://github.com/user-attachments/assets/a4f22d3e-587f-4152-998b-0a8e7c18606c" />

5. Quy trình Vận hành
-Khởi động thiết bị: Bật nguồn 2 con USRP, đợi đèn mạng sáng xanh ổn định.
-Cấu hình nguồn/đích:
Ở máy phát (TX): Trỏ khối File Source vào file padded.txt (sản phẩm của script Python).
Ở máy thu (RX): Trỏ khối File Sink và đặt tên file đầu ra tương ứng (ví dụ: output.txt, output.ts).
-Phát sóng:
-Bật lưu đồ Thu (RX) trước.
<img width="1944" height="899" alt="image" src="https://github.com/user-attachments/assets/f5f9acb3-7484-47d4-9bd1-e80d14aa773d" />

-Bật lưu đồ Phát (TX) sau.
<img width="1962" height="911" alt="image" src="https://github.com/user-attachments/assets/7ec70e70-a351-4936-ae7e-3b05408de459" />


