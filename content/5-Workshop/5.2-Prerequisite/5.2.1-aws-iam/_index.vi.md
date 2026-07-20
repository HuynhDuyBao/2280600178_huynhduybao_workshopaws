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
