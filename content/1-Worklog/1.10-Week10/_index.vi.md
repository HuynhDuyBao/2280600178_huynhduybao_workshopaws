---
title: "Worklog Tuần 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Tích hợp Amazon S3 vào website Netflop để lưu trữ ảnh, poster, video input và HLS output.
* Tìm hiểu luồng xử lý video bằng MediaConvert và phát video qua CloudFront.
* Kiểm tra luồng upload media từ trang quản trị.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Kiểm tra bucket `netflop-input-source` và `netflop-output-source` <br> - Ghi chú vai trò của từng bucket trong luồng upload video | 22/06/2026 | 22/06/2026 | S3 |
| 3 | - Cấu hình IAM role/policy cho EC2, S3 và MediaConvert <br> - Hạn chế quyền theo đúng chức năng cần dùng | 23/06/2026 | 23/06/2026 | IAM, S3 Policy |
| 4 | - Kiểm thử upload poster, thumbnail và video từ trang quản trị <br> - Lưu URL hoặc object key vào database | 24/06/2026 | 24/06/2026 | Backend, S3 |
| 5 | - Tìm hiểu MediaConvert job convert MP4/MKV sang HLS <br> - Kiểm tra output `.m3u8` và segment trong S3 output | 25/06/2026 | 25/06/2026 | MediaConvert |
| 6 | - Kiểm tra CloudFront distribution phát HLS stream <br> - Ghi chú cơ chế signed cookies qua API `/api/stream/session` | 26/06/2026 | 26/06/2026 | CloudFront |

### Kết quả đạt được tuần 10:

* Tích hợp được S3 input/output vào luồng quản lý tài nguyên của website Netflop.
* Upload và hiển thị được poster, thumbnail hoặc file media mẫu.
* Hiểu cách lưu object key/URL vào database để backend và frontend cùng sử dụng.
* Hiểu vai trò của MediaConvert trong việc chuyển video sang HLS nhiều chất lượng.
* Nắm được vai trò của CloudFront trong việc phát HLS stream và bảo vệ video bằng signed cookies.
