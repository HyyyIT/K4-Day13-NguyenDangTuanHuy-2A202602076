# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: K4-DAY13-Nhom01
- Thành viên: xem `TEAMMATES.md` (Nguyễn Đăng Tuấn Huy - 02076, Chu Thái Hòa - 02083, Kim Nguyên Khôi - 02116, Cầm Vũ Ngọc Thạch - 02067).
- Trạng thái: `executed-by-group` (chạy qua Docker student runner trên kiến trúc amd64).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Nhóm vận hành (Nguyễn Khôi vận hành Lượt C, Thái Hòa Lượt B, Tuấn Huy Lượt A); 2026-10-01T08:07:23Z; Linux amd64 (4 CPUs, 4GB RAM).
- Image tag và image ID; phiên bản repo: Image `day13-pointpillars:lc-20261001-amd64` (ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`); Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1`.
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `demo.pcd` (frame_id: `demo`, mẫu KITTI 000008 đã chuyển đổi z+1.73m, sha256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`).
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp: `/opt/PointPillars/pretrained/epoch_160.pth` (sha256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`).
- Phạm vi: front-window (`--from KITTI`); score threshold: 0.3.
- Giả định kênh thứ tư/intensity và nguồn z_ground: PCD Student bỏ reflectance nguồn, thêm RGB=0, script dùng kênh hằng số; `z_ground = 0.075m` ước tính từ điểm mặt đường trong scan.

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `ket-qua-nhom-01/run-A/boxes-demo-delta-0-voxel-0.16.json` | Chỉ phát hiện duy nhất 1 box `vehicles` (score=0.322), mean_z=0.330m rất thấp, hộp chìm sát mặt đất. Hầu hết các xe và người đi bộ đều bị miss do point cloud bị hạ thấp so với dải anchor KITTI. |
| B | 1.73 | 0.16 | 13 | 1.034 | `ket-qua-nhom-01/run-B/boxes-demo-delta-1.73-voxel-0.16.json` | Số hộp tăng vọt lên 13 (vehicles=10, pedestrian=2, two-wheels=1), mean_z=1.034m (tăng ~0.704m so với A). Bù delta=1.73m đưa cloud về đúng dải học sensor KITTI, kích hoạt đầy đủ các anchor của các lớp. |
| C | 1.73 | 0.32 | 6 | 1.091 | `ket-qua-nhom-01/run-C/boxes-demo-delta-1.73-voxel-0.32.json` | Số hộp giảm xuống 6 (mất toàn bộ 10 vehicles, chỉ detect 6 pedestrian), mean_z=1.091m. Tăng kích thước pillar làm giảm độ phân giải không gian XY, làm mờ đặc trưng hình học của vật thể lớn như ô tô. |

- A/B: thay input trước model có khác dịch cùng một hằng số cho output không? Vì sao?
  - **Khác hoàn toàn.** Phép trừ delta trước model (`z_model = z_source - z_ground - delta`) là biến đổi biểu diễn không gian 3D của đám mây điểm trước khi đưa vào mạng nơ-ron (Voxel Feature Encoder & Backbone 2D). Model PointPillars KITTI có các anchor 3D cố định ở các dải cao độ xác định so với mặt đường. Khi `delta=0`, toàn bộ đám mây điểm bị tụt thấp, các đặc trưng hình học không kích hoạt được các bộ lọc và score rơi xuống dưới ngưỡng 0.3 (chỉ 1 hộp qua ngưỡng). Khi `delta=1.73`, đám mây điểm nằm đúng phân bố độ cao của sensor KITTI, model trích xuất đúng đặc trưng và nhận diện được 13 hộp. Nếu chỉ dịch output sau model bằng hằng số, số hộp sẽ không thể tự tăng từ 1 lên 13 (vẫn chỉ có 1 hộp bị tịnh tiến z).
- B/C: thấy gì khi đổi pillar? Có đủ bằng chứng để nói cấu hình nào tốt hơn không?
  - **Quan sát:** Tăng pillar XY từ 0.16m lên 0.32m làm số hộp giảm từ 13 xuống 6; toàn bộ 10 xe hơi (`vehicles`) biến mất, mạng chỉ phát hiện 6 người đi bộ (`pedestrian`).
  - **Đánh giá:** Chưa đủ bằng chứng để nói cấu hình nào "tốt hơn" tuyệt đối chỉ dựa trên số lượng hộp hay score. Voxel 0.32m làm gộp điểm thô hơn, giảm độ phân giải không gian khiến các đặc trưng dạng khối lớn của ô tô bị phân rã hoặc không khớp anchor xe, trong khi người đi bộ (kích thước hẹp, dạng cột thẳng) vô tình vẫn kích hoạt anchor ở một số cụm điểm. Tuy nhiên với mục tiêu phát hiện phương tiện giao thông chính, cấu hình voxel 0.16m rõ ràng phù hợp và chi tiết hơn nhiều.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
  - **Giới hạn ROI:** Lệnh chạy dùng `--from KITTI` chỉ quét phạm vi cửa sổ phía trước (front-window). Các đối tượng nằm phía sau hoặc ngoài góc nhìn này không thể kết luận là model bỏ sót (false negative).
  - **Góc nhìn Side:** Là hình chiếu 2D toàn cảnh trên mặt phẳng X-Z (hình chiếu cạnh), các đối tượng có cùng X, Z nhưng khác Y sẽ bị chồng lấp lên nhau. Do đó, không thể dùng riêng ảnh Side để thẩm định góc quay yaw hay vị trí hộp theo trục Y; bắt buộc phải kết hợp góc nhìn từ trên xuống (Top/BEV) và ảnh camera phối cảnh.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?
  - **Cả 3 file JSON** đều chưa đủ cơ sở để import vào các job Robotaxi thật trong CVAT vì đây là kết quả của model pretrained KITTI chạy trên một frame demo (khác domain xe, khác hệ cảm biến và khác frame).
  - Ngay cả trên scan demo này, file `run-B` tuy tốt nhất nhưng vẫn cần kiểm tra từng hộp: kiểm tra class (đặc biệt 2 pedestrians và 1 two-wheels ở vùng biên), kiểm tra độ ôm khít của cuboid và góc yaw trên Top/Front view và camera trước khi coi là đạt chuẩn.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0 m | Không đổi (giữ nguyên) | Chưa rõ / Dùng làm đối chứng | Giữ nguyên từ prediction B (`boxes-demo-delta-1.73-voxel-0.16.json`), mean_z=1.034m, các hộp nằm khớp mặt đất. |
| case-batch-z | 13 / 13 (100%) | -1.805 m (`-(delta + z_ground)`) | Không đổi | DỪNG BATCH / Dừng pipeline ngay lập tức | Toàn bộ 13 hộp đều bị trừ đúng 1.805m vào tâm z (hộp 1 z từ 0.921 xuống -0.884m; mean_z âm). Lỗi hệ thống do thiếu bước nghịch đảo `z_source = z_model + z_ground + delta`. Tuyệt đối không sửa tay từng hộp. |
| case-one-box-z | 1 / 13 | -1.805 m (chỉ hộp đầu tiên ID 0) | Không đổi | Kiểm từng hộp, không dừng batch | Chỉ duy nhất hộp thứ nhất (x=8.09, y=1.21) bị hạ z từ 0.921m xuống -0.884m, 12 hộp còn lại giữ nguyên tọa độ đúng. Đây là lỗi đối tượng cục bộ (object-local), cần kiểm tra đa góc nhìn của đối tượng này. |

Ghi rõ helper `practice/pipeline-qc-cases.py` tạo biến đổi có chủ đích từ prediction của lượt B (`boxes-demo-delta-1.73-voxel-0.16.json`), không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét cá nhân

### Nhận xét của Kim Nguyên Khôi — MSSV: 02116
- **Vai trò:** Lượt A: xem hình học | Lượt B: kiểm JSON/cấu hình | Lượt C: vận hành
- **Quan sát A↔B:**
  - Ở lượt A (`run-A/boxes-demo-delta-0-voxel-0.16.json`), model chỉ dự đoán 1 box duy nhất (`vehicles`, score=0.322, mean_z=0.330m). Qua ảnh `run-A/side-*.png`, hộp này bị chìm sát mặt đất và toàn bộ các xe khác đều bị bỏ sót.
  - Sang lượt B (`run-B/boxes-demo-delta-1.73-voxel-0.16.json`), số hộp tăng vọt lên 13 boxes (10 vehicles, 2 pedestrian, 1 two-wheels) với mean_z=1.034m (tăng ~0.704m). Việc bù delta=1.73m trước inference đã đưa điểm LiDAR vào đúng dải anchor chuẩn của KITTI, giúp model trích xuất đúng đặc trưng và phát hiện thêm 12 hộp mới, chứ không chỉ đơn thuần là tịnh tiến độ cao.
- **Quan sát B↔C:**
  - Giữ nguyên delta=1.73m nhưng tăng kích thước voxel từ 0.16m lên 0.32m ở lượt C (`run-C/boxes-demo-delta-1.73-voxel-0.32.json`), số hộp giảm từ 13 xuống còn 6 hộp (mean_z=1.091m).
  - Đáng chú ý, toàn bộ 10 xe hơi (`vehicles`) biến mất hoàn toàn, model chỉ detect 6 người đi bộ (`pedestrian`). Voxel kích thước lớn làm giảm độ phân giải không gian, làm mờ ranh giới cụm điểm của các vật thể lớn như ô tô.
- **Phép z thuận/ngược:**
  - Tiền xử lý (thuận): $z_{\text{model}} = z_{\text{source}} - z_{\text{ground}} - \delta$. Với $\delta=1.73\text{m}$ và $z_{\text{ground}}=0.075\text{m}$, điểm LiDAR được đưa về hệ tọa độ sensor của checkpoint KITTI.
  - Hậu xử lý (nghịch): $z_{\text{source}} = z_{\text{model}} + z_{\text{ground}} + \delta$. Tọa độ tâm z của bounding box từ model được chuyển ngược lại về hệ quy chiếu PCD nguồn.
- **Quyết định QC:**
  - Trong `case-batch-z.json`, toàn bộ 13 hộp đều bị dịch xuống đúng $1.805\text{m}$ ($=\delta + z_{\text{ground}}$) -> Đây là lỗi hệ thống do thiếu bước nghịch đảo z trong pipeline -> Quyết định: **DỪNG BATCH/PIPELINE**, báo LC kiểm tra code transform, tuyệt đối không sửa thủ công 13 hộp trong CVAT.
  - Trong `case-one-box-z.json`, chỉ 1 hộp duy nhất bị lệch z trong khi 12 hộp khác đúng -> Lỗi đối tượng cục bộ -> Quyết định: Kiểm tra kỹ đối tượng này từ nhiều góc nhìn (Top/Front/Side/Camera).
- **Điều chưa chắc:**
  - Chưa rõ tại sao khi tăng kích thước pillar lên 0.32m thì mạng lại mất toàn bộ class `vehicles` nhưng lại nhận diện được 6 `pedestrian` (có thể do pooling của voxel thô làm suy yếu đặc trưng của xe lớn hoặc do anchor tuning).
  - Chưa thể xác định chính xác góc quay yaw và phân loại của 2 pedestrian và 1 two-wheels ở lượt B nếu chỉ dựa vào ảnh Side 2D mà thiếu ảnh BEV/camera đối chiếu.

### Nhận xét của Nguyễn Đăng Tuấn Huy — MSSV: 02076
- **Vai trò:** Lượt A: vận hành | Lượt B: ghi log | Lượt C: xem hình học
- **Quan sát A↔B:** Lượt A chỉ detect 1 box (mean_z=0.330), lượt B detect 13 box (mean_z=1.034). File `run-B/boxes-*.json` cho thấy model "nhìn thấy" nhiều vật thể hơn khi cloud nằm ở độ cao khác — không phải chỉ dịch cùng 1.73m.
- **Quan sát B↔C:** Voxel 0.32m làm mất phân giải → mất vehicles=10, chỉ còn pedestrian=6. File `run-C/side-*.png` cho thấy ít hộp hơn rõ rệt.
- **Phép z:** $z_{\text{model}} = z_{\text{source}} - z_{\text{ground}} - \delta$. Lượt A: delta=0 nên $z_{\text{model}} \approx z_{\text{source}} - 0.075$. Lượt B: delta=1.73 → model thấy input cao hơn 1.73m.
- **Quyết định QC:** `case-batch-z` → cả batch lệch cùng lượng → dừng batch, kiểm phép chuyển pipeline. Không sửa tay từng hộp.
- **Điều chưa chắc:** Chưa rõ tại sao pillar to lại giữ pedestrian mà mất vehicles.

### Nhận xét của Chu Thái Hòa — MSSV: 02083
- **Vai trò:** Lượt A: kiểm JSON/cấu hình | Lượt B: vận hành | Lượt C: ghi log
- **Quan sát A↔B:** Ở lượt A `run-A/summary.csv` chỉ có 1 box, class vehicles. Sang lượt B với delta=1.73m, `run-B/summary.csv` cho thấy có 13 boxes với đủ cả 3 classes (10 vehicles, 2 pedestrian, 1 two-wheels), mean_z nâng từ 0.330m lên 1.034m.
- **Quan sát B↔C:** Lượt C đổi voxel từ 0.16m lên 0.32m làm số boxes giảm từ 13 xuống 6 (`run-C/summary.csv`). Toàn bộ 10 vehicles bị mất, chỉ còn 6 pedestrian.
- **Phép z:** Chuyển đổi $z_{\text{model}} = z_{\text{source}} - z_{\text{ground}} - \delta$ thay đổi phân bố điểm đưa vào mạng. Hậu xử lý phải cộng ngược lại $z_{\text{source}} = z_{\text{model}} + z_{\text{ground}} + \delta$.
- **Quyết định QC:** `case-batch-z` có 13/13 boxes bị trừ cùng lượng 1.805m → Dừng pipeline báo LC. `case-one-box-z` chỉ lệch 1 box → kiểm tra đối tượng cục bộ.
- **Điều chưa chắc:** Cần kiểm tra thêm ảnh camera phối cảnh để xác minh 1 xe hai bánh ở lượt B có đúng nhãn không.

### Nhận xét của Cầm Vũ Ngọc Thạch — MSSV: 02067
- **Vai trò:** Lượt A: ghi log | Lượt B: xem hình học | Lượt C: kiểm JSON/cấu hình
- **Quan sát A↔B:** Ảnh `side-demo-delta-0-voxel-0.16.png` chỉ có 1 hộp chìm sát vạch z=0. Sang `side-demo-delta-1.73-voxel-0.16.png` các hộp phân bố đều ở độ cao thực tế trên mặt đường và xuất hiện thêm nhiều cụm xe khác nhau.
- **Quan sát B↔C:** `run-C/boxes-*.json` cho thấy 6 hộp đều có label pedestrian, không còn xe nào. Kích thước voxel lớn làm mất các chi tiết hình khối dài của xe.
- **Phép z:** Phép dịch delta trước model ảnh hưởng trực tiếp đến việc kích hoạt anchor và trích xuất đặc trưng của mạng, khác hoàn toàn việc tịnh tiến z sau suy luận.
- **Quyết định QC:** Phát hiện lỗi hàng loạt ở `case-batch-z` thì phải dừng pipeline ngay lập tức, không import vào CVAT; `case-one-box-z` thì kiểm tra riêng hộp đó.
- **Điều chưa chắc:** Chưa rõ độ nhạy của ngưỡng score threshold 0.3 khi thay đổi kích thước voxel.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
