# Báo cáo: Khung tham chiếu an toàn cho các bản vá Tuần 3

## 1. Bản vá HOTRO-4: Băm mật khẩu có muối (Salted Password Hashing)

### 1.1. Hiện trạng và vị trí trong mã nguồn MiniShop gốc
- Trong tệp `db.py`, hàm băm mật khẩu được cài đặt như sau:
```python
  def hash_password(password):
      """Bam mat khau thanh chuoi thap luc phan."""
      return hashlib.md5(password.encode("utf-8")).hexdigest()

```

* **Lỗ hổng:** Mật khẩu được băm bằng thuật toán MD5 thuần (không có muối). Thuật toán MD5 có chi phí tính toán cực kỳ thấp và không có độ trễ, khiến hệ thống dễ bị tấn công duyệt trước bằng bảng cầu vồng (Rainbow Table) hoặc tấn công vét cạn (Brute-force) bằng GPU/ASIC.

### 1.2. Ánh xạ khung NIST SSDF (SP 800-218 v1.1)

* **Nhóm thực hành:** PW (Produce Well-Secured Software).
* **Mục tiêu PW.4:** Tái sử dụng các thành phần và giải thuật mật mã an toàn, đã được chứng nhận và kiểm chứng.
* **Nhiệm vụ PW.4.1 / PW.4.4:**
* Không tự thiết kế cơ chế mật mã riêng hoặc tiếp tục sử dụng các hàm băm nhanh đã bị coi là lỗi thời (MD5).
* Triển khai hàm dẫn xuất khóa tiêu chuẩn (KDF) có tham số làm chậm (work factor) kèm muối ngẫu nhiên (Salt) để bảo vệ dữ liệu xác thực lưu trữ trong cơ sở dữ liệu.



### 1.3. Ánh xạ khung OpenSSF (LFD121)

* **Mục tham chiếu:** *Implementation Overview - Cryptographic Storage*.
* **Nguyên tắc kỹ thuật:**
* Không lưu trữ mật khẩu dưới dạng văn bản rõ hoặc băm bằng hàm băm thông thường (MD5, SHA-1, SHA-256 trần).
* Áp dụng cơ chế băm chậm kèm muối mật mã ngẫu nhiên (Cryptographic Salt) với độ dài tối thiểu 16 byte (như PBKDF2-HMAC-SHA256, Argon2 hoặc bcrypt) nhằm vô hiệu hóa các cơ sở dữ liệu băm tính sẵn và làm suy kiệt tài nguyên tấn công vét cạn.



---

## 2. Bản vá HOTRO-5: Niêm phong phiên an toàn (Session Signing)

### 2.1. Hiện trạng và vị trí trong mã nguồn MiniShop gốc

* Trong tệp `app.py`, chữ ký niêm phong cookie phiên được tạo qua hàm:
```python
def _sign(sid):
    """Ky ma phien de gan vao cookie."""
    return hashlib.md5((SESSION_KEY + sid).encode("utf-8")).hexdigest()

```


* **Lỗ hổng:** Cơ chế chữ ký sử dụng phép ghép chuỗi thô `hashlib.md5(SESSION_KEY + sid)`. Cách làm này dễ bị tổn thương trước tấn công mở rộng chiều dài (Length Extension Attack) và sử dụng giải thuật MD5 vốn dễ xảy ra xung đột (Collision).

### 2.2. Ánh xạ khung NIST SSDF (SP 800-218 v1.1)

* **Nhóm thực hành:** PW (Produce Well-Secured Software).
* **Mục tiêu PW.4:** Bảo vệ tính toàn vẹn (Integrity) và tính xác thực của trạng thái phiên làm việc giữa máy khách và máy chủ.
* **Nhiệm vụ PW.4.1:** Sử dụng các giao thức mật mã và cơ chế kiểm tra tính toàn vẹn đã được chuẩn hóa (như chuẩn HMAC) nhằm ngăn chặn việc sửa đổi hoặc làm giả dữ liệu truyền tải.

### 2.3. Ánh xạ khung OpenSSF (LFD121)

* **Mục tham chiếu:** *Implementation Overview - Message Authentication & Integrity*.
* **Nguyên tắc kỹ thuật:**
* Tuyệt đối không tự ghép chuỗi bí mật với dữ liệu bằng hàm băm (`H(key || data)`).
* Sử dụng cơ chế Mã xác thực thông điệp dựa trên hàm băm (HMAC) theo chuẩn RFC 2104 (ví dụ `hmac.new(..., hashlib.sha256)`) kết hợp với hàm băm an toàn họ SHA-2 nhằm đảm bảo người dùng không thể can thiệp hay sửa đổi mã phiên trên cookie trình duyệt.



---

## 3. Bản vá HOTRO-8: Mã định danh phiên ngẫu nhiên (CSPRNG Session ID)

### 3.1. Hiện trạng và vị trí trong mã nguồn MiniShop gốc

* Trong tệp `app.py`, việc cấp phát mã phiên được thực hiện qua hàm:
 ``python
_next_sid = 0

def _new_session_id():
    """Sinh mot ma phien moi."""
    global _next_sid
    _next_sid += 1
    return str(_next_sid)

 ``


* **Lỗ hổng:** Mã phiên là số nguyên tự tăng dần (`1`, `2`, `3`...). Do tính chất hoàn toàn có thể đoán trước (predictable), kẻ tấn công chỉ cần quan sát mã phiên của mình là có thể suy đoán chính xác mã phiên của các người dùng đăng nhập trước hoặc sau để chiếm đoạt phiên (Session Hijacking).

### 3.2. Ánh xạ khung NIST SSDF (SP 800-218 v1.1)

* **Nhóm thực hành:** PW (Produce Well-Secured Software).
* **Mục tiêu PW.1:** Thiết kế kiến trúc phần mềm đáp ứng đầy đủ các yêu cầu an toàn, đặc biệt là cơ chế quản lý danh tính và phiên (Session Management).
* **Nhiệm vụ PW.1.2:** Giảm thiểu bề mặt tấn công bằng cách đảm bảo các định danh bảo mật sinh ra không thể đoán trước, ngăn chặn nguy cơ mạo danh tài khoản.

### 3.3. Ánh xạ khung OpenSSF (LFD121)

* **Mục tham chiếu:** *Development Processes - Defense-in-Breadth & Session Management*.
* **Nguyên tắc kỹ thuật:**
* Định danh phiên (Session ID) bắt buộc phải được sinh từ bộ tạo số ngẫu nhiên giả mật mã an toàn (CSPRNG, chẳng hạn module `secrets` trong Python thay vì thư viện `random` thông thường).
* Độ dài của mã phiên phải đạt entropy tối thiểu 128 bit (tương đương 32 ký tự hex) để đảm bảo không gian khóa đủ lớn, loại bỏ khả năng quét hoặc đoán mò mã phiên.



---

## 4. Tổng kết đối chiếu

| Mã bản vá | Lỗ hổng kỹ thuật | Mục tiêu NIST SSDF SP 800-218 | Thực hành OpenSSF LFD121 |
| --- | --- | --- | --- |
| **HOTRO-4** | MD5 trần, không muối | **PW.4:** Dùng KDF chuẩn hóa (PBKDF2) | *Cryptographic Storage:* Băm chậm + Muối ngẫu nhiên tối thiểu 16 bytes |
| **HOTRO-5** | Ký phiên ghép chuỗi MD5 | **PW.4.1:** Bảo vệ tính toàn vẹn | *Protecting Secrets:* Sử dụng HMAC-SHA256 chuẩn |
| **HOTRO-8** | Mã phiên tự tăng `_next_sid` | **PW.1.2:** Quản lý phiên an toàn | *Session Management:* Dùng CSPRNG (`secrets`), entropy $\ge$ 128-bit |
