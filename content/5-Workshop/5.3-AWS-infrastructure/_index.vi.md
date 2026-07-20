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
