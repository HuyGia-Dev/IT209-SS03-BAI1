# Báo cáo Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

## 1. Mục tiêu
Thay đổi cổng kết nối SSH mặc định từ **22** sang **2222** cho tài khoản người dùng `devops` có quyền `sudo` nhằm hạn chế các cuộc quét cổng tự động và nâng cao bảo mật hệ thống.

---

## 2. Các bước thực hiện

### Bước 1: Cho phép cổng 2222/tcp qua tường lửa UFW
Trước khi thay đổi cấu hình dịch vụ SSH, mở cổng mới trên tường lửa UFW để tránh nguy cơ bị khóa truy cập (Lockout).

```bash
devops@vps-server:~$ sudo ufw allow 2222/tcp
[sudo] password for devops: 
Rules updated
Rules updated (v6)

devops@vps-server:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
2222/tcp                   ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
2222/tcp (v6)              ALLOW       Anywhere (v6)             
```

### Bước 2: Thay đổi cấu hình dịch vụ SSH
Mở tệp tin cấu hình `/etc/ssh/sshd_config` bằng trình soạn thảo `nano` dưới quyền `sudo`:

```bash
sudo nano /etc/ssh/sshd_config
```

Tìm dòng `#Port 22` hoặc `Port 22` và đổi thành:
```text
Port 2222
```

Lưu và đóng tệp tin (`Ctrl + O`, `Enter`, `Ctrl + X`).

### Bước 3: Khởi động lại dịch vụ SSH
Thực hiện khởi động lại dịch vụ SSH daemon để áp dụng thay đổi cấu hình:

```bash
devops@vps-server:~$ sudo systemctl restart ssh
```

### Bước 4: Điều chỉnh tường lửa UFW (Xóa cho phép cổng 22)
Sau khi chắc chắn dịch vụ SSH đã nhận cổng mới, thu hồi quyền truy cập qua cổng 22 trên UFW:

```bash
devops@vps-server:~$ sudo ufw delete allow 22/tcp
Rule deleted
Rule deleted (v6)
```

---

## 3. Nhật ký kiểm tra kết nối (Terminal Logs)

Thực hiện kiểm tra từ máy tính cá nhân (Client):

### Test 1: Kết nối qua cổng mới (2222) -> **Thành công**
```bash
user@local-pc:~$ ssh -p 2222 devops@103.x.x.x
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Sun Oct 15 10:20:15 2023 from 113.x.x.x
devops@vps-server:~$ 
```

### Test 2: Kết nối qua cổng cũ (22) -> **Thất bại (Bị chặn/Từ chối)**
```bash
user@local-pc:~$ ssh -p 22 devops@103.x.x.x
ssh: connect to host 103.x.x.x port 22: Connection refused
```

---

## 4. Kết luận
- Dịch vụ SSH đã chuyển sang hoạt động trên cổng `2222/tcp`.
- Tường lửa UFW đã mở cổng `2222/tcp` và chặn cổng `22/tcp`.
- Đã hoàn thành yêu cầu bài tập bằng tài khoản người dùng `devops`.