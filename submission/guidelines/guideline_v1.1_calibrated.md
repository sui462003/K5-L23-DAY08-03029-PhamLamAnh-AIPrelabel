# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03029 - Phạm Lâm Anh

> **Bản bổ sung v1.1 so với v1.0:** Làm rõ các câu hỏi mở và chuẩn hoá các ca khó phát sinh từ lỗi nhận diện của máy (YOLO26, YOLOE), bẫy lọt và phản hồi từ thực tế gán nhãn.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện, xe máy điện: phân loại vào `bicycle` hay `motorbike`? | Xe máy điện có kết cấu khung xe máy (không bàn đạp) xếp vào `motorbike`. Xe đạp điện có bàn đạp trợ lực xếp vào `bicycle`. | `img_013.jpg` | CAL-02 |
| 2 | Xe ba gác, tuk-tuk, xích lô có chở hàng cồng kềnh thì box tính thế nào? | Box bao gồm cả xe và toàn bộ phần hàng hoá gắn liền trên xe, không tách rời hàng. | `img_004.jpg` | CAL-01 |

## 2. Ca khó mới từ lỗi máy và bẫy phát sinh

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_004.jpg` | Xe tuk-tuk và xe ba gác chở khách | YOLO26 nhận nhầm thành `car` hoặc `truck` | Sửa lại thành `three_wheeler` (sử dụng YOLOE để hỗ trợ phát hiện). | Mục 2.3: Tất cả xe 3 bánh cơ giới hoặc thô sơ đều thuộc `three_wheeler`. |
| `img_018.jpg` | Cụm người và xe máy ở hậu cảnh xa (< 20px) | AI bỏ sót hoặc gán box rời rạc không chính xác | Không vẽ box đơn lẻ cho từng đối tượng dưới 20px; vẽ 1 box `ignore_region` bao phủ cả cụm. | Mục 3.1: Quy định ngưỡng kích thước tối thiểu và vùng bỏ qua (`ignore_region`). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản guideline gốc của giảng viên | Khung quy tắc ban đầu |
| v1.1 | Chốt phân loại xe điện; chuẩn hoá nhãn `three_wheeler` cho xe ba bánh; siết chặt quy tắc `ignore_region` cho vật < 20px. | CAL-01 (`img_004.jpg`), CAL-02 (`img_013.jpg`), CAL-03 (`img_018.jpg`) |
