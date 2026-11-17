# Thông tin học viên

- Họ và tên: Nguyen Thi Hong Hanh
- MSHV: 2A202603272
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: walnut1712
- Repo bài nộp (URL): https://github.com/walnut1712/K5-L23-Day08-NguyenThiHongHanh-2A202603272-AI-Assisted-Labeling
- Commit chốt bài (hash): Chưa có; các thay đổi hiện tại chưa được commit.
- Máy: Windows, Python 3.13.17; NVIDIA GeForce RTX 5070 (12 GB VRAM), driver 616.87, CUDA UMD 13.4. Log chạy model ghi device=cuda.
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 515 / 13 / 154 (mẫu số: 689 box máy); added: 21 (mẫu số: 549 đối tượng sau duyệt).
- So với gold: precision / recall của **máy** 54.5% / 88.9% (TP=48, FP=40, FN=6; 54 đối tượng gold trên 4 ảnh).
- Bắt bẫy: 8/12 = 66.7%.
- Thời gian: gán tay 7.5 phút/ảnh (30 phút/4 ảnh); duyệt có pre-label 3.1 phút/ảnh (75 phút/24 ảnh). Tính từ các khoảng giờ đã cung cấp.
- Lỗi QC: Chưa đo; chưa có `qc_sample.csv`, `qc_review.csv` hoặc `qc_report_auto.md`.
- **Quyết định:** Hold (tạm thời) — bắt bẫy 66.7%, dưới ngưỡng 80%, và chưa hoàn tất QC mẫu.
