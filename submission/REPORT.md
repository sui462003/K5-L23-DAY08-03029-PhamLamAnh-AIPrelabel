# Báo cáo Day 08 — 03029 - Phạm Lam Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26n / YOLO26m | 0.50 | Đạt recall 100% trên gold, cân bằng precision (33%), dẫn từ `run_02` |
| motorbike | YOLO26n / YOLO26m | 0.35 | Recall 100% trên gold, precision 67%, giữ được xe máy ở xa (`run_01`) |
| car / bicycle / bus / truck | YOLO26n / YOLO26m | 0.30 (mặc định) | Lớp hiếm (< 5 vật trong gold), giữ ngưỡng mặc định an toàn (`run_01`) |
| three_wheeler | YOLOE-26s (prompt: three_wheeler) | 0.15 | Recall 100%, precision 78% trên 7 vật gold, bù đắp việc COCO thiếu nhãn (`run_03`) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn:** SAM 2.1 (`sam2.1_t.pt` / `sam2.1_b.pt`). Lý do: SAM 2.1 tạo đường viền mask sắc nét hơn hẳn SAM 1, tốc độ xử lý nhanh, không bị tràn bộ nhớ RAM, rất phù hợp khi tương tác qua click điểm (`run_04`).
- Lỗi nào **không sửa được bằng đổi ngưỡng:** Xe tuk-tuk và xe ba gác bị YOLO26 nhận diện thành `car` hoặc `truck` vì tập COCO không có lớp `three_wheeler`. Sửa bằng cách dùng model open-vocabulary YOLOE với prompt chữ `three_wheeler` và cấu hình mức ưu tiên đè box khi gộp qua `merge_prelabels.py`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng:** Trong `img_004.jpg`, một xe ba gác chở hàng bị YOLO26 tự tin gán `truck` (điểm 0.72) nhưng thực tế là `three_wheeler`. Ngược lại, YOLOE nhận diện tuk-tuk trong `img_013.jpg` ở điểm chỉ 0.18 nhưng box bám rất chuẩn.
- Giới hạn: Việc quét và chọn ngưỡng thực hiện trên 4 ảnh gold (24 đối tượng) nên chỉ số precision/recall trên tập gold mang tính lạc quan (in-sample).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 382 | 382/429 = 89.0% | 429 box máy |
| edited (sửa) | 35 | 35/429 = 8.2% | 429 box máy |
| deleted (xoá) | 12 | 12/429 = 2.8% | 429 box máy |
| added (vẽ thêm) | 8 | 8/425 = 1.9% | 425 đối tượng sau duyệt |

- Lớp nào máy tệ nhất: Lớp `three_wheeler` (box của YOLOE có xu hướng bị lỏng, cần co lại khoảng 10-15%) và lớp `person` ở các góc ảnh bị che khuất một phần.
- So với gold: precision của máy đạt 52.3% (23/44), recall của **máy** đạt 95.8% (23/24) (mẫu số: 24 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91.7% (KTC 95%: 65–99%). Loại bẫy lọt: 1 bẫy missing ở `img_018.jpg` (người đi bộ nhỏ kích thước 25x38 px nằm sâu trong hậu cảnh đông đúc xe máy, bị mắt bỏ qua khi quét nhanh).
- Accepted 89.0% đi kèm tỉ lệ bắt bẫy cao 91.7% chứng minh người duyệt đã thực hiện rà soát chủ động theo checklist 4 bước chứ không chấp nhận thụ động mọi đề xuất của AI.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay 1.75 phút/ảnh (7 phút / 4 ảnh) so với duyệt có pre-label 1.17 phút/ảnh (28 phút / 24 ảnh). Duyệt máy giúp tiết kiệm ~33% thời gian.
- Kết luận: **RẤT ĐÁNG DÙNG** cho các lớp phương tiện phổ biến (`car`, `motorbike`, `person`). Với các phương tiện đặc thù đường phố châu Á (`three_wheeler`), bắt buộc phải kết hợp YOLOE và người kiểm tra kỹ.
- **Ba quy tắc giảm thiên kiến:**
  1. *Ẩn nhãn máy trước:* Luôn tắt hiển thị box máy trong 10-15 giây đầu để mắt quét độc lập toàn cảnh ảnh.
  2. *Kiểm tra đa bẫy:* Tập trung soi kỹ các vùng rìa xe máy, kiểm tra các xe có 3 bánh để phân loại đúng `three_wheeler`.
  3. *Quy tắc vùng mờ:* Nhất quán sử dụng `ignore_region` cho các cụm phương tiện/người nhỏ hơn 20px ở xa, không cố vẽ từng vật nhỏ gây nhiễu.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_t.pt`) | 2 pos, 1 neg | Tách riêng thân xe tuk-tuk, không gồm người lái | Ranh giới kính chắn gió phía trước đôi khi bị lẹm nhẹ | `submission/masks/tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_t.pt`) | 2 pos, 1 neg | Gồm cả xe đạp/xích lô và người điều khiển | Vùng nan hoa bánh xe khó lấy trọn viền | `submission/masks/rickshaw_img004.jpg` |
| `img_005.jpg` | SAM 2.1 (`sam2.1_t.pt`) | 2 pos, 1 neg | Tách đúng xe máy bị che một phần | Cần thêm 1 click âm để loại bỏ xe đỗ liền kề | `submission/masks/motorbike_img005.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc gộp người điều khiển vào xe đối với xe đạp/xe máy giúp SAM định hình đối tượng rõ ràng hơn. Tuy nhiên với xe ba bánh có cabin mở, cần hướng dẫn cụ thể việc tách riêng tài xế và khung xe.
- Trải nghiệm SAM trong CVAT (Interactor): SAM tương tác trực tiếp qua click dương/âm giúp tạo mask nhanh gấp nhiều lần vẽ polygon thủ công.

## 6. Calibration (Part 6)

- Số quyết định: 3 quyết định chuẩn hoá (`calibration_decisions.csv`); quan trọng nhất là `CAL-01` (`img_004.jpg`): phân biệt dứt khoát `three_wheeler` với `car`/`truck` cho toàn bộ các phương tiện 3 bánh chở hàng/khách.
- Quy tắc mới đã bổ sung vào `guideline_v1.1_calibrated.md`: Phân định xe máy điện / xe đạp điện; định nghĩa phạm vi bao bọc hàng hoá của xe ba gác; siết quy tắc `ignore_region` cho vật < 20px.
- Nguyên nhân gốc: Do giới hạn nhãn của model COCO kết hợp guideline v1.0 chưa bao quát đầy đủ sự phong phú của giao thông đô thị châu Á.

## 7. QC và quyết định (Part 6)

- Mẫu QC: 5 ảnh ngẫu nhiên (seed 42), gồm 98 đối tượng.
- Lỗi theo **đối tượng**: 0/98 = 0.0% · theo **ảnh**: 0/5 = 0.0% (đạt ngưỡng chấp thuận ≤ 2%).
- Top lỗi: Không phát hiện lỗi nghiêm trọng trong mẫu kiểm tra sau khi đã qua bước review kỹ lưỡng.
- **Giới hạn mẫu nhỏ:** Với mẫu 5 ảnh không có lỗi, khoảng tin cậy 95% Wilson cho cận trên tỉ lệ lỗi thực tế là **43%**. Cần mở rộng quy mô kiểm tra ở các lô sản xuất lớn hơn.
- **Quyết định: RELEASE**, ngày 2026-11-17.
- Lý do: Tỉ lệ lỗi trong mẫu đạt 0.0% (dưới ngưỡng 2%), tỉ lệ bắt bẫy đạt 91.7% (vượt xa ngưỡng yêu cầu 80%), quy trình calibration đã cập nhật guideline rõ ràng.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Sử dụng trợ lý AI Antigravity để hỗ trợ định dạng script kiểm thử, phân tích dữ liệu thống kê bẫy và rà soát các tiêu chí nộp bài.
- Gold có bị nhiễm không: **Không**. 4 ảnh gold được gán tay 100% bằng công cụ vẽ hộp của CVAT trước khi chạy bất kỳ model pre-label nào.
- Môi trường: Windows 11, GPU NVIDIA GeForce RTX 3050 Laptop, CVAT Docker 2.74.1.
- Dùng cache: Có sử dụng kết quả dự đoán cache từ `data/cache/` kết hợp chạy thử nghiệm cục bộ trên GPU.
- Một điều làm khác lần sau: Sẽ chủ động áp dụng công cụ Interactor SAM sớm hơn để tăng tốc độ gán nhãn các đối tượng có hình dáng phức tạp.
