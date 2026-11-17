# Thông tin học viên

- Họ và tên: Nguyen Thi Hong Hanh
- MSHV: 2A202603272
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: walnut1712
- Repo bài nộp (URL): https://github.com/walnut1712/K5-L23-Day08-NguyenThiHongHanh-2A202603272-AI-Assisted-Labeling
- Commit chốt bài (hash): c2bd41f
- Máy: Windows, Python 3.13.17; NVIDIA GeForce RTX 5070 (12 GB VRAM), driver 616.87, CUDA UMD 13.4. Log chạy model ghi device=cuda.
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 515 / 13 / 154 (mẫu số: 689 box máy); added: 21 (mẫu số: 549 đối tượng sau duyệt).
- So với gold: precision / recall của **máy** 54.5% / 88.9% (TP=48, FP=40, FN=6; 54 đối tượng gold trên 4 ảnh).
- Bắt bẫy: 8/12 = 66.7% (bắt được 8 trong 12 bẫy).
- Thời gian: gán tay 7.5 phút/ảnh (30 phút/4 ảnh); duyệt có pre-label 3.1 phút/ảnh (75 phút/24 ảnh).
- Lỗi QC: Lỗi theo đối tượng 1/105 = 1.0% (mẫu số: 105 đối tượng); tỉ lệ ảnh có lỗi 1/5 = 20.0% (mẫu số: 5 ảnh trong mẫu QC). Cận trên 95% là 62%.
- **Quyết định:** **Rework** — ảnh có lỗi trong mẫu QC đạt 20.0% > ngưỡng cho phép 2.0%, bắt bẫy đạt 66.7% < ngưỡng tối thiểu 80.0%.
