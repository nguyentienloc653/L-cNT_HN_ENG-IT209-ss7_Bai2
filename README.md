# Bài 2: Quản trị Tường lửa UFW cho Cụm Dịch vụ Multi-port

## 1. Mục tiêu
- Thiết lập tường lửa **UFW (Uncomplicated Firewall)** để bảo mật hệ thống máy chủ chạy nhiều cổng dịch vụ đồng thời.
- Nắm vững cách cấu hình mở các cổng dịch vụ web công khai và cô lập cổng cơ sở dữ liệu nội bộ.
- Ngăn chặn các cuộc tấn công quét cổng (port scanning) và truy cập trái phép từ Internet.

---

## 2. Yêu cầu & Kịch bản triển khai
Máy chủ dự kiến chạy 4 dịch vụ:
- **SSH (Port 22/tcp)**: Dùng để truy cập quản trị máy chủ từ xa -> **Cho phép (ALLOW)**.
- **Nginx Web HTTP (Port 80/tcp)**: Phục vụ dịch vụ web công khai -> **Cho phép (ALLOW)**.
- **Spring Boot App (Port 8082/tcp)**: Dịch vụ ứng dụng backend để kiểm tra từ xa -> **Cho phép (ALLOW)**.
- **MySQL Database (Port 3306/tcp)**: Dịch vụ cơ sở dữ liệu nội bộ -> **Chặn hoàn toàn từ Internet (DENY)** bằng chính sách mặc định (không thêm luật allow).

### Ràng buộc bảo mật
1. Chặn mặc định toàn bộ lưu lượng đi vào (`deny incoming`).
2. Cho phép lưu lượng đi ra (`allow outgoing`).
3. Mở đúng 3 cổng: `22/tcp`, `80/tcp`, `8082/tcp`.
4. Không cấu hình cho phép cổng `3306/tcp`.

---

## 3. Các bước thực hiện

### Bước 1: Khai báo chính sách mặc định (Default Policies)
Thiết lập chính sách chặn toàn bộ gói tin đi vào và cho phép gói tin đi ra từ server:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Bước 2: Cho phép các cổng dịch vụ quy định
Cấu hình mở các cổng tương ứng theo giao thức TCP:
```bash
# Cho phép SSH quản trị
sudo ufw allow 22/tcp

# Cho phép Web HTTP tiêu chuẩn
sudo ufw allow 80/tcp

# Cho phép ứng dụng Spring Boot
sudo ufw allow 8082/tcp
```
> **Lưu ý về cổng MySQL (3306/tcp):** Không thực hiện lệnh mở cổng 3306. Do chính sách mặc định là `deny incoming`, cổng 3306 sẽ tự động bị chặn đối với các kết nối từ Internet, chỉ các tiến trình cục bộ (localhost/127.0.0.1) trên máy chủ mới có thể kết nối.

### Bước 3: Kích hoạt UFW
Bật tường lửa và áp dụng các chính sách đã tạo:
```bash
sudo ufw enable
```

---

## 4. Kiểm tra và Xác nhận cấu hình

### Lệnh kiểm tra:
```bash
sudo ufw status verbose
```

### Kết quả đầu ra thực tế (Output):
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
8082/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
8082/tcp (v6)              ALLOW IN    Anywhere (v6)             
```

---

## 5. Đánh giá & Kết luận
- **Trạng thái tường lửa:** `Status: active` — Tường lửa đang hoạt động và giám sát mạng.
- **Chính sách vào/ra:** `Default: deny (incoming), allow (outgoing)` — Đảm bảo nguyên tắc bảo mật tối thiểu (Principle of Least Privilege).
- **Các cổng dịch vụ mở:**
  - `22/tcp`: Cho phép kết nối SSH quản trị.
  - `80/tcp`: Cho phép lưu lượng Web HTTP.
  - `8082/tcp`: Cho phép kết nối trực tiếp đến ứng dụng Spring Boot.
- **Cô lập cơ sở dữ liệu:** Cổng `3306/tcp` hoàn toàn không xuất hiện trong danh sách `ALLOW IN`, được bảo vệ an toàn trước các cuộc tấn công dò quét cổng từ bên ngoài Internet.
