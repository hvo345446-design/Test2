# Lab OpenSSF "deserialization", buổi 4

Tệp mẫu của Phần A. Lưu thành `bao-cao/tuan04/openssf-deserialization.md` trong kho cá nhân, rồi thay mọi chỗ `[điền ...]` bằng bài của mình. Tệp nộp còn chữ `[điền` là tệp chưa xong. Cách làm lab nằm ở Mục 9 của tài liệu thực hành buổi 4.

## 1. Lời giải của tôi

Dán nội dung hai ô đã sửa, đúng như lúc trang báo `COMPLETE!`: dòng thứ nhất là ô một dòng, các dòng sau là ô ba dòng.

```javascript
[const data = JSON.parse(base64Decoded);]
[if (data.username && typeof data.username == 'string' && data.username.length < 20) {]
```

## 2. Dòng xác nhận của trang

Chép nguyên dòng bắt đầu bằng `Completed` hiện dưới ba nút Hint, Reset, Give up. Dòng có đuôi `(GA)` vẫn được nhận.

```text
[điền: dòng Completed ...]
```

## 3. Giải thích bằng tiếng Việt

Viết ba tới năm câu, bằng lời của mình, trả lời ít nhất ba trong bốn câu hỏi: (a) `eval()` làm gì với chuỗi lấy từ cookie, và cookie vượt ranh giới tin cậy nào; (b) `JSON.parse()` khác `eval()` ở chỗ nào; (c) vì sao đọc bằng `JSON.parse()` rồi vẫn phải kiểm kiểu và độ dài của `data.username`; (d) MiniShop giữ gì trong cookie, và vì sao cách này an toàn hơn.

[điền: ba tới năm câu tiếng Việt]
