# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 435; đối tượng sau duyệt: 416; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 375 | 375/435 = 86.2% | 435 box máy |
| edited (sửa) | 24 | 24/435 = 5.5% | 435 box máy |
| deleted (xoá) | 31 | 31/435 = 7.1% | 435 box máy |
| added (vẽ thêm) | 17 | 17/416 = 4.1% | 416 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 8.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 186 | 2 | 0 | 0 |
| bicycle | 7 | 0 | 1 | 2 |
| motorbike | 119 | 15 | 4 | 7 |
| car | 47 | 0 | 5 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 1 | 3 |
| three_wheeler | 10 | 7 | 20 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 44 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 40 | 20 | 4 | 66.7% (KTC 95%: 54–77%) | 90.9% (KTC 95%: 79–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 2/3
  - missing: 1/3
  - loose: 2/3
  - wrong_class: 3/3
  - lọt: img_022.jpg (missing), img_028.jpg (extra), img_004.jpg (loose), img_024.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
