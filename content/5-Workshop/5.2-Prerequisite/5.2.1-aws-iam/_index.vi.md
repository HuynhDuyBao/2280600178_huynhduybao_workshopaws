---
title: "Chuẩn bị AWS và IAM"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.2.1. </b> "
---

#### Yêu cầu tài khoản AWS

Tài khoản AWS cần có quyền thao tác các dịch vụ:

* EC2, RDS, S3
* MediaConvert, CloudFront
* Lambda, EventBridge
* CloudWatch, SNS
* IAM, Systems Manager

#### Tài nguyên chính đã xác nhận

| Nhóm | Tài nguyên |
| --- | --- |
| EC2 | `netflop-web`, instance ID `i-059adda84b4d65714`, type `t3.micro`, public IP `18.143.150.109` |
| RDS | `netflop-db`, MySQL, `db.t4g.micro`, database `web_xem_phim_final`, port `3306` |
| S3 input | `netflop-input-source` |
| S3 output | `netflop-output-source` |
| CloudFront chính | `d11jdx7259stge.cloudfront.net` |
| Lambda | `netflop-mediaconvert-notifier`, `netflop-subtitle-converter` |
| EventBridge | `netflop-mediaconvert-job-state-change` |
| SNS | `netflop-alerts` |

#### Lưu ý bảo mật

* Không dùng root account để triển khai.
* Không đưa access key, secret key, password database hoặc private key vào báo cáo.
* IAM role cho EC2, Lambda và MediaConvert chỉ nên có quyền đúng nhu cầu.

![role](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.2-Prerequisite/5.2.1-aws-iam/role.png)

<!-- NETFLOP_DETAIL_START -->

#### Cách chuẩn bị quyền IAM

Trong dự án Netflop, không nên dùng một tài khoản có toàn quyền cho tất cả chức năng. Nên tách quyền theo từng thành phần để dễ kiểm soát và giảm rủi ro bảo mật.

| Thành phần | Quyền cần có |
| --- | --- |
| EC2 application | Đọc/ghi các bucket S3 được dùng bởi backend và gọi các dịch vụ AWS cần thiết |
| MediaConvert | Đọc file video từ S3 input và ghi HLS output vào S3 output |
| Lambda notifier | Nhận event từ MediaConvert và gọi webhook của backend |
| Lambda subtitle converter | Đọc file `.srt` từ S3 input và ghi file `.vtt` vào S3 output |
| Systems Manager | Gửi lệnh deploy/reload tới EC2 |

#### Policy S3 mẫu

Policy dưới đây là ví dụ quyền S3 tối thiểu cho bucket input/output của Netflop:

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

#### Cách kiểm tra

1. Gắn IAM Role phù hợp cho EC2 thay vì lưu access key trong source code.
2. Kiểm tra MediaConvert có role đọc được bucket input và ghi được bucket output.
3. Kiểm tra Lambda có quyền ghi log CloudWatch và truy cập đúng bucket S3.
4. Chạy thử upload file nhỏ lên S3 để xác nhận quyền hoạt động.

<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### IAM Role cần có trong hệ thống

Netflop nên dùng IAM Role thay vì hard-code AWS access key trong file env. EC2 được gắn role để backend có quyền thao tác với S3, MediaConvert, CloudFront signed key cấu hình bằng biến môi trường và Lambda/EventBridge.

#### Role cho EC2 backend

Các quyền tối thiểu:

* Đọc/ghi S3 input bucket và output bucket.
* Tạo multipart upload lên S3.
* Gọi MediaConvert CreateJob/GetJob.
* Pass role cho MediaConvert.
* Ghi log CloudWatch nếu dùng agent hoặc SDK logging.

#### Policy mẫu rút gọn

~~~json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::netflop-input-source",
        "arn:aws:s3:::netflop-input-source/*",
        "arn:aws:s3:::netflop-output-source",
        "arn:aws:s3:::netflop-output-source/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["mediaconvert:CreateJob", "mediaconvert:GetJob"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::<account-id>:role/<mediaconvert-role-name>"
    }
  ]
}
~~~

#### Role cho MediaConvert

MediaConvert cần một service role riêng có quyền đọc S3 input và ghi S3 output. Trust policy của role này phải cho phép service <code>mediaconvert.amazonaws.com</code> assume role.

#### Cách kiểm tra

~~~bash
aws sts get-caller-identity
aws s3 ls s3://netflop-input-source
aws s3 ls s3://netflop-output-source
aws mediaconvert describe-endpoints --region ap-southeast-1
~~~

{{% notice info %}}
Cần thêm ảnh: EC2 instance profile, IAM role policy, MediaConvert role trust relationship và kết quả AWS CLI kiểm tra quyền.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
