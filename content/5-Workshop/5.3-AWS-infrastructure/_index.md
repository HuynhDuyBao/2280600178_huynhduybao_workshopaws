---
title: "Create AWS infrastructure"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This section presents the steps to create the base infrastructure for Netflop: the domain, EC2, Nginx, backend runtime, RDS MySQL, and Security Group. This is the core application layer before configuring the media pipeline.

![rds](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/rds.png)

#### Contents

1. [Configure Cloudflare domain](5.3.1-cloudflare-domain/)
2. [Configure EC2, Nginx, and backend](5.3.2-ec2-nginx/)
3. [Configure RDS and Security Group](5.3.3-rds-security/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Infrastructure creation order

Create infrastructure from foundational resources to dependent resources:

1. Create S3 input and output buckets.
2. Create IAM roles for EC2 and MediaConvert.
3. Create RDS MySQL and security groups.
4. Create EC2, attach the IAM role, and install Node.js, Git, Nginx, and PM2.
5. Deploy backend and frontend.
6. Configure Cloudflare DNS to point to EC2.
7. Create CloudFront distribution for S3 output.
8. Configure EventBridge and Lambda for MediaConvert events.
9. Create CloudWatch alarms and SNS notifications.

#### Dependency flow

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

#### Verification after each stage

| Stage | Verification |
| --- | --- |
| EC2 | <code>pm2 status</code>, <code>sudo systemctl status nginx</code> |
| RDS | Backend API returns movie data |
| S3 | Upload image/video/subtitle successfully |
| MediaConvert | Job is SUBMITTED/COMPLETE |
| CloudFront | HLS URL opens through the distribution |
| CloudWatch | Alarm status is OK or ALARM |
<!-- NETFLOP_IMPLEMENTATION_END -->
