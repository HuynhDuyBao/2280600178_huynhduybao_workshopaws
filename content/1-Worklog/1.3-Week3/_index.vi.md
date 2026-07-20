---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu nền tảng mạng trong AWS thông qua Amazon VPC.
* Thực hành tạo subnet, route table, Internet Gateway và Security Group.
* Củng cố kiến thức EC2 bằng cách triển khai máy chủ trong VPC tự cấu hình.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu VPC, CIDR block, public subnet, private subnet <br> - Vẽ sơ đồ mạng đơn giản cho một ứng dụng web | 04/05/2026 | 04/05/2026 | VPC |
| 3 | - Tạo VPC thực hành <br> - Tạo public subnet, route table và Internet Gateway <br> - Gắn route ra Internet cho public subnet | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/1-explore/> |
| 4 | - Tạo EC2 trong public subnet <br> - Cấu hình Security Group cho SSH và HTTP <br> - Kiểm tra kết nối SSH từ máy cá nhân | 06/05/2026 | 06/05/2026 | EC2, VPC |
| 5 | - Thực hành gắn thêm EBS volume <br> - Format, mount volume và kiểm tra lưu trữ trên EC2 <br> - Ghi chú khác biệt giữa instance storage và EBS | 07/05/2026 | 07/05/2026 | EBS |
| 6 | - Kiểm tra toàn bộ luồng kết nối từ Internet vào EC2 <br> - Chụp lại sơ đồ, câu lệnh và cấu hình chính <br> - Xóa tài nguyên lab sau khi hoàn tất | 08/05/2026 | 08/05/2026 | AWS Console, AWS CLI |

### Kết quả đạt được tuần 3:

* Hiểu vai trò của VPC trong việc cô lập và tổ chức mạng trên AWS.
* Tự tạo được một mô hình mạng cơ bản gồm VPC, subnet, route table và Internet Gateway.
* Triển khai được EC2 trong subnet tự cấu hình và truy cập thành công qua SSH/HTTP.
* Biết cách gắn thêm EBS volume và kiểm tra dung lượng lưu trữ.
* Có khả năng đọc sơ đồ mạng AWS đơn giản và giải thích luồng truy cập vào ứng dụng.
