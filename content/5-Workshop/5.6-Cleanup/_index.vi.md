---
title: "Dọn dẹp tài nguyên"
date: 2026-07-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Phần cleanup dùng để kiểm tra và dọn các tài nguyên không còn sử dụng sau khi hoàn thành workshop hoặc demo. Với website xem phim, cần đặc biệt chú ý S3 và MediaConvert vì video dung lượng lớn có thể tạo chi phí lưu trữ và xử lý đáng kể.
![budget](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/budget.png)
![budget](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/cost.png)


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

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Mục tiêu cleanup

Cleanup dùng để giảm chi phí sau khi demo hoặc khi không còn sử dụng môi trường. Không nên xóa tài nguyên khi chưa backup dữ liệu quan trọng.

#### Nguyên tắc dọn dẹp

1. Backup database trước khi xóa RDS.
2. Export hoặc giữ lại video quan trọng trước khi xóa S3.
3. Tắt EC2 nếu chỉ nghỉ tạm thời, terminate nếu không dùng nữa.
4. Xóa CloudFront sau cùng vì cần disable trước khi delete.
5. Kiểm tra Cost Explorer sau khi cleanup.

#### Thứ tự đề xuất

~~~text
Backup RDS
-> Stop PM2/Nginx nếu cần
-> Empty S3 test prefixes
-> Disable CloudFront test distribution
-> Delete Lambda/EventBridge test rules
-> Stop/terminate EC2
-> Delete RDS snapshot hoặc DB nếu không dùng
~~~
<!-- NETFLOP_IMPLEMENTATION_END -->
