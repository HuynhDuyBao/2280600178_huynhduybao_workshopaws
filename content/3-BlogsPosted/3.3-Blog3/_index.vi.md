---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
📝 Bài 3: Chia sẻ một trải nghiệm khi xây dựng Media Pipeline trên AWS

Xin chào mọi người,

Trong dự án **Netflop**, sau khi video được **AWS Elemental MediaConvert** xử lý xong, nhóm cần cập nhật trạng thái phim để người dùng có thể xem ngay.

Ban đầu nhóm nghĩ đến việc Backend liên tục kiểm tra trạng thái Job (Polling), nhưng cách này vừa tốn tài nguyên vừa không hiệu quả.

Sau đó nhóm chuyển sang mô hình **Event-Driven Architecture** với:

**MediaConvert → EventBridge → Lambda → Backend Webhook**

Luồng hoạt động như sau:

📌 MediaConvert hoàn thành Job.

📌 EventBridge tự động nhận sự kiện.

📌 Lambda được kích hoạt và gọi Webhook về Backend.

📌 Backend cập nhật trạng thái tập phim từ **Processing** sang **Ready**.

Nhờ đó:

✅ Không cần Polling liên tục.

✅ Backend nhẹ hơn.

✅ Hệ thống phản hồi gần như ngay khi encode hoàn tất.

Qua dự án này, nhóm mình thấy **EventBridge** là một dịch vụ rất hữu ích để xây dựng các workflow tự động giữa các dịch vụ AWS.

👉 Anh/chị và các bạn thường sử dụng **EventBridge**, **Amazon SQS** hay **AWS Step Functions** trong các hệ thống Event-Driven? Rất mong được trao đổi thêm!

📚 Link tham khảo
[https://docs.aws.amazon.com/mediaconvert/latest/ug/eventbridge_events.html](https://docs.aws.amazon.com/mediaconvert/latest/ug/eventbridge_events.html?utm_source=chatgpt.com)
[https://docs.aws.amazon.com/eventbridge/latest/ref/events-ref-mediaconvert.html](https://docs.aws.amazon.com/eventbridge/latest/ref/events-ref-mediaconvert.html?utm_source=chatgpt.com)
[https://docs.aws.amazon.com/mediaconvert/latest/ug/mediaconvert_event_list.html](https://docs.aws.amazon.com/mediaconvert/latest/ug/mediaconvert_event_list.html?utm_source=chatgpt.com)
[https://docs.aws.amazon.com/lambda/latest/dg/welcome.html](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
