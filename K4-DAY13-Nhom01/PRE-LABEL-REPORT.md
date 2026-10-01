# Báo cáo thực hành PointPillars — Day 13

## Nhóm và provenance

- Mã nhóm/phòng: K4-DAY13-Vui_Ve
- Thành viên: xem `TEAMMATES.md`.
- Trạng thái: `executed-by-group` (nhóm xác nhận tự chạy lệnh)
- Người thực sự chạy: nhóm Vui_Ve, vận hành theo bảng vai (A: Tuấn Huy, B: Thái Hòa, C: Nguyên Khôi); ngày/giờ: 2026-10-01 08:07–08:08 UTC (theo `smoke.json`); hệ máy: linux/amd64
- Image tag: `day13-pointpillars:lc-20261001-amd64`; image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`; repo revision `0831856` (working tree dirty: true)
- PCD: `input/demo.pcd` (KITTI 000008 bản Student), frame_id `demo`; input sha256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`, sha256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`
- Phạm vi: front-window; score threshold 0.3; giới hạn container 4 CPU / 4 GB
- Giả định kênh thứ tư/intensity và nguồn z_ground: reflectance thật bị bỏ, kênh hằng theo lớp (RGB=0 là placeholder); z_ground = 0.075 m, ước lượng từ PCD (theo JSON/manifest)

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | run-A/boxes-demo-delta-0-voxel-0.16.json, side-demo-delta-0-voxel-0.16.png, summary.csv | 1 hộp vehicles (x=13.2, y=-0.45, z=0.330, h=1.46, score 0.32). Trên ảnh Side, đáy hộp ≈ 0.33-0.73 = -0.40 m, tức chìm dưới đường z=0 trong khi điểm mặt đất nằm ở z≈0 |
| B | 1.73 | 0.16 | 13 | 1.034 | run-B/boxes-demo-delta-1.73-voxel-0.16.json, side-...png, summary.csv | 10 vehicles, 1 two-wheels, 2 pedestrian; score 0.32–0.93. Đáy hộp xe lớn nằm sát z≈0 (vd. hộp 1: 0.92-0.77 = 0.15 m) |
| C | 1.73 | 0.32 | 6 | 1.091 | run-C/boxes-demo-delta-1.73-voxel-0.32.json, side-...png, summary.csv | Cả 6 hộp đều là pedestrian (score 0.30–0.81); không còn vehicles/two-wheels |

- A/B: [nhóm trả lời — dịch z trước inference làm model chạy lại trên input khác nên số hộp đổi 1→13, không phải mọi hộp dịch đúng 1.73 m; mean_z đổi 0.330→1.034 gồm cả thay đổi tập hộp]
- B/C: [nhóm trả lời — chỉ đổi pillar 0.16→0.32 làm 13→6 hộp và mất toàn bộ vehicles; số liệu không đủ để nói cấu hình nào tốt hơn, vì không có reference]
- Giới hạn ROI và góc Side: [nhóm trả lời — Side là hình chiếu x-z toàn scene, hộp chồng nhau theo y; ROI chỉ phía trước nên vật ngoài ROI không phải bằng chứng model bỏ sót]
- JSON chưa đủ cơ sở để import: [nhóm trả lời]

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Quyết định | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0/13 | 0 | Không | Không lệch so với B (đây chỉ là bản sao prediction B, không phải đáp án) | mean_z 1.034 = B |
| case-batch-z | 13/13 | -1.805 m (delta 1.73 + z_ground 0.075) | Không (class/x/y/yaw/length giữ nguyên) | Dừng batch, báo LC kiểm phép chuyển | mean_z -0.771; mọi hộp z âm (-1.11…-0.38) |
| case-one-box-z | 1/13 (hộp 1) | -1.805 m | Không | Kiểm riêng đối tượng đó nhiều góc | hộp 1 z 0.92→-0.88; 12 hộp còn lại giữ nguyên; mean_z 0.895 |

Helper `pipeline-qc-cases.py` tạo các ca bằng biến đổi có chủ đích từ prediction B (manifest: `training_only: true`); đây không phải kết quả inference riêng hay nhãn đúng.

## Nhận xét cá nhân

### Cầm Vũ Ngọc Thạch — 02067

- Vai trò: Lượt A ghi log; Lượt B xem hình học (ảnh Side); Lượt C kiểm JSON/cấu hình. Trạng thái: `executed-by-group` **[nhóm xác nhận]**.

**Lượt A — ghi log** (delta=0, voxel=0.16; `run-A/`)
- `summary.csv`: n_boxes=1, mean_z=0.330. JSON `boxes-demo-delta-0-voxel-0.16.json`: 1 hộp vehicles, x=13.15, y=-0.45, z=0.330, length 3.62, width 1.52, height 1.46, yaw 2.67, score 0.32.
- Ảnh `side-demo-delta-0-voxel-0.16.png`: đáy hộp ≈ 0.330 - 1.46/2 = -0.40 m, hộp nằm dưới đường z=0 trong khi điểm mặt đất nằm ở z≈0; chỉ có 1 hộp trên toàn scene trong khi các cụm điểm xe khác không có hộp.

**Lượt B — xem hình học** (delta=1.73, voxel=0.16; `run-B/`)
- `summary.csv`: n_boxes=13, mean_z=1.034; 10 vehicles, 1 two-wheels, 2 pedestrian; score từ 0.32 đến 0.93.
- Ảnh `side-demo-delta-1.73-voxel-0.16.png`: các hộp xe lớn có đáy gần z≈0 (vd. hộp 1: z=0.92, h=1.54 → đáy ≈ 0.15 m), không còn hộp chìm rõ như lượt A. Hộp mới xuất hiện ở nhiều vị trí x (3.7 → 55.6 m). So với A: 1 → 13 hộp, mean_z +0.704.
- Hộp pedestrian/two-wheels trên Side là các hộp hẹp, cao ≈ 1.65–1.81 m.

**Lượt C — kiểm JSON/cấu hình** (delta=1.73, voxel=0.32; `run-C/`)
- `summary.csv`: n_boxes=6, mean_z=1.091. Cả 6 hộp là pedestrian, score 0.30–0.81, height 1.68–1.79; không còn vehicles và two-wheels. Cấu hình khác B đúng một biến (voxel 0.16 → 0.32); frame_id, dataset, delta, z_ground (0.075) giống B.
- So B → C: 13 → 6 hộp, mất toàn bộ 10 vehicles và 1 two-wheels; mean_z 1.034 → 1.091.

**Phép z:** z_model = z_source - z_ground - delta; z_source = z_model + z_ground + delta (z_ground=0.075; delta=0 ở A, 1.73 ở B/C). Đổi delta trước inference thay input của mạng nên số hộp có thể đổi (A→B: 1→13); khác với việc cộng/trừ một hằng số lên mọi hộp sau inference.

**Quyết định QC:** case-batch-z có 13/13 hộp cùng lệch -1.805 m (z âm toàn bộ; class/x/y/yaw không đổi) → dừng, không sửa tay từng hộp, báo LC kiểm phép chuyển z. case-one-box-z chỉ hộp 1 lệch (0.92 → -0.88), 12 hộp còn lại giữ nguyên → kiểm riêng đối tượng đó qua Top/Side/Front và camera.

**Điều chưa chắc:** Chưa biết vì sao pillar 0.32 giữ pedestrian mà mất vehicles (chỉ quan sát số liệu, chưa có bằng chứng nguyên nhân). Chưa chắc hộp lượt A chìm do delta=0 hay do model; Side chỉ là hình chiếu x-z toàn scene nên chưa đủ để kết luận. Không có reference nên chưa nói được cấu hình nào tốt hơn.

### Nguyễn Đăng Tuấn Huy — 02076

- Vai trò: Lượt A vận hành; Lượt B ghi log; Lượt C xem hình học (ảnh Side). Trạng thái: `executed-by-group`.
- Quan sát A/B/C: Lượt A (`run-A/summary.csv`) 1 hộp, mean_z 0.330; lượt B 13 hộp, mean_z 1.034 (`run-B/boxes-demo-delta-1.73-voxel-0.16.json`); trên `run-C/side-demo-delta-1.73-voxel-0.32.png` chỉ còn 6 hộp hẹp, cao ≈ 1.7–1.8 m (pedestrian), so với 13 hộp của B.
- Phép z: z_model = z_source - z_ground - delta; z_source = z_model + z_ground + delta (z_ground=0.075). A/B đổi delta trước inference nên model chạy lại trên input khác, không phải dịch mọi hộp cùng một lượng.
- Quyết định QC: case-batch-z lệch 13/13 hộp cùng -1.805 m → dừng batch, báo LC kiểm pipeline; case-one-box-z chỉ hộp 1 lệch → kiểm đối tượng đó từ nhiều góc.
- Điều chưa chắc: Side chỉ là hình chiếu x-z nên chưa đủ kết luận hộp nào đúng; không có reference để nói cấu hình nào tốt hơn.

### Chu Thái Hòa — 02083

- Vai trò: Lượt A kiểm JSON/cấu hình; Lượt B vận hành; Lượt C ghi log. Trạng thái: `executed-by-group`.
- Quan sát A/B/C: `run-A/boxes-demo-delta-0-voxel-0.16.json` có 1 hộp vehicles (z=0.330, score 0.32); `run-B` có 13 hộp (10 vehicles, 1 two-wheels, 2 pedestrian); `run-C` có 6 hộp, toàn pedestrian, không còn vehicles. B và C chỉ khác voxel (0.16 → 0.32).
- Phép z: z_model = z_source - z_ground - delta; JSON đã ở hệ nguồn nên không đổi z thêm lần nữa khi đọc.
- Quyết định QC: batch lệch cùng 1.805 m (= delta 1.73 + z_ground 0.075), class/x/y/yaw không đổi → dừng pipeline, không sửa tay; một hộp lệch → kiểm riêng.
- Điều chưa chắc: Chưa rõ vì sao pillar 0.32 làm mất vehicles; chưa có bằng chứng nguyên nhân.

### Kim Nguyên Khôi — 02116

- Vai trò: Lượt A xem hình học; Lượt B kiểm JSON/cấu hình; Lượt C vận hành. Trạng thái: `executed-by-group`.
- Quan sát A/B/C: Trên `run-A/side-demo-delta-0-voxel-0.16.png`, hộp duy nhất có đáy ≈ -0.40 m, chìm dưới đường z=0. `run-B/boxes-...json` có 13 hộp, mean_z 1.034, đáy các hộp xe gần z≈0 (vd. hộp 1 đáy ≈ 0.15 m). Lượt C chỉ còn 6 pedestrian.
- Phép z: z_source = z_model + z_ground + delta; delta=0 ở A, 1.73 ở B/C, nên A/B khác nhau cả về input mạng lẫn số hộp (1 → 13).
- Quyết định QC: case-batch-z có mean_z -0.771 (mọi hộp z âm) → dừng batch; case-one-box-z mean_z 0.895, chỉ hộp 1 xuống -0.88 → kiểm từng hộp.
- Điều chưa chắc: chưa biết hộp chìm ở lượt A do delta=0 hay do model; cần Top/Front và camera để kết luận.
