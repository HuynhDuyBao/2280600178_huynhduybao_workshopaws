---
title: "Prerequisites"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Before deploying Netflop to AWS, prepare the AWS account, source code, environment variables, database dump, and necessary IAM permissions. This step is important because the system uses many linked services: EC2, RDS, S3, MediaConvert, CloudFront, Lambda, EventBridge, and CloudWatch.

#### Contents

1. [AWS and IAM preparation](5.2.1-aws-iam/)
2. [Source code and environment variables](5.2.2-source-env/)

#### Preparation checklist

* AWS CLI is configured for the correct account.
* Main deployment region: `ap-southeast-1`.
* Production domain: `netflop.win`.
* Source code includes the React frontend and Node.js backend.
* Local database has been exported to SQL from `web_xem_phim_final`.
* EC2 has Node.js, Git, Nginx, PM2, and IAM Role access to AWS.
* Do not hard-code personal access keys in source code or production `.env` files.

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Prerequisites before deployment

Before configuring AWS, prepare both the local project and the cloud account:

1. AWS account with permissions for EC2, RDS, S3, CloudFront, MediaConvert, IAM, Lambda, EventBridge, and CloudWatch.
2. AWS CLI configured with region <code>ap-southeast-1</code> for the main infrastructure.
3. Node.js 20+, npm, and Git.
4. Netflop source code can run locally.
5. Database dump file <code>web_xem_phim_final_dump.sql</code>.
6. Domain <code>netflop.win</code> points to the EC2 instance.

#### Local verification commands

~~~bash
node -v
npm -v
aws sts get-caller-identity
aws configure get region
~~~

#### Project check before deployment

~~~bash
npm --prefix backend install
npm --prefix frontend install
npm --prefix frontend run build
~~~

If the frontend builds successfully and the backend can connect to local/RDS MySQL, the infrastructure setup can begin.

{{% notice info %}}
Screenshots needed: <code>aws sts get-caller-identity</code>, Node.js/npm versions, and successful frontend build.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
