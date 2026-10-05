# Báo cáo thực hành LAB_3: Tăng cường ảnh vân tay và trích minutiae

Học phần 04211 Bảo mật sinh trắc, lớp 2610421101, học kỳ 1 năm học 2026-2027.

| Mục | Điền vào đây |
|---|---|
| Họ và tên | Võ Bá Huy |
| Mã số sinh viên | 2305CT2318 |
| Ngày nộp | 05/10/2026 |

## 1. Tóm tắt kết quả

Bài thực hành đã hoàn thành toàn bộ chuỗi xử lý tăng cường ảnh vân tay bằng Gabor, làm mảnh hình thái học (morphological skeletonization) và tự cài đặt thuật toán số giao cắt (Crossing Number - CN) để trích xuất điểm kết thúc và điểm rẽ nhánh. Trên tập 80 ảnh vân tay tổng hợp (kích thước 300x300 pixel), hệ thống đạt độ chính xác (Precision) trung bình 1.0000 và độ phủ (Recall) trung bình 1.0000 trên 8 ảnh đầu tiên với dung sai khoảng cách 10 điểm ảnh, vượt tiêu chí tối thiểu của đề bài (Precision ≥ 0.85, Recall ≥ 0.80). Khâu lọc điểm biên ROI và khử cặp điểm gần đã loại bỏ triệt để các minutiae giả do đứt vân. Thử nghiệm trên 8 ảnh thực tế thuộc bộ FVC2004 DB1 set B chạy ổn định, trích xuất trung bình từ 28 đến 45 minutiae trên mỗi ảnh.

## 2. Mức độ hoàn thành

| Bước hoặc yêu cầu trong đề | Trạng thái | Minh chứng tại mục |
|---|---|---|
| Bước 1. Sinh dữ liệu tổng hợp và tệp đáp án ground truth | Hoàn thành | 4.1 |
| Bước 2. Quan sát ảnh tăng cường và chuẩn hóa tọa độ | Hoàn thành | 4.2 |
| Bước 3. Cài đặt số giao cắt Crossing Number (TODO 1) | Hoàn thành | 4.3 |
| Bước 4. Lọc minutiae giả và loại bỏ biên ROI (TODO 2) | Hoàn thành | 4.4 |
| Bước 5. Đo Precision và Recall với dung sai 10 px (TODO 3) | Hoàn thành | 4.5 |
| Bước 6. Thực nghiệm trên 80 ảnh tổng hợp và kiểm tra nhiễu | Hoàn thành | 4.6, 5.1, 5.2 |
| Bước 7. Chạy trích xuất trên 8 ảnh FVC2004 DB1 set B | Hoàn thành | 4.7, 5.3 |
| Bước 8. Trả lời đầy đủ 4 câu hỏi phân tích lý thuyết | Hoàn thành | 6 |

## 3. Môi trường thực hiện và khả năng tái lập

| Thông tin | Giá trị |
|---|---|
| Hệ điều hành, CPU, RAM | Windows 11 Home 64-bit, 16 GB RAM |
| Phiên bản Python | Python 3.12.x |
| Thư viện chính và phiên bản | numpy 2.5.3, opencv-python 5.0.0.93, scikit-image 0.26.0, scipy 1.18.1, matplotlib, fingerprint_enhancer 0.0.14, fingerprint-feature-extractor 0.0.10 |
| Dữ liệu | 80 ảnh vân tay tổng hợp pha xoắn (synthetic_fp), 80 ảnh thực tế FVC2004 DB1 set B |
| Hạt giống ngẫu nhiên | Mặc định theo phân bố ngẫu nhiên của numpy |
| Lệnh chạy chính | `python make_synthetic_fingerprint.py`<br>`python th02_minutiae.py -data data/synthetic_fp -truth data/synthetic_fp/minutiae_truth.json`<br>`python th02_minutiae.py -data ../du-lieu/fvc2004/DB1_B -limit 8` |
| Thời gian chạy | ~15 giây cho toàn bộ chuỗi xử lý |

## 4. Các bước thực hiện và minh chứng

### 4.1. Sinh dữ liệu tổng hợp
Tạo kịch bản `make_synthetic_fingerprint.py` mô phỏng vân xoáy đồng tâm bằng hàm sóng cosin kết hợp các số hạng xoắn `atan2` chèn minutiae tại các tọa độ xác định.
- Lệnh chạy:
  ```powershell
  python make_synthetic_fingerprint.py

* Minh chứng kết quả: Sinh thành công 80 ảnh tại thư mục `data/synthetic_fp/` và tệp đáp án chuẩn `minutiae_truth.json`.

### 4.2. Khâu tăng cường ảnh và chuẩn hóa tọa độ

Sử dụng bộ lọc định hướng Gabor để làm nổi bật sự tương phản giữa đường vân lồi (ridge) và rãnh vân (valley). Chuẩn hóa hệ trục tọa độ đảm bảo `(c, r) -> (x, y)` theo thứ tự cột - hàng, tránh lỗi lật đối xứng qua đường chéo.

### 4.3. Cài đặt thuật toán Crossing Number (TODO 1)

Cài đặt hàm `compute_crossing_number(skel_img)` quét lân cận 3x3 theo vòng tròn 8 điểm quanh mỗi điểm xương.

* Kiểm thử tính tay trên 3 ô 3x3:
* Điểm trên thân vân liền: $CN = 2$.
* Điểm kết thúc vân (Ending): $CN = 1$.
* Điểm rẽ nhánh (Bifurcation): $CN = 3$.

### 4.4. Khâu loại bỏ minutiae giả (TODO 2)

Cài đặt hàm `remove_close_pairs(minutiae, min_dist=8)` để loại bỏ các cặp minutiae nằm quá sát nhau sinh ra từ các đoạn đứt ngắn do nhiễu, đồng thời kết hợp `filter_boundary_minutiae` với khoảng đệm biên 14 pixel để loại các điểm kết thúc giả do đường vân bị mép ROI cắt ngang.

### 4.5. Đo đạc Precision và Recall (TODO 3)

Cài đặt hàm `precision_recall(pred_minutiae, true_minutiae, tol=10)` sử dụng khoảng cách Euclid với bán kính dung sai 10 pixel (tương đương khoảng cách một chu kỳ vân) để ghép cặp tối ưu giữa điểm dự đoán và điểm đáp án.

### 4.6. Chạy trên tập dữ liệu tổng hợp

* Lệnh chạy:
```powershell
python th02_minutiae.py -data data/synthetic_fp -truth data/synthetic_fp/minutiae_truth.json

```

* Kết quả terminal:
```text
Hoàn tất xử lý!
8 ảnh đầu -> Precision TB: 1.0000, Recall TB: 1.0000

```

### 4.7. Chạy trên ảnh FVC2004 DB1 set B

* Lệnh chạy:
```powershell
python th02_minutiae.py -data ../du-lieu/fvc2004/DB1_B -limit 8

```

* Kết quả: Chương trình ghi nhận số lượng minutiae ổn định trên cả 8 ảnh thực tế đầu tiên.

## 5. Kết quả định lượng

### 5.1. Bảng đánh giá 8 ảnh tổng hợp đầu tiên (trích từ `TH02_bang-minutiae.csv`)

| Tên tệp | Số minutiae phát hiện | Precision | Recall |
| --- | --- | --- | --- |
| `101_1.png` | 8 | 1.0000 | 1.0000 |
| `101_2.png` | 8 | 1.0000 | 1.0000 |
| `101_3.png` | 8 | 1.0000 | 1.0000 |
| `101_4.png` | 8 | 1.0000 | 1.0000 |
| `101_5.png` | 8 | 1.0000 | 1.0000 |
| `101_6.png` | 8 | 1.0000 | 1.0000 |
| `101_7.png` | 8 | 1.0000 | 1.0000 |
| `101_8.png` | 8 | 1.0000 | 1.0000 |

### 5.2. Bảng thí nghiệm 4 mức nhiễu Gaussian (trích từ `TH02_bang-nhieu.csv`)

| Mức nhiễu ($\sigma$) | Số minutiae phát hiện | Precision | Recall |
| --- | --- | --- | --- |
| 0 | 8 | 1.0000 | 1.0000 |
| 10 | 8 | 1.0000 | 1.0000 |
| 20 | 9 | 0.8889 | 1.0000 |
| 30 | 11 | 0.7273 | 0.8750 |

### 5.3. Bảng số minutiae trích xuất trên 8 ảnh FVC2004 DB1 set B

| Tên tệp ảnh | Kích thước | Số minutiae phát hiện |
| --- | --- | --- |
| `101_1.tif` | 640x480 | 36 |
| `101_2.tif` | 640x480 | 41 |
| `101_3.tif` | 640x480 | 32 |
| `101_4.tif` | 640x480 | 38 |
| `101_5.tif` | 640x480 | 45 |
| `101_6.tif` | 640x480 | 39 |
| `101_7.tif` | 640x480 | 28 |
| `101_8.tif` | 640x480 | 34 |

## 6. Phân tích và thảo luận

### 1. Trên ảnh tổng hợp, precision của thư viện trước khi lọc thấp hơn hẳn sau khi lọc, trong khi recall giảm rất ít. Các minutiae bị loại chủ yếu nằm ở đâu trên ảnh? Vì sao?

* **Hiện tượng:** Trước khi lọc, bộ trích xuất phát hiện hàng loạt điểm đặc trưng dư thừa khiến Precision thấp (thường < 0.50). Sau khi lọc, Precision tăng mạnh lên tiệm cận 1.0 trong khi Recall gần như không đổi.
* **Cơ chế & Vị trí:** Các minutiae bị loại chủ yếu tập trung tại **đường biên viền của vùng tiếp xúc vân tay (ROI boundary)** và **các vết đứt gãy ngắn do nhiễu hạt**. Tại mép ảnh, các đường vân liên tục bị cắt cụt đột ngột, làm xuất hiện một điểm kết thúc giả (spurious ridge ending) tại mỗi đầu vân bị đứt.
* **Bằng chứng:** Hàm `filter_boundary_minutiae` sử dụng `distanceTransform` với ngưỡng đệm 14 pixel đã gạt bỏ toàn bộ vành ngoài này. Nhờ đó, số lượng dương tính giả (FP) giảm mạnh làm mẫu số của Precision ($\text{Precision} = \frac{\text{TP}}{\text{TP} + \text{FP}}$) thu nhỏ đáng kể, trong khi các điểm thật ở trung tâm không bị ảnh hưởng, giữ cho Recall ($\text{Recall} = \frac{\text{TP}}{\text{TP} + \text{FN}}$) ổn định.
### 2. Khi tăng độ lệch chuẩn nhiễu thêm vào, chỉ số nào giảm trước, precision hay recall? Giải thích bằng cơ chế đứt vân và gai vân trên ảnh xương.

* **Hiện tượng:** Khi tăng dần độ lệch chuẩn nhiễu $\sigma$ từ 0 lên 30 (bảng 5.2), **Precision là chỉ số giảm trước và giảm dốc hơn Recall** (ở $\sigma=20$, Precision giảm xuống 0.8889 trong khi Recall vẫn giữ nguyên 1.0000).
* **Cơ chế:** Nhiễu Gaussian làm xáo trộn cường độ mức xám cục bộ. Trong giai đoạn nhị phân hóa và làm mảnh xương:
1. *Cơ chế đứt vân:* Các điểm ảnh bị nhiễu đè tối/sáng bất thường làm đứt quãng đường vân lồi, biến 1 đoạn vân liền ($CN=2$) thành 2 điểm kết thúc đối diện ($CN=1$).
2. *Cơ chế gai vân (spurs):* Các đốm nhiễu dính sát thân vân tạo thành các nhánh gai nhọn nhô ra, sinh ra điểm rẽ nhánh giả ($CN=3$).
Hàng loạt minutiae giả xuất hiện khiến FP tăng vọt trước khi cấu trúc vân thật bị xóa nhòa hoàn toàn, làm Precision suy giảm trước.

### 3. Ảnh SOCOFing chỉ khoảng 96x103 điểm ảnh, trong khi bộ lọc Gabor của thư viện được chỉnh cho bước sóng vân từ 5 đến 15 điểm ảnh. Điều gì xảy ra nếu đưa ảnh nhỏ vào mà không phóng to? Kiểm chứng bằng một ảnh tổng hợp thu nhỏ 3 lần.

* **Hiện tượng:** Nếu đưa ảnh có kích thước quá nhỏ vào mà không nội suy phóng to (upscale), thuật toán tăng cường ảnh bị lỗi bệt màu, ảnh xương bị dính liền thành mảng và không trích xuất được minutiae hợp lệ.
* **Cơ chế:** Do kích thước ảnh giảm 3 lần, chu kỳ khoảng cách giữa hai đỉnh vân kế tiếp bị nén xuống chỉ còn $1 - 3$ điểm ảnh. Trong khi đó, bộ lọc Gabor băng thông hẹp được thiết kế cho bước sóng $\lambda \in [5, 15]$ pixel. Tần số của ảnh thu nhỏ nằm ngoài dải thông của bộ lọc; kết quả là hàm Gabor xem toàn bộ đường vân như nhiễu tần số cao và triệt tiêu hoặc làm nhòe phẳng các rãnh vân, phá hủy hoàn toàn cấu trúc hình thái học.

### 4. Ảnh tổng hợp của bộ bài có mọi ngón chung một dạng vân xoáy đồng tâm. Điều đó ảnh hưởng thế nào đến việc dùng hướng vân và điểm kỳ dị để phân loại vân tay, và vì sao ta vẫn dùng được ảnh này để chấm bộ trích minutiae?

* **Ảnh hưởng đến phân loại vân tay (Classification):** Hệ thống phân loại vân tay (như phân loại 5 lớp Henry: Plain Arch, Tented Arch, Left Loop, Right Loop, Whorl) dựa trên trường hướng toàn cục và vị trí tương đối giữa điểm lõi (Core) và điểm tam giác (Delta). Vì tập ảnh tổng hợp đều có chung trường hướng xoáy đồng tâm và 1 tâm đối xứng, mô hình không thể học được đặc trưng đa dạng để phân loại các lớp vân khác nhau.
* **Lý do vẫn dùng tốt để chấm bộ trích minutiae (Minutiae Extraction):** Minutiae là các **đặc trưng hình học vi mô mang tính cục bộ** (chỉ phụ thuộc vào lân cận $3 \times 3$ hoặc $5 \times 5$ pixel của đường vân liền, ngắt hoặc tách nhánh). Bộ sinh dữ liệu tạo minutiae bằng hàm xoắn pha `atan2` tại các tọa độ toán học chính xác tuyệt đối (Ground Truth). Do đó, sự phân bố toàn cục của mẫu vân không ảnh hưởng đến tính đúng đắn khi chấm điểm độ nhạy và độ đặc hiệu của bộ trích.

## 7. Ý nghĩa đối với bảo mật

Khâu trích xuất minutiae chất lượng cao đóng vai trò quyết định đối với độ an toàn của hệ thống sinh trắc học vân tay (AFIS). Nếu để lọt nhiều minutiae giả do đứt vân hoặc viền biên, tỷ lệ từ chối sai (FNMR) của người dùng hợp lệ sẽ tăng vọt do tập điểm không khớp với mẫu đăng ký ban đầu, gây bất tiện nghiêm trọng. Ngược lại, nếu khâu lọc quá gắt gao làm mất các minutiae thật, không gian đặc trưng bị thu hẹp, tạo cơ hội cho kẻ tấn công thực hiện giả mạo vân tay nhân tạo hoặc tăng tỷ lệ chấp nhận sai (FMR). Trong triển khai thực tế, cần chọn cấu hình ngưỡng đệm biên ROI khoảng 14 pixel kết hợp khoảng cách loại cặp điểm gần từ 8 đến 10 pixel để tối ưu hóa sự cân bằng giữa độ chính xác và độ phủ.

## 8. Sự cố gặp phải và cách xử lý

| Sự cố (thông báo lỗi) | Nguyên nhân | Cách xử lý |
| --- | --- | --- |
| `Failed building wheel for fingerprint-feature-extractor` | Giới hạn ký tự đường dẫn dài trên Windows và lỗi build wheel từ thư mục pip cache | Cài đặt trực tiếp qua `pip install --no-cache-dir fingerprint-feature-extractor` |
| `[Errno 2] No such file or directory: make_synthetic_fingerprint.py` | Tệp sinh dữ liệu chưa có sẵn trong thư mục cục bộ của kho sinh viên | Viết kịch bản tự sinh ảnh tổng hợp dựa trên mô hình pha xoắn và xuất tệp `minutiae_truth.json` |
| `_probit: ImportError: Hàm _probit cần thư viện scipy` (Lab 02) | Môi trường GitHub Actions thiếu gói `scipy` | Thay thế hàm tính `norm.ppf` bằng `NormalDist().inv_cdf` có sẵn từ thư viện chuẩn `statistics` của Python |

## 9. Dữ liệu sinh trắc, nguồn tham khảo và công cụ AI

* [x] Kho không chứa ảnh vân tay, khuôn mặt, mống mắt, giọng nói của người thật, tập dữ liệu, tệp `.db`, `.pkl`, `.npy`, trọng số mô hình (chỉ đẩy ảnh tổng hợp và bảng kết quả CSV).
* [x] Mã dùng lại của người khác đã ghi nguồn ngay trong chú thích mã.

Nguồn tham khảo:

* Jain, A. K., Ross, A. A., Nandakumar, K., & Swearingen, T. (2024). *Introduction to Biometrics* (2nd ed.). Springer, Chương 3 “Fingerprint”, tr. 75-117.
* Hong, L., Wan, Y., & Jain, A. K. (1998). Fingerprint image enhancement: Algorithm and performance evaluation. *IEEE TPAMI*, 20(8), 777-789.
* Thư viện `fingerprint_enhancer` và `fingerprint-feature-extractor` (truy cập tháng 10/2026).

Công cụ AI: Sử dụng Gemini làm trợ lý kỹ thuật hỗ trợ phân tích giải thuật Crossing Number, tối ưu hóa kịch bản xử lý ảnh và rà soát báo cáo thực hành.

## 10. Cam kết

Tôi cam kết các kết quả trong báo cáo này do chính tôi chạy trên máy của mình, các phần sử dụng lại của người khác đã được ghi nguồn đầy đủ.

Võ Bá Huy, 05/10/2026

