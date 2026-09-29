# Bảng yêu cầu an toàn (security requirements) của MiniShop, tuần 2

Họ tên: Võ Bá Huy

MSSV: 2305ct2318

Sửa tệp này ngay trên trình duyệt: bấm biểu tượng cây bút (**Edit this file**), gõ vào giữa hai dấu `|`, rồi bấm **Commit changes**. Mỗi hàng của bảng phải nằm trên đúng một dòng. Gõ vài hàng thì bấm **Commit changes** một lần, để không mất bài nếu lỡ đóng trang.

## 1. Mười hai yêu cầu (tầng L2, Mục 5 của tài liệu thực hành buổi 2)

Hàng YC-10 là hàng mẫu, chép từ Mục 5.3 của tài liệu thực hành buổi 2 và điền sẵn để đối chiếu. Giữ nguyên hàng ấy; mười một hàng còn lại là bài của sinh viên.

| Mã | Phát biểu | Điều kiện nghiệm thu (acceptance condition) | Cách kiểm chứng (verification) | Vị trí trong mã nguồn |
|---|---|---|---|---|
| YC-01 | Dữ liệu đầu vào từ người dùng không được nối trực tiếp vào câu lệnh SQL để làm thay đổi cấu trúc truy vấn. | Mọi truy vấn cơ sở dữ liệu có chứa tham số từ người dùng phải dùng cơ chế tham số hóa (parameterized query), không dùng phép cộng chuỗi. | Nhập ' OR '1'='1 vào ô tìm kiếm hoặc đăng nhập; hệ thống xử lý như chuỗi ký tự thuần, không làm sai lệch kết quả truy vấn. | `db.py`, dòng `return conn.execute(sql).fetchall()` |
| YC-02 | Dữ liệu do người dùng nhập không được đưa thẳng ra giao diện HTML mà chưa qua mã hóa thực thể (HTML escape). | Toàn bộ chuỗi hiển thị ra trình duyệt xuất phát từ tham số người dùng phải được chuyển đổi ký tự đặc biệt thành thực thể HTML an toàn trước khi nối vào thân trang. | Tìm kiếm với từ khóa `<script>alert(1)</script>`; chuỗi script hiển thị dạng văn bản thuần trên trang, không bị trình duyệt thực thi. | `views.py`, dòng `Ten san pham va ten nguoi dung deu duoc thoat ky tu bang html.escape,` |
| YC-03 | Mật khẩu tài khoản không được băm bằng thuật toán yếu (MD5) và không có muối (salt). | Mật khẩu phải được băm bằng thuật toán băm mật mã an toàn, có muối ngẫu nhiên cho từng tài khoản nhằm chống tra cứu bảng cầu vồng. | Kiểm tra giá trị lưu trong cột password_md5 của bảng users; chuỗi băm không được tạo từ hàm MD5 thuần túy. | `db.py`, dòng `def hash_password(password):` |
| YC-04 | Chuỗi bí mật dùng để ký phiên xác thực không được viết cứng (hardcoded) trực tiếp trong mã nguồn. | Khóa bí mật dùng để ký phiên phải được nạp động từ biến môi trường hệ thống hoặc tệp cấu hình bảo mật bên ngoài, không lưu tĩnh trong mã nguồn. | Mở mã nguồn tệp app.py; không tìm thấy giá trị chuỗi khóa bí mật cố định viết sẵn trong tệp mã. | `app.py`, dòng `SESSION_KEY = "minishop-secret-key-2026"` |
| YC-05 | Quyết định phân quyền truy cập chức năng quản trị không được chỉ phụ thuộc vào việc ẩn/hiện liên kết phía người dùng. | Máy chủ phải kiểm tra vai trò (role) của tài khoản ở mọi điểm cuối quản trị (/admin) trước khi trả về dữ liệu hoặc thực thi logic quản trị. | Dùng tài khoản thường (như lan), truy cập trực tiếp đường dẫn /admin; máy chủ phải từ chối truy cập hoặc chuyển hướng, không hiển thị dữ liệu quản trị. | `views.py`, dòng `if user["role"] == "admin":` |
| YC-06 | Người dùng không được phép xem hoặc can thiệp vào đơn hàng của người dùng khác khi chỉ biết mã đơn hàng (lỗ hổng IDOR). | Máy chủ phải xác thực quyền sở hữu: user_id của tài khoản đang đăng nhập phải khớp với user_id của người tạo đơn hàng trước khi trả về chi tiết. | Đăng nhập tài khoản lan (user_id = 2), truy cập trực tiếp /don/4 (đơn hàng của hung); hệ thống phải từ chối hiển thị và báo lỗi quyền truy cập. | `app.py`, dòng `elif path.startswith("/don/"):` |
| YC-07 | Mã định danh phiên (Session ID) không được sinh theo quy luật tuần tự có thể đoán trước. | Mã phiên phải là chuỗi ngẫu nhiên có độ dài và độ hỗn loạn (entropy) đủ lớn, sinh bằng bộ sinh số ngẫu nhiên an toàn mật mã. | Đăng nhập liên tiếp nhiều lần bằng các tài khoản khác nhau; các giá trị session ID thu được không phải là các số nguyên tăng dần liên tiếp (1, 2, 3...). | `app.py`, dòng `def _new_session_id():` |
| YC-08 | Cookie xác thực phiên không được để thiếu các thuộc tính bảo vệ an toàn (HttpOnly, SameSite). | Tiêu đề HTTP Set-Cookie gửi về trình duyệt khi đăng nhập phải chứa đầy đủ các cờ HttpOnly và SameSite để chống tấn công XSS và CSRF. | Kiểm tra tiêu đề HTTP Set-Cookie trả về khi đăng nhập thành công qua công cụ Network của trình duyệt; tiêu đề phải chứa cờ HttpOnly và thuộc tính SameSite. | `app.py`, dòng `self.send_header("Set-Cookie", cookie["session"].OutputString())` |
| YC-09 | Thông báo lỗi phản hồi cho người dùng không được làm lộ chi tiết ngoại lệ nội bộ hoặc cấu trúc kỹ thuật của hệ thống. | Khi phát sinh ngoại lệ, giao diện web chỉ trả về thông báo lỗi chung chung thân thiện cho người dùng; chi tiết kỹ thuật chỉ được ghi vào nhật ký máy chủ. | Gửi một yêu cầu làm phát sinh lỗi hệ thống; trang hiển thị lỗi trên trình duyệt không chứa chuỗi ngoại lệ kỹ thuật (Loi he thong: ...). | `app.py`, dòng `self._send(500, views.message_page("Loi", "Loi he thong: " + str(exc), None))` |
| YC-10 (hàng mẫu, điền sẵn) | Máy chủ MiniShop không được phục vụ yêu cầu đến từ máy khác trong mạng phòng máy | Ứng dụng chỉ lắng nghe trên địa chỉ vòng lặp nội bộ, không lắng nghe trên địa chỉ mà máy khác gọi tới được | Chạy MiniShop, rồi từ máy bên cạnh mở địa chỉ IP của máy này kèm cổng 8000; trình duyệt máy bên cạnh phải báo không kết nối được | `app.py`, dòng `HOST = "127.0.0.1"` |
| YC-11 | Thao tác kiểm tra tồn kho và tạo đơn hàng không được xảy ra tình trạng tranh chấp (race condition) dẫn đến bán vượt số lượng tồn kho. | Thao tác kiểm tra số lượng và trừ hàng tồn kho phải thực thi nguyên tử (atomic transaction) hoặc có ràng buộc chống âm số lượng ở mức cơ sở dữ liệu. | Gửi đồng thời hai yêu cầu mua sản phẩm chỉ còn 1 mặt hàng tồn kho; hệ thống chỉ cho phép một yêu cầu thành công, số lượng tồn kho không bị âm. | `db.py`, dòng `if row["stock"] >= quantity:` |
| YC-12 | Mật khẩu mẫu của các tài khoản mặc định và tài khoản quản trị không được đặt đơn giản, dễ đoán hoặc cố định trong mã nguồn. | Mật khẩu khởi tạo ban đầu phải có độ phức tạp cao hoặc hệ thống bắt buộc người dùng đổi mật khẩu ngay trong lần đăng nhập đầu tiên. | Kiểm tra danh sách tài khoản nạp mẫu trong cơ sở dữ liệu; mật khẩu quản trị không được đặt bằng chuỗi thông dụng viết sẵn như admin123. | `seed.py`, dòng `USERS = [` |

## 2. Hai tiêu chí chấp nhận (acceptance criteria), tầng L3 phần a

### Tiêu chí thứ nhất, cho yêu cầu YC-07

- Đầu vào: Gửi liên tiếp hai yêu cầu POST đăng nhập thành công với hai tài khoản hợp lệ khác nhau (`lan` và `hung`) vào endpoint `/login`.
- Kết quả quan sát được phải là: Giá trị session ID trích xuất từ tiêu đề `Set-Cookie` của hai lần đăng nhập phải là hai chuỗi ngẫu nhiên độc lập, có độ dài tối thiểu 32 ký tự hexa (hoặc tương đương 128 bit entropy) và không có quan hệ số học tăng dần.
- Kết quả chứng tỏ chưa đạt: Hai mã phiên trả về là hai số nguyên liên tiếp nhau (ví dụ: phiên thứ nhất là `1`, phiên tiếp theo là `2`).

### Tiêu chí thứ hai, cho yêu cầu YC-06

- Đầu vào: Đăng nhập bằng tài khoản `lan` (có `user_id = 2`), lấy cookie phiên hợp lệ rồi gửi yêu cầu HTTP GET đến đường dẫn `/don/4` (đơn hàng thuộc sở hữu của người dùng `hung`, có `user_id = 3`).
- Kết quả quan sát được phải là: Máy chủ phản hồi mã trạng thái 403 Forbidden hoặc 404 Not Found kèm thông báo không có quyền truy cập, hoàn toàn không hiển thị thông tin sản phẩm, số lượng hay giá tiền của đơn hàng số 4.
- Kết quả chứng tỏ chưa đạt: Máy chủ phản hồi mã trạng thái 200 OK và hiển thị đầy đủ chi tiết đơn hàng số 4 của người dùng `hung`.

## 3. Yêu cầu thứ mười ba, tầng L3 phần b

| Mã | Phát biểu | Điều kiện nghiệm thu | Cách kiểm chứng |
|---|---|---|---|
| YC-13 | Số lượng sản phẩm đặt mua trong một đơn hàng không được là số âm hoặc bằng 0. | Hệ thống phải kiểm tra tính hợp lệ của tham số `quantity` ở phía máy chủ trước khi xử lý, chỉ chấp nhận các số nguyên dương (`quantity >= 1`). | Dùng công cụ gửi yêu cầu HTTP POST tới `/mua` với tham số `quantity=-5`; máy chủ phải từ chối xử lý, không tạo bản ghi đơn hàng mới và không làm tăng số lượng tồn kho của sản phẩm. |

Vì sao MiniShop cần yêu cầu này: MiniShop là ứng dụng bán hàng trực tuyến có tính năng trừ kho và tính tiền dựa trên biểu thức `stock = stock - quantity` và `total = price * quantity`. Nếu thiếu khâu kiểm tra giá trị cận dưới của số lượng hàng đặt mua, hệ thống sẽ cho phép người dùng tự do gửi số lượng âm lên máy chủ.

Điều gì xảy ra nếu thiếu nó: Kẻ tấn công có thể cố tình gửi tham số `quantity` mang giá trị âm, dẫn đến việc phép trừ `stock - quantity` trở thành phép cộng, làm tăng khống số lượng hàng tồn kho trái phép hoặc sinh ra các đơn hàng có giá trị tiền âm, gây sai lệch nghiêm trọng toàn bộ dữ liệu tài chính và logic kinh doanh của cửa hàng.
