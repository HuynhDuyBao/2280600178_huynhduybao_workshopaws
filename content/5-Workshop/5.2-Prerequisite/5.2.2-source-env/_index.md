---
title: "Source code and environment variables"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

#### Goal

Prepare the Netflop source code and environment variables so that the backend can connect to RDS, S3, MediaConvert, CloudFront, Cognito/Google OAuth, and the production frontend domain.

#### Main source tree

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

#### Backend environment variables

Important variables for the backend `.env`:

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

#### Frontend environment variables

The production frontend should point API calls to the production domain:

```text
VITE_API_BASE_URL=https://netflop.win/api
```

If using OAuth callbacks, synchronize these URLs:

```text
GOOGLE_REDIRECT_URI=https://netflop.win/auth/callback
AWS_COGNITO_REDIRECT_URI=https://netflop.win/auth/callback
AWS_COGNITO_LOGOUT_URI=https://netflop.win/
```

#### Security principles

* Do not commit a real `.env` file to Git.
* Do not screenshot access keys, client secrets, database passwords, or private keys.
* In production, use IAM Role for EC2 instead of access keys.
* Secrets can later be moved to AWS Systems Manager Parameter Store or Secrets Manager.

<!-- NETFLOP_DETAIL_START -->
#### How to check source code before deployment

Before deploying Netflop to EC2, verify the following:

1. Frontend can build successfully.
2. Backend can start without syntax/runtime errors.
3. Database variables point to RDS.
4. S3 input/output bucket names are correct.
5. CloudFront domain and signed-cookie values are configured.
6. MediaConvert endpoint and role ARN are configured.

#### Sample backend config

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

#### Local validation commands

~~~bash
npm --prefix frontend install
npm --prefix frontend run build
npm --prefix backend install
npm --prefix backend start
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Environment variable groups

Production environment variables should be grouped so they are easy to verify:

* Server: port, Node environment, CORS origin.
* Database: RDS host, database name, user, password.
* JWT/session: secret and token lifetime.
* AWS media: region, S3 buckets, MediaConvert role, endpoint, CloudFront domain.
* Stream security: CloudFront key pair id, private key path/base64, signed cookie TTL.
* OAuth/Cognito: client id, client secret, redirect URI.

#### Production env example

~~~env
NODE_ENV=production
PORT=5000
CORS_ORIGIN=https://netflop.win

DB_HOST=netflop-db.xxxxxx.ap-southeast-1.rds.amazonaws.com
DB_PORT=3306
DB_NAME=web_xem_phim_final
DB_USER=admin
DB_PASSWORD=<do-not-put-the-real-password-in-the-report>

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

#### Apply env on EC2

1. Create or upload <code>/home/ubuntu/netflop/backend/.env</code>.
2. Do not commit this file to Git.
3. Restart the backend with PM2.
4. Check API and logs.

~~~bash
pm2 restart netflop-api --update-env
pm2 logs netflop-api
curl -I https://netflop.win/api/health
~~~

{{% notice warning %}}
Only show placeholder values in the report. Do not expose access keys, database passwords, client secrets, JWT secrets, or CloudFront private keys.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
