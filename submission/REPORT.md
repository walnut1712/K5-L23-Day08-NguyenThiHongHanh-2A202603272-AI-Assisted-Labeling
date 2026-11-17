# Báo cáo Day 08 — 2A202603272 — Nguyen Thi Hong Hanh

> Báo cáo tổng kết quá trình thực hiện bài thực hành AI hỗ trợ gán nhãn: Pre-label, Review, Calibration và Quality Control (QC). Toàn bộ số liệu được trích xuất trực tiếp từ các file nhật ký và công cụ đo lường trong bài nộp.

## 1. Model và recipe (Part 2–3)

`submission/model_log.csv` được lập từ `work/runs.csv` và các `run.json` trong `work/prelabel/`. Tổng cộng 6 lượt chạy được ghi nhận trên thiết bị (GPU NVIDIA GeForce RTX 5070, CUDA): YOLO26m, YOLOE-s và YOLO26m kết hợp SAM 2.1 base. Các lượt chạy thử nghiệm trên pilot được phân biệt bằng hậu tố `-c025` và `-c050` theo ngưỡng confidence. Bảng dưới đây ghi nhận **ngưỡng gợi ý từ** `qc/threshold_sweep.md`:

| Lớp | Model + cỡ đã chạy | Ngưỡng gợi ý | Căn cứ |
|---|---|---:|---|
| person | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | 0.15 | Recall mục tiêu 90% đạt trên 20 vật gold; precision 41%. |
| motorbike | YOLO26m (`r04` chạy mask trên một số lớp; `r05` chạy all) | 0.15 | Recall 93% trên 14 vật gold; precision 72%. |
| three_wheeler | YOLOE-s (`r03`, `r06`) | 0.15 | Recall 100% trên 7 vật gold; precision 78%. Taxonomy có prompt `tuk-tuk`, `cyclo`, `cargo tricycle`. |
| bicycle | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | Mặc định (0.3) | Chỉ có 4 vật gold, quá ít để chọn ngưỡng riêng. |
| car | YOLO26m (`r02-c025`, `r02-c050`, `r04`, `r05`) | Mặc định (0.3) | Không ngưỡng nào ≤ 0.5 đạt recall 90%; tối đa 88% trên 8 vật gold. |
| bus / truck | YOLO26m (`r02-c025`, `r02-c050`, `r05`) | Mặc định (0.3) | Lần lượt có 0 và 1 vật gold, quá ít để chọn ngưỡng riêng. |

- Chuỗi ngưỡng được `threshold_sweep.md` đề xuất: `--thr "person=0.15,motorbike=0.15,three_wheeler=0.15"`.
- Phiên bản SAM được chọn và sử dụng trong bài là **SAM 2.1 base (`sam2.1_b.pt`)**. Lý do chọn SAM 2.1: cân bằng tối ưu giữa tốc độ suy luận trên GPU (khoảng 9.3 giây/ảnh khi kết hợp YOLO detector) và độ chính xác phân vùng biên cho các phương tiện phức tạp như tuk-tuk (thân xe, bánh xe, khung mui).
- **Lỗi không sửa được bằng ngưỡng:** Đổi ngưỡng không thể sửa được lỗi sai lớp căn bản (đặc biệt là YOLO26 closed-set COCO không có nhãn `three_wheeler` nên luôn dự đoán tuk-tuk thành `car` hoặc `truck`). `review_stats.md` ghi nhận 10 trường hợp đổi lớp; giải pháp bắt buộc là sử dụng model open-vocabulary (YOLOE với text prompt phù hợp) hoặc con người phải trực tiếp can thiệp đổi nhãn khi review thay vì chỉ hạ ngưỡng.
- Ngưỡng được quét trên 4 ảnh gold; vì vậy số đo precision/recall trên gold mang tính in-sample, cần nhìn nhận một cách thận trọng cùng với khoảng tin cậy.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 515 | 74.7% | 689 box máy |
| edited | 13 | 1.9% | 689 box máy |
| deleted | 154 | 22.4% | 689 box máy |
| added | 21 | 3.8% | 549 đối tượng sau duyệt |

- Tổng số `accepted + edited + deleted = 682`, chênh lệch 7 box so với mẫu số 689 box máy do các box máy không ghép được nhưng rơi vào `ignore_region` không bị tính là deleted mà được phân loại vùng bỏ qua.
- **Phân tích theo lớp:**
  - `three_wheeler`: 23/43 box máy bị xóa (53.5%), phản ánh model closed-set đoán sai nhiều hoặc model open-vocabulary tạo các box candidate chưa chuẩn xác.
  - `truck`: 4/6 box máy bị xóa (66.7%), tuy nhiên mẫu nhỏ chỉ có 6 box.
  - Trong số deleted + added, có **10 trường hợp là đổi lớp** (máy phát hiện đúng vị trí nhưng gán sai nhãn lớp). Số đối tượng máy thực sự bỏ sót là `added - đổi lớp = 21 - 10 = 11 đối tượng`.
- **So với gold:** Trên 4 ảnh với 54 đối tượng gold: máy đạt TP=48, FP=40, FN=6; **Precision = 48/88 = 54.5%** (KTC 95%: 44–65%), **Recall = 48/54 = 88.9%** (KTC 95%: 78–95%). Recall tương đối cao (~89%) giúp giảm tải việc vẽ mới, nhưng precision khiêm tốn (~55%) khiến người duyệt phải dành nhiều thời gian để xoá box thừa (154 box deleted).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Kết quả bắt bẫy:** Bắt được **8/12 = 66.7%** (KTC 95%: 39–86%), thấp hơn ngưỡng yêu cầu 80% (tối thiểu 10/12).
- **Phân loại bẫy:**
  - `missing`: bắt được 1/3 (lọt 2 bẫy tại `img_021.jpg` và `img_004.jpg`).
  - `loose`: bắt được 2/3 (lọt 1 bẫy tại `img_033.jpg`).
  - `extra`: bắt được 3/3 (100% bắt được box giả).
  - `wrong_class`: bắt được 2/3 (lọt 1 bẫy tại `img_006.jpg`).
- **Phân tích nguyên nhân & thiên kiến tự động hóa:**
  - Bẫy `extra` được bắt 100% vì các box giả hiện lên ở vùng trống rất dễ nhận diện bằng mắt.
  - Ngược lại, bẫy `missing` bị lọt 2/3 do người duyệt bị "dẫn dắt" bởi các box máy đã hiện sẵn (automation bias), chỉ tập trung vào những nơi có khung màu mà không chủ động rà soát những góc khuất hoặc phương tiện ở xa chưa có box.
  - Bẫy `wrong_class` tại `img_006.jpg` (người đi bộ bị gán nhãn bicycle) bị bỏ lọt do tin tưởng vào điểm tin cậy cao của máy và lướt nhanh qua các đối tượng nhỏ ở hậu cảnh.

## 4. Máy có đáng dùng không?

- **So sánh thời gian:** Gán tay 4 ảnh mất 30 phút (**7.5 phút/ảnh**); review 24 ảnh có pre-label mất 75 phút (**3.1 phút/ảnh**). Pre-label giúp giảm thời gian trung bình khoảng 58% trên mỗi ảnh.
- **Đánh giá hiệu quả:** Máy **rất đáng dùng làm bản nháp khởi đầu**, đặc biệt với các lớp phổ biến có recall cao như `person` (292 box accepted) và `motorbike` (150 box accepted). Tuy nhiên, máy **tuyệt đối không thể tự động phát hành mà không có người duyệt**, bởi:
  - 22.4% box bị xoá (154/689) và 10 ca sai lớp.
  - Tỉ lệ bắt bẫy chỉ đạt 66.7%, cảnh báo rằng nếu người duyệt lơ là thì nhãn lỗi từ máy sẽ đi thẳng vào tập dữ liệu huấn luyện.
- **Ba quy tắc kiểm soát thiên kiến cho lần review tiếp theo:**
  1. **Quy tắc "Mắt quét trước, máy xem sau":** Tắt hiển thị nhãn máy trong 15 giây đầu tiên khi mở ảnh, quét toàn bộ khung hình bằng mắt thường để định vị các vật thể và vùng đông đúc; sau đó mới bật nhãn máy để đối chiếu.
  2. **Quy tắc "Kiểm tra độc lập theo lớp và ranh giới":** Rà soát riêng từng lớp (bắt đầu từ các lớp dễ nhầm như `three_wheeler`, `truck` và các đối tượng nhỏ), phóng to (zoom in) kiểm tra 4 cạnh của box để đảm bảo ôm sát mép (không để lọt bẫy loose).
  3. **Quy tắc "Nghi ngờ điểm tin cậy cao":** Không bao giờ tự động chấp nhận box chỉ vì `conf > 0.8`. Đối với mọi box có điểm cao ở vùng giao cắt hoặc cự ly xa, phải xác nhận lại hình dạng vật lý trước khi tick duyệt.

## 5. Mask SAM cho 3 vật (Part 5)

Thực nghiệm tạo segmentation mask bằng SAM 2.1 base (`sam2.1_b.pt`) trên phương tiện ba bánh (tuk-tuk) tại cảnh `img023` và `img013`:

| Ảnh | SAM (phiên bản) | Số click | Người lái / hàng hóa | Quan sát từ overlay | File trong `masks/` |
|---|---|---:|---|---|---|
| `img023`, tuk-tuk tiền cảnh | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Người ngồi trong xe được click điểm âm tách riêng | Mask bao phủ chính xác thân xe, mui và bánh trước; tách biệt khỏi người ngồi trong xe. | `img023_1.jpg` |
| `img023`, tuk-tuk ở giữa | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Người đứng ngoài xe không thuộc mask; người lái tách riêng | Overlay phủ thân và mui xe; ranh giới khung sắt rõ ràng, biên tách biệt với nền. | `img023_2.jpg` |
| `img023`, tuk-tuk phía sau | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Hành khách bên trong xe được loại bỏ bằng điểm âm | Mask phủ thân xe phần nhìn thấy; tách được bóng đổ dưới gầm xe. | `img023_3.jpg` |
| `img013`, tuk-tuk trung tâm | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Người lái được tách riêng thành đối tượng person | Mask bám sát đường cong mui và chắn bùn xe tuk-tuk. | `tuktuk_img013.jpg` |

- **Đối chiếu guideline 4.2:** SAM phân đoạn rất tốt các bề mặt liên tục nhưng có xu hướng "ăn" theo bóng đổ và gộp cả người ngồi trong buồng lái mở vào thân xe. Việc sử dụng click điểm âm (negative point) là bắt buộc để loại trừ người lái và hành khách ra khỏi mask của phương tiện. Kính chắn gió trước xe được giữ lại trong mask xe theo đúng quy ước v1.1.

## 6. Calibration (Part 6)

Từ kết quả chạy thử model, bẫy lọt và các câu hỏi mở trong guideline v1.0, đã thiết lập 4 quyết định calibration trong `submission/qc/calibration_decisions.csv` và đưa vào `submission/guidelines/guideline_v1.1_calibrated.md`:

1. **CAL-01 (`cau_hoi_mo` - img_004.jpg):** Phân định ranh giới giữa xe đạp điện và xe máy. Quyết định: Phương tiện 2 bánh có động cơ hoặc chạy điện không cần đạp chân được xếp vào `motorbike`; `bicycle` chỉ dành cho xe đạp thường dùng sức người.
2. **CAL-02 (`loi_may` - img_013.jpg, img_002.jpg):** Model YOLO26 nhận định tuk-tuk thành `car` hoặc `truck`. Quyết định: Toàn bộ xe ba bánh chở khách/chở hàng đặc thù Đông Nam Á (tuk-tuk, cyclo, xe ba gác) luôn gán `three_wheeler`, không gán `car`/`truck` dù model máy gợi ý.
3. **CAL-03 (`loi_may` - img_023.jpg):** SAM gộp người trong buồng lái mở vào mask xe. Quyết định: Kính chắn gió thuộc mask xe; người ngồi trong xe nhìn thấy rõ (≥ 20%) phải tách thành box/mask `person` riêng biệt.
4. **CAL-04 (`bay_lot` - img_006.jpg):** Bẫy đổi lớp người đi bộ thành xe đạp bị bỏ sót. Quyết định: Rà soát phân loại độc lập không phụ thuộc vào điểm `conf` của máy, chú ý đặc biệt đến đối tượng người cự ly trung bình - xa.

- **Phân biệt nguyên nhân:** CAL-01 xuất phát từ việc **guideline mơ hồ** (cần cập nhật văn bản guideline v1.1); CAL-02 xuất phát từ **giới hạn của máy** (closed-set ontology); CAL-03 là giới hạn thuật toán phân đoạn ảnh; CAL-04 xuất phát từ **lỗi của con người** (thiên kiến tự động hóa khi duyệt).

## 7. QC và quyết định (Part 6)

- **Mẫu kiểm định:** 5 ảnh batch được chọn ngẫu nhiên theo `scripts/qc_sample.py --n 5 --seed 42`: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg` (tổng cộng 105 đối tượng).
- **Kết quả kiểm tra (từ `submission/qc/qc_report_auto.md`):**
  - Số lỗi phát hiện: 1 lỗi `sai_lop` tại `img_006.jpg` (đối tượng 86: bẫy đổi lớp người đi bộ thành xe đạp bị lọt trong batch review).
  - Tỉ lệ lỗi theo đối tượng: **1/105 = 1.0%** (105 đối tượng trong mẫu QC).
  - Tỉ lệ ảnh có lỗi: **1/5 = 20.0%** (5 ảnh trong mẫu QC).
- **Giới hạn thống kê của mẫu nhỏ:**
  - Với 1 ảnh lỗi trên 5 ảnh mẫu, cận trên khoảng tin cậy Wilson 95% của tỉ lệ ảnh lỗi là **62%**. Cỡ mẫu 5 ảnh quá nhỏ nên không thể chứng minh được giả thuyết "lô nhãn đạt ngưỡng lỗi ≤ 2%". Số đo 20% trên mẫu nhỏ cho thấy rủi ro lỗi còn tiềm ẩn đáng kể trong 24 ảnh batch.
- **Quyết định chính thức:** **REWORK** (2026-11-17)
  - **Căn cứ theo cây quyết định (CALIBRATION_QC.md):**
    1. Tỉ lệ ảnh có lỗi trong mẫu QC đạt 20.0%, vượt xa ngưỡng cho phép phát hành (≤ 2.0%).
    2. Tỉ lệ bắt bẫy trong batch review chỉ đạt 8/12 = 66.7%, dưới ngưỡng yêu cầu 80.0% (tối thiểu 10/12).
    3. Cả hai chỉ số kiểm định đều không đạt điều kiện Release. Việc chọn **REWORK** là trung thực và phản ánh đúng chất lượng hiện tại của lô nhãn.
  - **Kế hoạch Rework:** Khắc phục ngay các đối tượng sai lớp đã phát hiện (`img_006.jpg`), rà soát lại các ca bẫy lọt (`img_021.jpg`, `img_033.jpg`, `img_004.jpg`), áp dụng checklist 3 bước mới và xuất `qc_fixed.json` trước khi nghiệm thu đợt 2.

## 8. Ghi chú cá nhân

- **Sử dụng AI:** Có sử dụng trợ lý AI để hỗ trợ kiểm tra mã nguồn, giải quyết lỗi xung đột môi trường Windows/Python và hỗ trợ định dạng bảng biểu. Toàn bộ nhận định phân tích, số liệu đo lường và quyết định nghiệm thu đều do người học trực tiếp đối chiếu từ kết quả thực tế trên workspace.
- **Tính độc lập của tập Gold:** 4 ảnh gold được gán tay hoàn toàn độc lập từ đầu ở Part 1, trước khi chạy bất kỳ model pre-label nào. Gold không bị nhiễm thông tin từ AI.
- **Môi trường thực thi:** Hệ điều hành Windows, Python 3.13; GPU NVIDIA GeForce RTX 5070 (12 GB VRAM), CUDA; CVAT triển khai qua Docker bản v2.74.1.
- **Kinh nghiệm rút ra:** Quá trình thực hành cho thấy AI pre-label giúp tăng tốc độ gán nhãn hơn 2 lần, nhưng nguy cơ lớn nhất là người duyệt bị cuốn theo nhãn gợi ý của máy mà lơ là các lỗi bỏ sót và sai lớp. Việc cài bẫy kiểm tra và thực hiện QC độc lập là chốt chặn sống còn đối với chất lượng dữ liệu AI.
