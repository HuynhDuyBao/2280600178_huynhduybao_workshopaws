---
title: "Worklog Tuần 2"
date: 2026-04-27
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Tìm hiểu AWS Identity and Access Management (IAM) và nguyên tắc phân quyền tối thiểu.
* Làm quen với Amazon EC2, máy chủ ảo, AMI, instance type, key pair và Security Group.
* Thực hành tạo tài nguyên đơn giản nhưng vẫn kiểm soát chi phí.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Ôn lại nội dung tuần 1 <br> - Tìm hiểu IAM User, Group, Role và Policy <br> - Phân biệt quyền gán trực tiếp và quyền thông qua role | 27/04/2026 | 27/04/2026 | IAM |
| 3 | - Thực hành tạo IAM user phục vụ lab <br> - Cấu hình quyền theo nguyên tắc least privilege <br> - Đọc ví dụ policy JSON cơ bản | 28/04/2026 | 28/04/2026 | <https://cloudjourney.awsstudygroup.com/1-explore/> |
| 4 | - Tìm hiểu Amazon EC2 <br> - Ghi chú các khái niệm: AMI, instance type, EBS volume, public IP, Elastic IP, key pair | 29/04/2026 | 29/04/2026 | EC2 |
| 5 | - Tạo EC2 instance thử nghiệm trong Free Tier <br> - Cấu hình Security Group cho SSH/HTTP <br> - Kết nối vào máy chủ bằng SSH | 30/04/2026 | 30/04/2026 | EC2, Security Group |
| 6 | - Cài web server cơ bản trên EC2 <br> - Kiểm tra truy cập qua trình duyệt <br> - Dừng hoặc xóa tài nguyên không cần thiết để tránh phát sinh chi phí | 01/05/2026 | 01/05/2026 | AWS Console, AWS CLI |

### Kết quả đạt được tuần 2:

* Nắm được IAM là dịch vụ quản lý danh tính và quyền truy cập trong AWS.
* Hiểu vì sao không nên dùng tài khoản root cho công việc hằng ngày.
* Tạo được EC2 instance cơ bản và kết nối SSH thành công.
* Biết cách mở port phù hợp bằng Security Group và kiểm tra truy cập ứng dụng đơn giản.
* Có thói quen dọn dẹp tài nguyên sau khi thực hành để kiểm soát chi phí.
