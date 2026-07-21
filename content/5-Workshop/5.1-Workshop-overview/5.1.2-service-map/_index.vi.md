---
title: "Bảng dịch vụ và vai trò"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.1.2. </b> "
---

#### Các dịch vụ đang sử dụng

| Dịch vụ | Trạng thái | Vai trò trong Netflop |
| --- | --- | --- |
| Cloudflare | Đang dùng | Quản lý domain `netflop.win`, DNS và HTTPS phía người dùng |
| Amazon EC2 | Đang chạy | Host Nginx, React frontend build và Node.js backend API |
| Nginx trên EC2 | Đang chạy | Reverse proxy, trả frontend và chuyển tiếp request API |
| PM2 trên EC2 | Đang dùng | Quản lý tiến trình backend Node.js |
| Amazon RDS MySQL | Đang chạy | Database chính `web_xem_phim_final` |
| Amazon S3 | Đang dùng | Lưu video input, HLS output, poster, banner, avatar và phụ đề |
| AWS Elemental MediaConvert | Đang dùng | Convert MP4/MKV sang HLS 360p/480p/720p/1080p |
| Amazon CloudFront | Đang dùng | Phân phối HLS stream, giảm tải S3 và bảo vệ stream bằng signed cookies |
| AWS Lambda | Đang dùng | Nhận event MediaConvert và convert phụ đề SRT sang VTT |
| Amazon EventBridge | Đang dùng | Bắt event MediaConvert COMPLETE/ERROR/CANCELED |
| Amazon CloudWatch | Đang dùng | Theo dõi EC2, RDS, Lambda, logs và alarm |
| Amazon SNS | Đang dùng | Gửi cảnh báo qua topic `netflop-alerts` |
| AWS IAM Role | Đang dùng | Cấp quyền cho EC2, MediaConvert và Lambda theo từng vai trò |
| AWS Systems Manager | Đang dùng | Hỗ trợ deploy/reload EC2 từ AWS CLI |
| Amazon Cognito | Có cấu hình | Hỗ trợ social login/OAuth, cần đồng bộ đúng domain callback khi triển khai production |

#### Thành phần chưa dùng làm phần chính

| Dịch vụ | Trạng thái | Ghi chú |
| --- | --- | --- |
| Amazon API Gateway | Chưa dùng | Backend hiện chạy trực tiếp trên EC2 qua Nginx; API Gateway phù hợp nếu tách thêm API serverless |
| AWS WAF | Chưa dùng | Có thể bổ sung sau để lọc request độc hại ở CloudFront/ALB |
| Application Load Balancer | Chưa dùng | Chưa cần khi chỉ có một EC2; cần khi scale nhiều instance |
| AWS Certificate Manager | Chưa dùng trực tiếp | HTTPS hiện đi qua Cloudflare; sẽ cần nếu dùng ALB/CloudFront domain tùy chỉnh trực tiếp trên AWS |
| Route 53 | Chưa dùng | Domain đang quản lý ở Cloudflare |

#### Ghi chú kiến trúc

* EC2 là nơi chạy ứng dụng chính, Lambda chỉ dùng cho tác vụ tự động nhỏ.
* S3 không dùng để lưu trạng thái nghiệp vụ; trạng thái phim/tập phim vẫn lưu trong RDS.
* CloudFront signed cookies giúp hạn chế việc public trực tiếp link HLS.
* Hướng hoàn thiện bảo mật là giới hạn Security Group của RDS chỉ cho Security Group EC2 truy cập.

<!-- NETFLOP_DETAIL_START -->
#### Cách ánh xạ chức năng với dịch vụ AWS

Khi làm báo cáo kiến trúc, không chỉ liệt kê tên dịch vụ mà cần giải thích dịch vụ đó giải quyết vấn đề gì trong website:

| Vấn đề của website xem phim | Dịch vụ/Thành phần giải quyết |
| --- | --- |
| File video lớn không nên lưu trong EC2 | Amazon S3 |
| Video MP4/MKV cần phát tốt trên web/mobile | AWS Elemental MediaConvert sang HLS |
| Người dùng ở xa server vẫn cần tải video nhanh | Amazon CloudFront |
| Không muốn lộ trực tiếp link stream | CloudFront signed cookies |
| Convert xong phải tự cập nhật trạng thái | EventBridge + Lambda + backend webhook |
| Phụ đề SRT không chạy trực tiếp trong player | Lambda/backend convert SRT sang WebVTT |
| Cần biết server/database có lỗi không | CloudWatch alarms + SNS |

#### Gợi ý trình bày trong sơ đồ

Trong sơ đồ architecture, nên nhóm các dịch vụ thành 5 vùng:

1. User Edge: Cloudflare, browser.
2. Application: EC2, Nginx, React, Node.js.
3. Data: RDS MySQL.
4. Media Pipeline: S3 input, MediaConvert, S3 output, CloudFront.
5. Operations: Lambda, EventBridge, CloudWatch, SNS, Systems Manager.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Bản đồ dịch vụ theo chức năng website

| Chức năng Netflop | Dịch vụ AWS dùng | Lý do sử dụng |
| --- | --- | --- |
| Host frontend/backend | EC2 | Dễ triển khai Node.js, Nginx và PM2 trên một server |
| Database phim/người dùng | RDS MySQL | Managed database, backup và security group riêng |
| Lưu video gốc | S3 input bucket | Lưu file MP4/MKV dung lượng lớn bền vững |
| Lưu HLS output | S3 output bucket | Lưu manifest, segment và subtitle |
| Chuyển đổi video | MediaConvert | Xuất HLS nhiều bitrate 360p-1080p |
| Cập nhật trạng thái job | EventBridge + Lambda | Tự động báo backend khi job COMPLETE/ERROR |
| Phát video nhanh | CloudFront | CDN cho HLS segment |
| Bảo vệ stream | CloudFront signed cookies | Giới hạn thời gian truy cập HLS |
| Giám sát | CloudWatch + SNS | Theo dõi CPU, lỗi API, dung lượng, cảnh báo email |

#### Dịch vụ chưa dùng trong kiến trúc chính

API Gateway, ALB, WAF và Route 53 chưa bắt buộc trong bản triển khai hiện tại. Website đang dùng Nginx reverse proxy trên EC2 và DNS/SSL qua Cloudflare. Các dịch vụ này có thể bổ sung ở giai đoạn mở rộng khi cần tách nhiều EC2, rate limit nâng cao hoặc quản lý domain hoàn toàn trong AWS.

#### Cách chứng minh trong báo cáo

1. Với mỗi dịch vụ, chụp ảnh resource thật trên AWS console.
2. Ghi rõ tên resource đang dùng, ví dụ <code>netflop-db</code>, <code>netflop-input-source</code>, <code>netflop-output-source</code>.
3. Nêu một chức năng website phụ thuộc vào resource đó.
4. Đưa code backend/frontend liên quan ở các mục sau.
<!-- NETFLOP_IMPLEMENTATION_END -->
