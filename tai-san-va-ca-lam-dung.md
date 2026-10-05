# Danh mục tài sản và ca lạm dụng

Họ tên: Võ Bá Huy  
MSSV: 2305ct2318  

## 7.1. Sáu tài sản, có xếp hạng

| Thứ hạng | Tài sản | Chủ của tài sản | Một câu: mất nó thì thiệt hại là gì |
|---|---|---|---|
| 1 | Cơ sở dữ liệu người dùng và mật khẩu băm (`users.password_md5`) | MiniShop và Người dùng | Kẻ xấu dùng bảng tra cứu rainbow table để bẻ khóa mật khẩu hàng loạt, chiếm đoạt tài khoản quản trị và xâm hại thông tin cá nhân khách hàng. |
| 2 | Khóa bí mật dùng để ký cookie phiên (`SESSION_KEY`) | Hệ thống máy chủ MiniShop | Kẻ xấu biết khóa có thể tự tính chữ ký giả mạo cookie cho bất kỳ `user_id` nào để chiếm quyền toàn bộ tài khoản trong hệ thống mà không cần mật khẩu. |
| 3 | Phiên làm việc hợp lệ của người dùng (`Session ID` / `Cookie`) | Người dùng đang đăng nhập | Kẻ xấu chiếm đoạt phiên để giả mạo danh tính nạn nhân thực hiện đặt mua hàng trái phép hoặc đánh cắp thông tin tài khoản. |
| 4 | Dữ liệu lịch sử đơn hàng (`orders`) | Khách hàng | Kẻ xấu khai thác lỗ hổng xem trộm đơn hàng để theo dõi lịch sử mua sắm và thói quen tiêu dùng cá nhân của người khác (lỗi IDOR). |
| 5 | Dữ liệu tồn kho và giá bán sản phẩm (`products.stock`, `products.price`) | Cửa hàng MiniShop | Tranh chấp dữ liệu (race condition) hoặc tham số âm làm sai lệch số lượng hàng tồn và doanh thu, gây thiệt hại tài chính cho cửa hàng. |
| 6 | Mã nguồn ứng dụng MiniShop | Đội ngũ phát triển MiniShop | Đối thủ sao chép sản phẩm hoặc kẻ tấn công phân tích mã nguồn để tìm kiếm các điểm yếu bảo mật chưa được vá. |

## 7.2. Bốn ca lạm dụng, mẫu ba phần

| # | Kẻ thực hiện là ai, chạm được tới đâu | Hành vi, viết bằng động từ chủ động | Kết quả mà kẻ ấy mong muốn |
|---|---|---|---|
| 1 | Kẻ tấn công lấy được bản sao lưu cơ sở dữ liệu `minishop.db` (ứng với HOTRO-4) | Dùng bảng tra cứu (rainbow table) để đối chiếu trực tiếp chuỗi băm MD5 không muối của người dùng | Khôi phục lại mật khẩu gốc dạng văn bản rõ của quản trị viên và các khách hàng. |
| 2 | Bất kỳ ai đọc được mã nguồn `app.py` trên kho GitHub (ứng với HOTRO-5) | Dùng chuỗi khóa bí mật ghi cứng `minishop-secret-key-2026` để tự tạo chữ ký MD5 hợp lệ cho mã phiên của quản trị viên | Mạo danh tài khoản quản trị `quantri` truy cập trái phép vào trang `/admin` mà không cần biết mật khẩu. |
| 3 | Người dùng bình thường đã đăng nhập, quan sát được cookie phiên của mình (ứng với HOTRO-8) | Dự đoán mã phiên tiếp theo bằng cách lấy mã phiên hiện tại cộng thêm 1 từ quy luật biến đếm tuần tự `_next_sid` | Chiếm đoạt phiên làm việc của người dùng vừa đăng nhập ngay sau đó để xem thông tin hồ sơ của họ. |
| 4 | Người dùng thông thường đã đăng nhập vào hệ thống | Thay đổi mã định danh đơn hàng trên thanh địa chỉ URL từ `/don/2` thành `/don/4` | Đọc trộm chi tiết đơn hàng (tên sản phẩm, số lượng, tổng tiền) của khách hàng khác. |