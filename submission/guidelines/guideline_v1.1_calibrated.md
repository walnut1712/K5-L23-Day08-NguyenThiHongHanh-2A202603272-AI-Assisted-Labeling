# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603272 - Nguyen Thi Hong Hanh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: bicycle hay motorbike? | Phương tiện hai bánh có động cơ hoặc chạy bằng điện không cần đạp chân xếp vào `motorbike`; xe đạp thường đạp bằng sức người xếp vào `bicycle`. | img_004.jpg | CAL-01 |
| 5 | Kính chắn gió / cửa sổ xe: thuộc mask xe hay không? | Kính chắn gió trước xe thuộc mask xe; khoảng không hở bên hông nhìn xuyên qua thấy người lái/hành khách thì vùng người đó gán `person` riêng. | img_023.jpg | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk chở khách đường phố Đông Nam Á | YOLO26 nhận diện nhầm thành `car` do closed-set COCO | Gán đúng lớp `three_wheeler`, bao gồm cả thân xe và khung mui | Mục 2: Xe ba bánh có mui chở khách hoặc chở hàng (tuk-tuk, xe lam, xích lô, ba gác) luôn gán `three_wheeler`, không gán `car` hay `truck` dù model máy gợi ý. |
| img_023.jpg | Tách mask chi tiết cho xe ba bánh bằng SAM | SAM ban đầu gộp cả người ngồi trong buồng lái vào mask xe | Dùng click điểm âm để tách người ra khỏi mask xe, gán box/mask `person` riêng | Mục 4.2: Kính chắn gió thuộc mask phương tiện; người ngồi trong xe nhìn thấy rõ (≥20%) phải gán `person` riêng, không gộp vào mask xe. |
| img_006.jpg | Người đi bộ ở cự ly xa bị bẫy gán nhầm thành xe đạp | Bẫy đổi lớp máy gán nhãn `bicycle` với độ tin cậy cao | Bị lọt trong lượt review ban đầu; hiệu chỉnh lại thành `person` | Mục 4.3: Không dựa vào điểm tin cậy cao của máy để bỏ qua kiểm tra lớp; chú ý phân biệt người đi bộ cự ly xa với xe đạp. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Quy định xe đạp điện thuộc `motorbike`; chốt quy tắc kính xe và người ngồi trong tuk-tuk; chốt lớp `three_wheeler` độc lập với dự đoán của detector; bổ sung lưu ý kiểm tra bẫy đổi lớp | Hiệu chỉnh sau review 24 ảnh batch và 4 ảnh gold, tour 3 model và phân tích 12 bẫy (CAL-01, CAL-02, CAL-03, CAL-04) |
