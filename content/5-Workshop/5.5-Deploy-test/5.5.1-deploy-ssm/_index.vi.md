---
title: "Deploy/reload bằng EC2 và Systems Manager"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Mục tiêu

Deploy source code Netflop lên EC2 và reload ứng dụng mà không cần thao tác thủ công quá nhiều trên server. EC2 chạy frontend build, backend API và Nginx.

#### Quy trình deploy

1. Build frontend React.
2. Copy frontend build vào thư mục phục vụ bởi Nginx.
3. Cập nhật source backend trên EC2.
4. Cài package backend nếu có thay đổi dependency.
5. Reload backend bằng PM2.
6. Reload Nginx nếu cấu hình reverse proxy thay đổi.
7. Kiểm tra website và API.

#### Lệnh kiểm tra trên EC2

```bash
pm2 status
pm2 logs netflop-api
sudo nginx -t
sudo systemctl reload nginx
curl -I https://netflop.win
curl https://netflop.win/api/health
```

#### Vai trò của Systems Manager

AWS Systems Manager hỗ trợ gửi lệnh đến EC2 từ AWS CLI, giúp deploy hoặc kiểm tra server mà không cần mở SSH rộng rãi. Đây là hướng tốt hơn cho production vì giảm phụ thuộc vào key pair và giảm bề mặt tấn công.

#### Kết quả mong đợi

* Frontend load từ `https://netflop.win`.
* Backend API hoạt động qua `/api`.
* PM2 hiển thị process backend online.
* Nginx test config thành công.

{{% notice info %}}
Cần thêm ảnh: lệnh deploy thành công, `pm2 status`, `sudo systemctl status nginx`, output health check và giao diện website sau deploy.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.1-deploy-ssm/ngix%20status.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.1-deploy-ssm/pm2%20status.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách deploy frontend

1. Build frontend ở local.
2. Đóng gói thư mục <code>frontend/dist</code>.
3. Upload artifact lên S3 staging.
4. Dùng Systems Manager gửi lệnh cho EC2 tải artifact.
5. Giải nén vào <code>/var/www/netflop</code>.
6. Reload Nginx.

#### Lệnh mẫu

~~~bash
npm --prefix frontend run build
tar -czf netflop-frontend.tgz -C frontend/dist .
aws s3 cp netflop-frontend.tgz s3://netflop-input-source/deploy/frontend/netflop-frontend.tgz
~~~

Trên EC2, lệnh SSM sẽ thực hiện ý tưởng sau:

~~~bash
curl -fL '<presigned-url>' -o /tmp/netflop-deploy/frontend.tgz
mkdir -p /tmp/netflop-frontend-build
tar -xzf /tmp/netflop-deploy/frontend.tgz -C /tmp/netflop-frontend-build
sudo cp -a /tmp/netflop-frontend-build/. /var/www/netflop/
sudo nginx -t
sudo systemctl reload nginx
~~~

#### Cách deploy backend

Backend deploy bằng cách cập nhật source, cài dependency nếu cần và restart PM2:

~~~bash
cd /home/ubuntu/netflop/backend
npm install --omit=dev
pm2 restart netflop-api --update-env
pm2 logs netflop-api
~~~
<!-- NETFLOP_DETAIL_END -->
