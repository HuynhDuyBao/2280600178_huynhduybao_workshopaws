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

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Kịch bản thực hiện workshop

Workshop được xây dựng theo kịch bản một website xem phim có hai nhóm người dùng:

* Người dùng thường: đăng ký, đăng nhập, xem phim, chọn tập, bật phụ đề, tiếp tục xem, yêu thích và đánh giá phim.
* Quản trị viên: thêm phim, thêm tập phim, upload video lớn lên S3, theo dõi tiến trình MediaConvert và quản lý phụ đề/banner tập.

#### Luồng demo nên trình bày

1. Truy cập domain <code>https://netflop.win</code>.
2. Kiểm tra danh sách phim, trang chi tiết phim và trang xem phim.
3. Đăng nhập tài khoản admin.
4. Upload một tập phim MP4/MKV ở trang quản trị.
5. Kiểm tra file đã vào S3 input bucket.
6. Kiểm tra MediaConvert tạo HLS trong S3 output bucket.
7. Mở tập phim trên website để phát từ CloudFront.
8. Upload phụ đề SRT/VTT và kiểm tra subtitle selector.
9. Kiểm tra lịch sử xem lưu theo tài khoản.

#### Các thành phần project liên quan

| Thành phần | Đường dẫn trong source |
| --- | --- |
| Backend API | <code>backend/src</code> |
| Frontend React | <code>frontend/src</code> |
| Admin upload | <code>frontend/src/admin</code> |
| Lambda MediaConvert notifier | <code>aws/lambda/mediaconvert-notifier</code> |
| Lambda subtitle converter | <code>aws/lambda/subtitle-converter</code> |
| Database dump | <code>database/web_xem_phim_final_dump.sql</code> |

{{% notice info %}}
Cần thêm ảnh: trang chủ, trang chi tiết phim, trang watch, trang admin và sơ đồ phân quyền user/admin.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
