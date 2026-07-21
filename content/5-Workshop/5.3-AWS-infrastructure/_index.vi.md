---
title: "Tạo hạ tầng AWS"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Phần này trình bày các bước tạo hạ tầng nền cho Netflop: domain, EC2, Nginx, backend runtime, RDS MySQL và Security Group. Đây là lớp chạy ứng dụng chính trước khi cấu hình pipeline media.

{{% notice info %}}
Cần thêm ảnh: Cloudflare DNS cho `netflop.win`, EC2 `netflop-web` đang running, Nginx/PM2 running và RDS `netflop-db` available.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/rds.png)

#### Nội dung

1. [Cấu hình Cloudflare domain](5.3.1-cloudflare-domain/)
2. [Cấu hình EC2, Nginx và backend](5.3.2-ec2-nginx/)
3. [Cấu hình RDS và Security Group](5.3.3-rds-security/)

<!-- NETFLOP_DETAIL_START -->
#### Thứ tự thực hiện hạ tầng

Nên triển khai hạ tầng theo thứ tự sau:

1. Tạo EC2 và Security Group cho web server.
2. Cài Node.js, Git, Nginx, PM2 trên EC2.
3. Tạo RDS MySQL và import database.
4. Cấu hình domain Cloudflare trỏ về EC2.
5. Cấu hình Nginx reverse proxy.
6. Build frontend và chạy backend bằng PM2.
7. Kiểm tra API health check và giao diện web.

#### Lệnh kiểm tra tổng hợp

~~~bash
sudo systemctl status nginx --no-pager
sudo nginx -t
pm2 status
curl -I https://netflop.win
curl https://netflop.win/api/catalog/genres
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Thứ tự tạo hạ tầng

Thứ tự triển khai nên đi từ tài nguyên nền tảng đến tài nguyên phụ thuộc:

1. Tạo S3 input/output bucket.
2. Tạo IAM Role cho EC2 và MediaConvert.
3. Tạo RDS MySQL và Security Group.
4. Tạo EC2, gắn IAM Role, cài Node.js/Git/Nginx/PM2.
5. Deploy backend/frontend.
6. Cấu hình Cloudflare DNS trỏ về EC2.
7. Tạo CloudFront distribution cho S3 output.
8. Cấu hình EventBridge + Lambda cho MediaConvert.
9. Tạo CloudWatch alarm và SNS.

#### Luồng phụ thuộc

~~~text
RDS + S3 + IAM
      |
      v
EC2 backend
      |
      v
MediaConvert + CloudFront
      |
      v
User player + Admin upload
~~~

#### Kiểm tra cuối mỗi bước

| Bước | Cách kiểm tra |
| --- | --- |
| EC2 | <code>pm2 status</code>, <code>sudo systemctl status nginx</code> |
| RDS | Backend gọi API trả dữ liệu phim |
| S3 | Upload thử ảnh/video/phụ đề |
| MediaConvert | Có job SUBMITTED/COMPLETE |
| CloudFront | Mở URL HLS qua distribution |
| CloudWatch | Alarm có trạng thái OK/ALARM |
<!-- NETFLOP_IMPLEMENTATION_END -->
