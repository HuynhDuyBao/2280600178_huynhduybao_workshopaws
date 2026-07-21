---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
![So sanh FFmpeg tren EC2 voi AWS Elemental MediaConvert](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/3.1-Blog1/ffmpeg-vs-mediaconvert.svg)

Bài 1: Tại sao nhóm mình chọn AWS Elemental MediaConvert thay vì FFmpeg?

Xin chào mọi người,

Trong quá trình thực hiện dự án **Netflop - Website xem phim trên nền tảng AWS**, nhóm mình đã có cơ hội tìm hiểu và triển khai nhiều dịch vụ AWS để xây dựng một media pipeline hoàn chỉnh. Một trong những quyết định khiến nhóm mất khá nhiều thời gian để cân nhắc là lựa chọn giải pháp encode video.

Ban đầu, nhóm dự định sử dụng **FFmpeg chạy trực tiếp trên Amazon EC2** để chuyển đổi video sau khi upload. Tuy nhiên, sau khi thử nghiệm với nhiều video có dung lượng lớn, CPU của EC2 thường xuyên hoạt động ở mức cao, thời gian xử lý kéo dài và việc quản lý nhiều tác vụ encode đồng thời trở nên khá phức tạp.

Sau khi tìm hiểu thêm, nhóm quyết định chuyển sang sử dụng **AWS Elemental MediaConvert**, kết hợp với **Amazon S3, Amazon CloudFront, Amazon EventBridge và AWS Lambda** để xây dựng quy trình xử lý video tự động.

Sau một thời gian triển khai, nhóm nhận thấy một số lợi ích rõ rệt:

✅ Tự động chuyển đổi video sang chuẩn **HLS** với nhiều chất lượng (360p, 480p, 720p, 1080p).

✅ Giảm tải đáng kể cho EC2 vì toàn bộ quá trình encode được thực hiện bởi MediaConvert.

✅ Tự động cập nhật trạng thái phim sau khi encode hoàn tất thông qua **EventBridge + Lambda**.

✅ Kiến trúc dễ mở rộng khi số lượng video hoặc người dùng tăng lên.

Qua dự án này, nhóm mình nhận ra rằng việc sử dụng các **Managed Services** của AWS không chỉ giúp giảm khối lượng vận hành mà còn giúp hệ thống ổn định và dễ mở rộng hơn so với việc tự triển khai mọi thứ trên máy chủ.

Đây là một trải nghiệm rất thú vị đối với nhóm trong quá trình học tập và thực hiện dự án.

Không biết anh/chị và các bạn trong cộng đồng đã từng sử dụng **AWS Elemental MediaConvert** hoặc giải pháp nào khác cho hệ thống video streaming chưa? Rất mong được lắng nghe những chia sẻ và góp ý để nhóm có thể tiếp tục hoàn thiện dự án.

Xin cảm ơn mọi người đã dành thời gian đọc bài!

📚 Link tham khảo
[https://aws.amazon.com/mediaconvert/](https://aws.amazon.com/mediaconvert/)
[https://docs.aws.amazon.com/mediaconvert/latest/ug/what-is.html](https://docs.aws.amazon.com/mediaconvert/latest/ug/what-is.html?utm_source=chatgpt.com)
[https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-jobs.html](https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-jobs.html)
