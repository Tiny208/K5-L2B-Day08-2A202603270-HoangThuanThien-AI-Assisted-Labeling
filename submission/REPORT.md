# Báo cáo Day 08 — 2A202603270 - Hoàng Thuận Thiên

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / car / bus / truck / bicycle | YOLO26m | 0.30 (mặc định) | Theo `run_01`, YOLO26m nhận diện tốt ô tô, người đi bộ và xe máy ở cự ly gần/trung bình; các lớp ít mẫu (<5 trong gold) giữ ngưỡng mặc định 0.30 để tránh overfitting |
| motorbike | YOLO26m | 0.25 | Theo kết quả `threshold_sweep.md`, tại ngưỡng 0.25 lớp xe máy đạt Recall 90.9% (10/11 vật gold) và Precision 76.9% (10/13 box) |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | Theo `run_03`, YOLOE là mô hình open-vocabulary nhận diện đúng tuk-tuk/ba gác; quét ngưỡng đạt Recall 100% (7/7 vật gold) và Precision 77.8% (7/9 box) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** và lý do: Tôi chọn **SAM 2.1** (`sam2.1_b.pt`) vì đây là phiên bản cân bằng xuất sắc giữa chất lượng mask sắc nét và tốc độ xử lý (~0.45s/vật), hỗ trợ cơ chế prompt đa điểm dương/âm (positive/negative clicks) linh hoạt hơn SAM 1 và không đòi hỏi GPU chuyên dụng như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?
  Lỗi nhầm lẫn phân loại do tập từ vựng đóng (closed vocabulary): COCO không có nhãn xe ba bánh, nên YOLO26 luôn tự tin dán nhãn tuk-tuk thành `car` hoặc `truck` dù nâng hay hạ ngưỡng. Cách sửa: Phối hợp nhánh mô hình open-vocabulary YOLOE cho lớp `three_wheeler`, sau đó trong `merge_prelabels.py` thiết lập luật ưu tiên cho phép box `three_wheeler` của YOLOE ghi đè lên box `car`/`truck` của YOLO26 tại cùng vị trí không gian (IoU > 0.6).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Trong `img_013.jpg`, chiếc xe tuk-tuk bị YOLO26 gán nhãn `car` với độ tin cậy rất cao (0.82) nhưng sai hoàn toàn bản chất đối tượng.
  - *Điểm thấp mà đúng:* Trong `img_006.jpg`, xe ba gác chở cồng kềnh bị YOLOE gán nhãn `three_wheeler` với điểm tin cậy chỉ 0.18 nhưng lại phát hiện đúng vị trí và loại phương tiện.
- Giới hạn: Quét ngưỡng được thực hiện trên chính 4 ảnh gold (56 đối tượng gold), tức là chọn ngưỡng và đánh giá trên cùng một tập dữ liệu (in-sample), do đó các chỉ số Precision/Recall của máy trên gold mang tính lạc quan; các lớp có ít hơn 5 mẫu (`bus`, `truck`, `bicycle`) không đủ độ tin cậy thống kê để tinh chỉnh ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 446 | 446/532 = 83.8% | 532 box máy |
| edited | 26 | 26/532 = 4.9% | 532 box máy |
| deleted | 50 | 50/532 = 9.4% | 532 box máy |
| added | 52 | 52/524 = 9.9% | 524 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  Lớp `three_wheeler` là lớp máy hoạt động tệ nhất: có tới 23 box bị xóa (`deleted = 23/50` tổng số box xóa). Nguyên nhân do mô hình YOLOE ở ngưỡng thấp dễ bị ảo giác gán nhãn vào các góc biển hiệu hoặc bóng râm kéo dài trên mặt đường (như trong `img_005.jpg` và `img_021.jpg`), đồng thời box sinh ra thường bị lỏng, đòi hỏi người duyệt phải xóa bỏ hoặc nắn lại khung bao.
- So với gold: Precision của máy đạt **61.1%** (34/56, KTC 95%: 50–72%); Recall của máy đạt **78.6%** (44/56, KTC 95%: 66–87%) trên mẫu số **56 đối tượng gold**.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 bẫy = 66.7%** (KTC 95%: 39–86%).
  - Bắt trọn 100% bẫy sai lớp (`wrong_class`: 3/3).
  - Bắt 2/3 bẫy box thừa (`extra`: 2/3).
  - Bắt 2/3 bẫy bỏ sót (`missing`: 2/3).
  - Bắt 1/3 bẫy hộp bao lỏng (`loose`: 1/3).
  - Các bẫy bị lọt gồm: `img_008.jpg` (extra), `img_013.jpg` (missing), `img_035.jpg` (loose), `img_031.jpg` (loose).
  - *Vì sao lọt:* Tôi đã bỏ qua **Bước 1 của checklist (quét mắt toàn cảnh trước khi mở nhãn máy)** trên các frame đông xe. Khi thấy máy đã vẽ sẵn cụm box trông rất hợp lý, mắt tôi bị "neo" vào các box có sẵn và lướt qua nhanh mà không đối chiếu kỹ 4 cạnh tiếp xúc cực trị, dẫn đến lọt 2 bẫy box lỏng và 1 bẫy bỏ sót xe nhỏ ở hậu cảnh xa.
- Ý nghĩa: Tỉ lệ `accepted` cao (83.8%) trong khi tỉ lệ bắt bẫy chỉ đạt 66.7% (dưới ngưỡng yêu cầu 80%) là minh chứng kinh điển cho **Thiên Kiến Tự Động Hóa (Automation Bias)**: Người duyệt có xu hướng chủ quan tin tưởng vào kết quả đề xuất của AI, ngộ nhận rằng máy đã bắt đủ và vẽ chuẩn, từ đó chuyển từ tâm thế "kiểm định khắt khe" sang tâm thế "xác nhận thụ động".

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`):
  - Gán tay thủ công (gold): 14 phút / 4 ảnh = **3.50 phút/ảnh**.
  - Duyệt có AI pre-label: 37 phút / 24 ảnh = **1.54 phút/ảnh**.
  - Tốc độ duyệt nhanh hơn **2.27 lần** (tiết kiệm ~56% thời gian lao động).
  - *Thiên lệch:* Khi duyệt lô 24 ảnh, tôi đã quen thuộc với guideline và các loại phương tiện đường phố hơn so với lúc gán 4 ảnh gold đầu tiên, nên tốc độ có phần tăng thêm nhờ hiệu ứng đường cong học tập (learning curve).
- Kết luận: **CÓ ĐÁNG DÙNG**, nhưng phải áp dụng chiến lược phân tầng (routing matrix):
  - Với các đối tượng phổ biến rõ nét (`car`, `person`, `motorbike` ở tiền/trung cảnh), AI giúp tăng tốc độ gán nhãn vượt bậc với tỷ lệ chấp nhận > 85%.
  - Với các đối tượng ranh giới phức tạp (`three_wheeler`, cụm xe máy ùn tắc xa), không được tự động chấp nhận (cấm auto-accept), bắt buộc phải có quy trình con người rà soát False Negative kỹ lưỡng.
- **Ba quy tắc giảm thiên kiến tự động hóa:**
  1. *Quy tắc 5 giây mù (Blind Scan Rule):* Khi chuyển sang frame mới, bắt buộc ẩn toàn bộ annotation trong 5 giây đầu tiên để mắt quét tự nhiên từ trái sang phải, tìm các vật thể bị bỏ sót trước khi bật nhãn AI.
  2. *Quy tắc 4 điểm tiếp xúc (Touch-point Verification):* Khi duyệt box ô tô/xe máy, bắt buộc dùng mắt kiểm tra nhanh 4 mép cực trị (nóc xe, 2 gương/sườn xe, điểm chạm mặt đường); nếu khoảng hở > 10px phải bấm phím tắt co lại ngay.
  3. *Quy tắc chống mỏi mắt 10-1 (Fatigue Management):* Cứ sau mỗi 10 frame duyệt liên tục, annotator bắt buộc phải dừng lại nghỉ mắt 60 giây để phục hồi độ tập trung thị giác, tránh hội chứng "bấm duyệt vô thức".

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Đã tách được người lái ra ngoài, chỉ bao trọn thân xe tuk-tuk | Khi chưa có click âm (neg point), SAM có xu hướng gộp luôn áo người lái vào mui xe | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Chỉ bao trọn chiếc ô tô bị che, không ăn sang xe bên cạnh | Vùng bóng đổ dưới gầm xe hơi bị lẹm một góc nhỏ | `car_occluded_img004.jpg` |
| `img_006.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Bao trọn thùng xe ba gác và hàng hóa cồng kềnh | Chi tiết nan hoa bánh xe ba gác bị đứt đoạn do độ tương phản thấp | `three_wheeler_img006.jpg` |

- Đối chiếu guideline mục 4.2: Guideline quy định đối với phương tiện cơ giới, phần hàng hóa gắn liền trên xe được tính vào thể tích phương tiện, nhưng người điều khiển phải tách riêng. SAM 2.1 mặc định nếu chỉ dùng box prompt sẽ gom tất cả thành 1 khối mask duy nhất; việc sử dụng click âm (negative click) tại vị trí người lái là bắt buộc để đáp ứng đúng yêu cầu của guideline.
- Khác biệt so với Interactor trong CVAT: Chạy script offline qua PyTorch cho phép lưu ảnh kiểm tra trực tiếp và thử nghiệm linh hoạt các tọa độ click; trên máy Mac Apple Silicon cần lưu ý chạy trên CPU (`--device cpu`) do hàm nội suy `grid_sampler` của PyTorch MPS chưa hỗ trợ chế độ `border padding`.

## 6. Calibration (Part 6)

- Số quyết định: **3 quyết định** trong `submission/qc/calibration_decisions.csv`.
- Quyết định quan trọng nhất: **`CAL-02` (Rider Rule)** và **`CAL-01` (Ranh giới Three-wheeler)**.
  - `CAL-02` giải quyết dứt điểm câu hỏi mở lớn nhất: Quy định rõ ràng người ngồi trên xe hai bánh phải vẽ 2 box độc lập (1 box `person` và 1 box `motorbike`), chấm dứt việc annotator thì gộp chung, người thì tách riêng làm sai lệch ground truth.
  - `CAL-01` chốt chặn lỗi mô hình COCO: Mọi phương tiện ba bánh (tuk-tuk, ba gác) dù có cabin kín vẫn dán nhãn `three_wheeler`, cấm dán `car` hay `truck`.
- Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md` tại các mục 2.1, 3.2 và 4.1.
- Nguyên nhân gốc: Bất đồng xuất phát từ 3 nguồn: (1) Lỗ hổng định nghĩa Guideline v1.0 để mở ranh giới Rider Rule; (2) Giới hạn của mô hình pre-label COCO chưa được fine-tune trên dữ liệu giao thông Đông Nam Á; (3) Thiên kiến tự động hóa của người duyệt đối với các bẫy loose box.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** ngẫu nhiên (`img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`) với tổng số **102 đối tượng** kiểm tra.
- Lỗi theo **đối tượng**: **1/102 = 1.0%** (1 lỗi box lỏng ở `img_031.jpg`).
- Lỗi theo **ảnh**: **1/5 = 20.0%** (vượt xa ngưỡng an toàn Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 lỗi tại `img_031.jpg`, hộp bao ô tô bị lỏng viền tiếp xúc bánh xe với mặt đường).
- **Giới hạn mẫu nhỏ:** Với mẫu kiểm tra chỉ 5 ảnh, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi lên tới **62.0%**. Về mặt thống kê, mẫu kích thước nhỏ không thể chứng minh một cách chắc chắn rằng dữ liệu đạt chuẩn ≤ 2% lỗi toàn lô.
- Script kiểm tra được định dạng, schema, tọa độ hợp lệ nhưng **không kiểm tra được tính bao sát thực tế (tightness) và các đối tượng mà cả AI lẫn người duyệt cùng bỏ sót**.

**Quyết định: REWORK**, ngày 2026-11-17.
- *Lý do (đi theo cây quyết định Hold → Rework → Release):*
  1. Guideline v1.1 đã được hiệu chỉnh rõ ràng, không có tắc nghẽn định nghĩa (không rơi vào Hold).
  2. Tuy nhiên, tỉ lệ ảnh có lỗi trong mẫu QC đạt **20.0%** (vượt ngưỡng cho phép 2.0%), đồng thời tỉ lệ bắt bẫy trong lô chỉ đạt **66.7%** (< 80%). Điều này chứng tỏ lô dữ liệu sau duyệt vẫn còn tồn tại các lỗi loose box và bỏ sót do thiên kiến tự động hóa.
  3. Theo đúng nguyên tắc của cây quyết định, hệ thống phải chuyển sang trạng thái **Rework**.
- *Việc cần làm tiếp:*
  - Cập nhật Guideline v1.1 cho toàn bộ đội ngũ.
  - Mở lại 4 frame có bẫy bị lọt (`img_008`, `img_013`, `img_031`, `img_035`) trên CVAT để nắn lại các box lỏng và vẽ bổ sung xe bị sót.
  - Tiến hành rút mẫu kiểm tra vòng 2 trước khi ban hành chính thức.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Tôi sử dụng trợ lý Antigravity AI để hỗ trợ tự động hóa việc khởi chạy container CVAT Docker, kết nối API xuất nhập nhãn COCO 1.0, chạy các script tính toán toán học (`review_stats.py`, `threshold_sweep.py`, `sam2_points.py`), và định dạng báo cáo. Toàn bộ thao tác vẽ nhãn 4 ảnh gold thủ công và rà soát duyệt 24 ảnh trên giao diện CVAT đều do chính tôi trực tiếp thực hiện bằng mắt và tay.
- Gold có bị nhiễm không: **Hoàn toàn KHÔNG bị nhiễm**. Tôi đã thực hiện gán tay 4 ảnh gold độc lập từ đầu trên CVAT trước khi chạy bất kỳ mô hình pre-label nào.
- Máy / GPU / CVAT dùng: macOS Sonoma trên chip Apple Silicon; CVAT Docker chạy phiên bản v2.74.1; PyTorch chạy trên MPS và CPU.
- Dùng cache `data/cache/` thay vì tự chạy? Tôi có sử dụng dữ liệu dự đoán gốc của YOLO26m và YOLOE trong `data/cache/` để làm nguồn dữ liệu đầu vào cho bước quét ngưỡng và gộp nhãn, sau đó tự chạy script `merge_prelabels.py` và `inject_traps.py` trên máy.
- Lỗi / bug script đã gặp: Lỗi `RuntimeError: MPS: Unsupported Border padding mode` khi chạy SAM 2.1 trên Apple Silicon MPS; đã xử lý bằng cách cấu hình `--device cpu` theo đúng chỉ dẫn trong tài liệu.
- Một điều bạn làm khác lần sau: Lần sau khi duyệt nhãn máy, tôi sẽ nghiêm túc thực hiện quy tắc "5 giây mù" (tắt hiển thị nhãn máy ở những giây đầu) để rèn luyện thói quen chủ động tìm lỗi bỏ sót của AI thay vì bị cuốn vào các box có sẵn.
