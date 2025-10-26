# Webstrike Lab — SOC Write-up (CyberDefender)

Tổng quan
- Mục tiêu: Phân tích PCAP để xác định nguồn tấn công, kỹ thuật upload web shell, thư mục lưu file tải lên, cổng kết nối ra ngoài và file bị cố gắng trích xuất.
- Tệp phân tích: `/D:/H4ck DOC/xinloihuy.github.io/posts/webstrike-lab.md`
- PCAP được cung cấp bởi đội mạng sau khi phát hiện file đáng ngờ trên web server.

Mô tả nhanh quy trình
1. Kiểm tra thống kê giao thức để xác định lưu lượng HTTP bất thường.
    ![image](../assets/webstrike-lab/1.png)
2. Export các HTTP object để tìm file PHP khả nghi.
    ![image](../assets/webstrike-lab/2.png)

Phát hiện ban đầu
- Có hai file upload PHP thu được từ lưu lượng:
  - `upload.php` (phiên bản đầu, upload trực tiếp thất bại)
     ![image](../assets/webstrike-lab/3.png)
  - `upload.php` (phiên bản thứ hai, cho thấy kỹ thuật double extension)
     ![image](../assets/webstrike-lab/4.png)
- File thực tế tải lên thành công: `image.jpg.php`
  ![image](../assets/webstrike-lab/5.png)

Phân tích chi tiết & trả lời câu hỏi (Q1–Q6)

Q1 — Nguồn địa lý của cuộc tấn công  
- Địa chỉ IP tấn công: `117.11.88.124` (trích từ packet capture).  
- Tra cứu geolocation (ví dụ: iplocation.net) cho kết quả: Quốc gia: Trung Quốc, Thành phố: Tianjin.  
- Kết luận: Tianjin, China.

Q2 — Full User‑Agent của attacker  
- User‑Agent thu được từ HTTP request:  
  Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0  
  ![image](../assets/webstrike-lab/7.png)

Q3 — Tên web shell đã được upload thành công  
- File web shell: `image.jpg.php` (double extension để né kiểm tra định dạng).  
  ![image](../assets/webstrike-lab/5.png)

Q4 — Thư mục lưu file upload trên website  
- Thư mục đích: `/reviews/uploads/` (đường dẫn request/path trong HTTP).  
  (Xác nhận trong capture/HTTP request)

Q5 — Cổng trên máy attacker được web shell dùng để kết nối outbound  
- Cổng được web shell cố gắng kết nối/listen là: `8080`.  
  ![image](../assets/webstrike-lab/4.png)

Q6 — File bị cố gắng exfiltrate  
- File mục tiêu: `/etc/passwd` (được gửi/ghi trong payload của session).  
  ![image](../assets/webstrike-lab/8.png)

Tổng kết (Kết quả nhanh)
- Source city: Tianjin, China  
- User‑Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0  
- Web shell: image.jpg.php  
- Upload directory: /reviews/uploads/  
- Port targeted: 8080  
- File targeted for exfiltration: /etc/passwd

Khuyến nghị ngắn gọn cho SOC
- Khóa IP nguồn/giám sát thêm các kết nối từ `117.11.88.124` và các IP liên quan.  
- Xóa file web shell khỏi `/reviews/uploads/` và rà soát các file tải lên khác.  
- Cập nhật quy tắc upload để kiểm tra MIME type và không chấp nhận double extensions.  
- Kiểm tra log web server và hệ thống để phát hiện các truy vấn/command shell tương tự.  

Các file/tham chiếu trong bài
- Hình ảnh và trích xuất packet: `../assets/webstrike-lab/0.png` … `../assets/webstrike-lab/8.png`

Hoàn tất — lab đã giải quyết 6 câu hỏi.
