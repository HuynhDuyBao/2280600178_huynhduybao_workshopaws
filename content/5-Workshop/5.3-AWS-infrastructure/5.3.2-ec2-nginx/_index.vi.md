---
title: "Cấu hình EC2, Nginx và backend"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### Mục tiêu

EC2 `netflop-web` là máy chủ chính chạy ứng dụng Netflop. Máy chủ này phục vụ frontend React, backend Node.js và Nginx reverse proxy.

#### Thông tin EC2

| Thuộc tính | Giá trị |
| --- | --- |
| Instance name | `netflop-web` |
| Instance ID | `i-059adda84b4d65714` |
| Instance type | `t3.micro` |
| Public IP hiện tại | `18.143.150.109` |
| Security Group | `netflop-ec2-sg` |

#### Thành phần cài trên EC2

* Node.js để chạy backend.
* Git để kéo source code.
* Nginx để reverse proxy và phục vụ frontend.
* PM2 để quản lý process backend `netflop-api`.
* AWS CLI/SSM Agent để hỗ trợ thao tác deploy.

#### Luồng request trên EC2

```text
Request https://netflop.win
   |
   v
Nginx
   |-- /              -> React build trong /var/www/netflop
   |-- /api/*         -> Node.js backend localhost:5000
   |-- /uploads/*     -> Backend hoặc static media fallback nếu còn dữ liệu cũ
```

#### Các bước kiểm tra

```bash
sudo systemctl status nginx
pm2 status
pm2 logs netflop-api
curl -I https://netflop.win
curl https://netflop.win/api/health
```

#### Security Group EC2

EC2 cần mở các cổng tối thiểu:

| Port | Nguồn | Mục đích |
| --- | --- | --- |
| 80 | Internet | HTTP/Cloudflare |
| 443 | Internet | HTTPS nếu Nginx xử lý TLS |
| 22 | IP quản trị hoặc SSM thay thế | SSH quản trị |

Nếu dùng Systems Manager Session Manager thì có thể hạn chế SSH hơn nữa.



![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/ec2running.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/security%20gr.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/ngix%20status.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách cài và kiểm tra EC2

Sau khi tạo EC2 Ubuntu, cài các thành phần chính:

~~~bash
sudo apt update
sudo apt install -y nginx git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
~~~

#### Cấu hình Nginx reverse proxy mẫu

~~~nginx
server {
  listen 80;
  server_name netflop.win www.netflop.win;

  root /var/www/netflop;
  index index.html;

  location /api/ {
    proxy_pass http://127.0.0.1:5000/api/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  location / {
    try_files $uri $uri/ /index.html;
  }
}
~~~

#### Chạy backend bằng PM2

~~~bash
cd /home/ubuntu/netflop/backend
npm install
pm2 start src/server.js --name netflop-api
pm2 save
~~~

#### Kiểm tra trạng thái

~~~bash
sudo nginx -t
sudo systemctl status nginx --no-pager
pm2 status
pm2 logs netflop-api
~~~

Nếu Nginx hiện <code>active (running)</code> và PM2 hiện <code>netflop-api online</code>, lớp application đã chạy.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Vai trò của EC2 trong hệ thống

EC2 chạy cả frontend đã build và backend Node.js. Nginx đứng phía trước để phục vụ file tĩnh React và reverse proxy các request API sang backend chạy ở localhost.

#### Các bước cài đặt trên EC2

~~~bash
sudo apt update
sudo apt install -y nginx git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
~~~

#### Nginx reverse proxy mẫu

~~~nginx
server {
  listen 80;
  server_name netflop.win www.netflop.win;

  root /var/www/netflop;
  index index.html;

  location /api/ {
    proxy_pass http://127.0.0.1:5000/api/;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
  }

  location / {
    try_files $uri $uri/ /index.html;
  }
}
~~~

#### Deploy backend bằng PM2

~~~bash
cd /home/ubuntu/netflop/backend
npm install --omit=dev
pm2 start src/server.js --name netflop-api
pm2 save
pm2 startup
~~~

#### Deploy frontend

~~~bash
cd /home/ubuntu/netflop/frontend
npm install
npm run build
sudo rm -rf /var/www/netflop/*
sudo cp -r dist/* /var/www/netflop/
sudo systemctl reload nginx
~~~

#### Kiểm tra

~~~bash
sudo nginx -t
sudo systemctl status nginx --no-pager
pm2 status
pm2 logs netflop-api
curl -I http://127.0.0.1:5000/api/health
~~~


<!-- NETFLOP_IMPLEMENTATION_END -->
