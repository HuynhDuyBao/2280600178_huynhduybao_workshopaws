---
title: "Workshop"
date: 2026-07-10
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai website xem phim Netflop trên AWS

Phần workshop mô tả quá trình triển khai website xem phim **Netflop** lên môi trường AWS. Nội dung được viết theo luồng thực tế của hệ thống hiện tại: người dùng truy cập domain `netflop.win`, frontend/backend chạy trên EC2, dữ liệu lưu trong RDS MySQL, media được lưu trên S3, video được chuyển mã bằng AWS Elemental MediaConvert và phát qua CloudFront.

Mục tiêu của chương này là trình bày được cách xây dựng một hệ thống xem phim có khả năng upload video dung lượng lớn, tự động chuyển đổi sang HLS nhiều độ phân giải, phát video qua CDN, lưu lịch sử xem theo tài khoản, hỗ trợ phụ đề và theo dõi trạng thái vận hành bằng CloudWatch.

{{% notice info %}}
Cần thêm ảnh: sơ đồ kiến trúc tổng quan Netflop theo luồng Cloudflare -> EC2/Nginx -> RDS/S3 -> MediaConvert -> CloudFront -> Video Player. Không chụp hoặc đưa vào báo cáo các secret như access key, client secret, JWT secret, webhook secret.
{{% /notice %}}

#### Nội dung

1. [Tổng quan workshop](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường](5.2-Prerequisite/)
3. [Tạo hạ tầng AWS](5.3-AWS-infrastructure/)
4. [Cấu hình lưu trữ, database và xử lý media](5.4-Storage-database/)
5. [Triển khai và kiểm thử ứng dụng](5.5-Deploy-test/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)

#### Kết quả sau workshop

Sau khi hoàn thành, hệ thống có các thành phần chính:

* Website Netflop chạy tại `https://netflop.win`.
* EC2 chạy Nginx, React frontend và Node.js backend API.
* RDS MySQL lưu database `web_xem_phim_final`.
* S3 lưu video gốc, HLS output, ảnh phim, avatar và phụ đề.
* MediaConvert tự động tạo HLS 360p, 480p, 720p và 1080p.
* CloudFront phân phối HLS và bảo vệ stream bằng signed cookies.
* Lambda xử lý tác vụ tự động: nhận event MediaConvert và chuyển phụ đề `.srt` sang `.vtt`.
* CloudWatch/SNS theo dõi EC2, RDS, Lambda và cảnh báo khi có lỗi.

<!-- NETFLOP_DETAIL_START -->
#### Cách sử dụng phần workshop

Khi trình bày chương này trong báo cáo, mỗi mục nên đi theo cùng một cấu trúc:

1. Nêu chức năng cần triển khai.
2. Chỉ ra dịch vụ AWS hoặc thành phần ứng dụng liên quan.
3. Mô tả các bước thực hiện.
4. Đưa code mẫu hoặc lệnh kiểm tra ngắn.
5. Chụp ảnh kết quả sau khi thực hiện.

Các đoạn code mẫu trong chương này được rút gọn từ source Netflop. Mục đích của code mẫu là giải thích cách hệ thống hoạt động, không đưa các giá trị bí mật như access key, private key, JWT secret, database password hoặc OAuth client secret vào báo cáo.

#### Các chức năng chính cần chứng minh

| Chức năng | Thành phần thực hiện | Kết quả cần chứng minh |
| --- | --- | --- |
| Người dùng truy cập web | Cloudflare, EC2, Nginx, React | Website mở được tại https://netflop.win |
| API backend | Nginx reverse proxy, Node.js, PM2 | API trả JSON và backend online |
| Lưu dữ liệu | RDS MySQL | Dữ liệu phim, tập phim, tài khoản được đọc từ RDS |
| Upload video | React admin, Node.js, S3 | File gốc vào S3 input |
| Convert video | MediaConvert | HLS 360p/480p/720p/1080p trong S3 output |
| Cập nhật trạng thái tự động | EventBridge, Lambda, webhook backend | Tập phim tự chuyển sang ready |
| Phát video | CloudFront, signed cookies, VideoPlayer | Player phát HLS qua HTTPS |
| Phụ đề | S3, Lambda, WebVTT | SRT được chuyển sang VTT và hiển thị trên player |
| Theo dõi hệ thống | CloudWatch, SNS | Alarm và log hoạt động |
<!-- NETFLOP_DETAIL_END -->
