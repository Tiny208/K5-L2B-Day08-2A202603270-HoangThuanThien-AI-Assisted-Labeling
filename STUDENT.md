# Thông tin học viên

- Họ và tên: Hoàng Thuận Thiên
- MSHV: 2A202603270
- Lớp (L2 / L3 / khác): L2-3
- Tài khoản GitHub: Tiny208
- Repo bài nộp (URL): https://github.com/Tiny208/K5-L2B-HoangThuanThien-2A202603270-Day08-AI-Assisted-Labeling
- Commit chốt bài (hash): a73ff5a
- Máy: macOS Sonoma / Apple Silicon (MPS)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 446/532 = 83.8% / 26/532 = 4.9% / 50/532 = 9.4% (mẫu số: 532 box máy); added: 52/524 = 9.9% (mẫu số: 524 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 61.1% (34/56) / 78.6% (44/56) (mẫu số: 56 đối tượng gold)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 3.50 phút/ảnh (14 phút / 4 ảnh); duyệt có pre-label 1.54 phút/ảnh (37 phút / 24 ảnh)
- Lỗi QC: 20.0% theo ảnh (1/5), 1.0% theo đối tượng (1/102)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh có lỗi trong mẫu QC đạt 20.0% (>2%) và tỉ lệ bắt bẫy đạt 66.7% (<80%), cần rà soát lại các hộp bao bị lỏng viền.
