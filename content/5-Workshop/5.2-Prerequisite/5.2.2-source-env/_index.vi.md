---
title: "Chuẩn bị source code và biến môi trường"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

#### Mục tiêu

Chuẩn bị source code Netflop và các biến môi trường để backend có thể kết nối RDS, S3, MediaConvert, CloudFront, Cognito/Google OAuth và frontend production domain.

#### Source code chính

```text
netflop/
  backend/
    src/
    package.json
  frontend/
    src/
    package.json
  database/
    web_xem_phim_final_dump.sql
```

#### Nhóm biến môi trường backend

Các biến quan trọng cần có trong `.env` backend:

```text
NODE_ENV=production
PORT=5000

DB_HOST=netflop-db.c76we6m8scy0.ap-southeast-1.rds.amazonaws.com
DB_PORT=3306
DB_NAME=web_xem_phim_final
DB_USER=<db-user>
DB_PASSWORD=<db-password>

CORS_ORIGIN=https://netflop.win
APP_BASE_URL=https://netflop.win

AWS_REGION=ap-southeast-1
AWS_S3_INPUT_BUCKET=netflop-input-source
AWS_S3_OUTPUT_BUCKET=netflop-output-source
AWS_MEDIACONVERT_ENDPOINT=https://mediaconvert.ap-southeast-1.amazonaws.com
AWS_MEDIACONVERT_ROLE_ARN=<mediaconvert-role-arn>

AWS_CLOUDFRONT_DOMAIN=<cloudfront-domain>
AWS_CLOUDFRONT_KEY_PAIR_ID=<key-pair-id>
AWS_CLOUDFRONT_PRIVATE_KEY_PATH=<private-key-path>

JWT_SECRET=<jwt-secret>
AWS_MEDIACONVERT_WEBHOOK_SECRET=<webhook-secret>
```

#### Nhóm biến môi trường frontend

Frontend production cần trỏ API về domain production:

```text
VITE_API_BASE_URL=https://netflop.win/api
```

Nếu dùng OAuth callback, các URL cần đồng bộ:

```text
GOOGLE_REDIRECT_URI=https://netflop.win/auth/callback
AWS_COGNITO_REDIRECT_URI=https://netflop.win/auth/callback
AWS_COGNITO_LOGOUT_URI=https://netflop.win/
```

#### Nguyên tắc bảo mật

* Không commit file `.env` thật lên Git.
* Không chụp màn hình access key, client secret, database password hoặc private key.
* Production nên dùng IAM Role cho EC2 thay vì access key.
* Các secret có thể chuyển sang AWS Systems Manager Parameter Store hoặc AWS Secrets Manager ở bước hoàn thiện.

<!-- NETFLOP_DETAIL_START -->

#### Cách kiểm tra source code trước khi deploy

Trước khi đưa Netflop lên EC2, cần kiểm tra source code theo từng nhóm chức năng:

| Nhóm chức năng | File/thiết lập cần kiểm tra | Kết quả mong muốn |
| --- | --- | --- |
| Frontend | `frontend/package.json`, biến `VITE_API_BASE_URL` | Build production thành công và gọi đúng API domain |
| Backend API | `backend/package.json`, file `.env` | Backend start được trên port `5000` |
| Database | `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` | Backend kết nối được RDS MySQL |
| Upload video | Bucket input/output, IAM Role | Backend upload được file lên S3 |
| Convert video | MediaConvert endpoint và role ARN | Tạo được MediaConvert job |
| Stream video | CloudFront domain và key pair | Frontend phát được HLS qua CloudFront |
| Subtitle | Lambda converter và bucket S3 | File `.srt` được đổi sang `.vtt` |

#### Code cấu hình AWS mẫu

Đoạn cấu hình dưới đây giúp backend đọc các thông tin AWS từ biến môi trường:

~~~js
export const awsConfig = {
  region: process.env.AWS_REGION || 'ap-southeast-1',
  s3InputBucket: process.env.AWS_S3_INPUT_BUCKET,
  s3OutputBucket: process.env.AWS_S3_OUTPUT_BUCKET,
  mediaConvertEndpoint: process.env.AWS_MEDIACONVERT_ENDPOINT,
  mediaConvertRoleArn: process.env.AWS_MEDIACONVERT_ROLE_ARN,
  cloudFrontDomain: process.env.AWS_CLOUDFRONT_DOMAIN
};
~~~

#### Lệnh kiểm tra local

Chạy các lệnh sau trước khi deploy để phát hiện lỗi sớm:

~~~bash
npm --prefix frontend install
npm --prefix frontend run build
npm --prefix backend install
npm --prefix backend start
~~~

Nếu backend cần kết nối RDS thật khi chạy local, cần mở security group tạm thời cho IP cá nhân hoặc kiểm tra trực tiếp trên EC2 để tránh mở database quá rộng.

<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Nhóm biến môi trường cần cấu hình

File env production nên chia thành từng nhóm để dễ kiểm tra:

* Server: port, node env, CORS origin.
* Database: host RDS, database, user, password.
* JWT/session: secret, thời hạn token.
* AWS media: region, S3 buckets, MediaConvert role, endpoint, CloudFront domain.
* Stream security: CloudFront key pair id, private key path/base64, signed cookie TTL.
* OAuth/Cognito: client id, client secret, redirect URI.

#### Mẫu env production

~~~env
NODE_ENV=production
PORT=5000
CORS_ORIGIN=https://netflop.win

DB_HOST=netflop-db.xxxxxx.ap-southeast-1.rds.amazonaws.com
DB_PORT=3306
DB_NAME=web_xem_phim_final
DB_USER=admin
DB_PASSWORD=<khong-ghi-mat-khau-vao-bao-cao>

AWS_REGION=ap-southeast-1
AWS_S3_INPUT_BUCKET=netflop-input-source
AWS_S3_OUTPUT_BUCKET=netflop-output-source
AWS_CLOUDFRONT_DOMAIN=https://dxxxxxxxxxxxxx.cloudfront.net
AWS_MEDIACONVERT_ENDPOINT=https://mediaconvert.ap-southeast-1.amazonaws.com
AWS_MEDIACONVERT_ROLE_ARN=arn:aws:iam::<account-id>:role/<mediaconvert-role>

APP_URL=https://netflop.win
GOOGLE_REDIRECT_URI=https://netflop.win/auth/callback
AWS_COGNITO_REDIRECT_URI=https://netflop.win/auth/callback
~~~

#### Cách áp dụng env trên EC2

1. Upload hoặc tạo file <code>/home/ubuntu/netflop/backend/.env</code>.
2. Không commit file env lên Git.
3. Restart backend bằng PM2.
4. Kiểm tra API và log.

~~~bash
pm2 restart netflop-api --update-env
pm2 logs netflop-api
curl -I https://netflop.win/api/health
~~~

{{% notice warning %}}
Trong báo cáo chỉ đưa biến mẫu. Không chụp hoặc ghi access key, database password, client secret, JWT secret và CloudFront private key.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
