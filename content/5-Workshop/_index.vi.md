---
title: "Workshop"
date: 2026-07-10
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai website xem phim Netflop trên AWS

Phần workshop mô tả quá trình triển khai website xem phim **Netflop** lên môi trường AWS. Nội dung được viết theo luồng thực tế của hệ thống hiện tại: người dùng truy cập domain `netflop.win`, frontend/backend chạy trên EC2, dữ liệu lưu trong RDS MySQL, media được lưu trên S3, video được chuyển mã bằng AWS Elemental MediaConvert và phát qua CloudFront.

Mục tiêu của chương này là trình bày được cách xây dựng một hệ thống xem phim có khả năng upload video dung lượng lớn, tự động chuyển đổi sang HLS nhiều độ phân giải, phát video qua CDN, lưu lịch sử xem theo tài khoản, hỗ trợ phụ đề và theo dõi trạng thái vận hành bằng CloudWatch.

#### Nội dung

1. [Tổng quan workshop](5.1-Workshop-overview/)
2. [Chuẩn bị môi trường](5.2-Prerequisite/)
3. [Tạo hạ tầng AWS](5.3-AWS-infrastructure/)
4. [Cấu hình lưu trữ, database và xử lý media](5.4-Storage-database/)
5. [Triển khai và kiểm thử ứng dụng](5.5-Deploy-test/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)

#### Kết quả sau workshop

Sau khi hoàn thành, hệ thống có các thành phần chính:

* Website Netflop chạy tại `https://netflop.win`.
* EC2 chạy Nginx, React frontend và Node.js backend API.
* RDS MySQL lưu database `web_xem_phim_final`.
* S3 lưu video gốc, HLS output, ảnh phim, avatar và phụ đề.
* MediaConvert tự động tạo HLS 360p, 480p, 720p và 1080p.
* CloudFront phân phối HLS và bảo vệ stream bằng signed cookies.
* Lambda xử lý tác vụ tự động: nhận event MediaConvert và chuyển phụ đề `.srt` sang `.vtt`.
* CloudWatch/SNS theo dõi EC2, RDS, Lambda và cảnh báo khi có lỗi.

<!-- NETFLOP_DETAIL_START -->
#### Cách sử dụng phần workshop

Khi trình bày chương này trong báo cáo, mỗi mục nên đi theo cùng một cấu trúc:

1. Nêu chức năng cần triển khai.
2. Chỉ ra dịch vụ AWS hoặc thành phần ứng dụng liên quan.
3. Mô tả các bước thực hiện.
4. Đưa code mẫu hoặc lệnh kiểm tra ngắn.
5. Chụp ảnh kết quả sau khi thực hiện.

Các đoạn code mẫu trong chương này được rút gọn từ source Netflop. Mục đích của code mẫu là giải thích cách hệ thống hoạt động, không đưa các giá trị bí mật như access key, private key, JWT secret, database password hoặc OAuth client secret vào báo cáo.

#### Các chức năng chính cần chứng minh

| Chức năng | Thành phần thực hiện | Kết quả cần chứng minh |
| --- | --- | --- |
| Người dùng truy cập web | Cloudflare, EC2, Nginx, React | Website mở được tại https://netflop.win |
| API backend | Nginx reverse proxy, Node.js, PM2 | API trả JSON và backend online |
| Lưu dữ liệu | RDS MySQL | Dữ liệu phim, tập phim, tài khoản được đọc từ RDS |
| Upload video | React admin, Node.js, S3 | File gốc vào S3 input |
| Convert video | MediaConvert | HLS 360p/480p/720p/1080p trong S3 output |
| Cập nhật trạng thái tự động | EventBridge, Lambda, webhook backend | Tập phim tự chuyển sang ready |
| Phát video | CloudFront, signed cookies, VideoPlayer | Player phát HLS qua HTTPS |
| Phụ đề | S3, Lambda, WebVTT | SRT được chuyển sang VTT và hiển thị trên player |
| Theo dõi hệ thống | CloudWatch, SNS | Alarm và log hoạt động |
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Bổ sung phạm vi triển khai thực tế

Phần workshop này mô tả lại quá trình triển khai website Netflop từ môi trường local lên AWS. Nội dung được viết theo hướng có thể thực hành lại, gồm các bước cấu hình hạ tầng, triển khai ứng dụng, xử lý video, bảo vệ stream, theo dõi vận hành và dọn dẹp tài nguyên.

Các đoạn code minh họa được rút gọn từ project Netflop hiện tại. Khi đưa vào báo cáo, chỉ trích những đoạn thể hiện logic chính, không đưa mật khẩu database, access key, private key CloudFront, JWT secret hoặc OAuth client secret.

#### Cách trình bày mỗi chức năng

1. Nêu mục tiêu chức năng.
2. Nêu dịch vụ AWS hoặc thành phần ứng dụng liên quan.
3. Mô tả các bước thực hiện trên console/CLI.
4. Mô tả luồng xử lý trong source code.
5. Đưa code mẫu ngắn từ project.
6. Ghi kết quả kiểm thử và ảnh minh họa cần chụp.

#### Chức năng chính trong workshop

| Nhóm | Chức năng triển khai | Bằng chứng cần có |
| --- | --- | --- |
| Hạ tầng | EC2, Nginx, PM2, RDS, Security Group | Web truy cập được, API health check thành công |
| Lưu trữ media | S3 input/output, CloudFront | File gốc ở input bucket, HLS ở output bucket |
| Xử lý video | MediaConvert, EventBridge, Lambda notifier | Job COMPLETE và trạng thái tập phim tự cập nhật |
| Phụ đề | Upload VTT/SRT, Lambda chuyển SRT sang VTT | Track phụ đề hiển thị trong player |
| Bảo mật stream | CloudFront signed cookies | HLS phát qua CloudFront, cookie có TTL |
| Vận hành | CloudWatch, SNS, Cost Explorer | Có alarm, log, dashboard hoặc ảnh chi phí |

#### Demo stream

<div class="netflop-demo-player">
  <video id="netflop-demo-video-vi" controls playsinline preload="metadata" style="width:100%;max-width:960px;background:#000;border-radius:8px;"></video>
  <p id="netflop-demo-status-vi">
    <a href="https://customer-mq3bsojkqgoa0nyg.cloudflarestream.com/8bc4e084d7f19b8303da087366e0fe91/manifest/video.m3u8">Mở HLS demo stream</a>
  </p>
</div>

<script src="https://cdn.jsdelivr.net/npm/hls.js@1/dist/hls.min.js"></script>
<script>
(function () {
  const source = "https://customer-mq3bsojkqgoa0nyg.cloudflarestream.com/8bc4e084d7f19b8303da087366e0fe91/manifest/video.m3u8";
  const video = document.getElementById("netflop-demo-video-vi");
  const status = document.getElementById("netflop-demo-status-vi");
  if (!video) return;

  if (video.canPlayType("application/vnd.apple.mpegurl")) {
    video.src = source;
    return;
  }

  if (window.Hls && window.Hls.isSupported()) {
    const hls = new Hls();
    hls.loadSource(source);
    hls.attachMedia(video);
    hls.on(Hls.Events.ERROR, function () {
      if (status) status.textContent = "Không tải được HLS demo stream trên trình duyệt này.";
    });
    return;
  }

  if (status) status.textContent = "Trình duyệt này không hỗ trợ phát HLS.";
})();
</script>
<!-- NETFLOP_IMPLEMENTATION_END -->
