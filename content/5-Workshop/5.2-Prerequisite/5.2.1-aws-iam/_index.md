---
title: "AWS and IAM preparation"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.2.1. </b> "
---

#### AWS account requirements

The AWS account must have permissions to use these services:

* EC2, RDS, S3
* MediaConvert, CloudFront
* Lambda, EventBridge
* CloudWatch, SNS
* IAM, Systems Manager

#### Confirmed core resources

| Group | Resource |
| --- | --- |
| EC2 | `netflop-web`, instance ID `i-059adda84b4d65714`, type `t3.micro`, public IP `18.143.150.109` |
| RDS | `netflop-db`, MySQL, `db.t4g.micro`, database `web_xem_phim_final`, port `3306` |
| S3 input | `netflop-input-source` |
| S3 output | `netflop-output-source` |
| CloudFront | `d11jdx7259stge.cloudfront.net` |
| Lambda | `netflop-mediaconvert-notifier`, `netflop-subtitle-converter` |
| EventBridge | `netflop-mediaconvert-job-state-change` |
| SNS | `netflop-alerts` |

#### Security notes

* Do not deploy with the AWS root account.
* Do not include access keys, secret keys, database passwords, or private keys in the report.
* IAM roles for EC2, Lambda, and MediaConvert should be scoped to least privilege.

![role](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.2-Prerequisite/5.2.1-aws-iam/role.png)

<!-- NETFLOP_DETAIL_START -->
#### How to prepare IAM permissions

The deployment should separate permissions by component:

| Component | Required access |
| --- | --- |
| EC2 application | Read/write selected S3 buckets and call AWS services needed by backend |
| MediaConvert | Read S3 input and write S3 output |
| Lambda notifier | Receive MediaConvert event and call backend webhook |
| Lambda subtitle converter | Read `.srt` from S3 input and write `.vtt` to S3 output |
| Systems Manager | Send deployment/reload commands to EC2 |

#### Sample S3 policy

~~~json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": [
        "arn:aws:s3:::netflop-input-source/*",
        "arn:aws:s3:::netflop-output-source/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::netflop-input-source",
        "arn:aws:s3:::netflop-output-source"
      ]
    }
  ]
}
~~~

For production, avoid long-lived access keys and prefer IAM roles attached to EC2/Lambda/MediaConvert.
<!-- NETFLOP_DETAIL_END -->

