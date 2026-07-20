---
title: "Dọn dẹp tài nguyên media và automation"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### Mục tiêu

Dọn dẹp tài nguyên media và automation không còn sử dụng để tránh phát sinh chi phí từ S3, CloudFront, Lambda, EventBridge và CloudWatch Logs.

#### S3

Các bucket cần kiểm tra:

```text
netflop-input-source
netflop-output-source
```

Tài nguyên cần chú ý:

* Video gốc MP4/MKV dung lượng lớn.
* HLS output gồm nhiều segment.
* File test upload lỗi.
* Avatar/banner/phụ đề test.

Nếu website còn chạy production thì không xóa media đang được database tham chiếu.

#### MediaConvert

MediaConvert tính phí theo thời lượng xử lý. Sau khi job hoàn thành, không cần cleanup job ngay, nhưng cần:

* Kiểm tra không có job bị retry liên tục.
* Kiểm tra backend không tạo job trùng cho cùng một tập.
* Ghi lại số lượng job demo trong báo cáo.

#### CloudFront

CloudFront distribution đang dùng để phát HLS nên chỉ disable/delete khi không còn chạy website. Nếu xóa S3 output nhưng giữ CloudFront, cần invalidation hoặc kiểm tra cache để tránh dữ liệu cũ.

#### Lambda và EventBridge

Các tài nguyên cần kiểm tra:

* Lambda `netflop-mediaconvert-notifier`.
* Lambda `netflop-subtitle-converter`.
* EventBridge rule `netflop-mediaconvert-job-state-change`.
* S3 notification trigger chuyển phụ đề.
* CloudWatch Logs của Lambda.

#### Kết quả mong đợi

* Không còn file media test dung lượng lớn.
* Không còn automation test chạy ngoài ý muốn.
* CloudFront/Lambda/EventBridge chỉ giữ lại phần đang dùng thật.

{{% notice info %}}
Cần thêm ảnh: S3 storage/object list, MediaConvert job history, CloudFront distribution, Lambda functions, EventBridge rule và CloudWatch log groups.
{{% /notice %}}

<!-- NETFLOP_DETAIL_START -->
#### Cách cleanup tài nguyên media

S3 input thường chứa video gốc dung lượng lớn. Nếu video đã convert xong và không cần giữ bản gốc, có thể chuyển sang lifecycle policy hoặc xóa sau một khoảng thời gian.

#### Lệnh kiểm tra dung lượng S3

~~~bash
aws s3 ls s3://netflop-input-source --recursive --summarize --human-readable
aws s3 ls s3://netflop-output-source --recursive --summarize --human-readable
~~~

#### Cleanup theo prefix test

Chỉ xóa prefix test/demo, không xóa prefix production đang được database tham chiếu:

~~~bash
aws s3 rm s3://netflop-input-source/deploy/frontend/ --recursive
aws s3 rm s3://netflop-input-source/tmp/ --recursive
~~~

#### Cleanup automation

Nếu không dùng pipeline nữa:

1. Disable EventBridge rule MediaConvert.
2. Remove S3 notification trigger của Lambda subtitle.
3. Xóa Lambda function demo.
4. Xóa CloudWatch log group không cần giữ.

Với website production, chỉ nên giữ lại hai Lambda đang dùng: MediaConvert notifier và subtitle converter.
<!-- NETFLOP_DETAIL_END -->
