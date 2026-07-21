---
title: "Kiểm tra chi phí sau cleanup"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.6.3. </b> "
---

#### Mục tiêu

Sau khi cleanup, cần kiểm tra Billing Dashboard để chắc rằng không còn tài nguyên phát sinh chi phí ngoài ý muốn. Với Netflop, các nhóm chi phí quan trọng nhất là EC2, RDS, S3, MediaConvert, CloudFront, CloudWatch và data transfer.

#### Các mục cần kiểm tra

| Dịch vụ | Điều cần kiểm tra |
| --- | --- |
| EC2 | Instance running, EBS volume, Elastic IP chưa gắn |
| RDS | DB instance, snapshot, backup retention |
| S3 | Dung lượng video input/output, số request |
| MediaConvert | Số phút xử lý video, job lỗi/retry |
| CloudFront | Data transfer và request |
| Lambda | Số lần invoke và duration |
| CloudWatch | Log storage, custom metrics, alarms |
| SNS | Số lượng notification |

#### Chi phí cần lưu ý khi upload video

Một phim gốc 5GB sau khi convert ra nhiều độ phân giải có thể tạo output lớn hơn file ban đầu. Ngoài chi phí lưu trữ S3 còn có:

* Phí MediaConvert theo thời lượng video và cấu hình output.
* Phí CloudFront data transfer khi người dùng xem.
* Phí S3 request và lưu trữ HLS segment.
* Phí CloudWatch Logs nếu log nhiều.

#### Các bước thực hiện

1. Vào AWS Billing Dashboard.
2. Kiểm tra chi phí theo service.
3. Vào Cost Explorer để xem chi tiết theo ngày.
4. Kiểm tra AWS Budgets nếu đã cấu hình.
5. Ghi lại các service còn phát sinh chi phí sau cleanup.

#### Kết quả mong đợi

* Không còn EC2/RDS chạy ngoài nhu cầu.
* Không còn S3 bucket demo dung lượng lớn.
* Không còn CloudFront/Lambda/EventBridge không dùng.
* Báo cáo có ảnh chứng minh đã kiểm tra chi phí.

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/budget.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách kiểm tra chi phí

Vào AWS Billing -> Cost Explorer và lọc theo service. Các service cần chú ý:

* EC2: phí instance, EBS, data transfer.
* RDS: phí database instance, storage, backup.
* S3: storage video gốc, HLS output và request.
* MediaConvert: phí theo thời lượng convert.
* CloudFront: data transfer khi người dùng xem phim.
* CloudWatch: log storage, metrics và alarm.

#### Lệnh CLI tham khảo

~~~bash
aws ce get-cost-and-usage \
  --time-period Start=2026-07-01,End=2026-07-31 \
  --granularity MONTHLY \
  --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
~~~

#### Cách ghi vào báo cáo

Nên ghi chi phí theo nhóm thay vì chỉ ghi tổng tiền. Ví dụ:

| Nhóm chi phí | Nguyên nhân |
| --- | --- |
| S3 | Lưu video gốc và HLS output |
| MediaConvert | Convert video sang nhiều độ phân giải |
| CloudFront | Người dùng tải HLS segment |
| RDS | Database chạy liên tục |
| EC2 | Server chạy frontend/backend |

Nếu upload nhiều phim 5GB, chi phí tăng chủ yếu ở S3, MediaConvert và CloudFront data transfer.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Kiểm tra chi phí sau workshop

Các dịch vụ có thể phát sinh chi phí chính:

* EC2 chạy liên tục.
* RDS chạy liên tục và storage backup.
* S3 lưu video gốc/HLS output.
* MediaConvert tính phí theo thời lượng video convert.
* CloudFront tính phí data transfer.
* CloudWatch tính phí log/metric/alarm.

#### Lệnh xem chi phí theo service

~~~bash
aws ce get-cost-and-usage \
  --time-period Start=2026-07-01,End=2026-07-31 \
  --granularity MONTHLY \
  --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
~~~

#### Cách ước tính khi upload phim lớn

Khi upload 5 phim, mỗi phim 5GB, chi phí tăng ở 3 phần:

1. S3 input: lưu khoảng 25GB video gốc.
2. S3 output: HLS nhiều bitrate có thể gần bằng hoặc lớn hơn dung lượng gốc tùy cấu hình.
3. MediaConvert: tính theo thời lượng nội dung và độ phân giải output.
4. CloudFront: tính theo lượng người xem tải segment.

#### Bảng trình bày trong báo cáo

| Nhóm chi phí | Nguyên nhân | Cách tối ưu |
| --- | --- | --- |
| S3 | Lưu video gốc và HLS | Xóa file test, lifecycle |
| MediaConvert | Convert nhiều chất lượng | Chỉ xuất quality cần thiết |
| CloudFront | Người dùng xem video | Cache đúng, dùng CDN thay vì EC2 |
| RDS | DB chạy 24/7 | Chọn instance nhỏ giai đoạn demo |
| EC2 | Server chạy 24/7 | Stop khi không dùng demo |
<!-- NETFLOP_IMPLEMENTATION_END -->
