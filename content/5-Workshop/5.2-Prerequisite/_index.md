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
