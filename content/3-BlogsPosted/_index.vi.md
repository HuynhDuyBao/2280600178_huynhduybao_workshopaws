---
title: "Các bài blogs đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---



Tại đây là phần liệt kê và giới thiệu các blog nhóm đã đăng trong quá trình thực hiện dự án **Netflop - Website xem phim trên nền tảng AWS**.

### [Blog 1 - Tại sao nhóm mình chọn AWS Elemental MediaConvert thay vì FFmpeg?](3.1-Blog1/)

![Bai dang Facebook Blog 1](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/blog1-facebook-post.svg)

Blog này chia sẻ lý do nhóm lựa chọn **AWS Elemental MediaConvert** thay vì tự chạy **FFmpeg trên Amazon EC2** để encode video. Nội dung tập trung vào các lợi ích như giảm tải cho EC2, tự động chuyển đổi video sang HLS nhiều chất lượng và giúp media pipeline dễ mở rộng hơn.


### [Blog 2 - Tại sao nhóm mình dùng Amazon CloudFront thay vì phát video trực tiếp từ S3?](3.2-Blog2/)

![Bai dang Facebook Blog 2](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/blog2-facebook-post.svg)

Blog này chia sẻ kinh nghiệm sử dụng **Amazon CloudFront** để phân phối video HLS trong website xem phim. Nhóm trình bày các lợi ích như giảm độ trễ, cache tại Edge Locations, giảm request trực tiếp đến S3, hỗ trợ HTTPS và bảo vệ nội dung bằng CloudFront Signed Cookies.

### [Blog 3 - Xây dựng Media Pipeline theo hướng Event-Driven trên AWS](3.3-Blog3/)

![AWS media workflow with S3, MediaConvert, CloudFront, Lambda, EventBridge, CloudWatch va SNS](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/3.3-Blog3/mediaconvert-workflow.svg)

Blog này trình bày cách nhóm sử dụng mô hình **Event-Driven Architecture** với luồng **MediaConvert → EventBridge → Lambda → Backend Webhook** để tự động cập nhật trạng thái tập phim sau khi encode hoàn tất, thay vì để Backend polling liên tục.
Nguồn Facebook: https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2218428612255432

