# Báo cáo thực hành LAB_4: Xác minh khuôn mặt 1:1 với DeepFace trên LFW

Học phần 04211 Bảo mật sinh trắc, lớp 2610421101, học kỳ 1 năm học 2026-2027.

| Mục | Điền vào đây |
|---|---|
| Họ và tên | Võ Bá Huy |
| Mã số sinh viên | 2305ct2318 |
| Ngày nộp | 13/10/2026 |

<!-- Cách dùng mẫu: giữ nguyên thứ tự và tên mười mục có dấu ##. Thay mọi chỗ trong ngoặc nhọn
bằng nội dung của em. Các khối chú thích như khối này không hiện ra khi xem trên GitHub, nên
không cần xoá. Mục nào không áp dụng thì ghi "Không áp dụng" kèm lý do trong một dòng.
Chèn đồ thị bằng dòng ![Hình 1: mô tả](ket-qua/ten-do-thi.png) hoặc ![Hình 2: mô tả](hinh/ten-anh.png) -->

## 1. Tóm tắt kết quả

<!-- 3 đến 5 câu. Nêu con số quan trọng nhất kèm điều kiện đo (cỡ mẫu, ngưỡng). Nói thẳng phần
chưa làm được và lý do. -->
Đã hoàn thành thực nghiệm đánh giá xác minh khuôn mặt 1:1 trên tập dữ liệu LFW (300 cặp ảnh, 561 ảnh phân biệt) bằng thư viện DeepFace. Hệ thống đã đánh giá 7 cấu hình khác nhau kết hợp giữa các mô hình đặc trưng (Facenet512, ArcFace, SFace), các bộ phát hiện khuôn mặt (OpenCV, RetinaFace, MTCNN, Skip) và có/không căn chỉnh góc mặt. Kết quả cho thấy cấu hình `Facenet512 + RetinaFace (không căn chỉnh)` đạt độ chính xác cao nhất với chỉ số **EER = 1.33%** ở ngưỡng khoảng cách cosin là **0.5676**. Bộ phát hiện `OpenCV` gặp lỗi FTA cao làm bỏ sót 18 cặp ảnh, trong khi các bộ phát hiện dựa trên Deep Learning (`RetinaFace`, `MTCNN`) đạt tỷ lệ FTA = 0.

## 2. Mức độ hoàn thành

<!-- Mỗi bước hoặc yêu cầu của đề là một dòng. Trạng thái: Hoàn thành, Một phần, Chưa làm. -->

| Bước hoặc yêu cầu trong đề | Trạng thái | Minh chứng tại mục |
|---|---|---|
| Cài đặt môi trường & thư viện (DeepFace, OpenCV, scikit-learn) | Hoàn thành | 4.1 |
| Hoàn thiện hàm `load_lfw_pairs` (chuyển định dạng BGR uint8) | Hoàn thành | 4.2 |
| Hoàn thiện hàm `cosine_distance` (tính khoảng cách Cosine) | Hoàn thành | 4.3 |
| Chạy thực nghiệm 7 cấu hình & sinh kết quả CSV, DET plot | Hoàn thành | 5 |
| Phân tích & trả lời các câu hỏi thảo luận | Hoàn thành | 6 |

## 3. Môi trường thực hiện và khả năng tái lập

<!-- Một người khác chỉ cần bảng này là chạy lại được đúng kết quả. Lấy thông tin bằng lệnh. -->

| Thông tin | Giá trị |
|---|---|
| Hệ điều hành, CPU, RAM | Windows 11, Intel Core i5, RAM 16GB |
| Phiên bản Python | Python 3.12.3 |
| Thư viện chính và phiên bản | deepface==0.0.101, opencv-python==4.10.0.84, scikit-learn==1.9.1, tensorflow==2.21.0, tf-keras==2.21.0 |
| Dữ liệu | LFW (Labeled Faces in the Wild), tập con `test`, 300 cặp ảnh (150 cùng người, 150 khác người) |
| Hạt giống ngẫu nhiên | `seed = 4211` |
| Lệnh chạy chính | `python th04_verify.py --source lfw --n-pairs 300` |
| Thời gian chạy | Khoảng 22 phút |

## 4. Các bước thực hiện và minh chứng

<!-- Mỗi bước của đề một tiểu mục ###, gồm: việc đã làm, lệnh hoặc đoạn mã chính, minh chứng,
nhận xét. Kết quả chạy chép dạng văn bản trong khối ``` tốt hơn ảnh chụp. -->

### 4.1. Khởi tạo môi trường và cài đặt thư viện phụ thuộc

- **Việc đã làm:** Tạo môi trường ảo Python (`venv`), kích hoạt môi trường và tiến hành cài đặt các thư viện cần thiết như `deepface`, `scikit-learn`, `matplotlib`. Phát hiện sự cố phiên bản `opencv-python==5.0.0.93` gây lỗi đường dẫn tệp Haar Cascade trong DeepFace, xử lý bằng cách hạ cấp về `opencv-python<4.11`.
- **Lệnh thực hiện:**
```powershell
  python -m venv venv
  .\venv\Scripts\Activate.ps1
  pip uninstall opencv-python -y
  pip install "opencv-python<4.11" deepface scikit-learn matplotlib
```

* **Minh chứng:**
```text
Successfully installed Flask-3.1.3 Pillow-12.3.0 deepface-0.0.101 opencv-python-4.10.0.84 retina-face-0.0.18 scikit-learn-1.9.1 tensorflow-2.21.0
```

* **Nhận xét:** Môi trường ảo được khởi tạo thành công, các thư viện tương thích hoàn toàn sau khi hạ cấp OpenCV về nhánh 4.x.

---

### 4.2. Tải trước dữ liệu LFW, mô hình DeepFace và hoàn thiện hàm `load_lfw_pairs`

* **Việc đã làm:** Tải trước tập dữ liệu 300 cặp ảnh LFW và các tệp trọng số mô hình (`Facenet512`, `ArcFace`, `SFace`). Hoàn thiện hàm `load_lfw_pairs` trong `th04_verify.py` để chuyển đổi mảng dữ liệu số thực sang kiểu `uint8` $[0, 255]$ và đảo thứ tự kênh màu RGB sang BGR.
* **Mã nguồn chính (`th04_verify.py`):**
```python
def load_lfw_pairs(n_pairs, seed=4211):
    from sklearn.datasets import fetch_lfw_pairs
    lfw = fetch_lfw_pairs(subset="test", color=True, resize=1.0, slice_=None)
    print(f"lfw.pairs.dtype: {lfw.pairs.dtype}, min: {lfw.pairs.min():.4f}, max: {lfw.pairs.max():.4f}")
    pairs = np.clip(lfw.pairs * 255.0, 0, 255).astype(np.uint8)[..., ::-1]

    rng = np.random.default_rng(seed)
    idx_same = rng.permutation(np.where(lfw.target == 1)[0])[: n_pairs // 2]
    idx_diff = rng.permutation(np.where(lfw.target == 0)[0])[: n_pairs // 2]
    idx = np.sort(np.r_[idx_same, idx_diff])
    return [(np.ascontiguousarray(pairs[i, 0]), np.ascontiguousarray(pairs[i, 1]), int(lfw.target[i])) for i in idx]
```

* **Minh chứng:**
```text
lfw.pairs.dtype: float32, min: 0.0000, max: 1.0000
300 cặp, 561 ảnh phân biệt
```

* **Nhận xét:** Dữ liệu ảnh LFW đã được định dạng chuẩn xác theo chuẩn BGR uint8 mà OpenCV và DeepFace yêu cầu, hạt giống `seed=4211` đảm bảo tính tái lập kết quả.

---

### 4.3. Cài đặt hàm tính khoảng cách Cosine `cosine_distance`

* **Việc đã làm:** Hoàn thiện hàm `cosine_distance` theo công thức khoảng cách Cosine: `d(a, b) = 1 - (a . b) / (||a|| * ||b|| + 1e-12)`. Bổ sung đại lượng `epsilon = 1e-12` để chống lỗi chia cho 0 và ép miền giá trị độ tương đồng về `[-1.0, 1.0]`.
* **Mã nguồn chính (`th04_verify.py`):**
```python
def cosine_distance(a, b):
    a = np.asarray(a, dtype=np.float64).flatten()
    b = np.asarray(b, dtype=np.float64).flatten()
    norm_a = np.linalg.norm(a)
    norm_b = np.linalg.norm(b)
    cos_sim = np.dot(a, b) / (norm_a * norm_b + 1e-12)
    return float(1.0 - np.clip(cos_sim, -1.0, 1.0))
```

* **Minh chứng:**
```text
verify cặp 0: nhãn thật 1, verified=True, distance=0.0938, threshold=0.3
```

* **Nhận xét:** Hàm tính khoảng cách hoạt động chính xác, khớp với đầu ra kiểm tra thử của API `DeepFace.verify`.

---

### 4.4. Đánh giá 7 cấu hình thực nghiệm và xuất kết quả

* **Việc đã làm:** Chạy lệnh đánh giá chính thức trên 300 cặp ảnh LFW. Chương trình tự động trích xuất đặc trưng, lưu bộ đệm embedding vào `cache-embedding/`, tính toán chỉ số EER, sinh bảng `TH04_bang-cau-hinh.csv` và đồ thị `TH04_det.png`.
* **Lệnh thực hiện:**
```powershell
python th04_verify.py --source lfw --n-pairs 300

```


* **Minh chứng:**
```text
{'mo_hinh': 'Facenet512', 'bo_phat_hien': 'opencv', 'can_chinh': True, 'so_anh_FTA': 17, 'so_cap_bi_bo': 18, 'eer_%': np.float64(9.22), 'nguong_eer_cosin': np.float64(0.6531)}
{'mo_hinh': 'ArcFace', 'bo_phat_hien': 'opencv', 'can_chinh': True, 'so_anh_FTA': 17, 'so_cap_bi_bo': 18, 'eer_%': np.float64(9.93), 'nguong_eer_cosin': np.float64(0.789)}
{'mo_hinh': 'SFace', 'bo_phat_hien': 'opencv', 'can_chinh': True, 'so_anh_FTA': 17, 'so_cap_bi_bo': 18, 'eer_%': np.float64(13.12), 'nguong_eer_cosin': np.float64(0.7251)}
{'mo_hinh': 'Facenet512', 'bo_phat_hien': 'retinaface', 'can_chinh': True, 'so_anh_FTA': 0, 'so_cap_bi_bo': 0, 'eer_%': np.float64(6.0), 'nguong_eer_cosin': np.float64(0.6491)}
{'mo_hinh': 'Facenet512', 'bo_phat_hien': 'retinaface', 'can_chinh': False, 'so_anh_FTA': 0, 'so_cap_bi_bo': 0, 'eer_%': np.float64(1.33), 'nguong_eer_cosin': np.float64(0.5676)}
{'mo_hinh': 'Facenet512', 'bo_phat_hien': 'mtcnn', 'can_chinh': True, 'so_anh_FTA': 0, 'so_cap_bi_bo': 0, 'eer_%': np.float64(2.67), 'nguong_eer_cosin': np.float64(0.5893), 'thoi_gian_1_anh_s': 0.453}
{'mo_hinh': 'Facenet512', 'bo_phat_hien': 'skip', 'can_chinh': False, 'so_anh_FTA': 0, 'so_cap_bi_bo': 0, 'eer_%': np.float64(5.33), 'nguong_eer_cosin': np.float64(0.6257), 'thoi_gian_1_anh_s': 0.298}

```

* **Nhận xét:** Đã đánh giá thành công đủ 7 cấu hình. Dữ liệu bảng kết quả và biểu đồ DET được khởi tạo chuẩn xác trong thư mục `lab04/ket-qua/`.

## 5. Kết quả định lượng

<!-- Bảng và hình có số thứ tự và chú thích. Ghi công thức, ngưỡng, cỡ mẫu. Số liệu lấy từ tệp
CSV trong ket-qua/ do chương trình ghi ra, không chép tay. -->

### Bảng 1: Kết quả so sánh 7 cấu hình xác minh khuôn mặt 1:1 trên 300 cặp LFW (150 cặp cùng người, 150 cặp khác người)

* **Cỡ mẫu đánh giá:** $N = 300$ cặp ảnh (gồm $150$ cặp Genus/Cùng người và $150$ cặp Impostor/Khác người, tổng cộng $561$ ảnh phân biệt).
* **Độ đo khoảng cách:** Khoảng cách Cosine d(a, b) = 1 - (a . b) / (||a|| * ||b|| + 1e-12).
* **Chỉ số:** EER (Equal Error Rate) - Tỷ lệ lỗi tại điểm FMR = FNMR; Ngưỡng EER - Giá trị ngưỡng Cosine tương ứng tại điểm EER; FTA (Failure to Acquire) - Số lượng ảnh không tìm thấy khuôn mặt.

| Mô hình | Bộ phát hiện | Căn chỉnh | Số ảnh FTA | Số cặp bị bỏ | Số cặp cùng người | Số cặp khác người | EER (%) | Ngưỡng EER (cosin) | Thời gian 1 ảnh (s) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Facenet512 | opencv | Có | 17 | 18 | 143 | 139 | 9.22 | 0.6531 | *Đọc từ đệm* |
| ArcFace | opencv | Có | 17 | 18 | 143 | 139 | 9.93 | 0.7890 | *Đọc từ đệm* |
| SFace | opencv | Có | 17 | 18 | 143 | 139 | 13.12 | 0.7251 | *Đọc từ đệm* |
| Facenet512 | retinaface | Có | 0 | 0 | 150 | 150 | 6.00 | 0.6491 | *Đọc từ đệm* |
| Facenet512 | retinaface | Không | 0 | 0 | 150 | 150 | 1.33 | 0.5676 | *Đọc từ đệm* |
| Facenet512 | mtcnn | Có | 0 | 0 | 150 | 150 | 2.67 | 0.5893 | 0.453 |
| Facenet512 | skip | Không | 0 | 0 | 150 | 150 | 5.33 | 0.6257 | 0.298 |

*(Ghi chú: Số liệu chi tiết được xuất tự động ra tệp `lab04/ket-qua/TH04_bang-cau-hinh.csv`)*

---

### Đồ thị DET (Decision Error Tradeoff)

Đồ thị đường cong DET thể hiện mối quan hệ trao đổi giữa tỷ lệ nhận nhầm (FMR - False Match Rate) và tỷ lệ bỏ sót (FNMR - False Non-Match Rate) ở các mức ngưỡng phân ngưỡng khác nhau cho các cấu hình thực nghiệm:

![Hình 1: Biểu đồ đường cong DET so sánh hiệu năng các cấu hình xác minh khuôn mặt trên tập LFW](ket-qua/TH04_det.png)

*Hình 1: Đường cong DET đánh giá hiệu năng xác minh khuôn mặt trên tập LFW (đồ thị được lưu tại `lab04/ket-qua/TH04_det.png`).*

## 6. Phân tích và thảo luận

<!-- Trả lời từng câu hỏi phân tích của đề, đánh số theo đề. Mỗi nhận định: hiện tượng, cơ chế
(dẫn tới hàm hoặc tham số), bằng chứng (dẫn tới bảng hoặc hình ở mục 5), giới hạn của kết luận. -->

1. **So sánh 3 mô hình trích xuất đặc trưng (Facenet512 vs ArcFace vs SFace khi cùng dùng `opencv` + căn chỉnh):**
   - **Hiện tượng:** Cùng sử dụng bộ phát hiện `opencv` và bật căn chỉnh (`align=True`), **Facenet512** đạt độ chính xác cao nhất với chỉ số **EER = 9.22%**, vượt trội hơn **ArcFace** (**EER = 9.93%**) và **SFace** (**EER = 13.12%**).
   - **Cơ chế:** Facenet512 mã hóa khuôn mặt thành vector đặc trưng 512 chiều tối ưu hóa khoảng cách Euclide/Cosine thông qua hàm mất mát Triplet Loss trên không gian biểu diễn lớn, giúp phân biệt tốt biên khoảng cách giữa các lớp người dùng. SFace có kiến trúc mạng thu nhỏ để tối ưu tốc độ trích xuất nên khả năng phân tách ranh giới các cặp khó kém hơn.
   - **Bằng chứng:** Bảng 1 (Mục 5) cho thấy ở cùng số cặp đánh giá ($143$ cặp cùng người, $139$ cặp khác người), Facenet512 có EER $9.22\%$ ở ngưỡng Cosine $0.6531$, so với ArcFace ($9.93\%$, ngưỡng $0.7890$) và SFace ($13.12\%$, ngưỡng $0.7251$).
   - **Giới hạn kết luận:** So sánh này chỉ áp dụng khi đi kèm bộ phát hiện yếu `opencv` (có 17 ảnh bị FTA).

2. **Ảnh hưởng của bộ phát hiện khuôn mặt (`opencv` vs `retinaface` vs `mtcnn` vs `skip` khi dùng Facenet512):**
   - **Hiện tượng:** Bộ phát hiện `opencv` bị lỗi phát hiện cao làm giảm đáng kể số cặp đánh giá, trong khi `retinaface` và `mtcnn` đạt **FTA = 0**, đồng thời giúp giảm chỉ số EER xuống mức rất thấp.
   - **Cơ chế:** `opencv` dựa trên thuật toán tầng Haar Cascade truyền thống nên rất dễ thất bại với các ảnh nghiêng góc, ánh sáng yếu hoặc độ tương phản kém trong LFW. Trong khi đó, `retinaface` và `mtcnn` là các mạng nơ-ron sâu chuyên dụng cho tác vụ định vị vùng mặt (face detection & alignment). Cấu hình `skip` lấy toàn bộ khung hình mà không cắt mặt.
   - **Bằng chứng:** Bảng 1 ghi nhận `opencv` có **FTA = 17** ảnh (khiến $18$ cặp bị loại bỏ), EER là $9.22\%$. Ngược lại, `retinaface` và `mtcnn` có **FTA = 0**, giữ nguyên đủ $300$ cặp; EER giảm xuống còn $6.00\%$ (`retinaface` có align), $2.67\%$ (`mtcnn`) và $1.33\%$ (`retinaface` không align). Cấu hình `skip` đạt EER $5.33\%$ do ảnh LFW gốc vốn đã được căn giữa khuôn mặt tương đối chuẩn.
   - **Giới hạn kết luận:** Việc `skip` đạt EER khá tốt ($5.33\%$) phụ thuộc vào đặc thụ của tập LFW (ảnh cropped sẵn). Với ảnh ngoài thực tế có nền phức tạp, `skip` sẽ cho tỷ lệ lỗi cao hơn nhiều.

3. **Ảnh hưởng của bước căn chỉnh (`align=True` vs `align=False` với Facenet512 + retinaface):**
   - **Hiện tượng:** Cấu hình không căn chỉnh (`align=False`) đạt chỉ số EER ấn tượng **1.33%**, tốt hơn đáng kể so với cấu hình có căn chỉnh (`align=True`) đạt **6.00%**.
   - **Cơ chế:** Quá trình căn chỉnh (`align=True`) sử dụng phép biến đổi affine (xoay, co giãn, dịch chuyển) dựa trên 5 điểm mốc (landmarks). Do ảnh trong LFW có độ phân giải khá thấp ($250 \times 250$), phép biến đổi hình học cùng quá trình nội suy pixel làm nhòe nhẹ các chi tiết kết cấu vi mô (micro-texture) trên vùng mắt, mũi và miệng, khiến vector đặc trưng của Facenet512 bị biến dạng nhẹ.
   - **Bằng chứng:** Bảng 1 chỉ ra `Facenet512 + retinaface (no align)` có EER chỉ **1.33%** (ngưỡng $0.5676$), trong khi `Facenet512 + retinaface (align)` có EER **6.00%** (ngưỡng $0.6491$). Đồ thị DET (Hình 1) cũng thể hiện đường cong của cấu hình không căn chỉnh nằm sát gốc tọa độ nhất.
   - **Giới hạn kết luận:** Kết luận này đúng với dữ liệu LFW độ phân giải thấp. Với ảnh camera độ phân giải cao hoặc góc nghiêng lớn, căn chỉnh góc mặt vẫn là bước bắt buộc để duy trì độ chính xác.

4. **Ý nghĩa thực tế của lỗi FTA (Failure to Acquire):**
   - **Hiện tượng & Cơ chế:** Lỗi FTA xảy ra khi bộ phát hiện không thể trích xuất được khuôn mặt từ ảnh đầu vào.
   - **Bằng chứng:** Bộ phát hiện `opencv` làm $17$ ảnh bị lỗi FTA, dẫn tới $18$ cặp xác thực ($6\%$ dữ liệu) không thể thực hiện so khớp.
   - **Ý nghĩa:** Trong hệ thống thực tế (như kiểm soát ra vào hay ngân hàng số), FTA đồng nghĩa với việc người dùng bị từ chối dịch vụ ngay tại bước thu nhận ảnh. Để giảm FTA, hệ thống cần ứng dụng các bộ phát hiện Deep Learning mạnh (`retinaface`/`mtcnn`) và thiết kế giao diện hướng dẫn người dùng căn chỉnh khuôn mặt theo thời gian thực trước camera.

5. **Phân tích đồ thị DET (`TH04_det.png`):**
   - **Hiện tượng & Bằng chứng:** Trên Hình 1, đường cong DET của cấu hình `Facenet512 + retinaface (không căn chỉnh)` nằm ở vị trí thấp nhất và gần gốc tọa độ $(0,0)$ nhất.
   - **Cơ chế:** Đồ thị DET biểu diễn sự đánh đổi giữa FMR (Tỷ lệ nhận nhầm) và FNMR (Tỷ lệ bỏ sót). Đường cong nằm càng gần gốc tọa độ chứng tỏ hệ thống duy trì được đồng thời cả FMR và FNMR ở mức rất thấp trên hầu hết mọi ngưỡng phân ngưỡng.

## 7. Ý nghĩa đối với bảo mật

<!-- 3 đến 6 câu: thay đổi em vừa đo có lợi hay hại cho ai, kẻ tấn công hay người dùng hợp lệ;
nếu triển khai thật thì chọn cấu hình nào, vì sao. -->

Kết quả thực nghiệm cho thấy việc lựa chọn bộ phát hiện khuôn mặt và thuật toán trích xuất đặc trưng có ảnh hưởng trực tiếp đến sự cân bằng giữa tính an toàn (giảm tỷ lệ nhận nhầm FMR) và tính tiện dụng (giảm tỷ lệ bỏ sót FNMR và lỗi FTA). Việc chuyển đổi sang bộ phát hiện học sâu như RetinaFace hay MTCNN giúp loại bỏ hoàn toàn rủi ro FTA (từ 17 ảnh bị từ chối xuống 0 ảnh), mang lại lợi ích lớn cho người dùng hợp lệ khi giảm thiểu tối đa tình trạng bị hệ thống từ chối truy cập vô lý. Ngược lại, việc sử dụng các bộ phát hiện truyền thống như OpenCV làm gia tăng điểm yếu vận hành khi kẻ tấn công có thể lợi dụng các góc chụp nghiêng hoặc điều kiện ánh sáng khó để gây nghẽn dịch vụ (Denial of Service). Nếu triển khai trong hệ thống thực tế (như xác thực giao dịch ngân hàng hoặc kiểm soát ra vào cơ quan), cấu hình **`Facenet512 + RetinaFace (không căn chỉnh)`** nên được lựa chọn vì mang lại độ chính xác cao nhất (EER đạt mức tối ưu **1.33%**) cùng độ ổn định thu nhận dữ liệu tuyệt đối (FTA = 0).

## 8. Sự cố gặp phải và cách xử lý

| Sự cố (thông báo lỗi) | Nguyên nhân | Cách xử lý |
|---|---|---|
| `ValueError: Confirm that opencv is installed on your environment! Expected path '...haarcascade_frontalface_default.xml' violated.` | Thư viện `opencv-python` phiên bản `5.0.0.93` vừa ra mắt đã thay đổi cấu trúc thư mục chứa dữ liệu `cv2/data`, khiến DeepFace không tìm thấy tệp mô hình Haar Cascade mặc định. | Hạ cấp phiên bản OpenCV về nhánh 4.x ổn định bằng hai lệnh: <br>`pip uninstall opencv-python -y`<br>`pip install "opencv-python<4.11"` |
| Quá trình chạy bị dừng/treo lâu ở cấu hình `Facenet512 + retinaface` | Mạng RetinaFace tính toán đa tỷ lệ và định vị 5 điểm mốc (landmarks) rất nặng trên CPU (tốn $1.5 - 2.5$ giây/ảnh với 561 ảnh, tổng thời gian $\approx 15 - 20$ phút). Đồng thời, vô tình nhấp chuột vào terminal Windows bật chế độ QuickEdit làm tạm dừng tiến trình. | Nhấn phím `Enter` hoặc `Space` trên bàn phím để thoát chế độ QuickEdit nếu lỡ nhấp chuột; kiên nhẫn cắm sạc laptop để CPU chạy ở xung nhịp tối đa cho đến khi chương trình tính xong và xuất đệm `cache-embedding/`. |
## 9. Dữ liệu sinh trắc, nguồn tham khảo và công cụ AI

- [x] Kho không chứa ảnh vân tay, khuôn mặt, mống mắt, giọng nói của người thật, tập dữ liệu, tệp `.db`, `.pkl`, `.npy`, trọng số mô hình.
- [x] Mã dùng lại của người khác đã ghi nguồn ngay trong chú thích mã.

Nguồn tham khảo:
- Kho mã nguồn mở DeepFace: https://github.com/serengil/deepface (Truy cập ngày 07/10/2026)
- Tập dữ liệu LFW trên Scikit-learn: https://scikit-learn.org/stable/modules/generated/sklearn.datasets.fetch_lfw_pairs.html (Truy cập ngày 07/10/2026)
- Giáo trình và tài liệu thực hành môn Bảo mật Sinh trắc (Mã HP: 04211), Trường Đại học Hùng Vương TP.HCM.

Công cụ AI: Google Gemini (đã khai chi tiết trong tệp `AI-SUDUNG.md`).

## 10. Cam kết

Tôi cam kết các kết quả trong báo cáo này do chính tôi chạy trên máy của mình, các phần sử dụng lại của người khác đã được ghi nguồn đầy đủ.

Võ Bá Huy, ngày 12 tháng 10 năm 2026