---
title: "Dọn dẹp tài nguyên ứng dụng"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### Mục tiêu

Dọn dẹp hoặc kiểm tra các tài nguyên thuộc lớp application: EC2, Nginx, PM2, domain, IAM và RDS. Nếu hệ thống vẫn tiếp tục chạy production thì không xóa tài nguyên chính, chỉ xóa tài nguyên thử nghiệm hoặc không còn dùng.

#### Tài nguyên cần kiểm tra

| Tài nguyên | Hành động |
| --- | --- |
| EC2 `netflop-web` | Giữ nếu website còn chạy; stop nếu chỉ demo |
| EBS volume | Kiểm tra volume không gắn instance |
| Elastic IP | Giữ nếu domain đang trỏ vào; release nếu không dùng |
| Security Group | Xóa rule dư thừa, đặc biệt SSH/RDS mở rộng |
| IAM user/access key | Vô hiệu hóa key không dùng; ưu tiên IAM Role |
| RDS `netflop-db` | Giữ production; snapshot trước khi xóa nếu cần |
| Cloudflare DNS | Giữ record production; xóa record test |

#### Lưu ý với RDS

Không xóa RDS production nếu chưa backup. Trước khi dọn:

1. Export database hoặc tạo snapshot.
2. Kiểm tra backend không còn phụ thuộc database đó.
3. Ghi lại endpoint và thông tin snapshot trong báo cáo.

#### Kết quả mong đợi

* Không còn instance/volume/key thử nghiệm gây chi phí.
* Security Group gọn và an toàn hơn.
* Database production có backup trước khi thay đổi lớn.

{{% notice info %}}
Cần thêm ảnh: EC2 instance list, EBS volumes, Elastic IP, Security Group inbound, RDS snapshot và IAM access key đã vô hiệu hóa nếu không dùng.
{{% /notice %}}

<!-- NETFLOP_DETAIL_START -->
#### Cách cleanup tài nguyên ứng dụng

Nếu chỉ dừng môi trường demo:

~~~bash
pm2 stop netflop-api
sudo systemctl stop nginx
~~~

Nếu xóa hẳn hạ tầng, cần làm theo thứ tự:

1. Backup database RDS hoặc tạo snapshot.
2. Tắt EC2 nếu không dùng.
3. Kiểm tra EBS volume không còn gắn.
4. Release Elastic IP nếu không còn trỏ domain.
5. Xóa Security Group không dùng.
6. Vô hiệu hóa access key không dùng.

#### Kiểm tra RDS trước khi xóa

~~~bash
aws rds describe-db-instances --region ap-southeast-1
aws rds create-db-snapshot \
  --db-instance-identifier netflop-db \
  --db-snapshot-identifier netflop-db-final-snapshot \
  --region ap-southeast-1
~~~

Không xóa RDS nếu chưa có snapshot hoặc chưa export dữ liệu cần giữ.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Dọn dẹp tài nguyên application

Nếu chỉ nghỉ tạm thời, nên stop backend/frontend local và có thể stop EC2. Nếu môi trường demo đã kết thúc, mới terminate EC2.

#### Lệnh trên EC2

~~~bash
pm2 status
pm2 stop netflop-api
sudo systemctl stop nginx
~~~

#### Stop EC2 bằng CLI

~~~bash
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxxxx --region ap-southeast-1
~~~

#### RDS

Trước khi xóa RDS, tạo snapshot:

~~~bash
aws rds create-db-snapshot \
  --db-instance-identifier netflop-db \
  --db-snapshot-identifier netflop-db-before-cleanup
~~~


<!-- NETFLOP_IMPLEMENTATION_END -->
