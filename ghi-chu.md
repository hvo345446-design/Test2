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
1. **Về HOTRO-4:** Tài liệu LFD121 xếp Argon2id trước PBKDF2 và nói rõ PBKDF2 là thuật toán dễ
bị tấn công bằng phần cứng chuyên dụng nhất trong ba thuật toán được khuyến nghị. Học phần dùng
PBKDF2 vì thư viện chuẩn của Python không có Argon2id và phòng máy không có quyền cài thêm gói.
Đây là giới hạn còn lại của bản vá, và nó thuộc về môi trường chứ không thuộc về tham số: số vòng
lặp dùng ở đây là sáu trăm nghìn, đúng mức bảng hướng dẫn của OWASP khuyến nghị cho PBKDF2
kèm HMAC-SHA256.
2. **Về HOTRO-5:** Biến môi trường là cách lưu bí mật yếu hơn một kho bí mật chuyên dụng, vì giá trị của
nó lộ ra cho toàn bộ tiến trình đã nạp nó. Nó vẫn tốt hơn hẳn việc ghi cứng trong mã nguồn.
3. **Về HOTRO-8:** Bản vá làm cho mã phiên không đoán được, nhưng nó không đặt hạn cho phiên và
không đặt thuộc tính an toàn cho cookie. Hai việc ấy nằm ngoài phạm vi tuần 3.
