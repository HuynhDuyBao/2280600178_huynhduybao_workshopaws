---
title: "Bản đề xuất"
date: 2026-04-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Website xem phim Netflop trên AWS
## Kiến trúc triển khai web xem phim với EC2, RDS, S3, MediaConvert, CloudFront và Lambda

### 1. Tóm tắt điều hành

Dự án **Netflop** là website xem phim được triển khai trên AWS. Hệ thống cho phép người dùng xem danh sách phim, tìm kiếm phim, xem chi tiết phim, phát video dạng HLS và xem phụ đề. Quản trị viên có thể upload phim, upload phụ đề, quản lý tập phim, poster, avatar và dữ liệu phim.

Kiến trúc hiện tại sử dụng **Cloudflare** cho domain `netflop.win`, **Amazon EC2** chạy Nginx, React frontend và Node.js API, **Amazon RDS MySQL** lưu database chính, **Amazon S3** lưu file video/ảnh/phụ đề, **AWS Elemental MediaConvert** chuyển video MP4/MKV sang HLS, **Amazon CloudFront** phân phối video, **Lambda** và **EventBridge** tự động cập nhật trạng thái xử lý video/phụ đề, đồng thời dùng **CloudWatch** và **SNS** để giám sát, cảnh báo.

Mục tiêu của dự án là xây dựng một hệ thống xem phim có kiến trúc gần với thực tế, giúp sinh viên hiểu cách triển khai ứng dụng web, xử lý media, phân phối nội dung và giám sát hệ thống trên AWS.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Website xem phim không chỉ cần lưu dữ liệu phim mà còn cần xử lý nhiều loại media như video gốc, HLS output, poster, avatar và phụ đề. Nếu chỉ lưu toàn bộ file trên một máy chủ EC2, hệ thống dễ gặp các vấn đề:

* Dung lượng ổ đĩa EC2 nhanh đầy khi video tăng.
* Video lớn khó phát ổn định cho nhiều người dùng.
* Quá trình convert video thủ công mất thời gian.
* Cần cơ chế tự động cập nhật trạng thái tập phim sau khi convert.
* Cần bảo vệ stream video thay vì public toàn bộ file.
* Cần giám sát EC2, RDS, Lambda và cảnh báo khi có lỗi.

#### Giải pháp đề xuất

Netflop được triển khai theo mô hình tách riêng ứng dụng, database, lưu trữ media, xử lý video và phân phối nội dung:

* **EC2/Nginx** chạy React frontend và Node.js backend API.
* **RDS MySQL** lưu dữ liệu chính của hệ thống.
* **S3 input bucket** lưu video/phụ đề gốc được admin upload.
* **MediaConvert** chuyển video MP4/MKV sang HLS nhiều chất lượng.
* **S3 output bucket** lưu HLS output, ảnh public, avatar và phụ đề VTT.
* **CloudFront** phát HLS stream và hỗ trợ bảo vệ stream bằng signed cookies.
* **EventBridge + Lambda** nhận event MediaConvert và gọi webhook backend để cập nhật trạng thái tập phim.
* **Lambda subtitle converter** chuyển phụ đề `.srt` sang `.vtt`.
* **CloudWatch + SNS** giám sát và gửi cảnh báo.

#### Lợi ích kỳ vọng

* Tách video/media khỏi EC2, giảm tải cho máy chủ ứng dụng.
* Có pipeline xử lý video tự động từ upload đến HLS.
* Phát video ổn định hơn thông qua CloudFront.
* Database được quản lý bằng RDS thay vì tự cài đặt thủ công.
* Có log, alarm và topic cảnh báo để theo dõi hệ thống.
* Có nền tảng để mở rộng thêm bảo mật, CI/CD, Auto Scaling hoặc CDN nâng cao.

### 3. Kiến trúc giải pháp

```text
User / Admin
   |
   v
Cloudflare DNS + HTTPS
   |
   v
netflop.win
   |
   v
EC2 netflop-web
   |-- Nginx reverse proxy
   |-- React frontend
   |-- Node.js backend API
   |
   |---- RDS MySQL netflop-db
   |
   |---- S3 netflop-input-source
   |          |
   |          v
   |      MediaConvert
   |          |
   |          v
   |---- S3 netflop-output-source
              |
              v
        CloudFront protected HLS
              |
              v
        Video Player
```

Luồng tự động cập nhật trạng thái video:

```text
MediaConvert Job State Change
   |
   v
EventBridge
   |
   v
Lambda netflop-mediaconvert-notifier
   |
   v
Backend webhook /api/uploads/mediaconvert/events
   |
   v
Update episode status in RDS
```

Luồng xử lý phụ đề:

```text
Admin upload .srt
   |
   v
S3 input subtitle-input/
   |
   v
Lambda netflop-subtitle-converter
   |
   v
S3 output subtitles/*.vtt
   |
   v
Video Player subtitles
```

#### Dịch vụ AWS sử dụng

| Dịch vụ | Vai trò trong hệ thống |
| --- | --- |
| Amazon EC2 | Host Node.js backend, React frontend build và Nginx reverse proxy |
| Amazon RDS MySQL | Database chính `web_xem_phim_final` |
| Amazon S3 | Lưu video input, HLS output, ảnh, avatar và phụ đề |
| AWS Elemental MediaConvert | Convert video MP4/MKV sang HLS 360p/480p/720p/1080p |
| Amazon CloudFront | Phát HLS stream và bảo vệ stream bằng signed cookies |
| AWS Lambda | Xử lý event MediaConvert và convert phụ đề SRT sang VTT |
| Amazon EventBridge | Bắt sự kiện MediaConvert COMPLETE/ERROR/CANCELED |
| Amazon CloudWatch | Monitoring EC2, RDS, Lambda |
| Amazon SNS | Topic cảnh báo `netflop-alerts` |
| IAM Role | Cấp quyền cho EC2, MediaConvert và Lambda |
| AWS Systems Manager | Hỗ trợ deploy/reload EC2 từ local AWS CLI |
| Amazon Cognito | Có cấu hình xác thực, cần xác nhận lại account/region |

### 4. Triển khai kỹ thuật

#### Tài nguyên chính

* **EC2**: `netflop-web`, instance type `t3.micro`, Security Group `netflop-ec2-sg`.
* **RDS**: `netflop-db`, MySQL, `db.t4g.micro`, database `web_xem_phim_final`, port `3306`.
* **S3 input**: `netflop-input-source`, lưu file upload gốc, video input và subtitle input.
* **S3 output**: `netflop-output-source`, lưu HLS output, ảnh public, avatar và phụ đề VTT.
* **CloudFront**: distribution chính `d11jdx7259stge.cloudfront.net` dùng phát HLS stream.
* **Lambda**:
  * `netflop-mediaconvert-notifier`
  * `netflop-subtitle-converter`
* **EventBridge**: rule `netflop-mediaconvert-job-state-change`.
* **SNS**: topic `netflop-alerts`.

#### Chức năng chính

* **Admin upload phim**: backend nhận file, upload lên S3 input, tạo MediaConvert job và lưu `jobId`, `hls_url`, `cloudfront_url`, `upload_status`.
* **Convert video**: MediaConvert tạo HLS nhiều chất lượng gồm 360p, 480p, 720p và 1080p.
* **Tự động cập nhật trạng thái**: EventBridge gọi Lambda khi job đổi trạng thái, Lambda gọi webhook backend để cập nhật tập phim.
* **Phát video**: frontend lấy thông tin phim/tập từ backend, player phát HLS qua CloudFront; stream bảo vệ dùng signed cookies qua `/api/stream/session`.
* **Upload phụ đề**: `.vtt` lưu trực tiếp vào S3 output; `.srt` được Lambda convert sang `.vtt`.
* **Upload ảnh/avatar**: file lưu trên S3, backend lưu URL vào database.
* **Monitoring**: CloudWatch theo dõi EC2, RDS, Lambda; SNS gửi cảnh báo khi có alarm.

### 5. Lộ trình và mốc triển khai

| Giai đoạn | Thời gian | Nội dung chính |
| --- | --- | --- |
| Giai đoạn 1 | Tuần 1-2 | Học AWS nền tảng, IAM, EC2, AWS CLI và bảo mật tài khoản |
| Giai đoạn 2 | Tuần 3-4 | Thực hành VPC, EC2, S3 và triển khai app cơ bản |
| Giai đoạn 3 | Tuần 5-6 | Tìm hiểu RDS, CloudWatch, Budgets và chuẩn bị database |
| Giai đoạn 4 | Tuần 7-8 | Phân tích Netflop, thiết kế frontend/backend và kiến trúc AWS |
| Giai đoạn 5 | Tuần 9-10 | Tích hợp RDS, S3, upload media và backend API |
| Giai đoạn 6 | Tuần 11-12 | Tích hợp MediaConvert, CloudFront, Lambda/EventBridge, kiểm thử và hoàn thiện báo cáo |

### 6. Ước tính ngân sách

Ước tính dưới đây áp dụng cho cấu hình demo nhỏ, **không dùng Free Tier/credit**, chạy tại region **Asia Pacific (Singapore) - ap-southeast-1**.

| Dịch vụ | Cấu hình dự kiến | Giá ước tính/tháng |
| --- | --- | --- |
| Amazon EC2 | 01 t3.micro Linux chạy Netflop | 10 USD |
| Amazon EBS | 20 GB gp3 cho EC2 | 2 USD |
| Amazon RDS MySQL | 01 db.t4g.micro Single-AZ | 18 USD |
| RDS Storage/Backup | 20 GB database và backup nhỏ | 3 USD |
| Amazon S3 | 2 bucket input/output, khoảng 20-30 GB media | 2 USD |
| AWS Elemental MediaConvert | Convert video demo sang HLS nhiều chất lượng | 5 USD |
| Amazon CloudFront | Phát HLS stream mức demo | 8 USD |
| Data Transfer Out | Khoảng 100 GB/tháng | 12 USD |
| AWS Lambda | 2 Lambda xử lý event/phụ đề | 1 USD |
| EventBridge + SNS | Event và cảnh báo mức demo | 1 USD |
| Amazon CloudWatch | Log, metric và alarm cơ bản | 3 USD |
| AWS Systems Manager | Deploy/reload EC2 mức cơ bản | 0 USD |

**Tổng chi phí ước tính:** khoảng **65 USD/tháng**.  
**Tổng chi phí cho 3 tháng thực tập:** khoảng **195 USD**.

### 7. Đánh giá rủi ro

| Rủi ro | Mức ảnh hưởng | Xác suất | Hướng xử lý |
| --- | --- | --- | --- |
| Vượt chi phí do video/data transfer | Cao | Trung bình | Dùng video mẫu dung lượng nhỏ, theo dõi Billing và Budgets |
| RDS public accessible | Cao | Trung bình | Hướng hoàn thiện là giới hạn Security Group chỉ cho EC2 truy cập |
| S3 bucket/object public sai | Cao | Thấp | Bật Block Public Access, cấp quyền qua IAM/CloudFront |
| MediaConvert job lỗi | Trung bình | Trung bình | Theo dõi job status, EventBridge và CloudWatch logs |
| Lambda notifier/webhook lỗi | Trung bình | Trung bình | Ghi log Lambda, kiểm tra endpoint `/api/uploads/mediaconvert/events` |
| CloudFront signed cookies cấu hình sai | Trung bình | Trung bình | Kiểm thử `/api/stream/session` và quyền truy cập stream |
| Cognito khác account/region | Thấp | Trung bình | Ghi chú là đang cấu hình và cần xác nhận lại tài khoản Cognito |

### 8. Kết quả kỳ vọng

* Website Netflop chạy được qua domain `netflop.win`.
* EC2 vận hành React frontend, Node.js API và Nginx reverse proxy.
* RDS MySQL lưu dữ liệu phim, tập phim, tài khoản, bình luận, lịch sử xem và yêu thích.
* Admin upload video lên S3 input và hệ thống tự tạo MediaConvert job.
* Video được convert sang HLS và phát qua CloudFront.
* EventBridge/Lambda tự động cập nhật trạng thái tập phim sau khi MediaConvert hoàn tất.
* Phụ đề `.srt` được Lambda chuyển sang `.vtt`.
* CloudWatch và SNS hỗ trợ giám sát, cảnh báo cho EC2, RDS và Lambda.
