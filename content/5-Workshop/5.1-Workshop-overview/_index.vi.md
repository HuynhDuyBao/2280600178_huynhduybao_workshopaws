---
title: "Tổng quan workshop"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Phần này giới thiệu kiến trúc tổng thể của website xem phim **Netflop** trên AWS và vai trò của từng nhóm dịch vụ. Đây là phần nền để người đọc hiểu cách request đi từ người dùng đến hệ thống, cách admin upload video và cách video được xử lý tự động trước khi phát trên web.

Netflop không chỉ là một website CRUD đơn giản. Hệ thống có pipeline media riêng gồm upload file lớn, lưu file gốc, chuyển mã, lưu HLS output, phát qua CDN, bảo vệ link stream và cập nhật trạng thái tập phim tự động.

{{% notice info %}}
Cần thêm ảnh: sơ đồ kiến trúc tổng quan Netflop trên AWS; nên thể hiện rõ nhóm Application, Database, Media Processing, CDN/Security và Monitoring.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.1-Workshop-overview/sodo.jpg)

#### Nội dung

1. [Kiến trúc tổng quan](5.1.1-architecture/)
2. [Bảng dịch vụ và vai trò](5.1.2-service-map/)

<!-- NETFLOP_DETAIL_START -->
#### Cách trình bày tổng quan

Ở phần tổng quan, nên trình bày theo hai góc nhìn:

1. Góc nhìn người dùng: mở web, đăng nhập, chọn phim, xem phim, tiếp tục xem.
2. Góc nhìn admin: thêm phim, upload tập phim, theo dõi tiến trình convert, thêm phụ đề.

Sau đó liên kết từng chức năng với dịch vụ AWS tương ứng. Ví dụ, chức năng upload tập phim không chỉ nằm ở frontend mà còn đi qua backend, S3 input, MediaConvert, S3 output, CloudFront và RDS.

#### Mẫu mô tả ngắn trong báo cáo

Netflop được triển khai theo mô hình ứng dụng web kết hợp media pipeline. EC2 chạy ứng dụng chính, RDS lưu dữ liệu nghiệp vụ, S3 lưu file media, MediaConvert xử lý video, CloudFront phân phối HLS và Lambda xử lý các tác vụ tự động. Cách triển khai này giúp giảm tải cho EC2 vì file video lớn không được lưu lâu dài trên ổ đĩa máy chủ.
<!-- NETFLOP_DETAIL_END -->
