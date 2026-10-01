# 🔍 BÁO CÁO THỰC HIỆN NHIỆM VỤ — CHU THÁI HÒA (ĐỢT A)
**Dự án:** Robotaxi LiDAR 3D Object Detection (Day 13)  
**Nhóm:** K4-DAY13-Nhom01  
**Học viên:** Chu Thái Hòa (MSSV: 02083)  
**Vai trò Lượt A:** 🔍 **Kiểm JSON / Cấu hình** (đọc boxes, class, mean_z)  

---

## 1. Mục tiêu và nhiệm vụ theo phân công (BƯỚC 1 — Lượt A)
Theo [HUONG-DAN-NHOM.md](../HUONG-DAN-NHOM.md):
- **Cấu hình thí nghiệm:** `delta = 0`, `voxel_size = 0.16 m`, `score_thresh = 0.3`, `dataset = KITTI`.
- **Nhiệm vụ:** Mở và kiểm tra cấu hình, nội dung file dự đoán `boxes-demo-delta-0-voxel-0.16.json`, file tổng hợp `summary.csv`, và metadata tại `smoke.json` trong thư mục `run-A/`.

---

## 2. Kết quả kiểm tra chi tiết JSON & Metadata

### 2.1. Cấu hình kiểm tra từ `boxes-demo-delta-0-voxel-0.16.json`
- **Frame ID:** `demo`
- **Dataset:** `KITTI`
- **Tham số sensor offset $\delta$ (`delta`):** `0.0` m
- **Kích thước Voxel (`voxel_size`):** `0.16` m (Pillar XY: 0.16m x 0.16m)
- **Ước lượng mặt đất (`z_ground`):** `0.0750` m (ước lượng tự động từ điểm mặt đường trong scan PCD)
- **Tổng số lượng Bounding Boxes:** `1` hộp

### 2.2. Chi tiết hộp phát hiện duy nhất (Cuboid 3D)
| Trường dữ liệu | Giá trị | Nhận xét của Người kiểm JSON |
|---|---|---|
| **Class (`label`)** | `vehicles` | Nhận diện đúng lớp xe hơi, không phát hiện được người đi bộ (`pedestrian`) hay xe hai bánh (`two-wheels`). |
| **Tâm X (`x`)** | `13.154 m` | Nằm trong tầm nhìn thẳng phía trước cảm biến (~13.15 m). |
| **Tâm Y (`y`)** | `-0.451 m` | Nằm hơi lệch nhẹ về bên phải trục dọc thân xe. |
| **Tâm Z (`z`)** | `0.330 m` | Độ cao tâm hộp là 0.33 m. |
| **Chiều dài (`length`)** | `3.620 m` | Kích thước xe du lịch thông dụng. |
| **Chiều rộng (`width`)** | `1.523 m` | Kích thước hợp lý cho bề ngang ô tô. |
| **Chiều cao (`height`)** | `1.459 m` | Chiều cao xe thực tế. |
| **Góc quay Yaw (`yaw`)** | `2.671 rad` | Tương đương ~153.02° trong mặt phẳng oxy. |
| **Điểm tin cậy (`score`)** | `0.322` | Điểm rất thấp, sát ngưỡng lọc `0.300`. |
| **Đáy hộp ($z_{bottom}$)** | `-0.399 m` | $z_{bottom} = z - \frac{height}{2} = 0.330 - 0.729 = -0.399\text{ m}$. Đáy hộp bị chìm xuống dưới mặt đất cục bộ ($z_{ground} \approx 0.075\text{ m}$). |

### 2.3. Đối chiếu với `summary.csv`
- Nội dung file: `demo,KITTI,0,0.16,1,0.330`
- Khớp hoàn toàn: `n_boxes = 1`, `mean_z = 0.330 m`.

### 2.4. Đối chiếu với `smoke.json` (Provenance & Môi trường)
- **Bước `run-A`:** trạng thái `passed`, thời gian chạy `6.07 s`, `boxes = 1`.
- **Hash prediction:** `02b1ba0ab8c2054852b74221a0ed31053d7447cded56f1e0fe1f784eb0d44880`.
- **Container limits:** 4 CPU cores, 4 GB RAM, kiến trúc `amd64`.

---

## 3. Phân tích nguyên nhân kỹ thuật (Góc nhìn Kiểm cấu hình / JSON)
Trong pipeline mô hình PointPillars KITTI:
$$z_{model} = z_{source} - z_{ground} - \delta$$
$$z_{source} = z_{model} + z_{ground} + \delta$$

- **Khi $\delta = 0$ (Lượt A):**
  Điểm LiDAR đưa vào mạng có cao độ $z_{model} \approx z_{source} - 0.075\text{ m}$.
  Trong khi đó, mô hình PointPillars (pretrained trên KITTI) kỳ vọng cảm biến đặt trên nóc xe cách mặt đường $\delta \approx 1.73\text{ m}$ (mặt đường ở $z \approx -1.73\text{ m}$).
- **Hệ quả phát hiện:**
  Do không bù $\delta = 1.73\text{ m}$, toàn bộ phân bố mây điểm bị dịch lên cao hơn $1.73\text{ m}$ so với không gian mà mô hình đã học.
  Kết quả là mô hình bị lệch phân bố (domain/coordinate shift), bỏ sót (miss) hầu như toàn bộ vật thể trong cảnh, chỉ còn sót lại duy nhất 1 xe với độ tin cậy thấp ($0.322$), và đáy hộp chìm xuống dưới mặt đất.

---

## 4. Bàn giao kết quả cho các thành viên trong nhóm (Handover)

1. **Gửi bạn Kim Nguyên Khôi (📐 Xem hình học):**
   - Đã xác nhận trên JSON chỉ có 1 box duy nhất tại vị trí $x=13.15\text{ m}, y=-0.45\text{ m}, z=0.33\text{ m}$.
   - Khôi đối chiếu ảnh `side-demo-delta-0-voxel-0.16.png` tại vùng $x \in [11, 15]\text{ m}$: hộp đỏ cắm đáy xuống dưới vạch $z=0$ (chìm ~0.4m), xác nhận hình học hoàn toàn khớp với tính toán từ JSON.

2. **Gửi bạn Cầm Vũ Ngọc Thạch (📝 Ghi log):**
   - Cung cấp số liệu chính xác để Thạch điền vào hàng A của bảng `Ba lượt inference thật` trong `PRE-LABEL-REPORT.md`:
     `| A | 0 | 0.16 | 1 | 0.330 | boxes-demo-delta-0-voxel-0.16.json | delta=0 khiến đám mây điểm vào model bị lệch cao độ ~1.73m; model miss hầu hết vật thể, chỉ bắt được 1 xe (score=0.322 sát ngưỡng), đáy chìm dưới đất. |`

3. **Chuẩn bị sẵn cho BƯỚC 5 (Nhận xét cá nhân của Chu Thái Hòa trong báo cáo):**
   - **Vai trò:** Lượt A: Kiểm JSON/cấu hình | Lượt B: Vận hành | Lượt C: Ghi log.
   - **Quan sát Lượt A:** File `boxes-demo-delta-0-voxel-0.16.json` chỉ có 1 box duy nhất (`vehicles`, score=0.322, mean_z=0.330). Đáy hộp tụt xuống $z=-0.399\text{ m}$, chìm dưới mặt đường $z_{ground}=0.075\text{ m}$.
   - **Bản chất phép z:** Đổi $\delta$ trước inference làm thay đổi tọa độ input đi vào PointPillars, làm thay đổi hoàn toàn đặc trưng voxel/pillar và phân bố không gian, dẫn đến số lượng box thay đổi (không phải chỉ tịnh tiến z sau đầu ra).
