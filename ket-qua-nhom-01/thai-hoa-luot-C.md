# 📝 BIÊN BẢN GHI LOG — CHU THÁI HÒA (LƯỢT C)
**Dự án:** Robotaxi LiDAR 3D Object Detection (Day 13)  
**Nhóm:** K4-DAY13-Nhom01  
**Học viên:** Chu Thái Hòa (MSSV: 02083)  
**Vai trò Lượt C:** 📝 **Ghi log** (ghi vào báo cáo, tổng hợp kết quả)  

---

## 1. Nhiệm vụ ghi log theo phân công (BƯỚC 3 — Lượt C)
Theo [HUONG-DAN-NHOM.md](../HUONG-DAN-NHOM.md):
- **Cấu hình thí nghiệm:** `delta = 1.73`, `voxel_size = 0.32 m`, `score_thresh = 0.3`, `dataset = KITTI`.
- **Người vận hành:** Kim Nguyên Khôi.
- **Người kiểm JSON:** Cầm Vũ Ngọc Thạch.
- **Người xem hình học:** Nguyễn Đăng Tuấn Huy.
- **Nhiệm vụ của Thái Hòa:** Điền dòng C trong bảng `Ba lượt inference thật` và phân tích so sánh B↔C vào báo cáo `PRE-LABEL-REPORT.md`.

---

## 2. Số liệu Lượt C thu thập từ các thành viên
- **File đầu ra:**
  - `run-C/boxes-demo-delta-1.73-voxel-0.32.json` (1,582 bytes, hash SHA-256: `8eb5011ad94d91c15937209c0a4c862d4921ca464c732b42d37c12f2e7e7d6ec`)
  - `run-C/side-demo-delta-1.73-voxel-0.32.png`
  - `run-C/summary.csv` (`demo,KITTI,1.73,0.32,6,1.091`)
- **Tổng số lượng boxes:** `6` hộp (giảm 7 hộp so với lượt B).
- **Phân bố lớp (Classes):** `pedestrian = 6` (mất sạch toàn bộ 10 `vehicles` và 1 `two-wheels` của lượt B!).
- **Giá trị trung bình cao độ tâm:** `mean_z = 1.091 m` (tăng nhẹ từ 1.034m ở lượt B).

---

## 3. Dòng điền bảng báo cáo Lượt C
```markdown
| C | 1.73 | 0.32 | 6 | 1.091 | ket-qua-nhom-01/run-C/boxes-demo-delta-1.73-voxel-0.32.json, ket-qua-nhom-01/run-C/summary.csv | Thái Hòa ghi log: Khi tăng voxel từ 0.16m lên 0.32m (pillar to hơn), số hộp giảm từ 13 xuống 6, mean_z tăng từ 1.034m lên 1.091m. Đặc biệt, mất toàn bộ 10 xe (vehicles) và 1 xe hai bánh (two-wheels), chỉ còn phát hiện 6 người đi bộ (pedestrian). Voxel to làm mờ ranh giới mây điểm của xe cộ, khiến anchor xe không đạt ngưỡng. |
```

---

## 4. Phân tích so sánh B↔C (Trả lời câu hỏi trong báo cáo)
**Câu hỏi:** *Thấy gì khi đổi pillar? Có đủ bằng chứng để nói cấu hình nào tốt hơn không? Tại sao pillar to hơn lại làm mất xe nhưng giữ người đi bộ?*

**Trả lời:**
1. **Hiện tượng khi tăng kích thước Pillar XY (từ 0.16m lên 0.32m):**
   - Diện tích đáy mỗi pillar tăng gấp 4 lần ($0.32 \times 0.32 = 0.1024\text{ m}^2$ so với $0.16 \times 0.16 = 0.0256\text{ m}^2$).
   - Số lượng hộp phát hiện giảm từ 13 xuống 6 hộp.
   - Cơ cấu lớp thay đổi đảo lộn: Lượt B có `vehicles = 10`, `pedestrian = 2`, `two-wheels = 1`. Lượt C chỉ còn duy nhất `pedestrian = 6`, mất hoàn toàn 10 xe hơi và 1 xe hai bánh.

2. **Cơ chế tại sao pillar to lại làm mất xe nhưng bắt người đi bộ:**
   - **Với xe hơi (`vehicles`):** Ô tô là vật thể lớn, đặc trưng nhận diện của PointPillars dựa vào các đường viền mép phản xạ sắc nét từ bề mặt kim loại phẳng và hình khối lăng trụ rộng. Khi voxel to ($0.32\text{ m}$), các điểm thuộc bề mặt vỏ xe và mặt đường lân cận bị gộp chung vào một pillar, làm suy giảm độ phân giải không gian trên bản đồ đặc trưng BEV (Bird's Eye View). Phép pooling trong Pillar Feature Net làm nhòe biên dạng hình học của xe, khiến anchor box lớp vehicle không đạt điểm tin cậy cần thiết (bị rớt xuống dưới threshold 0.3).
   - **Với người đi bộ (`pedestrian`):** Người đi bộ là đối tượng có kích thước nhỏ, phản xạ LiDAR là cụm điểm dọc hẹp. Khi dùng pillar to, một số cụm điểm nhiễu hoặc phần mép thưa của xe bị gom lại trong một pillar và vô tình tạo ra đặc trưng hình trụ đứng có kích thước tương đồng với anchor box người đi bộ, dẫn đến việc kích hoạt phát hiện giả (false positive) thành người đi bộ.

3. **Có đủ bằng chứng để nói cấu hình nào tốt hơn không?**
   - **KHÔNG thể khẳng định tuyệt đối** cấu hình nào tốt hơn chỉ dựa trên số lượng hộp hay điểm score.
   - Tuy nhiên, cấu hình lượt B ($0.16\text{ m}$) cho thấy tính cân bằng vượt trội vì phát hiện được đa dạng các lớp (xe hơi, người, xe hai bánh), các hộp xe khớp sát hình học và mặt đất.
   - Cấu hình lượt C ($0.32\text{ m}$) làm mất hẳn lớp đối tượng giao thông quan trọng nhất (ô tô), cho thấy cỡ pillar $0.32\text{ m}$ là quá thô đối với cảm biến KITTI ở cự ly gần và trung bình.
