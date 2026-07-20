---
title: "Dọn dẹp tài nguyên"
date: 2026-07-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Phần cleanup dùng để kiểm tra và dọn các tài nguyên không còn sử dụng sau khi hoàn thành workshop hoặc demo. Với website xem phim, cần đặc biệt chú ý S3 và MediaConvert vì video dung lượng lớn có thể tạo chi phí lưu trữ và xử lý đáng kể.

{{% notice info %}}
Cần thêm ảnh: danh sách tài nguyên AWS cần giữ/xóa, S3 storage usage, CloudFront distribution, RDS snapshot và Billing Dashboard sau khi cleanup.
{{% /notice %}}

#### Nội dung

1. [Dọn dẹp tài nguyên ứng dụng](5.6.1-application-resources/)
2. [Dọn dẹp tài nguyên media và automation](5.6.2-media-resources/)
3. [Kiểm tra chi phí sau cleanup](5.6.3-cost-check/)

<!-- NETFLOP_DETAIL_START -->
#### Nguyên tắc cleanup

Không xóa tài nguyên production khi website còn chạy. Cleanup trong báo cáo nên chia thành hai loại:

1. Tài nguyên demo/test có thể xóa ngay.
2. Tài nguyên production chỉ kiểm tra, backup và tối ưu chi phí.

Với Netflop, tài nguyên cần cẩn thận nhất là RDS và S3 output vì chúng chứa dữ liệu chính và media đang được website sử dụng.
<!-- NETFLOP_DETAIL_END -->
