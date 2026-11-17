# Báo cáo Day 08 — 2A202603272 — Nguyen Thi Hong Hanh

> **Bản nháp dựa trên các tệp hiện có.** Những mục ghi là chưa được ghi nhận cần được kiểm tra và bổ sung theo trải nghiệm thật trước khi nộp; không suy đoán model, thời gian hay kết quả QC.

## 1. Model và recipe (Part 2–3)

`submission/model_log.csv` được lập lại từ `work/runs.csv` và các `run.json` trong `work/prelabel/`. Có 6 lượt chạy được ghi nhận: YOLO26m, YOLOE-s và YOLO26m kết hợp SAM 2.1 base. Nguồn có hai lượt cùng mã `r02`; trong submission chúng được phân biệt bằng hậu tố `-c025` và `-c050` theo ngưỡng. Nhận xét và quyết định trong CSV được bổ sung sau khi xem các đầu ra; chúng không phải ghi chú gốc được tạo ngay lúc chạy. Bảng dưới đây ghi **ngưỡng gợi ý trong** `qc/threshold_sweep.md`, không khẳng định các ngưỡng đó đã được áp dụng trong merge.

| Lớp | Model + cỡ đã chạy | Ngưỡng gợi ý | Căn cứ |
|---|---|---:|---|
| person | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | 0.15 | Recall mục tiêu 90% đạt trên 20 vật gold; precision 41%. |
| motorbike | YOLO26m (`r04` chạy mask trên một số lớp; `r05` chạy all) | 0.15 | Recall 93% trên 14 vật gold; precision 72%. |
| three_wheeler | YOLOE-s (`r03`, `r06`) | 0.15 | Recall 100% trên 7 vật gold; precision 78%. Taxonomy có prompt `tuk-tuk`, `cyclo`, `cargo tricycle`; lệnh lưu trong work log không ghi prompt riêng. |
| bicycle | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | Mặc định (0.3) | Chỉ có 4 vật gold, quá ít để chọn ngưỡng riêng. |
| car | YOLO26m (`r02-c025`, `r02-c050`, `r04`, `r05`) | Mặc định (0.3) | Không ngưỡng nào ≤ 0.5 đạt recall 90%; tối đa 88% trên 8 vật gold. |
| bus / truck | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | Mặc định (0.3) | Lần lượt có 0 và 1 vật gold, quá ít để chọn ngưỡng riêng. |

- Chuỗi ngưỡng được `threshold_sweep.md` đề xuất: `--thr "person=0.15,motorbike=0.15,three_wheeler=0.15"`. Lệnh `merge_prelabels.py` thực tế đã chạy: **chưa được ghi lại**.
- Phiên bản SAM trong lượt `r04` là **SAM 2.1 base (`sam2.1_b.pt`)**, kết hợp YOLO26m để tạo mask trên 2 ảnh. Lý do chọn phiên bản này không được ghi trong log.
- Đổi ngưỡng không sửa được lỗi sai lớp. `review_stats.md` ghi nhận 10 trường hợp đổi lớp; cần kiểm tra lại ánh xạ/prompt và duyệt nhãn thay vì chỉ hạ ngưỡng. Tệp thống kê không chỉ rõ ảnh ví dụ cho các trường hợp thật này.
- Ví dụ box điểm cao nhưng sai / điểm thấp nhưng đúng: **chưa được ghi nhận**.
- Ngưỡng được quét trên chính 4 ảnh gold; vì vậy precision/recall trên gold là số đo in-sample, có thể lạc quan. Mỗi lớp cũng có rất ít ví dụ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 515 | 74.7% | 689 box máy |
| edited | 13 | 1.9% | 689 box máy |
| deleted | 154 | 22.4% | 689 box máy |
| added | 21 | 3.8% | 549 đối tượng sau duyệt |

Tổng accepted + edited + deleted là 682, thấp hơn mẫu số 689 đúng 7 box. Theo cách tính của `scripts/review_stats.py`, các box máy không ghép được nhưng nằm trong `ignore_region` của nhãn cuối không bị tính là deleted; chúng vẫn nằm trong mẫu số box máy. Vì vậy các tỉ lệ trạng thái không cộng thành 100%.

Theo lớp, `three_wheeler` có 23/43 box máy bị xóa (53.5%). `truck` có tỉ lệ 4/6 (66.7%), nhưng mẫu chỉ gồm 6 box. Đây là tín hiệu cần kiểm tra, không đủ để kết luận nguyên nhân hoặc nêu ảnh lỗi cụ thể. Ngoài ra, 10 trường hợp được ghép không gian nhưng đổi lớp là lỗi phân loại, không phải vật bị bỏ sót thật.

So với gold, trên 4 ảnh với 54 đối tượng: máy có TP=48, FP=40, FN=6; precision = 48/88 = **54.5%**, recall = 48/54 = **88.9%**. Khoảng tin cậy 95% trong `review_stats.md` lần lượt là 44–65% và 78–95%; mẫu nhỏ và ngưỡng được chọn trên cùng gold nên không nên diễn giải các điểm ước lượng này như kết quả tổng quát.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được **8/12 = 66.7%**, thấp hơn ngưỡng yêu cầu 80%. Lọt 4 bẫy: `img_021.jpg` (missing), `img_033.jpg` (loose), `img_004.jpg` (missing), `img_006.jpg` (wrong_class). Theo loại: missing 1/3, loose 2/3, extra 3/3, wrong_class 2/3.
- Các tệp không ghi lại bước checklist nào đã bị bỏ qua, nên không thể kết luận nguyên nhân cá nhân. Kết quả cho thấy chỉ nhìn tỉ lệ accepted (74.7%) là chưa đủ để tin nhãn; cần rà lại ảnh toàn cảnh và xác minh từng box, đặc biệt lỗi bỏ sót và sai lớp. Với 12 bẫy, khoảng tin cậy 95% rộng (39–86%).

## 4. Máy có đáng dùng không?

- Theo khoảng thời gian tôi cung cấp, gán tay 4 ảnh mất 30 phút (**7.5 phút/ảnh**); review 24 ảnh mất 75 phút (**3.1 phút/ảnh**). Đây là hai nhóm ảnh và tác vụ khác nhau, không phải phép so sánh có kiểm soát; không thể kết luận chính xác thời gian tiết kiệm được. Vì gán gold diễn ra trước review, sự quen thuộc với bộ ảnh có thể ảnh hưởng thời gian review.
- Với chất lượng hiện có, chỉ nên xem pre-label là bản nháp để người duyệt sửa, không tự động chấp nhận hoặc phát hành. Căn cứ: 515/689 box được giữ nguyên (74.7%), 154/689 bị xóa (22.4%) và chỉ bắt được 8/12 bẫy (66.7%).
- Ba quy tắc tôi sẽ áp dụng ở lần review tiếp theo:
  1. Quét toàn ảnh trước khi xem nhãn máy; sau đó đối chiếu lại từng vùng để tìm vật bị bỏ sót.
  2. Kiểm tra riêng lớp, vị trí và độ ôm sát của box; ưu tiên rà `three_wheeler` và các trường hợp đổi lớp.
  3. Chỉ cân nhắc phát hành sau khi hoàn tất mẫu QC; nếu kiểm tra lại bẫy thì đặt mục tiêu ít nhất 10/12 (83.3%), cao hơn ngưỡng 80%.

## 5. Mask SAM cho 3 vật (Part 5)

Ba ảnh dưới đây thuộc cùng cảnh `img023`. Đếm dấu trên ảnh overlay cho thấy mỗi ảnh có 2 điểm dương (xanh) và 1 điểm âm (đỏ), tổng cộng 3 click. Phiên bản SAM không được lưu kèm ảnh.

| Ảnh | SAM (phiên bản) | Số click | Người lái / hàng hóa | Quan sát từ overlay | File trong `masks/` |
|---|---|---:|---|---|---|
| `img023`, tuk-tuk tiền cảnh | Chưa ghi nhận | 3 (2 dương, 1 âm) | Người/hàng bên trong chưa xác nhận chắc chắn | Vùng mask phủ phần lớn thân và mui xe; cần rà biên các chi tiết mảnh. | `img023_1.jpg` |
| `img023`, tuk-tuk ở giữa | Chưa ghi nhận | 3 (2 dương, 1 âm) | Người/hàng bên trong chưa xác nhận chắc chắn; người đứng cạnh không nằm trong vùng mask | Overlay phủ thân và mui; kiểm tra lại vùng cửa mở/khung xe. | `img023_2.jpg` |
| `img023`, tuk-tuk phía sau | Chưa ghi nhận | 3 (2 dương, 1 âm) | Người/hàng bên trong chưa xác nhận chắc chắn; người ở ngoài xe không nằm trong vùng mask | Overlay phủ thân xe; cần đối chiếu ảnh gốc để rà phần bị che khuất. | `img023_3.jpg` |

Theo guideline mục 4.2, cần kiểm tra biên phần nhìn thấy, lỗ trong khung xe, bóng đổ và vùng tiếp xúc giữa các vật. Ảnh overlay không đủ để xác nhận sai lệch biên ≤3 px hoặc quyết định chính xác mọi phần người/hàng hóa bị che; cần rà lại từng mask trên ảnh gốc. Việc thử SAM Interactor trong CVAT và so sánh với cách offline chưa được ghi nhận.

## 6. Calibration (Part 6)

`qc/calibration_decisions.csv` hiện chỉ có dòng tiêu đề, tức **0 quyết định được ghi lại**. `guidelines/guideline_v1.1_calibrated.md` vẫn là khuôn chưa điền, chưa có quy tắc mới hoặc `case_id` để dẫn. Nguyên nhân gốc của từng ca khó vì vậy chưa thể kết luận. Cần ghi ít nhất 3 quyết định có ảnh ví dụ, rồi cập nhật phần bổ sung tương ứng trong guideline.

## 7. QC và quyết định (Part 6)

Trong `submission/qc/` hiện chưa có `qc_sample.csv`, `qc_review.csv` hoặc `qc_report_auto.md`; do đó chưa có cỡ mẫu QC, lỗi theo ảnh/đối tượng hay cận trên 95% để báo cáo. `submission/release/` hiện cũng chưa có đầu ra release.

**Quyết định hiện tại: Hold — 2026-11-17 (tạm thời, chưa phải kết quả QC hoàn tất).** Lý do: bắt bẫy 8/12, dưới ngưỡng 80%, và chưa có mẫu QC để đánh giá ngưỡng Release ≤2%. Việc tiếp theo là tạo và hoàn tất mẫu QC, ghi loại lỗi, chạy báo cáo QC, rồi mới quyết định Release / Hold / Rework chính thức. Không thể nêu top lỗi QC hoặc tỉ lệ lỗi theo ảnh/đối tượng khi chưa có các tệp này.

## 8. Ghi chú cá nhân

- **Dùng AI:** có. Tôi dùng trợ lý AI trong VS Code để hỗ trợ xử lý lỗi môi trường Python và tổng hợp báo cáo; các số liệu trong bản nháp được đối chiếu với các tệp log và QC. Tôi cần tự xác nhận phần mô tả trải nghiệm trước khi nộp.
- Gold có bị nhiễm thông tin từ pre-label hay không: **chưa được ghi nhận**, nên tôi không thể khẳng định gold hoàn toàn độc lập.
- Máy: Windows; Python 3.13.17. Lần kiểm tra môi trường trước đó, Windows Application Control chặn extension nhị phân `_umath_linalg` của NumPy, khiến các thư viện phụ thuộc không import được. GPU, cách chạy model sau đó và phiên bản CVAT: **chưa được ghi nhận trong các tệp submission**.
- Việc dùng cache `data/cache/`: **chưa được ghi nhận**.
- Vướng mắc đã xác nhận: lỗi chính sách Application Control khi import NumPy. Model log submission được khôi phục từ `work/runs.csv`/`run.json`. Thời lượng trong `time_log.csv` được tính từ các khoảng giờ tôi cung cấp sau đó, không phải log bấm giờ tự động.
- Lần sau tôi sẽ ghi `model_log.csv` và `time_log.csv` ngay sau mỗi lần chạy, lưu phiên bản SAM/số click, hoàn tất calibration và QC trước khi đưa ra quyết định phát hành.
