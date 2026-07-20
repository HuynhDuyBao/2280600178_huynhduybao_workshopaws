---
title: "Worklog Tuần 9"
date: 2026-06-15
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Xây dựng backend API cho website xem phim.
* Kết nối backend với database đã thiết kế ở tuần 5.
* Bổ sung chức năng đăng nhập, phân quyền cơ bản và quản lý dữ liệu phim.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khởi tạo backend project <br> - Thiết kế cấu trúc route, controller, service và database connection | 15/06/2026 | 15/06/2026 | Backend |
| 3 | - Xây dựng API quản lý phim: danh sách, chi tiết, thêm, sửa, xóa <br> - Kiểm thử API bằng công cụ test request | 16/06/2026 | 16/06/2026 | API |
| 4 | - Kết nối backend với RDS hoặc database local mô phỏng RDS <br> - Kiểm tra truy vấn danh sách phim, danh mục và tập phim | 17/06/2026 | 17/06/2026 | RDS |
| 5 | - Xây dựng chức năng đăng nhập cơ bản <br> - Phân quyền người dùng thường và quản trị viên <br> - Bảo vệ các API quản trị | 18/06/2026 | 18/06/2026 | IAM concept, Auth |
| 6 | - Triển khai thử backend trên EC2 <br> - Cấu hình Security Group, biến môi trường và kết nối database <br> - Ghi log lỗi để phục vụ kiểm thử | 19/06/2026 | 19/06/2026 | EC2, CloudWatch |

### Kết quả đạt được tuần 9:

* Xây dựng được backend API cơ bản cho hệ thống website xem phim.
* Kết nối được backend với database và truy xuất dữ liệu phim.
* Có cơ chế đăng nhập và phân quyền ở mức phù hợp với bài thực tập.
* Triển khai thử backend trên EC2 và hiểu các bước cấu hình môi trường chạy ứng dụng.
* Nhận diện được các vấn đề thường gặp khi backend kết nối database trên AWS như Security Group, endpoint, username/password và biến môi trường.
