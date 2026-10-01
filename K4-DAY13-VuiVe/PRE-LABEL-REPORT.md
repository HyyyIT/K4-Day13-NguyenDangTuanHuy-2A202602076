# Báo cáo thực hành PointPillars — Day 13

## Nhóm và provenance

- Mã nhóm/phòng: **K4-DAY13-VuiVe**.
- Thành viên/MSSV và vai trò A/B/C: xem [TEAMMATES.md](TEAMMATES.md).
- Trạng thái: **`executed-by-group`** — nhóm đã chạy ba lượt A/B/C ngày 01/10/2026.
- Phân vai vận hành: A — Nguyễn Đăng Tuấn Huy; B — Chu Thái Hòa; C — Kim Nguyên Khôi. Các vai kiểm JSON, xem hình học và ghi log được luân phiên theo bảng thành viên.
- Môi trường: Linux/amd64; giới hạn container 4 CPU / 4 GB. Đây là giới hạn chạy, không phải số đo RAM tối thiểu của máy.
- Thời điểm chạy: 2026-10-01 08:07–08:08 UTC (15:07–15:08 giờ Việt Nam), theo `output/smoke.json`.
- Image tag: `day13-pointpillars:lc-20261001-amd64`; image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`.
- Phiên bản repo lúc chạy: `0831856d921609312d42c7582c366e5a311bb7b1`; working tree dirty: true.
- PCD: `demo.pcd`, frame_id `demo`, mẫu KITTI 000008 bản Student; x/y giữ nguyên, z dịch +1.73 m, reflectance thật bị bỏ và RGB=0 là placeholder. Xem [provenance](../data/provenance.json) và [ghi nguồn/giấy phép](../data/ATTRIBUTION.md).
- Input SHA-256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA-256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Cấu hình giữ nguyên: dataset KITTI, cùng frame/checkpoint, score threshold 0.3, front-window, không thêm `--full-scene` giữa các lượt.
- Adapter intensity: dùng kênh hằng 0.0 cho vehicles và 0.7 cho pedestrian/two-wheels theo `practice/preannotate.py`; RGB không phải intensity được phục hồi. `z_ground = 0.075 m` là số được ghi từ JSON, được ước lượng từ scan.

## Ba lượt inference

Các đường dẫn kết quả dưới đây được ghi tương đối với thư mục nhóm `K4-DAY13-VuiVe/`.

| Lượt | delta (m) | Pillar XY (m) | Số hộp | mean_z (m) | File JSON/Side/CSV | Quan sát |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `output/run-A/boxes-demo-delta-0-voxel-0.16.json`; `output/run-A/side-demo-delta-0-voxel-0.16.png`; `output/run-A/summary.csv` | 1 vehicles, score khoảng 0.322; tâm x≈13.15, y≈-0.45, z≈0.330. Đáy hộp ≈0.330−1.46/2=−0.40 m, dưới đường tham chiếu z=0 trên ảnh Side. |
| B | 1.73 | 0.16 | 13 | 1.034 | `output/run-B/boxes-demo-delta-1.73-voxel-0.16.json`; `output/run-B/side-demo-delta-1.73-voxel-0.16.png`; `output/run-B/summary.csv` | 10 vehicles, 2 pedestrian, 1 two-wheels. Số hộp tăng từ 1 lên 13; mean_z tăng khoảng 0.704 m. Ví dụ hộp đầu z≈0.921, h≈1.54 cho đáy ≈0.15 m. |
| C | 1.73 | 0.32 | 6 | 1.091 | `output/run-C/boxes-demo-delta-1.73-voxel-0.32.json`; `output/run-C/side-demo-delta-1.73-voxel-0.32.png`; `output/run-C/summary.csv` | 6 pedestrian, không còn prediction vehicles hoặc two-wheels. So với B, số hộp giảm từ 13 xuống 6; mean_z tăng khoảng 0.057 m. |

### A/B: thay input trước model có khác dịch output không?

Có. Script dùng `z_model = z_source - z_ground - delta`. Với cùng điểm nguồn và z_ground, tăng delta từ 0 lên 1.73 làm input B **thấp hơn input A 1.73 m**. Vì thế input A cao hơn B 1.73 m; không phải input A bị hạ thấp hơn B. Model chạy lại trên input khác nên tập prediction có thể đổi số hộp, class, vị trí và score.

Nếu chỉ tịnh tiến mọi hộp sau inference, số hộp/class/score vẫn giữ nguyên. Số liệu A/B trong báo cáo là 1→13 hộp với tập class khác nhau; mean_z tăng 0.704 m trên hai tập hộp khác nhau, không chứng minh mọi hộp tăng đúng 1.73 m. Chưa có reference để khẳng định mọi prediction B đúng hoặc mọi đối tượng ngoài hộp A là miss.

### B/C: thấy gì khi đổi pillar? Có đủ chứng cứ chọn cấu hình tốt hơn không?

B/C giữ delta=1.73, chỉ đổi pillar XY 0.16→0.32 m; diện tích mỗi pillar tăng gấp 4 và độ phân giải XY giảm. Số hộp ở B là 13, ở C là 6; prediction vehicles và two-wheels không còn.

Chưa có reference, đánh giá nhiều frame hoặc phân tích đặc trưng để xác định nguyên nhân cụ thể hay kết luận cấu hình nào tốt hơn. Số hộp/confidence không tự chứng minh độ đúng. Các giải thích về anchor, pooling hoặc mất chi tiết xe chỉ là giả thuyết cần kiểm thêm.

### Giới hạn ROI và góc Side

Front-window chỉ bao phủ ROI phía trước; đối tượng ngoài ROI không phải bằng chứng model bỏ sót trong phạm vi bài này. Side là hình chiếu x-z toàn scene, các đối tượng khác y có thể chồng lên nhau. Không dùng riêng Side để duyệt yaw, tâm y, class hay mức khớp của từng hộp; cần Top/Front/Side và camera. Đường z=0 trên plot là đường tham chiếu, không chứng nhận mặt đường cục bộ ở mọi x/y.

### JSON nào chưa đủ cơ sở để import? Cần kiểm gì tiếp?

Cả ba JSON là prediction KITTI demo, khác frame và domain Robotaxi nên không được import vào job Robotaxi. Trên chính frame demo, vẫn cần đối chiếu cuboid với PCD qua nhiều view/camera, kiểm class, tâm, kích thước, yaw, ROI và schema. Prediction B có nhiều hộp hơn không phải đáp án chuẩn.

Script xuất `z_source = z_model + z_ground + delta` và đã đổi bottom-z sang center-z, yaw và class theo checkpoint. JSON đã ở hệ PCD nguồn; không cộng delta/z_ground hoặc chuyển center-z lần nữa khi đọc.

## Ca QC có kiểm soát — không import CVAT

Nguồn là prediction B. Helper `practice/pipeline-qc-cases.py` tạo biến đổi có chủ đích, không chạy detector thêm và không tạo nhãn đúng. Đường dẫn ca QC là `output/qc-cases/`; helper ghi nguồn prediction B và `training_only: true` trong `manifest.json`.

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw/kích thước có đổi? | Quyết định | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0/13 | 0 m | Không | Đối chứng giữ nguyên prediction B; chưa kết luận mọi cuboid đúng | `case-correct.json`, `side-correct.png`; mean_z≈1.034. |
| case-batch-z | 13/13 | −1.805 m (=−(1.73+0.075)) | Không | Dừng batch, không sửa tay từng hộp; báo LC kiểm phép chuyển frame/z | `case-batch-z.json`, `side-batch-z.png`; mean_z≈−0.771; hộp đầu z≈0.921→−0.884. Đây là dịch xuống, không phải nổi lên cao. |
| case-one-box-z | 1/13, phần tử index 0 | −1.805 m | Không | Kiểm riêng đối tượng qua nhiều view/camera; không kết luận lỗi toàn pipeline | `case-one-box-z.json`, `side-one-box-z.png`; hộp đầu z≈0.921→−0.884; 12 hộp còn lại giữ nguyên; mean_z≈0.895. Index 0 không phải ID QC trong CVAT. |

Khi mọi hộp cùng lệch z, cần kiểm transform/pipeline và tạo lại prediction đúng. Khi chỉ một hộp lệch, cần bằng chứng đa góc nhìn của đối tượng đó. Không import các ca mô phỏng lỗi vào CVAT; tên `case-correct` chỉ nghĩa giữ nguyên phép chuyển của prediction B.

## Nhận xét cá nhân

### Nhận xét của Kim Nguyên Khôi — MSSV: 02116

- **Vai trò:** Lượt A: xem hình học | Lượt B: kiểm JSON/cấu hình | Lượt C: vận hành
- **Quan sát A↔B:**
  - Ở lượt A (`output/run-A/boxes-demo-delta-0-voxel-0.16.json`), model chỉ dự đoán 1 box duy nhất (`vehicles`, score=0.322, mean_z=0.330m). Qua ảnh `output/run-A/side-*.png`, đáy hộp ở khoảng -0.40 m so với đường tham chiếu z=0. Nhiều cụm điểm không có hộp; cần đối chiếu thêm trước khi kết luận đối tượng bị bỏ sót.
  - Sang lượt B (`output/run-B/boxes-demo-delta-1.73-voxel-0.16.json`), số hộp tăng vọt lên 13 boxes (10 vehicles, 2 pedestrian, 1 two-wheels) với mean_z=1.034m (tăng ~0.704m). Tăng delta từ 0 lên 1.73m làm input của model thấp hơn 1.73m trên trục z; tập prediction thay đổi từ 1 lên 13 hộp. Đây là thay input trước model, không phải tịnh tiến các hộp đã có sau inference.
- **Quan sát B↔C:**
  - Giữ nguyên delta=1.73m nhưng tăng kích thước voxel từ 0.16m lên 0.32m ở lượt C (`output/run-C/boxes-demo-delta-1.73-voxel-0.32.json`), số hộp giảm từ 13 xuống còn 6 hộp (mean_z=1.091m).
  - Đáng chú ý, toàn bộ 10 xe hơi (`vehicles`) biến mất hoàn toàn, model chỉ detect 6 người đi bộ (`pedestrian`). Voxel kích thước lớn làm giảm độ phân giải XY. Nguyên nhân cụ thể khiến prediction không còn vehicles chưa được xác minh chỉ bằng số hộp hoặc ảnh Side.
- **Phép z thuận/ngược:**
  - Tiền xử lý (thuận): $z_{\text{model}} = z_{\text{source}} - z_{\text{ground}} - \delta$. Với $\delta=1.73\text{m}$ và $z_{\text{ground}}=0.075\text{m}$, điểm LiDAR được đưa về hệ tọa độ sensor của checkpoint KITTI.
  - Hậu xử lý (nghịch): $z_{\text{source}} = z_{\text{model}} + z_{\text{ground}} + \delta$. Tọa độ tâm z của bounding box từ model được chuyển ngược lại về hệ quy chiếu PCD nguồn.
- **Quyết định QC:**
  - Trong `case-batch-z.json`, toàn bộ 13 hộp đều bị dịch xuống đúng $1.805\text{m}$ ($=\delta + z_{\text{ground}}$) -> Đây là ca mô phỏng lỗi hệ thống do thiếu bước nghịch đảo z trong pipeline -> Quyết định: **DỪNG BATCH/PIPELINE**, báo LC kiểm tra code transform, tuyệt đối không sửa thủ công 13 hộp trong CVAT.
  - Trong `case-one-box-z.json`, chỉ 1 hộp duy nhất bị lệch z trong khi 12 hộp khác đúng -> Lỗi đối tượng cục bộ -> Quyết định: Kiểm tra kỹ đối tượng này từ nhiều góc nhìn (Top/Front/Side/Camera).
- **Điều chưa chắc:**
  - Chưa rõ tại sao khi tăng kích thước pillar lên 0.32m thì mạng lại mất toàn bộ class `vehicles` nhưng lại nhận diện được 6 `pedestrian` (có thể do pooling của voxel thô làm suy yếu đặc trưng của xe lớn hoặc do anchor tuning).
  - Chưa thể xác định chính xác góc quay yaw và phân loại của 2 pedestrian và 1 two-wheels ở lượt B nếu chỉ dựa vào ảnh Side 2D mà thiếu ảnh BEV/camera đối chiếu.

### Nhận xét của Chu Thái Hòa — 02083

- **Vai trò:** Lượt A: Kiểm JSON/cấu hình | Lượt B: Vận hành | Lượt C: Ghi log (theo bảng phân vai `TEAMMATES.md`).
- **Quan sát có bằng chứng từ các lượt:**
  - *Lượt A (Kiểm JSON):* File `output/run-A/boxes-demo-delta-0-voxel-0.16.json` chỉ có 1 box duy nhất thuộc lớp `vehicles` (score 0.322, mean_z=0.330m), đáy hộp $z_{bottom}=-0.399\text{ m}$ chìm dưới mặt đất $z_{ground}=0.075\text{ m}$. Lượt A chỉ cho 1 hộp; chưa có reference để xác nhận toàn bộ đối tượng bị bỏ sót hay nguyên nhân của từng prediction.
  - *Lượt B (Vận hành):* Chạy script với $\delta=1.73$, voxel $0.16\text{ m}$, container hoàn thành sau 5.41s. Output `output/run-B/boxes-demo-delta-1.73-voxel-0.16.json` tăng vọt lên 13 boxes (10 vehicles, 2 pedestrian, 1 two-wheels, mean_z=1.034m), đáy một số hộp nằm gần đường tham chiếu z=0 trên ảnh Side; cần nhiều view để xác nhận hình học.
  - *Lượt C (Ghi log):* Khi tăng voxel lên $0.32\text{ m}$, file `output/run-C/boxes-demo-delta-1.73-voxel-0.32.json` chỉ còn 6 boxes và toàn bộ là `pedestrian` (mất sạch 10 xe và 1 xe hai bánh). Ảnh `output/run-C/side-*.png` cho thấy không còn bất kỳ bounding box xe hơi nào.
- **Diễn giải phép z thuận/ngược:** Áp dụng $z_{model} = z_{source} - z_{ground} - \delta$ và $z_{source} = z_{model} + z_{ground} + \delta$. Ở lượt A, vì $\delta=0$ nên $z_{model} \approx z_{source} - 0.075\text{ m}$, làm đám mây điểm đưa vào mạng cao hơn $1.73\text{ m}$ so với hệ tọa độ chuẩn của KITTI LiDAR. Việc thay đổi $\delta$ trước inference làm thay đổi phân bố dữ liệu đưa vào mạng (input distribution shift), quyết định việc vật thể có được nhận diện hay không, khác hoàn toàn việc tịnh tiến z sau khi đã có output bounding box.
- **Quyết định lỗi batch và hành động:** Với ca `case-batch-z` trong `output/qc-cases/`, toàn bộ các hộp đều bị lệch cùng một lượng z do lỗi tham số phép chuyển hệ tọa độ trong pipeline. Quyết định: Dừng batch ngay lập tức, báo LC kiểm tra pipeline chuyển đổi; tuyệt đối không sửa thủ công từng hộp trong CVAT.
- **Điều chưa chắc:** Cần nghiên cứu sâu hơn về cơ chế trích xuất đặc trưng của Pillar Feature Net và mạng 2D Backbone của PointPillars để hiểu rõ hơn tại sao kích thước pillar $0.32\text{ m}$ lại làm điểm tin cậy của anchor xe bị rớt xuống dưới threshold 0.3 trong khi lại gom các điểm thưa kích hoạt anchor người đi bộ.

### Cầm Vũ Ngọc Thạch — 02067

- Vai trò: Lượt A ghi log; Lượt B xem hình học (ảnh Side); Lượt C kiểm JSON/cấu hình. Trạng thái: `executed-by-group`.

**Lượt A — ghi log** (delta=0, voxel=0.16; `output/run-A/`)

- `summary.csv`: n_boxes=1, mean_z=0.330. JSON `boxes-demo-delta-0-voxel-0.16.json`: 1 hộp vehicles, x=13.15, y=-0.45, z=0.330, length 3.62, width 1.52, height 1.46, yaw 2.67, score 0.32.
- Ảnh `side-demo-delta-0-voxel-0.16.png`: đáy hộp ≈ 0.330 - 1.46/2 = -0.40 m, hộp nằm dưới đường z=0 trong khi điểm mặt đất nằm ở z≈0; chỉ có 1 hộp trên toàn scene trong khi các cụm điểm xe khác không có hộp.

**Lượt B — xem hình học** (delta=1.73, voxel=0.16; `output/run-B/`)

- `summary.csv`: n_boxes=13, mean_z=1.034; 10 vehicles, 1 two-wheels, 2 pedestrian; score từ 0.32 đến 0.93.
- Ảnh `side-demo-delta-1.73-voxel-0.16.png`: các hộp xe lớn có đáy gần z≈0 (vd. hộp 1: z=0.92, h=1.54 → đáy ≈ 0.15 m), không còn hộp chìm rõ như lượt A. Hộp mới xuất hiện ở nhiều vị trí x (3.7 → 55.6 m). So với A: 1 → 13 hộp, mean_z +0.704.
- Hộp pedestrian/two-wheels trên Side là các hộp hẹp, cao ≈ 1.65–1.81 m.

**Lượt C — kiểm JSON/cấu hình** (delta=1.73, voxel=0.32; `output/run-C/`)

- `summary.csv`: n_boxes=6, mean_z=1.091. Cả 6 hộp là pedestrian, score 0.30–0.81, height 1.68–1.79; không còn vehicles và two-wheels. Cấu hình khác B đúng một biến (voxel 0.16 → 0.32); frame_id, dataset, delta, z_ground (0.075) giống B.
- So B → C: 13 → 6 hộp, mất toàn bộ 10 vehicles và 1 two-wheels; mean_z 1.034 → 1.091.

**Phép z:** z_model = z_source - z_ground - delta; z_source = z_model + z_ground + delta (z_ground=0.075; delta=0 ở A, 1.73 ở B/C). Đổi delta trước inference thay input của mạng nên số hộp có thể đổi (A→B: 1→13); khác với việc cộng/trừ một hằng số lên mọi hộp sau inference.

**Quyết định QC:** case-batch-z có 13/13 hộp cùng lệch -1.805 m (z âm toàn bộ; class/x/y/yaw không đổi) → dừng, không sửa tay từng hộp, báo LC kiểm phép chuyển z. case-one-box-z chỉ hộp 1 lệch (0.92 → -0.88), 12 hộp còn lại giữ nguyên → kiểm riêng đối tượng đó qua Top/Side/Front và camera.

**Điều chưa chắc:** Chưa biết vì sao pillar 0.32 giữ pedestrian mà mất vehicles (chỉ quan sát số liệu, chưa có bằng chứng nguyên nhân). Chưa chắc hộp lượt A chìm do delta=0 hay do model; Side chỉ là hình chiếu x-z toàn scene nên chưa đủ để kết luận. Không có reference nên chưa nói được cấu hình nào tốt hơn.

### Nhận xét của Nguyễn Đăng Tuấn Huy — 02076

- **Vai trò:** Lượt A: Vận hành | Lượt B: Ghi log | Lượt C: Xem hình học
- **Trạng thái:** `executed-by-group`.
- **Quan sát A↔B:** Lượt A (delta=0) chỉ detect 1 box vehicles (mean_z=0.330, score=0.322, file `output/run-A/boxes-demo-delta-0-voxel-0.16.json`). Lượt B (delta=1.73) detect 13 box: vehicles=10, pedestrian=2, two-wheels=1 (mean_z=1.034, file `output/run-B/boxes-demo-delta-1.73-voxel-0.16.json`). Kết luận: delta z thay đổi đặc trưng pillar đầu vào, không phải chỉ dịch output — model "nhìn thấy" hình dạng hoàn toàn khác của cùng đám mây điểm.
- **Quan sát B↔C:** Lượt C (voxel=0.32m) mất hoàn toàn vehicles=10, chỉ còn pedestrian=6 (file `output/run-C/boxes-demo-delta-1.73-voxel-0.32.json`). Ảnh `output/run-C/side-demo-delta-1.73-voxel-0.32.png` cho thấy ít hộp hơn rõ rệt, không còn hộp kích thước xe. Pillar to làm giảm độ phân giải XY; nguyên nhân cụ thể khiến prediction không còn vehicles chưa được xác minh.
- **Phép z:** `z_model = z_source - z_ground - delta`. Lượt A: delta=0 → z_model ≈ z_source - 0.075m. Lượt B: delta=1.73 → z_model = z_source - 0.075 - 1.73 → model nhận input thấp hơn ~1.805m → đặc trưng voxel pillar thay đổi lớn. Khi xuất JSON, script dùng `z_source = z_model + delta + z_ground`. JSON đã ở hệ PCD nguồn nên không cộng delta/z_ground lần nữa khi đọc hoặc chuyển tiếp.
- **Quyết định QC:** `case-batch-z` — toàn bộ 13 hộp lệch z cùng một lượng (−1.805m theo `manifest.json`). Đây là ca mô phỏng lỗi hệ thống trong phép chuyển pipeline, không phải lỗi từng hộp. → **Hành động: Dừng batch, không sửa tay, báo LC kiểm pipeline chuyển z.**
- **Điều chưa chắc:** Chưa rõ tại sao pillar to (0.32m) lại giữ pedestrian mà mất hoàn toàn vehicles — có thể do vehicles cần nhiều pillar liên tiếp để tạo đặc trưng hình dạng dài, trong khi pedestrian compact hơn nên vẫn nằm gọn trong 1 pillar to.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
