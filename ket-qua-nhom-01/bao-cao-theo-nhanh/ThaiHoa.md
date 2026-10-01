# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: K4-DAY13-Nhom01
- Thành viên: Chu Thái Hòa (MSSV: 02083) — các thành viên khác xem `TEAMMATES.md` (mỗi thành viên làm việc trên nhánh riêng).
- Trạng thái: `executed-by-group` (theo smoke.json ghi nhận runtime).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Nhóm K4-DAY13-Nhom01; 2026-10-01 08:07 UTC; Linux amd64 (4 CPUs, 4GB RAM).
- Image tag và image ID; phiên bản repo: `day13-pointpillars:lc-20261001-amd64` (ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`); phiên bản repo: `0831856d921609312d42c7582c366e5a311bb7b1`.
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `demo.pcd` / `demo`; input SHA-256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`.
- Checkpoint: PointPillars KITTI có sẵn trong image (`/opt/PointPillars/pretrained/epoch_160.pth`, hash: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`).
- Phạm vi: front-window; score threshold: 0.3.
- Giả định kênh thứ tư/intensity và nguồn z_ground: Kênh thứ tư gán hằng số theo adapter KITTI; z_ground ước lượng từ PCD (`z_ground = 0.075m`).

## Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | ket-qua-nhom-01/run-A/boxes-demo-delta-0-voxel-0.16.json, ket-qua-nhom-01/run-A/summary.csv | Thái Hòa kiểm JSON/cấu hình: Chỉ detect 1 box duy nhất thuộc lớp vehicles (x=13.15m, y=-0.45m, z=0.330m, l=3.62m, w=1.52m, h=1.46m, score=0.322). Đáy hộp tụt xuống z_bottom = 0.330 - 1.459/2 = -0.399m, chìm dưới mặt đất z_ground (0.075m). delta=0 làm mây điểm đi vào model bị nâng cao 1.73m so với hệ cảm biến KITTI chuẩn, khiến model miss gần như toàn bộ vật thể trong cảnh. |
| B | 1.73 | 0.16 | 13 | 1.034 | ket-qua-nhom-01/run-B/boxes-demo-delta-1.73-voxel-0.16.json, ket-qua-nhom-01/run-B/summary.csv | Thái Hòa vận hành lệnh (Tuấn Huy ghi log): 13 boxes (10 vehicles, 2 pedestrian, 1 two-wheels), mean_z=1.034m. Đáy các hộp nằm khớp sát mặt đất. Khi bù delta=1.73m, mây điểm vào model đúng hệ sensor KITTI, model nhận diện đầy đủ các lớp vật thể. |
| C | 1.73 | 0.32 | 6 | 1.091 | ket-qua-nhom-01/run-C/boxes-demo-delta-1.73-voxel-0.32.json, ket-qua-nhom-01/run-C/summary.csv | Thái Hòa ghi log: Khi tăng voxel từ 0.16m lên 0.32m (pillar to hơn), số hộp giảm từ 13 xuống 6, mean_z tăng từ 1.034m lên 1.091m. Đặc biệt, mất toàn bộ 10 xe (vehicles) và 1 xe hai bánh (two-wheels), chỉ còn phát hiện 6 người đi bộ (pedestrian). Voxel to làm mờ ranh giới mây điểm của xe cộ, khiến anchor xe không đạt ngưỡng. |

- A/B: thay input trước model có khác dịch cùng một hằng số cho output không? Vì sao?
  - *Trả lời (Chu Thái Hòa - Kiểm JSON Lượt A / Vận hành Lượt B):* Có, khác hoàn toàn. Khi thay delta từ 0 lên 1.73 trước model, tọa độ đám mây điểm đưa vào mạng thay đổi theo $z_{model} = z_{source} - z_{ground} - \delta$. Điều này làm thay đổi cách chia voxel, trích xuất đặc trưng pillar và phân bố không gian của các điểm. Vì thế mô hình phát hiện ra các cụm vật thể hoàn toàn mới (lượt B phát hiện 13 boxes thay vì chỉ 1 box ở lượt A), chứ không phải là vẫn giữ nguyên 1 box rồi tịnh tiến cao độ z thêm 1.73m.
- B/C: thấy gì khi đổi pillar? Có đủ bằng chứng để nói cấu hình nào tốt hơn không?
  - *Trả lời (Chu Thái Hòa - Ghi log Lượt C):* Đổi pillar XY từ 0.16m lên 0.32m (diện tích mỗi pillar tăng gấp 4) làm suy giảm độ phân giải không gian trên mặt phẳng BEV. Mạng PointPillars mất toàn bộ 10 xe hơi và 1 xe hai bánh do đặc trưng biên dạng xe bị gộp mờ trong pillar to, chỉ phát hiện 6 người đi bộ (do cụm điểm thưa bị gom lại vô tình khớp anchor người đi bộ). Không đủ bằng chứng để khẳng định cấu hình nào "tốt hơn" một cách tuyệt đối chỉ dựa trên số hộp, nhưng cấu hình B (0.16m) cân bằng vượt trội vì nhận diện được đa dạng lớp và không làm mất đối tượng xe cộ quan trọng.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
  - *(Thành viên phụ trách xem hình học điền trên nhánh riêng)*
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?
  - *Trả lời (Chu Thái Hòa - Kiểm JSON Lượt A):* File `boxes-demo-delta-0-voxel-0.16.json` của Lượt A hoàn toàn chưa đủ cơ sở để import vào CVAT vì thiếu tham số bù độ cao sensor ($\delta=0$), dẫn đến miss hầu hết vật thể và đáy box bị cắm xuống đất. Cần kiểm tra kỹ các file prediction với $\delta=1.73$ (lượt B), đồng thời đối chiếu đa góc nhìn (Side, Top, Front, Camera) trước khi import.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | | | | | *(Phần nhóm thảo luận/thành viên QC điền trên nhánh riêng)* |
| case-batch-z | | | | | *(Phần nhóm thảo luận/thành viên QC điền trên nhánh riêng)* |
| case-one-box-z | | | | | *(Phần nhóm thảo luận/thành viên QC điền trên nhánh riêng)* |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét cá nhân

### Nhận xét của Chu Thái Hòa — 02083
- **Vai trò:** Lượt A: Kiểm JSON/cấu hình | Lượt B: Vận hành | Lượt C: Ghi log (theo bảng phân vai `TEAMMATES.md`).
- **Quan sát có bằng chứng từ các lượt:**
  - *Lượt A (Kiểm JSON):* File `run-A/boxes-demo-delta-0-voxel-0.16.json` chỉ có 1 box duy nhất thuộc lớp `vehicles` (score 0.322, mean_z=0.330m), đáy hộp $z_{bottom}=-0.399\text{ m}$ chìm dưới mặt đất $z_{ground}=0.075\text{ m}$. Lượt A miss hầu như toàn bộ vật thể vì $\delta=0$.
  - *Lượt B (Vận hành):* Chạy script với $\delta=1.73$, voxel $0.16\text{ m}$, container hoàn thành sau 5.41s. Output `run-B/boxes-demo-delta-1.73-voxel-0.16.json` tăng vọt lên 13 boxes (10 vehicles, 2 pedestrian, 1 two-wheels, mean_z=1.034m), đáy các hộp nằm khớp sát mặt đất.
  - *Lượt C (Ghi log):* Khi tăng voxel lên $0.32\text{ m}$, file `run-C/boxes-demo-delta-1.73-voxel-0.32.json` chỉ còn 6 boxes và toàn bộ là `pedestrian` (mất sạch 10 xe và 1 xe hai bánh). Ảnh `run-C/side-*.png` cho thấy không còn bất kỳ bounding box xe hơi nào.
- **Diễn giải phép z thuận/ngược:** Áp dụng $z_{model} = z_{source} - z_{ground} - \delta$ và $z_{source} = z_{model} + z_{ground} + \delta$. Ở lượt A, vì $\delta=0$ nên $z_{model} \approx z_{source} - 0.075\text{ m}$, làm đám mây điểm đưa vào mạng cao hơn $1.73\text{ m}$ so với hệ tọa độ chuẩn của KITTI LiDAR. Việc thay đổi $\delta$ trước inference làm thay đổi phân bố dữ liệu đưa vào mạng (input distribution shift), quyết định việc vật thể có được nhận diện hay không, khác hoàn toàn việc tịnh tiến z sau khi đã có output bounding box.
- **Quyết định lỗi batch và hành động:** Với ca `case-batch-z` trong `qc-cases/`, toàn bộ các hộp đều bị lệch cùng một lượng z do lỗi tham số phép chuyển hệ tọa độ trong pipeline. Quyết định: Dừng batch ngay lập tức, báo LC kiểm tra pipeline chuyển đổi; tuyệt đối không sửa thủ công từng hộp trong CVAT.
- **Điều chưa chắc:** Cần nghiên cứu sâu hơn về cơ chế trích xuất đặc trưng của Pillar Feature Net và mạng 2D Backbone của PointPillars để hiểu rõ hơn tại sao kích thước pillar $0.32\text{ m}$ lại làm điểm tin cậy của anchor xe bị rớt xuống dưới threshold 0.3 trong khi lại gom các điểm thưa kích hoạt anchor người đi bộ.

*(Các thành viên khác trong nhóm sẽ tự điền nhận xét của mình trên nhánh làm việc riêng).*

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
