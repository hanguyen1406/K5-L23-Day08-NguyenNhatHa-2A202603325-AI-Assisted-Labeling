# Báo cáo Day 08 — 2A202603325 - Nguyễn Nhật Hạ

> Mọi con số kèm mẫu số và nguồn (file nào). Viết bằng lời của học viên.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.5 | r01/r05: sweep đạt recall 94% / precision 68% trên 16 vật gold; nâng từ 0.25 để bớt box trùng ở đám đông |
| motorbike / car | YOLO26m | 0.3 (mặc định) | r01/r05: không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 89% / 88%); giữ mặc định và ghi rõ giới hạn |
| bicycle / bus / truck | YOLO26m | 0.3 (mặc định) | r01/r05: chỉ 1–3 vật trong 4 ảnh gold (< 5) nên không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03/r06: recall 100%, precision 67% (6 vật gold); YOLO26 bỏ sót hoàn toàn tuk-tuk xanh img_002 |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` (510 box)
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: **SAM 2.1 (`sam2.1_b.pt`)** — cân bằng chất lượng/tốc độ, chạy được trên CPU (14.65 s/ảnh); SAM 3 cần GPU và quyền truy cập gate.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk bị gọi `car`/`truck` hoặc bỏ sót (img_002, img_013) là lỗi **ngữ nghĩa closed-set**, ngưỡng chỉ làm box biến mất chứ không đổi tên. Cách sửa: chạy YOLOE với prompt `three_wheeler`, ưu tiên file YOLOE trước YOLO26 khi merge để box three_wheeler thắng box car/truck chồng lên.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): img_013 box `car` 0.42 ôm đúng tuk-tuk nhưng sai lớp (điểm khá cao, vẫn sai); img_002 YOLOE `three_wheeler` 0.40 nằm trên xe đạp bán hàng (dương tính giả) — phải review tay mọi box dưới 0.5, không tin điểm.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng được chọn và chấm trên cùng 4 ảnh; lớp ít vật (bicycle 3, bus 1, truck 1) chỉ là ước lượng thô; phần lớn ngưỡng vẫn là mặc định 0.3 vì không đạt recall 90%.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 375 | 375/435 = 86.2% | 435 box máy |
| edited | 24 | 24/435 = 5.5% | 435 box máy |
| deleted | 31 | 31/435 = 7.1% | 435 box máy |
| added | 17 | 17/416 = 4.1% | 416 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? `three_wheeler`: chỉ 10 accepted / 37 box máy, 7 edited, 20 deleted — YOLO26 gọi tuk-tuk/xích lô là `car`/`truck`/`motorbike` nên mỗi lần đổi lớp bị tính là xoá + thêm. `car` có 5 deleted (box thừa ở hàng xe đỗ, img_014/img_035).
- So với gold: precision / recall của **máy** 66.7% / 90.9% (mẫu số: 44 đối tượng gold; TP 40, FP 20, FN 4). Precision thấp vì box thừa ở hàng xe đỗ và box sai lớp; recall khá vì gold là cảnh quen.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 = 66.7%** (lần 1: 5/12 = 41.7%; lần 2 duyệt lại: 8/12). Lọt: `missing` 2/3 (img_022, img_024), `extra` 1/3 (img_028), `loose` 1/3 (img_004). `wrong_class` đã bắt 3/3 sau khi duyệt lại.
- Loại lọt nhiều nhất là **missing**: lần đầu không có box nào được vẽ thêm cho xe đạp chở hàng (img_031) và xe/xích lô ở xa (img_022, img_024) — bước 0 "ẩn nhãn, quét ảnh trước" và bước 1 "bỏ sót" bị bỏ qua, mắt chỉ đối chiếu box có sẵn. Lọt `loose` vì không kiểm độ ôm sát của box ở cụm xe đỗ.
- Accepted cao (86.2%) mà bắt bẫy thấp (66.7%) là biểu hiện của **thiên kiến tự động hóa**: nhãn máy trông hợp lý nên bấm giữ, không quay lại ảnh gốc. Accepted không có nghĩa là đúng — lỗi máy **không hiện** trên điểm accepted là các box đúng chỗ nhưng sai lớp và các vật bị bỏ sót hoàn toàn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 7,5 phút/ảnh (30 phút/4 ảnh); duyệt có pre-label **1,7 phút/ảnh** (24 ảnh trong **41 phút**, đo từ timestamp PATCH annotations của CVAT; wall-clock 50 phút, trừ 9m40s không thao tác giữa 2 lượt). Tiết kiệm khoảng **77%** thời gian so với gán tay. Thiên lệch: 4 ảnh gold được gán trước nên đã quen cảnh, thời gian gán tay có thể nhanh hơn người mới; số phút duyệt là thời gian server ghi nhận, gồm cả thời gian nhìn ảnh không sửa.
- Kết luận: **có, nhưng có điều kiện** — pre-label tiết kiệm thời gian cho `person`/`motorbike`/`car` (accepted 86.2% trên toàn bộ), nhưng `three_wheeler` gần như phải làm lại (20/37 box bị xoá) và phải có YOLOE + review tay. Không dùng pre-label cho lớp ba bánh nếu chỉ có YOLO26.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn toàn bộ nhãn và tự đếm vật trước khi hiện nhãn ở **mọi** ảnh — kiểm bằng số bẫy `missing` bắt được (lần 2: 1/3, mục tiêu 3/3 ở lô sau).
  2. Kiểm tra tay mọi box `conf` < 0.5 và mọi cụm xe đỗ — kiểm bằng số `deleted`/`edited` theo lớp trong `review_stats.md`.
  3. Với `three_wheeler`, luôn kiểm lại lớp và đếm bánh/mui trước khi giữ box — kiểm bằng số `three_wheeler` bị xoá (20/37 box ở lô này).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk đỏ/xanh) | SAM 2.1_b | 1 dương + 2 âm + box gợi ý | Không gồm người lái và khách (click âm); gồm mui, thân, bánh | Box YOLO gọi `car 0.42` và ôm cả người lái; mask đúng hình nhưng vẫn sai lớp | `masks/tuktuk_img013.jpg` (27.3% ảnh, 1 polygon) |
| img_002 (tuk-tuk xanh) | SAM 2.1_b | 1 dương + 3 âm + box gợi ý | Không gồm người lái/khách; gồm thân và mui | Mui và thân ra 2 polygon rời; box YOLOE đúng lớp nhưng cần click âm để tách người | `masks/tuktuk_img002.jpg` (6.3%, 2 polygon) |
| img_032 (xích lô) | SAM 2.1_b | 1 dương + 2 âm + box gợi ý | **Gồm hàng hoá** (đệm vải) nằm trong khoang; không gồm người đứng cạnh | Hàng gắn trong khoang nên mask gộp vào xe; nếu coi hàng nhô > 20% thì phải sửa theo guideline 4.2 | `masks/cyclo_img032.jpg` (9.0%, 1 polygon) |

- Đối chiếu guideline mục 4.2: quy tắc "polygon bám phần nhìn thấy" và "mỗi vật một polygon, không chồng nhau" giúp tách người bằng click âm. Quy tắc còn thiếu: mask có gồm hàng hoá gắn trong khoang không và kính xe — đã bổ sung ở v1.1 (CAL-05).
- Mask đúng hình **không cứu được nhãn lớp sai**: img_013 mask tuk-tuk rất sát nhưng nhãn vẫn là `car` nếu không sửa lớp.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa làm (không có GPU cho model trong CVAT) — so sánh offline: click trực tiếp trong CVAT tiện hơn nhưng phụ thuộc model server.

## 6. Calibration (Part 6)

- Số quyết định: **5** (`qc/calibration_decisions.csv`, CAL-01…CAL-05); quan trọng nhất: **CAL-01** — YOLO26 gọi tuk-tuk/xích lô là `car`/`truck`/`motorbike`, phải ưu tiên YOLOE cho `three_wheeler`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: ưu tiên YOLOE cho xe ba bánh; hàng gắn cố định thuộc box xe, hàng nhô > 20% không; ngưỡng `ignore_region` 20 px / ≥ 5 vật; bắt buộc bước 0 khi duyệt; mask tách người bằng click âm.
- Nguyên nhân gốc: guideline mơ hồ (câu 2, 3, 5 — đã chốt), máy sai ngữ nghĩa closed-set (CAL-01), và lỗi quy trình của người duyệt (bỏ bước 0 — CAL-04). Câu 1 và 6 chưa chốt vì thiếu ảnh ví dụ.
- Hạn chế: `can_review` không được dùng trong lượt duyệt (0 box), nên ca khó lấy từ log lỗi và bẫy lọt thay vì từ đánh dấu trong CVAT.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), **92 đối tượng**; người kiểm: học viên (**hạn chế**: vừa là người duyệt).
- Lỗi theo **đối tượng**: 1/92 = 1.1% · theo **ảnh**: **1/5 = 20%** (ngưỡng Release ≤ 2%).
- Top lỗi: `thua` 1 lỗi — img_003 có hai box `motorbike` chồng nhau trên cùng một xe đỗ (#11 và #15); các loại bỏ sót/sai lớp/sai vị trí không gặp trong mẫu.
- **Giới hạn mẫu nhỏ** (`qc_report_auto.md`): với 1/5 ảnh lỗi, cận trên 95% (Wilson) là **62%**; mẫu 5 ảnh không thể chứng minh "dưới 2%".
- Script kiểm được: tên ảnh, lớp, box trong ảnh, đủ ảnh, các số đo. Script **không** kiểm được: box có ôm sát vật, lớp có đúng thực tế, vật cả người và máy cùng bỏ sót, việc duyệt có theo checklist thật không.

**Quyết định: Rework**, ngày 2026-11-19. Lý do (theo cây Hold → Rework → Release): ảnh có lỗi 20% > 2%, bắt bẫy 66.7% < 80%, và `can_review` chưa được dùng. Lô nhãn chưa đủ tin cậy để Release.
Nếu Rework: duyệt lại toàn bộ 24 ảnh với bước 0 bắt buộc, sửa các box thừa ở cụm xe đỗ, export `qc_fixed.json` nếu có thay đổi, chạy lại `build_release.py` → `qc_sample.py --force` → `qc_report.py`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: dùng Qoder để chạy script, đọc số đo và soạn nháp báo cáo; nhãn được duyệt tay trong CVAT, không nhờ AI gán hộ; mọi số liệu được kiểm lại bằng `review_stats.py`, `qc_report.py` và đối chiếu ảnh gốc.
- Gold có bị nhiễm không: **không** — gán 4 ảnh gold trước khi nạp bất kỳ pre-label nào.
- Máy / GPU / CVAT: Windows, **CPU only** (không GPU), CVAT 2.81.0 bằng Docker; vướng mắc: thiếu `pyyaml`/`transformers` nên `check_env.py` báo FAIL hai mục, nhưng YOLO26/YOLOE/SAM vẫn chạy.
- Dùng cache `data/cache/` thay vì tự chạy? **Không** — tự chạy YOLO26, YOLOE và SAM trên CPU.
- Lỗi / bug script đã gặp: không có lỗi script; thời gian SAM trên CPU chậm (14.65 s/ảnh) nhưng chấp nhận được.
- Một điều làm khác lần sau: bắt buộc bước 0 (ẩn nhãn, quét ảnh) trước mỗi ảnh và bấm giờ ngay từ đầu, vì thiếu bước này làm bắt bẫy chỉ đạt 66.7%.
