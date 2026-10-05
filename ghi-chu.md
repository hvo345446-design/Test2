# Ghi chú bài thực hành Buổi 3

## 1. Thông tin sinh viên và ca thực hành
- Họ tên: Võ Bá Huy
- MSSV: 2305ct2318
- Lớp: 261042100101_A (Lớp A)
- Bản đề: Đề A
- Số máy phòng thực hành: PM601-Máy 33

## 2. Ghi nhận sử dụng tệp cứu hộ
- Không sử dụng tệp cứu hộ (tự hoàn thành toàn bộ ba bản vá).

## 3. Ba giới hạn của ba bản vá
1. **Về HOTRO-4:** Tài liệu LFD121 xếp Argon2id trước PBKDF2 và nói rõ PBKDF2 là thuật toán dễ bị tấn công bằng phần cứng chuyên dụng nhất trong ba thuật toán được khuyến nghị. Học phần dùng PBKDF2 vì thư viện chuẩn của Python không có Argon2id và phòng máy không có quyền cài thêm gói bên ngoài. Đây là giới hạn còn lại của bản vá thuộc về môi trường thực thi; số vòng lặp dùng ở đây là 600.000, đúng mức chuẩn khuyến nghị của OWASP cho PBKDF2-HMAC-SHA256.
2. **Về HOTRO-5:** Biến môi trường là cách lưu bí mật yếu hơn so với các kho quản lý bí mật chuyên dụng (như Vault, KMS), vì giá trị biến môi trường bị lộ ra cho toàn bộ các tiến trình con đã nạp nó. Dù vậy, phương án này tốt hơn hẳn việc ghi cứng trong mã nguồn.
3. **Về HOTRO-8:** Bản vá chỉ giải quyết việc làm cho mã phiên ngẫu nhiên và không đoán được, nhưng chưa đặt thời gian hết hạn cho phiên làm việc (session timeout) và chưa đặt các thuộc tính an toàn bắt buộc cho cookie (`HttpOnly`, `SameSite`, `Secure`).