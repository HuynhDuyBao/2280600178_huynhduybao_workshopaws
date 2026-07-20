---
title: "Cấu hình lưu trữ, database và xử lý media"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Phần này tập trung vào pipeline media của Netflop. Đây là phần quan trọng nhất của website xem phim vì admin cần upload video dung lượng lớn, hệ thống tự convert sang HLS nhiều chất lượng, lưu output lên S3 và phát qua CloudFront.

{{% notice info %}}
Cần thêm ảnh: S3 input/output buckets, MediaConvert job hoàn thành, CloudFront HLS distribution, Lambda subtitle converter và video player đang phát phim.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/lamda.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/inputsub.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/outputsub.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/player1.png)

#### Nội dung

1. [Cấu hình S3 input/output buckets](5.4.1-s3-buckets/)
2. [Cấu hình MediaConvert](5.4.2-mediaconvert/)
3. [Cấu hình CloudFront phát HLS](5.4.3-cloudfront-hls/)
4. [Cấu hình Lambda chuyển phụ đề](5.4.4-subtitle-converter/)

<!-- NETFLOP_DETAIL_START -->
#### Cách triển khai pipeline media

Pipeline media của Netflop gồm 6 bước:

1. Admin chọn video trong trang quản trị.
2. Frontend upload file lên backend/S3 và hiển thị progress.
3. Backend lưu video gốc vào S3 input.
4. Backend tạo MediaConvert job.
5. MediaConvert xuất HLS vào S3 output.
6. CloudFront phát HLS cho VideoPlayer.

Với file lớn, upload được chia thành nhiều part để tránh lỗi <code>413 Request Entity Too Large</code> và giảm rủi ro timeout.

#### API chính của pipeline

| API | Mục đích |
| --- | --- |
| POST <code>/api/uploads/videos</code> | Upload video thường |
| POST <code>/api/uploads/videos/multipart/start</code> | Bắt đầu multipart upload |
| POST <code>/api/uploads/videos/multipart/parts</code> | Upload từng part |
| POST <code>/api/uploads/videos/multipart/complete</code> | Ghép part và tạo MediaConvert job |
| POST <code>/api/uploads/videos/:id/sync</code> | Kiểm tra trạng thái thủ công nếu cần |
| POST <code>/api/stream/session</code> | Cấp signed cookies cho CloudFront stream |
<!-- NETFLOP_DETAIL_END -->
