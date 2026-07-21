---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
![S3, MediaConvert, CloudFront va website streaming flow](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/3.2-Blog2/s3-cloudfront-streaming.svg)

📝 Bài 2: Chia sẻ một kinh nghiệm nhỏ khi xây dựng website xem phim trên AWS

Xin chào mọi người,

Trong quá trình phát triển dự án **Netflop**, nhóm mình từng đặt câu hỏi:

**Tại sao không phát video trực tiếp từ Amazon S3 mà lại phải dùng thêm Amazon CloudFront?**

Sau khi tìm hiểu và triển khai thực tế, nhóm nhận thấy CloudFront mang lại rất nhiều lợi ích.

Thay vì người dùng truy cập trực tiếp vào S3, toàn bộ video **HLS** được phân phối thông qua **Amazon CloudFront**.

Một số lợi ích mà nhóm nhận thấy:

✅ Giảm độ trễ khi xem video.

✅ Tăng tốc độ tải nhờ cơ chế Cache tại Edge Locations.

✅ Giảm số lượng request trực tiếp đến S3.

✅ Hỗ trợ HTTPS mặc định.

✅ Có thể bảo vệ nội dung bằng **CloudFront Signed Cookies**, chỉ người dùng đã được xác thực mới xem được video.

Đối với các website có nhiều nội dung media như xem phim hoặc học trực tuyến, CloudFront thực sự là một dịch vụ rất đáng để tìm hiểu.

👉 Không biết mọi người thường sử dụng **CloudFront**, **S3 trực tiếp** hay CDN khác khi xây dựng hệ thống streaming? Rất mong được học hỏi thêm từ mọi người.

📚 Link tham khảo
[https://aws.amazon.com/cloudfront/](https://aws.amazon.com/cloudfront/)
[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html?utm_source=chatgpt.com)
[https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html?utm_source=chatgpt.com)


