---
title: "Worklog Tuần 5"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Tìm hiểu Amazon RDS và cách AWS hỗ trợ quản lý cơ sở dữ liệu quan hệ.
* Thực hành tạo database, cấu hình bảo mật và kết nối từ ứng dụng.
* Chuẩn bị mô hình dữ liệu cơ bản cho website xem phim.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu Amazon RDS <br> - So sánh database tự cài trên EC2 và database được quản lý bởi RDS | 18/05/2026 | 18/05/2026 | RDS |
| 3 | - Tạo RDS database thử nghiệm <br> - Cấu hình engine, instance class, storage và backup phù hợp lab | 19/05/2026 | 19/05/2026 | <https://cloudjourney.awsstudygroup.com/1-explore/> |
| 4 | - Cấu hình Security Group cho RDS <br> - Kết nối từ máy client hoặc EC2 vào database <br> - Kiểm tra truy vấn cơ bản | 20/05/2026 | 20/05/2026 | RDS, VPC |
| 5 | - Thiết kế bảng dữ liệu cho website xem phim: users, movies, categories, episodes, watch_history <br> - Xác định quan hệ giữa các bảng chính | 21/05/2026 | 21/05/2026 | Tài liệu dự án |
| 6 | - Viết script tạo bảng mẫu <br> - Nhập dữ liệu thử nghiệm <br> - Ghi chú cách backup và xóa tài nguyên RDS sau khi lab | 22/05/2026 | 22/05/2026 | SQL, RDS |

### Kết quả đạt được tuần 5:

* Hiểu được RDS giúp giảm công việc vận hành database như backup, patching và cấu hình cơ bản.
* Tạo và kết nối được database thử nghiệm trên AWS.
* Biết cách giới hạn truy cập database bằng Security Group.
* Xây dựng được mô hình dữ liệu ban đầu cho website xem phim.
* Có kinh nghiệm kiểm soát chi phí khi thực hành với RDS bằng cách chọn cấu hình nhỏ và xóa tài nguyên sau lab.
