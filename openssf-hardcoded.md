
# Báo cáo: Phân tích và khắc phục bí mật viết cứng (Hardcoded Secrets) theo chuẩn OpenSSF

## 1. Hiện trạng và vị trí lỗ hổng trong mã nguồn MiniShop
Trong tệp `app.py`, khóa bí mật dùng để ký và niêm phong cookie phiên đang bị viết cứng trực tiếp dưới dạng chuỗi:

```python
SESSION_KEY = "minishop-secret-key-2026"

```

Khóa này được trực tiếp đưa vào hàm `_sign(sid)` để tạo chữ ký MD5:

```python
def _sign(sid):
    """Ky ma phien de gan vao cookie."""
    return hashlib.md5((SESSION_KEY + sid).encode("utf-8")).hexdigest()

```

## 2. Đánh giá rủi ro an ninh theo khung OpenSSF (LFD121)

Căn cứ các khuyến nghị của tài liệu OpenSSF Secure Software Development Fundamentals (LFD121) tại các phần *Risk Management* và *Implementation Overview*:

### Lộ lọt qua hệ thống quản lý mã nguồn (VCS Exposure)

* Khi mã nguồn MiniShop được đẩy lên GitHub, bất kỳ ai có quyền truy cập kho đều đọc được khóa bí mật này.
* Ngay cả khi khóa được sửa trong các commit tiếp theo, khóa cũ vẫn tồn tại vĩnh viễn trong lịch sử Git (`git log`).

### Rủi ro mạo danh phiên làm việc (Session Forgery)

* Kẻ tấn công biết được giá trị `SESSION_KEY` có thể tự tính toán chữ ký hợp lệ thông qua hàm `_sign(sid)`, kết hợp với mã phiên tự tăng để mạo danh tài khoản `admin` hoặc người dùng khác mà không cần mật khẩu.

### Bất khả thi trong xoay vòng khóa (Key Rotation)

* Khóa gắn chặt vào mã nguồn khiến việc thu hồi và thay mới khi bị lộ buộc phải can thiệp trực tiếp vào mã, thực hiện commit và tái triển khai toàn bộ ứng dụng.

## 3. Giải pháp khắc phục chuẩn hóa theo OpenSSF

1. **Tách biệt bí mật khỏi mã nguồn:** Đọc khóa từ biến môi trường hệ thống, sinh khóa ngẫu nhiên an toàn trong bộ nhớ bằng module `secrets` (CSPRNG) nếu chưa cấu hình:
```python
import os
import secrets

SESSION_KEY = os.environ.get("MINISHOP_SESSION_KEY", secrets.token_hex(32))
```
2. **Loại trừ tệp cấu hình chứa bí mật khỏi Git:** Đưa các tệp cấu hình môi trường cục bộ (`.env`) vào tệp `.gitignore`.
3. **Sử dụng thuật toán xác thực thông điệp chuẩn (HMAC):** Áp dụng HMAC-SHA256 thay vì nối chuỗi với MD5 để triệt tiêu tấn công mở rộng chiều dài (Length Extension Attack).

## 4. Kết luận

Đưa bí mật ra biến môi trường và sử dụng CSPRNG giúp bảo vệ tính toàn vẹn của cơ chế phiên, đáp ứng trực tiếp yêu cầu của khung OpenSSF LFD121 và nhóm thực hành PW (NIST SSDF SP 800-218).
