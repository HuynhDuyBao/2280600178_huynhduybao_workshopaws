---
title: "Cấu hình Lambda chuyển phụ đề"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Mục tiêu

Video player trên web cần phụ đề định dạng WebVTT (`.vtt`). Vì admin có thể upload phụ đề `.srt`, hệ thống sử dụng Lambda `netflop-subtitle-converter` để tự động chuyển `.srt` sang `.vtt`.

#### Luồng xử lý phụ đề

```text
Admin upload .srt
  |
  v
S3 input subtitle-input/
  |
  v
Lambda netflop-subtitle-converter
  |
  v
S3 output subtitles/*.vtt
  |
  v
Backend lưu URL phụ đề
  |
  v
Video Player hiển thị subtitle track
```

#### Trường hợp upload

| Loại file | Cách xử lý |
| --- | --- |
| `.vtt` | Upload trực tiếp lên S3 output và lưu vào database |
| `.srt` | Upload vào S3 input, Lambda convert sang `.vtt`, sau đó backend/player dùng file `.vtt` |

#### Trigger S3

Lambda được kích hoạt khi có object mới:

```text
Bucket: netflop-input-source
Prefix: subtitle-input/
Suffix: .srt
Event: s3:ObjectCreated:*
```

#### Lưu ý cho admin UI

Trang quản trị nên hiển thị rõ:

* Tập phim đang gắn phụ đề.
* Ngôn ngữ phụ đề, ví dụ Tiếng Việt.
* File upload là `.srt` hay `.vtt`.
* Trạng thái convert nếu upload `.srt`.
* URL phụ đề sau khi đã chuyển sang `.vtt`.

#### Kiểm tra

1. Upload một file `.srt` từ trang quản trị.
2. Kiểm tra file xuất hiện trong `netflop-input-source/subtitle-input/`.
3. Vào CloudWatch Logs của Lambda `netflop-subtitle-converter`.
4. Kiểm tra file `.vtt` được tạo trong S3 output.
5. Mở video player và bật phụ đề.
6. Kiểm tra phụ đề chỉ hiển thị một lớp, không bị trùng trên mobile.

{{% notice info %}}
Cần thêm ảnh: admin upload phụ đề, file `.srt` input, S3 trigger, Lambda log, file `.vtt` output và phụ đề hiển thị đúng trên player desktop/mobile.
{{% /notice %}}

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện upload và chuyển phụ đề

Khi admin upload phụ đề:

1. Nếu file là <code>.vtt</code>, backend upload trực tiếp vào S3 output.
2. Nếu file là <code>.srt</code>, backend upload source vào S3 input với metadata output bucket/key.
3. Backend có thể chuyển nhanh SRT sang VTT để trả URL cho web.
4. Lambda <code>netflop-subtitle-converter</code> vẫn xử lý tự động khi có object <code>.srt</code> ở prefix <code>subtitle-input/</code>.

#### Code mẫu backend khi nhận file SRT

~~~js
if (isSubtitle && extension === '.srt') {
  const { inputKey, outputKey } = subtitleObjectKeys(body, req.file.originalname);
  const uploadedSource = await awsS3Service.uploadSubtitleSource(req.file, {
    inputKey,
    outputBucket: awsConfig.s3OutputBucket,
    outputKey,
    metadata
  });

  const vttText = convertSrtBufferToVtt(req.file.buffer);
  const uploadedVtt = await awsS3Service.uploadPublicObject({
    key: outputKey,
    body: Buffer.from(vttText, 'utf8'),
    contentType: 'text/vtt; charset=utf-8'
  });
}
~~~

#### Code mẫu Lambda chuyển SRT sang VTT

~~~js
function srtToWebVtt(srtText) {
  const body = String(srtText || '')
    .replace(/\r\n/g, '\n')
    .replace(/(\d{2}:\d{2}:\d{2}),(\d{3})\s*-->\s*(\d{2}:\d{2}:\d{2}),(\d{3})/g, '$1.$2 --> $3.$4');

  return 'WEBVTT\n\n' + body + '\n';
}

await s3.send(new PutObjectCommand({
  Bucket: outputBucket,
  Key: outputKey,
  Body: vttText,
  ContentType: 'text/vtt; charset=utf-8'
}));
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Chức năng phụ đề

Admin có thể thêm phụ đề cho từng tập phim bằng URL hoặc upload file. Website hỗ trợ <code>.vtt</code> trực tiếp và <code>.srt</code> bằng cách chuyển sang WebVTT.

#### Luồng upload phụ đề

1. Admin chọn tập phim.
2. Nhập mã ngôn ngữ, ví dụ <code>vi</code>.
3. Upload file <code>.vtt</code> hoặc <code>.srt</code>.
4. Backend upload file lên S3.
5. Nếu là SRT, backend tạo VTT trả ngay cho web và Lambda cũng có thể xử lý serverless khi object SRT vào input bucket.
6. Backend lưu URL phụ đề vào database.
7. Player nhận danh sách subtitles và hiển thị trong selector CC.

#### Code backend xử lý SRT

~~~js
if (isSubtitle && extension === '.srt') {
  const { inputKey, outputKey } = subtitleObjectKeys(body, req.file.originalname);
  const uploadedSource = await awsS3Service.uploadSubtitleSource(req.file, {
    inputKey,
    outputBucket: awsConfig.s3OutputBucket,
    outputKey,
    metadata
  });

  const vttText = convertSrtBufferToVtt(req.file.buffer);
  const uploadedVtt = await awsS3Service.uploadPublicObject({
    key: outputKey,
    body: Buffer.from(vttText, 'utf8'),
    contentType: 'text/vtt; charset=utf-8'
  });
}
~~~

#### Code Lambda chuyển SRT sang VTT

~~~js
function srtToWebVtt(srtText) {
  const body = String(srtText || '')
    .replace(/^\uFEFF/, '')
    .replace(/\r\n/g, '\n')
    .replace(/\r/g, '\n')
    .replace(/(\d{2}:\d{2}:\d{2}),(\d{3})\s*-->\s*(\d{2}:\d{2}:\d{2}),(\d{3})/g, '$1.$2 --> $3.$4');

  return 'WEBVTT\n\n' + body + '\n';
}
~~~

#### Code player nhận subtitle

~~~js
const tracks = subtitles
  .filter((item) => item?.url)
  .map((item) => ({
    languageCode: item.languageCode,
    label: item.label,
    url: subtitleMediaUrl(item.url)
  }));
~~~

{{% notice info %}}
Cần thêm ảnh: giao diện admin upload phụ đề, file SRT ở S3 input, file VTT ở S3 output, Lambda invocation log và selector CC trong player.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
