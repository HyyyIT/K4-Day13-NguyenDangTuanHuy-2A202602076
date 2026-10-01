# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: K4-DAY13-Vui_Ve
- Thành viên: xem `TEAMMATES.md` (họ tên/MSSV, vai trò từng lượt).
- Trạng thái: `provided-results` (kết quả được cấp sẵn, nhóm phân tích).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: provided-results / 2026-10-01 / amd64 (Intel/AMD)
- Image tag và image ID; phiên bản repo: xem image LC cấp
- PCD được cấp / frame_id: frame_id = "demo" / nơi chạy: máy phòng LC
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp: theo LC cấp
- Phạm vi: front-window; score threshold: 0.3 (hộp thấp nhất score=0.301 trong run-C)
- Giả định kênh thứ tư/intensity và nguồn z_ground: z_ground = 0.075m (từ JSON), intensity không đổi giữa các lượt

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | boxes-demo-delta-0-voxel-0.16.json / side-demo-delta-0-voxel-0.16.png | 1 xe (vehicles, score=0.322). Hộp nằm thấp sát mặt đất (z=0.330). Chỉ detect được 1 vật thể gần (x=13.15m, y=-0.45m). Không có pedestrian, two-wheels. |
| B | 1.73 | 0.16 | 13 | 1.034 | boxes-demo-delta-1.73-voxel-0.16.json / side-demo-delta-1.73-voxel-0.16.png | vehicles=10, pedestrian=2, two-wheels=1. mean_z tăng lên 1.034 (+0.704m so với A). Hộp phân tán rộng hơn (xa đến x=55.6m). Không phải chỉ dịch cùng 1.73m — số class và số hộp thay đổi hoàn toàn. |
| C | 1.73 | 0.32 | 6 | 1.091 | boxes-demo-delta-1.73-voxel-0.32.json / side-demo-delta-1.73-voxel-0.32.png | pedestrian=6, vehicles=0, two-wheels=0. Pillar to hơn (0.32m) làm mất toàn bộ xe (vehicles=10→0). Chỉ giữ lại người đi bộ. mean_z tương đương B (1.091 vs 1.034). |

- A/B: **Thay delta z trước model KHÔNG tương đương dịch cùng một hằng số cho output.** Lượt A (delta=0) chỉ detect 1 box vehicles. Lượt B (delta=1.73) detect 13 box với 3 class khác nhau. Nguyên nhân: delta z thay đổi phân bố điểm trên trục z trong từng pillar → đặc trưng pillar hoàn toàn khác → model "nhìn thấy" hình dạng khác nhau của cùng một đám mây điểm.
- B/C: **Pillar to hơn (0.32m vs 0.16m) làm giảm phân giải không gian → mất khả năng detect xe.** B detect vehicles=10, C mất toàn bộ xe (0 vehicles). Nguyên nhân có thể: xe có kích thước lớn hơn pedestrian nhưng cần phân giải pillar nhỏ để phân biệt cấu trúc; khi pillar to, đặc trưng xe bị trộn lẫn vào background. Chưa đủ bằng chứng để kết luận cấu hình nào tốt hơn — cần đánh giá trên nhiều frame với ground truth.
- Giới hạn ROI và góc Side: ROI front-window cắt bỏ vật thể phía sau/bên xe — không thể đọc miss ngoài ROI. Góc Side (nhìn từ cạnh) khó phân biệt yaw của xe chạy thẳng vs xe đỗ nếu hộp dài tương đương.
- JSON chưa đủ cơ sở để import: run-A (1 hộp, score=0.322 thấp, chưa rõ có đủ coverage không). run-C (pillar to, mất xe → chưa đủ để dùng cho dataset có xe). Cần kiểm thêm: nhiều frame, nhiều góc nhìn, so sánh với camera RGB.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0/13 | Không có hộp lệch z | Class/x/y/yaw giữ nguyên theo prediction B | Không cần hành động thêm — prediction trông hợp lý | side-correct.png: hộp nằm đúng chiều cao, phân bố hợp lý |
| case-batch-z | 13/13 | Tất cả hộp bị trừ z xuống ~1.805m (= delta + z_ground = 1.73 + 0.075) | Class/x/y/yaw không đổi | **Dừng batch, kiểm phép chuyển pipeline** — lỗi hệ thống, không phải lỗi từng hộp | side-batch-z.png: toàn bộ hộp nổi lên cao bất thường so với side-correct.png |
| case-one-box-z | 1/13 | Chỉ hộp đầu tiên (vehicles x=8.09) bị trừ z xuống ~1.805m | Class/x/y/yaw của 12 hộp còn lại không đổi | **Kiểm đối tượng đó từ nhiều góc** (Front, Top, Side) trước khi quyết định; không dừng cả batch | side-one-box-z.png: 12 hộp bình thường, 1 hộp xe nằm quá thấp so với mặt đất |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng. Nguồn: `manifest.json` — `case-batch-z` dịch toàn bộ z xuống delta+z_ground=1.805m; `case-one-box-z` chỉ dịch hộp index 0.

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục: vai trò đã làm; một quan sát A/B/C có dẫn file hoặc hộp/vùng; diễn giải phép z thuận/ngược; một quyết định lỗi batch và hành động; điều chưa chắc. Chỉ đọc kết quả chuẩn bị trước thì ghi rõ chưa tự chạy.

## Nhận xét của Nguyễn Đăng Tuấn Huy — 02076

- **Vai trò:** Lượt A: Vận hành | Lượt B: Ghi log | Lượt C: Xem hình học
- **Trạng thái:** Phân tích kết quả `provided-results`; chưa tự chạy inference.
- **Quan sát A↔B:** Lượt A (delta=0) chỉ detect 1 box vehicles (mean_z=0.330, score=0.322, file `run-A/boxes-demo-delta-0-voxel-0.16.json`). Lượt B (delta=1.73) detect 13 box: vehicles=10, pedestrian=2, two-wheels=1 (mean_z=1.034, file `run-B/boxes-demo-delta-1.73-voxel-0.16.json`). Kết luận: delta z thay đổi đặc trưng pillar đầu vào, không phải chỉ dịch output — model "nhìn thấy" hình dạng hoàn toàn khác của cùng đám mây điểm.
- **Quan sát B↔C:** Lượt C (voxel=0.32m) mất hoàn toàn vehicles=10, chỉ còn pedestrian=6 (file `run-C/boxes-demo-delta-1.73-voxel-0.32.json`). Ảnh `run-C/side-demo-delta-1.73-voxel-0.32.png` cho thấy ít hộp hơn rõ rệt, không còn hộp kích thước xe. Pillar to làm mất phân giải phân biệt xe vs nền.
- **Phép z:** `z_model = z_source - z_ground - delta`. Lượt A: delta=0 → z_model ≈ z_source - 0.075m. Lượt B: delta=1.73 → z_model = z_source - 0.075 - 1.73 → model nhận input thấp hơn ~1.805m → đặc trưng voxel pillar thay đổi lớn. Ngược lại khi export ra CVAT: z_cvat = z_model + delta + z_ground.
- **Quyết định QC:** `case-batch-z` — toàn bộ 13 hộp lệch z cùng một lượng (−1.805m theo `manifest.json`). Đây là lỗi hệ thống trong phép chuyển pipeline, không phải lỗi từng hộp. → **Hành động: Dừng batch, không sửa tay, báo LC kiểm pipeline chuyển z.**
- **Điều chưa chắc:** Chưa rõ tại sao pillar to (0.32m) lại giữ pedestrian mà mất hoàn toàn vehicles — có thể do vehicles cần nhiều pillar liên tiếp để tạo đặc trưng hình dạng dài, trong khi pedestrian compact hơn nên vẫn nằm gọn trong 1 pillar to.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
