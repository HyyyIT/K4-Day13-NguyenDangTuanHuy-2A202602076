# 🖥️ NHẬT KÝ VẬN HÀNH — CHU THÁI HÒA (LƯỢT B)
**Dự án:** Robotaxi LiDAR 3D Object Detection (Day 13)  
**Nhóm:** K4-DAY13-Nhom01  
**Học viên:** Chu Thái Hòa (MSSV: 02083)  
**Vai trò Lượt B:** 🖥️ **Vận hành** (gõ lệnh, chạy script)  

---

## 1. Cấu hình thực thi Lượt B
- **Đầu vào:** `demo.pcd` (mẫu KITTI đã chuyển đổi, CC BY-NC-SA 3.0)
- **Image Docker:** `day13-pointpillars:lc-20261001-amd64` (Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`)
- **Tham số thực thi:**
  - Sensor offset $\delta$: `1.73` m (giả định chuẩn của sensor KITTI)
  - Kích thước Voxel/Pillar: `0.16` m
  - Score threshold: `0.30`
  - ROI / Detection mode: `--from KITTI` (front-window)

## 2. Lệnh thực thi Container
```powershell
docker run --rm --network none --cpus 4 --memory 4g `
  --mount "type=bind,source=$DATA_DIR,target=/data,readonly" `
  --mount "type=bind,source=$OUT_DIR,target=/out" `
  "$IMAGE" --data /data/demo.pcd --out /out/run-B --from KITTI --deltas 1.73 --voxel-size 0.16 --score-thresh 0.3
```

## 3. Nhật ký kiểm tra runtime & Output
- **Thời gian chạy:** Bắt đầu lúc `2026-10-01T08:07:52.462Z`, hoàn thành lúc `2026-10-01T08:07:57.875Z` (tổng thời gian: `5.41 s`).
- **Trạng thái:** `passed`
- **Tài nguyên sử dụng:** 4 CPUs, 4GB RAM, kiến trúc `amd64`.
- **Output sinh ra đầy đủ tại thư mục `ket-qua-nhom-01/run-B/`:**
  1. `boxes-demo-delta-1.73-voxel-0.16.json` (kích thước 3,270 bytes, hash SHA-256: `c2a8db247353f0ef00299ff50acff997b7b4a87bdcfcefe792815acb5652cc80`)
  2. `side-demo-delta-1.73-voxel-0.16.png` (ảnh chiếu cạnh Side view)
  3. `summary.csv` (`demo,KITTI,1.73,0.16,13,1.034`)

## 4. Bàn giao kết quả Lượt B cho đồng đội
- **Chuyển giao cho Kim Nguyên Khôi (🔍 Kiểm JSON/Cấu hình):**
  - Xác nhận output Lượt B sinh ra 13 boxes, gồm 10 xe (`vehicles`), 2 người đi bộ (`pedestrian`), 1 xe hai bánh (`two-wheels`).
  - Giá trị `mean_z = 1.034 m` (tăng ~0.704m so với Lượt A).
- **Chuyển giao cho Cầm Vũ Ngọc Thạch (📐 Xem hình học):**
  - Đã có ảnh `side-demo-delta-1.73-voxel-0.16.png`. Các hộp có đáy nằm sát mặt đường $z \approx 0$ thay vì bị chìm như Lượt A.
- **Chuyển giao cho Nguyễn Đăng Tuấn Huy (📝 Ghi log):**
  - Cung cấp số liệu để Tuấn Huy điền dòng B và so sánh A↔B trong báo cáo.
