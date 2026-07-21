---
title: "Kiến trúc tổng quan"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.1.1. </b> "
---

#### Mục tiêu kiến trúc

Kiến trúc Netflop tách ứng dụng web, database, lưu trữ media, xử lý video, phát nội dung và giám sát thành các lớp riêng. Cách tách này giúp hệ thống dễ vận hành hơn, hạn chế việc lưu file lớn trên EC2 và tạo nền tảng để mở rộng về sau.

#### Luồng truy cập chính

```text
User / Admin
   |
   v
Cloudflare DNS + HTTPS
   |
   v
netflop.win
   |
   v
EC2 netflop-web
   |-- Nginx reverse proxy
   |-- React frontend build
   |-- Node.js backend API
   |
   |---- RDS MySQL netflop-db
   |
   |---- S3 netflop-input-source
   |          |
   |          v
   |      AWS Elemental MediaConvert
   |          |
   |          v
   |---- S3 netflop-output-source
              |
              v
        CloudFront protected HLS
              |
              v
        Video Player
```

#### Luồng người dùng

1. Người dùng truy cập `https://netflop.win`.
2. Cloudflare xử lý DNS và HTTPS phía public domain.
3. Request được chuyển đến EC2.
4. Nginx trả frontend React hoặc proxy request `/api/*` đến backend Node.js.
5. Backend đọc/ghi dữ liệu phim, tài khoản, lịch sử xem, tiếp tục xem, yêu thích, đánh giá và bình luận trong RDS MySQL.
6. Khi người dùng xem phim, backend cấp phiên stream và trình phát lấy HLS từ CloudFront.

#### Luồng admin upload video

1. Admin chọn phim, tập phim và file video MP4/MKV trên trang quản trị.
2. Backend tạo bản ghi tập phim ở trạng thái đang xử lý.
3. File gốc được upload lên S3 bucket `netflop-input-source`.
4. Backend tạo job MediaConvert.
5. MediaConvert chuyển video sang HLS nhiều độ phân giải: 360p, 480p, 720p và 1080p.
6. Output HLS được lưu vào S3 bucket `netflop-output-source`.
7. EventBridge bắt trạng thái MediaConvert và gọi Lambda.
8. Lambda gọi webhook backend để cập nhật trạng thái tập phim trong database.
9. Website tự hiển thị trạng thái hoàn thành, không cần admin bấm sync thủ công.

#### Luồng tự động cập nhật trạng thái

```text
MediaConvert COMPLETE / ERROR / CANCELED
   |
   v
EventBridge rule netflop-mediaconvert-job-state-change
   |
   v
Lambda netflop-mediaconvert-notifier
   |
   v
Backend webhook /api/uploads/mediaconvert/events
   |
   v
Update episode upload_status in RDS
```

#### Luồng phụ đề

```text
Admin upload .srt hoặc .vtt
   |
   v
S3 input/output
   |
   v
Lambda chuyển .srt sang .vtt nếu cần
   |
   v
Video Player load subtitle track
```

{{% notice info %}}
Cần thêm ảnh: sơ đồ luồng truy cập chính; sơ đồ upload video -> MediaConvert -> CloudFront; sơ đồ EventBridge -> Lambda -> Backend webhook; sơ đồ phụ đề SRT -> VTT.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.1-Workshop-overview/5.1.1-architecture/sodo.jpg)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện kiến trúc

Kiến trúc được triển khai theo từng lớp:

1. Lớp domain: Cloudflare quản lý DNS của <code>netflop.win</code>.
2. Lớp application: EC2 chạy Nginx, frontend React build và backend Node.js.
3. Lớp database: RDS MySQL lưu dữ liệu phim và người dùng.
4. Lớp media: S3 input lưu video gốc, MediaConvert tạo HLS, S3 output lưu kết quả.
5. Lớp delivery: CloudFront phân phối HLS và bảo vệ stream bằng signed cookies.
6. Lớp automation: EventBridge và Lambda cập nhật trạng thái MediaConvert.
7. Lớp monitoring: CloudWatch và SNS theo dõi hệ thống.

#### File code liên quan trong source

| Thành phần | File liên quan |
| --- | --- |
| Kết nối RDS | <code>backend/src/config/database.js</code> |
| Upload S3 | <code>backend/src/services/awsS3.service.js</code> |
| Tạo MediaConvert job | <code>backend/src/services/mediaConvert.service.js</code> |
| Webhook MediaConvert | <code>backend/src/controllers/upload.controller.js</code> |
| Signed cookies | <code>backend/src/services/cloudFrontSignedCookie.service.js</code> |
| Player HLS | <code>frontend/src/components/VideoPlayer.jsx</code> |
| Lambda phụ đề | <code>aws/lambda/subtitle-converter/index.mjs</code> |

#### Code mẫu: luồng backend tạo job xử lý video

~~~js
const uploaded = await awsS3Service.uploadVideo(req.file, { movieId, episodeName });
const pendingEpisode = await episodeModel.createUploadEpisode({
  movieId,
  name: episodeName,
  sourceUrl: uploaded.s3Uri,
  uploadStatus: 'uploaded'
});

const job = await mediaConvertService.createHlsJob({
  inputS3Uri: uploaded.s3Uri,
  movieId,
  episodeId: pendingEpisode.MaTap
});
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Kiến trúc triển khai theo project Netflop

Kiến trúc hiện tại vẫn là mô hình web truyền thống chạy trên EC2, kết hợp các dịch vụ managed của AWS cho database, media, CDN và monitoring. Backend Node.js không lưu file video lớn trên ổ đĩa EC2, mà chỉ điều phối upload và lưu metadata vào RDS.

#### Luồng request chính

1. Người dùng truy cập <code>netflop.win</code> qua Cloudflare.
2. Nginx trên EC2 phục vụ frontend React đã build.
3. Các request <code>/api/*</code> được reverse proxy về Node.js backend chạy bằng PM2.
4. Backend đọc/ghi dữ liệu phim, người dùng, tập phim, lịch sử xem trong RDS MySQL.
5. Video admin upload đi vào S3 input bucket.
6. Backend tạo MediaConvert job để xuất HLS sang S3 output bucket.
7. Player phát HLS qua CloudFront, có thể bảo vệ bằng signed cookies.

#### Sơ đồ luồng upload và xem phim

~~~text
Admin browser
  -> React admin
  -> Backend API /uploads/videos/multipart/*
  -> S3 input bucket
  -> MediaConvert
  -> S3 output bucket
  -> CloudFront
  -> Video player
~~~

#### File source thể hiện kiến trúc

| Mục | File |
| --- | --- |
| Khởi tạo server | <code>backend/src/server.js</code> |
| Route tổng | <code>backend/src/routes/index.js</code> |
| Kết nối RDS | <code>backend/src/config/database.js</code> |
| Upload S3 | <code>backend/src/services/awsS3.service.js</code> |
| Tạo MediaConvert job | <code>backend/src/services/mediaConvert.service.js</code> |
| Player HLS | <code>frontend/src/components/VideoPlayer.jsx</code> |

#### Code mẫu kết nối database

~~~js
const pool = mysql.createPool({
  host: env.db.host,
  port: env.db.port,
  database: env.db.database,
  user: env.db.user,
  password: env.db.password,
  waitForConnections: true,
  connectionLimit: env.db.connectionLimit,
  charset: 'utf8mb4',
  namedPlaceholders: true
});

pool.execute = pool.query.bind(pool);
~~~

{{% notice info %}}
Cần thêm ảnh: sơ đồ kiến trúc tự vẽ, EC2 đang chạy, RDS available, S3 buckets, CloudFront distribution và MediaConvert job.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
