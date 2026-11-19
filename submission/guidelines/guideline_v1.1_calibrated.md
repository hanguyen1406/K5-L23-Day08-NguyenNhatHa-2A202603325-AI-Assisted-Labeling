# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603325 - Nguyễn Nhật Hạ

> Chỉ phần thay đổi so với v1.0. Mỗi quyết định khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Xe rõ và cao ≥ 20 px: vẽ từng xe. Cụm ≥ 5 vật nhỏ < 20 px sát nhau: một `ignore_region` bao cụm. Không dùng `ignore_region` cho vật rõ. | img_005, img_018, img_024 | CAL-03 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Giỏ/rổ **gắn cố định** thuộc box xe. Hàng nhô ra **> 20% chiều dài xe** không tính vào box xe. Người bán vẫn có box `person` riêng. | img_031 | CAL-02 |
| 4 | Người ngồi trong xe ba bánh có mui che, chỉ thấy thấp thoáng | Giữ ngưỡng "còn nhìn thấy ≥ 20%" của mục 4.1.2; nếu lưỡng lự thì tick `can_review`. Không gán nếu chỉ thấy bóng/mảng nhỏ hơn 20%. | img_013 | CAL-05 |
| 5 | Kính chắn gió / cửa sổ xe: thuộc mask xe? | Thuộc mask xe. Người nhìn xuyên qua kính là polygon `person` riêng. | img_013 | CAL-05 |
| 7 | Có đặt ngưỡng tin cậy tối thiểu để chấp nhận box máy? | Không đặt ngưỡng chất lượng theo `conf`. `conf` chỉ để xếp thứ tự nhìn kỹ; box điểm cao vẫn phải kiểm, box điểm thấp vẫn phải kiểm. | img_002, img_013 | CAL-01 |

**Chưa chốt (nói rõ lý do):**
- **Câu 1 (xe đạp điện nhỏ):** trong 32 ảnh gold + batch không có ca xe đạp điện rõ ràng để làm ví dụ; chưa đủ bằng chứng đổi quy tắc. Khi gặp: chọn lớp tin nhất và tick `can_review`.
- **Câu 6 (box máy chồng nhau trong ảnh đêm):** lô ảnh này là ban ngày, không có ca đêm để đối chiếu.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013 | Tuk-tuk đỏ/xanh có người lái, đỗ sát lề | YOLO26 gọi `car 0.42`, box ôm cả người lái | Gán `three_wheeler` cho cả xe; người lái là `person` riêng | Mục 2: xe ba bánh có mui vẫn là `three_wheeler`, không dùng `car` |
| img_002 | Tuk-tuk xanh giữa đường | YOLO26 không có box nào cho xe; YOLOE `three_wheeler 0.93` đúng nhưng còn dương tính giả 0.40 ở xe đạp bán hàng | Dùng YOLOE cho `three_wheeler`, review tay mọi box < 0.5 | Mục 2 + mục 4.3: box điểm thấp vẫn phải kiểm |
| img_031 | Xe đạp chở hàng, giỏ cà chua và bao long nhãn | Ban đầu bị bẫy `missing` xoá box xe, người duyệt không thêm lại | Box `bicycle` gồm giỏ gắn cố định; người bán và người ngồi là `person` riêng | Mục 4.1.5 và mục 5 (ca khó) |
| img_022, img_024 | Xe/xích lô ở xa, bị che một phần | YOLO26 bỏ sót hoàn toàn (bẫy `missing` lọt cả 2 lượt) | Bắt buộc quét ảnh trước khi hiện nhãn; đếm xe ba bánh và xe đạp trước | Mục 5: bước 0 của checklist là bắt buộc |
| img_028, img_004 | Box ma / box lỏng ở cụm xe | Bẫy `extra`, `loose` lọt | Kiểm riêng cụm xe đỗ: từng box phải có vật thật và ôm sát | Mục 4.1.1–4.1.2 |
| img_005, img_018, img_024 | Cụm xe máy nhỏ đỗ sát nhau | Nhiều box chồng, vật < 20 px | Bọc một `ignore_region` cho cụm < 20 px; xe rõ vẫn vẽ từng chiếc | Mục 4.1.6 |
| img_013, img_032 | Mask tuk-tuk và xích lô có người ngồi trong | SAM 2.1 phủ cả khoang chứa người nếu box ôm cả người lái | Mask chỉ thân xe; dùng click âm lên người lái/khách để tách | Mục 4.2 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 3, 4, 5, 7; thêm quy tắc ưu tiên YOLOE cho `three_wheeler`; bắt buộc bước 0 khi duyệt; quy tắc mask tách người | CAL-01 (lỗi YOLO26 gọi tuk-tuk là car), CAL-02 (hàng trên xe đạp), CAL-03 (cụm xe nhỏ), CAL-04 (bẫy missing/extra/loose lọt), CAL-05 (mask có người) |
