---
title: "Chuẩn bị môi trường"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Trước khi triển khai Netflop lên AWS, cần chuẩn bị tài khoản AWS, source code, biến môi trường, database dump và các quyền IAM cần thiết. Đây là bước quan trọng vì hệ thống sử dụng nhiều dịch vụ liên kết với nhau: EC2, RDS, S3, MediaConvert, CloudFront, Lambda, EventBridge và CloudWatch.

{{% notice info %}}
Cần thêm ảnh: cấu trúc source code Netflop, AWS CLI đã đăng nhập đúng account/region, và file `.env` mẫu đã che secret.
{{% /notice %}}

#### Nội dung

1. [Chuẩn bị AWS và IAM](5.2.1-aws-iam/)
2. [Chuẩn bị source code và biến môi trường](5.2.2-source-env/)

#### Checklist chuẩn bị

* AWS CLI đã cấu hình đúng account.
* Region triển khai chính: `ap-southeast-1`.
* Domain production: `netflop.win`.
* Source code đã có frontend React và backend Node.js.
* Database local đã export thành file SQL từ database `web_xem_phim_final`.
* EC2 có Node.js, Git, Nginx, PM2 và quyền truy cập AWS qua IAM Role.
* Không dùng access key cá nhân hard-code trong source hoặc `.env` production.

<!-- NETFLOP_DETAIL_START -->
#### Cách chuẩn bị trước khi làm workshop

Trước khi triển khai, cần xác nhận 4 nhóm thông tin:

1. AWS account và region đang dùng.
2. Source code frontend/backend chạy được ở local.
3. Database local đã export được file dump.
4. Các biến môi trường production không bị thiếu.

#### Lệnh kiểm tra nhanh

~~~bash
aws sts get-caller-identity
aws configure get region
node -v
npm -v
npm --prefix frontend run build
~~~

Nếu frontend build thành công và AWS CLI trả đúng account, có thể chuyển sang bước tạo hạ tầng.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Chuẩn bị trước khi triển khai

Trước khi cấu hình AWS, cần chuẩn bị môi trường local và tài khoản cloud:

1. AWS account có quyền tạo EC2, RDS, S3, CloudFront, MediaConvert, IAM, Lambda, EventBridge và CloudWatch.
2. AWS CLI đã cấu hình region <code>ap-southeast-1</code> cho phần hạ tầng chính.
3. Node.js 20+, npm, Git.
4. Source code Netflop đã chạy được local.
5. Database dump <code>web_xem_phim_final_dump.sql</code>.
6. Domain <code>netflop.win</code> đã trỏ DNS về EC2.

#### Lệnh kiểm tra local

~~~bash
node -v
npm -v
aws sts get-caller-identity
aws configure get region
~~~

#### Kiểm tra project trước deploy

~~~bash
npm --prefix backend install
npm --prefix frontend install
npm --prefix frontend run build
~~~

Nếu build frontend thành công và backend kết nối được database local/RDS, có thể chuyển sang triển khai hạ tầng.

{{% notice info %}}
Cần thêm ảnh: kết quả <code>aws sts get-caller-identity</code>, phiên bản Node.js/npm và frontend build thành công.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
