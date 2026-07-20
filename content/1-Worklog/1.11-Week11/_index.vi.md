---
title: "Worklog Tuần 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

* Kiểm thử toàn bộ hệ thống Netflop sau khi tích hợp AWS.
* Kiểm thử luồng tự động MediaConvert, EventBridge, Lambda và phụ đề.
* Bổ sung giám sát, ghi log, cảnh báo và kiểm soát chi phí.
* Hoàn thiện chức năng chính trước giai đoạn tổng kết.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Kiểm thử luồng người dùng: xem danh sách phim, tìm kiếm, xem chi tiết, phát HLS stream <br> - Ghi nhận lỗi giao diện và API | 29/06/2026 | 29/06/2026 | Test case dự án |
| 3 | - Kiểm thử luồng quản trị: upload phim, tạo MediaConvert job, cập nhật poster/phụ đề <br> - Kiểm tra dữ liệu lưu vào RDS và S3 | 30/06/2026 | 30/06/2026 | RDS, S3, MediaConvert |
| 4 | - Kiểm thử EventBridge gọi Lambda khi MediaConvert COMPLETE/ERROR/CANCELED <br> - Kiểm tra webhook `/api/uploads/mediaconvert/events` | 01/07/2026 | 01/07/2026 | EventBridge, Lambda |
| 5 | - Kiểm thử upload phụ đề `.srt` và Lambda convert sang `.vtt` <br> - Rà soát Security Group, IAM policy, bucket permission | 02/07/2026 | 02/07/2026 | Lambda, IAM |
| 6 | - Kiểm tra CloudWatch alarms, SNS topic `netflop-alerts`, Billing Dashboard và AWS Budgets <br> - Tổng hợp lỗi đã sửa và phần còn hạn chế | 03/07/2026 | 03/07/2026 | CloudWatch, SNS |

### Kết quả đạt được tuần 11:

* Hoàn thiện các chức năng chính của website Netflop ở mức demo thực tập.
* Kiểm thử được luồng người dùng và luồng quản trị.
* Kiểm thử được pipeline upload video, MediaConvert, CloudFront HLS và cập nhật trạng thái qua Lambda/EventBridge.
* Biết cách theo dõi log, metric và cảnh báo bằng CloudWatch/SNS.
* Rà soát được các điểm quan trọng về bảo mật như Security Group, IAM policy, S3 bucket permission và RDS public access.
* Dọn dẹp tài nguyên không cần thiết và kiểm soát tốt hơn chi phí trong tài khoản AWS.
