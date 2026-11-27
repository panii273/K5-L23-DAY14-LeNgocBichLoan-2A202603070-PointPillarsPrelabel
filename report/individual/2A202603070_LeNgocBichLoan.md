# Báo cáo Day 14 — Lê Ngọc Bích Loan — 2A202603070

> Sao chép file này thành `report/individual/<MSSV>_<HoTen>.md` (không dấu), ví dụ `2A20260000_NguyenVanAn.md`.

- Mã học viên dùng với `--student`: `070`
- `seed_tag` trong `frames_A.txt`: `393590736f`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

### Quan sát baseline trên tập A

Sau khi kiểm tra 3 frame tập A bằng point cloud, ảnh camera và công cụ QC bổ sung, tôi nhận thấy:

- **Lỗi phổ biến nhất tôi ghi nhận là box thừa.** Với cấu hình gốc (`score_thr=0.1`), 40/94 box có score dưới `0.4`, trong đó 14 box có score dưới `0.3`. Tôi ưu tiên rà các box có score thấp và đối chiếu camera, point cloud để xác định box nào thực sự thừa.
- **Nhóm thường cần kiểm tra kỹ** là Cyclist, Pedestrian và Car ở xa LiDAR. Các box Cyclist và Pedestrian tôi quan sát có score trong khoảng `0.14–0.45`. Trong 21 box cách LiDAR trên 35 m, có 11 box có score dưới `0.4`.
- **Nhóm dễ bị bỏ sót** gồm xe lớn (Truck, Bus) và các vật thể nhỏ hoặc bị che khuất. Mô hình PointPillars huấn luyện trên KITTI không dự đoán trực tiếp hai lớp Truck và Bus.
- **Các box tôi nghi dự đoán sai** thường có score thấp, khoảng `0.14–0.35`, hoặc nằm xa LiDAR. Tôi gặp dạng nghi vấn này ở các box Cyclist và Pedestrian.

### Dự đoán ảnh hưởng của tham số

| Tham số | Mức dự đoán | Tác động kỳ vọng | Rủi ro |
|---|---:|---|---|
| `score_thr` | `0.3` | Tăng ngưỡng từ `0.1` lên `0.3` có thể giảm số box thừa phải xóa. Tôi kỳ vọng vẫn giữ được nhiều box Car có score cao. | Có thể loại nhầm Cyclist hoặc Pedestrian thật có score thấp. |
| `min_points` | `5` | Yêu cầu ít nhất 5 điểm LiDAR trong mỗi box có thể lọc bớt các box nhiễu chứa rất ít điểm. | Vật thể thật ở xa hoặc bị che khuất cũng có thể chứa dưới 5 điểm và bị loại nhầm. |

### Cấu hình tôi dự đoán tốt nhất

- `score_thr = 0.3`
- `min_points = 5`

Tôi chọn ngưỡng `0.3` vì muốn loại bớt box thừa mà vẫn hạn chế nguy cơ bỏ sót vật thể thật. Ở baseline, 40 box có score dưới `0.4`, nên tôi dự đoán ngưỡng `0.4` có rủi ro lọc nhầm cao hơn. Thêm lại một box bị sót tốn 1,0 điểm công sửa, hơn ba lần chi phí xóa một box thừa (0,3 điểm). Tôi kỳ vọng cấu hình trên giảm tổng công sửa tính theo số box phải xóa, thêm và chỉnh sửa.

Tôi đưa ra dự đoán này dựa trên quan sát baseline của 3 frame tập A, trước khi chạy Sweep và chưa sử dụng dữ liệu tập B.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | | | | |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_<ID>/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --gt d14_<ID>/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Nêu ít nhất 2 giới hạn (ví dụ: chỉ 3 frame tune; model KITTI không có Truck/Bus; chi phí là công thức, chưa phải thời gian sửa thật): ...
