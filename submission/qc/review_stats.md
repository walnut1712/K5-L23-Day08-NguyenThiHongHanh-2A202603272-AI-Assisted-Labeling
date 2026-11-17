# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 689; đối tượng sau duyệt: 549; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 515 | 515/689 = 74.7% | 689 box máy |
| edited (sửa) | 13 | 13/689 = 1.9% | 689 box máy |
| deleted (xoá) | 154 | 154/689 = 22.4% | 689 box máy |
| added (vẽ thêm) | 21 | 21/549 = 3.8% | 549 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 292 | 6 | 73 | 5 |
| bicycle | 6 | 0 | 2 | 3 |
| motorbike | 150 | 2 | 44 | 2 |
| car | 45 | 2 | 8 | 2 |
| bus | 3 | 0 | 0 | 4 |
| truck | 2 | 0 | 4 | 3 |
| three_wheeler | 17 | 3 | 23 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 54 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 40 | 6 | 54.5% (KTC 95%: 44–65%) | 88.9% (KTC 95%: 78–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - loose: 2/3
  - extra: 3/3
  - wrong_class: 2/3
  - lọt: img_021.jpg (missing), img_033.jpg (loose), img_004.jpg (missing), img_006.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
