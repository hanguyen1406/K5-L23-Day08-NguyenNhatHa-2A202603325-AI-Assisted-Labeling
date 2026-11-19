# Thông tin học viên

- Họ và tên: Nguyễn Nhật Hạ
- MSHV: 2A202603325
- Lớp (L2 / L3 / khác): <<>>
- Tài khoản GitHub: hanguyen1406
- Repo bài nộp (URL): https://github.com/hanguyen1406/K5-L23-Day08-NguyenNhatHa-2A202603325-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Windows, CPU only (không GPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 375 / 24 / 31 (mẫu số: 435 box máy); added: 17 (mẫu số: 416 đối tượng)
- So với gold: precision / recall của **máy** 66.7% / 90.9% (44 đối tượng gold)
- Bắt bẫy: 8/12 = 66.7% (lần 1: 5/12 = 41.7%)
- Thời gian: gán tay 7,5 phút/ảnh (30 phút/4 ảnh); duyệt có pre-label 1,7 phút/ảnh (41 phút/24 ảnh)
- Lỗi QC: 20% theo ảnh (1/5), 1.1% theo đối tượng (1/92)
- **Quyết định:** Rework — ảnh lỗi 20% > 2% và bắt bẫy 66.7% < 80%, lô nhãn chưa đủ tin cậy để Release
