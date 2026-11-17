# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 532; đối tượng sau duyệt: 524; ngưỡng accepted: IoU >= 0.9
- Đã **loại 32 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 446 | 446/532 = 83.8% | 532 box máy |
| edited (sửa) | 26 | 26/532 = 4.9% | 532 box máy |
| deleted (xoá) | 50 | 50/532 = 9.4% | 532 box máy |
| added (vẽ thêm) | 52 | 52/524 = 9.9% | 524 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 42.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 238 | 14 | 11 | 13 |
| bicycle | 6 | 0 | 1 | 3 |
| motorbike | 141 | 5 | 5 | 14 |
| car | 45 | 2 | 6 | 13 |
| bus | 3 | 0 | 0 | 3 |
| truck | 1 | 0 | 4 | 4 |
| three_wheeler | 12 | 5 | 23 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 44 | 28 | 12 | 61.1% (KTC 95%: 50–72%) | 78.6% (KTC 95%: 66–87%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 2/3
  - missing: 2/3
  - loose: 1/3
  - lọt: img_008.jpg (extra), img_013.jpg (missing), img_035.jpg (loose), img_031.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
