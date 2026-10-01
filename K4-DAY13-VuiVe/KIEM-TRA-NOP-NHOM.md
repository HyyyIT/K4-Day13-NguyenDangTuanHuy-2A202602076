# Rà soát bộ nộp nhóm K4-DAY13-VuiVe

Ngày rà soát: 01/10/2026. Căn cứ: [PRE-LABEL.md](../PRE-LABEL.md), [rubric phần nhóm](../RUBRIC.md) và [hướng dẫn gói Student](../bundle/README-STUDENT.md).

## Kết quả

Sau khi chỉnh, phần nội dung báo cáo có đủ các mục được yêu cầu: thành viên/phân vai, provenance, A/B/C, phân tích phép z và pillar, giới hạn ROI/Side, ba ca QC và nhận xét riêng của bốn người. Trạng thái `executed-by-group` ghi theo xác nhận Huy đã chạy A/B/C; không còn mâu thuẫn với mục Huy chỉ đọc `provided-results`.

Việc rà soát này kiểm tra tài liệu và code trong repo. Output thực tế không có trong bản repo hiện tại nên không xác nhận các JSON/PNG/CSV hoặc `smoke.json` đã được đối chiếu trực tiếp. Các số hộp, mean_z, thời gian, image/checkpoint ID trong báo cáo được giữ theo ghi nhận của thành viên; riêng hash PCD đã khớp với `data/demo.pcd`.

| Yêu cầu | Kết quả rà soát |
| --- | --- |
| Thư mục nhóm đúng tên | Đã thống nhất `K4-DAY13-VuiVe/` trong tên thư mục, báo cáo và bảng thành viên. |
| Thành viên/MSSV và đổi vai | Đủ bốn người, phân vai A/B/C trong `TEAMMATES.md`. |
| Mô tả thực hiện trung thực | Đã cập nhật nhóm và Huy thành `executed-by-group` theo xác nhận mới. Không tự ghi LC đã duyệt. |
| PCD/môi trường/checkpoint/cấu hình | Có frame, kiến trúc, image ID, checkpoint hash, repo revision, score, ROI và giả định intensity/z_ground. Hash PCD khớp dữ liệu repo. |
| Ba cấu hình A/B/C | A: delta=0, pillar=0.16; B: delta=1.73, pillar=0.16; C: delta=1.73, pillar=0.32. Đổi đúng một biến cho A/B và B/C. |
| Đọc kết quả | Có số hộp/class/mean_z và tên JSON/Side/CSV cho từng lượt; không kết luận cấu hình tốt hơn chỉ bằng số hộp. |
| Phép z và hệ nguồn | Đã sửa chiều dịch input A/B. JSON đã ở hệ nguồn, không cộng delta/z_ground thêm lần nữa. |
| ROI/Side và khả năng import | Đã nêu giới hạn hình chiếu x-z, cần nhiều view/camera; không import KITTI demo vào Robotaxi. |
| Ba ca QC | Có 0/13, 13/13 và 1/13 hộp lệch; delta+z_ground=1.805 m; phân biệt dừng batch với kiểm riêng một hộp; case-correct không phải đáp án. |
| Nhận xét cá nhân | Giữ nhận xét riêng của Khôi, Hòa, Thạch và Huy, gồm vai trò, quan sát, phép z, quyết định QC và điều chưa chắc. |
| File gốc kèm bộ nộp | Nhóm đã chạy theo xác nhận; ghép các output gốc vào thư mục nộp riêng cho LC. Không tạo file giả để lấp chỗ trống. |
| LC thu/duyệt | LC ghi nhận thủ công tại nơi thu bài được cấp. Push GitHub không thay cho bước nộp và xác nhận này. |

## Cấu trúc bộ nộp để gửi LC

```text
K4-DAY13-VuiVe/
├── TEAMMATES.md
├── PRE-LABEL-REPORT.md
├── KIEM-TRA-NOP-NHOM.md
└── output/
    ├── smoke.json
    ├── run-A/
    │   ├── boxes-demo-delta-0-voxel-0.16.json
    │   ├── side-demo-delta-0-voxel-0.16.png
    │   └── summary.csv
    ├── run-B/
    │   ├── boxes-demo-delta-1.73-voxel-0.16.json
    │   ├── side-demo-delta-1.73-voxel-0.16.png
    │   └── summary.csv
    ├── run-C/
    │   ├── boxes-demo-delta-1.73-voxel-0.32.json
    │   ├── side-demo-delta-1.73-voxel-0.32.png
    │   └── summary.csv
    └── qc-cases/
        ├── manifest.json
        ├── case-correct.json
        ├── side-correct.png
        ├── case-batch-z.json
        ├── side-batch-z.png
        ├── case-one-box-z.json
        └── side-one-box-z.png
```

Đây là cấu trúc ghép bộ nộp, không phải danh sách file hiện có trong Git. `smoke.json` cần có `status: passed`; manifest ca QC phải trỏ đúng prediction B. Giữ output gốc tại nơi thu riêng của LC theo hướng dẫn bài học.

## Các điểm đã sửa

- Đổi tên thư mục và mã nhóm; bỏ đường dẫn tới thư mục `ket-qua-nhom-01` đã bị xóa trên main.
- Đưa báo cáo tổng hợp và nhận xét đủ bốn người vào thư mục nộp; file report ở gốc repo dẫn tới bản này.
- Điền mục JSON chưa đủ cơ sở để import, thay các ghi chú `[nhóm trả lời]` bằng nội dung cụ thể.
- Sửa diễn giải input A/B và mô phỏng batch-z dịch xuống; phân biệt index 0 với ID QC.
- Giữ giới hạn bằng chứng: Side và số hộp không xác nhận cuboid đúng; nguyên nhân mất prediction vehicles khi đổi pillar chưa được chứng minh.
