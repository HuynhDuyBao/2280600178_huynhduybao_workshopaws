---
title: "Configure EC2, Nginx, and backend"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

#### Goal

The EC2 instance `netflop-web` is the main host for the Netflop application. It serves the React frontend, Node.js backend, and Nginx reverse proxy.

#### EC2 information

| Attribute | Value |
| --- | --- |
| Instance name | `netflop-web` |
| Instance ID | `i-059adda84b4d65714` |
| Instance type | `t3.micro` |
| Public IP | `18.143.150.109` |
| Security Group | `netflop-ec2-sg` |

#### Installed components on EC2

* Node.js for running the backend.
* Git for pulling source code.
* Nginx for reverse proxy and serving the frontend.
* PM2 for managing the `netflop-api` backend process.
* AWS CLI/SSM Agent for deployment support.

#### EC2 request flow

```text
Request https://netflop.win
   |
   v
Nginx
   |-- /              -> React build in /var/www/netflop
   |-- /api/*         -> Node.js backend at localhost:5000
   |-- /uploads/*     -> Backend or static media fallback if legacy data exists
```

#### Verification commands

```bash
sudo systemctl status nginx
pm2 status
pm2 logs netflop-api
curl -I https://netflop.win
curl https://netflop.win/api/health
```

#### EC2 security group

Minimum inbound ports for EC2:

| Port | Source | Purpose |
| --- | --- | --- |
| 80 | Internet | HTTP / Cloudflare |
| 443 | Internet | HTTPS if Nginx handles TLS |
| 22 | Admin IP or SSM only | SSH administration |

If using Systems Manager Session Manager, SSH access can be further restricted.



![ec2](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/ec2running.png)
![ec2s](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/security%20gr.png)
![Nginx](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.2-ec2-nginx/ngix%20status.png)

<!-- NETFLOP_DETAIL_START -->
#### How to install and check EC2

After creating an Ubuntu EC2 instance, install the required components:

~~~bash
sudo apt update
sudo apt install -y nginx git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
~~~

#### Sample Nginx reverse proxy

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

#### Run backend with PM2

~~~bash
cd /home/ubuntu/netflop/backend
npm install
pm2 start src/server.js --name netflop-api
pm2 save
~~~

#### Check status

~~~bash
sudo nginx -t
sudo systemctl status nginx --no-pager
pm2 status
pm2 logs netflop-api
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### EC2 role in the system

EC2 hosts both the built frontend and the Node.js backend. Nginx serves static React files and reverse proxies API requests to the backend running locally.

#### Install dependencies on EC2

~~~bash
sudo apt update
sudo apt install -y nginx git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
~~~

#### Nginx reverse proxy example

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

#### Run backend with PM2

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

#### Verification

~~~bash
sudo nginx -t
sudo systemctl status nginx --no-pager
pm2 status
pm2 logs netflop-api
curl -I http://127.0.0.1:5000/api/health
~~~

{{% notice info %}}
Screenshots needed: EC2 instance running, inbound security group, <code>pm2 status</code>, <code>systemctl status nginx</code>, and API health check.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
