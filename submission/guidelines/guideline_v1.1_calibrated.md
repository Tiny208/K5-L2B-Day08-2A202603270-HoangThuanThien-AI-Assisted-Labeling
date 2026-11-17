# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603270 - Hoàng Thuận Thiên

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người ngồi trên xe máy / xe đạp: gán chung hay tách rời? | **Tách 2 box độc lập chồng lên nhau**: 1 box `person` bao trọn cơ thể người điều khiển/người ngồi sau, và 1 box `motorbike` (hoặc `bicycle`) bao trọn toàn bộ khung và bánh xe. Không gom người vào xe. | `img_001.jpg`, `img_020.jpg` | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk chở khách có cabin/mui che kín và xe ba gác chở hàng | YOLO26 gọi là `car` hoặc `truck`; YOLOE bắt đúng lớp nhưng box bị lỏng trùm cả vỉa hè | Bắt buộc dán `three_wheeler` cho mọi phương tiện cơ giới 3 bánh; nắn box sát 4 mép bánh xe và mui | **Mục 2.1**: Phương tiện ba bánh dù có cabin, mui bạt hay gắn thêm thùng chở hàng phía sau vẫn thuộc lớp `three_wheeler`, cấm dán `car`/`truck` (CAL-01). |
| `img_035.jpg`, `img_031.jpg` | Hộp bao bị lỏng (loose box) quá 20% diện tích bao quanh phương tiện | Pre-label vẽ box rộng bao trùm cả vệt bóng đổ và dải phân cách; annotator dễ lọt mắt khi duyệt nhanh | Bổ sung quy tắc Tightness: Box phải chạm sát ít nhất 3 điểm cực trị của vật thể; khe hở biên > 10px bị tính lỗi | **Mục 3.2**: Tiêu chuẩn tiếp xúc biên: Hộp bao phải chạm sát mép trên nóc, mép dưới bánh tiếp xúc đường và 2 mép bên; khoảng trống thừa > 10px tính là lỗi `sai_vi_tri_kich_thuoc` (CAL-03). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt Rider Rule tách riêng `person` và `motorbike`/`bicycle`; Định nghĩa ranh giới lớp `three_wheeler` cho tuk-tuk/ba gác; Bổ sung chuẩn tiếp xúc biên Tightness Contract chống lỗi box lỏng. | Xử lý bất đồng giữa các annotator (CAL-02); khắc phục lỗi phân loại cố hữu của mô hình COCO (CAL-01); ngăn chặn hiện tượng lọt bẫy loose box do thiên kiến tự động hóa (CAL-03). |
