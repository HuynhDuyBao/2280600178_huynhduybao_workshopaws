---
title: "Cấu hình S3 input/output buckets"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Mục tiêu

Amazon S3 được dùng để lưu file media thay vì lưu trực tiếp trên ổ đĩa EC2. Điều này giúp tránh lỗi upload file lớn, giảm tải server và phù hợp khi deploy production.

#### Buckets đang sử dụng

| Bucket | Vai trò |
| --- | --- |
| `netflop-input-source` | Lưu video gốc MP4/MKV và file phụ đề `.srt` cần convert |
| `netflop-output-source` | Lưu HLS output, ảnh phim, banner tập, avatar và phụ đề `.vtt` |

#### Quy ước thư mục

```text
netflop-input-source/
  movies/{movieId}/episodes/{episodeId}/source.{ext}
  subtitle-input/{episodeId}/{file}.srt

netflop-output-source/
  movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
  movies/{movieId}/episodes/{episodeId}/hls/*.ts
  subtitles/{episodeId}/{file}.vtt
  avatars/{userId}/{file}
  episode-banners/{episodeId}/{file}
```

#### Upload file lớn

Đối với video 5GB hoặc lớn hơn, frontend/backend cần dùng luồng upload trực tiếp lên S3 hoặc multipart upload. Nếu upload qua Nginx/backend như request thường, có thể gặp lỗi:

```text
413 Request Entity Too Large
```

Trong Netflop, hướng phù hợp là:

1. Backend tạo thông tin upload/presigned URL hoặc multipart upload.
2. Frontend upload file lên S3.
3. Backend lưu metadata tập phim.
4. Backend tạo MediaConvert job sau khi file đã có trên S3.

#### Các loại media được lưu

* Video gốc của tập phim.
* HLS output sau khi convert.
* Poster/banner phim.
* Banner riêng cho từng tập.
* Avatar người dùng.
* Phụ đề `.vtt`.

#### Kiểm tra

```bash
aws s3 ls s3://netflop-input-source
aws s3 ls s3://netflop-output-source
```

{{% notice info %}}
Cần thêm ảnh: danh sách hai bucket S3, object video gốc trong input, thư mục HLS output, ảnh avatar/banner trên S3 và màn hình upload file lớn ở admin.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.1-s3-buckets/s3input.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.1-s3-buckets/s3ouput.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện upload lên S3

Backend dùng AWS SDK để ghi object vào S3. Video gốc đi vào bucket input, còn HLS output và ảnh/phụ đề public đi vào bucket output.

#### Code mẫu upload object

~~~js
async function putObject({ bucket, key, body, contentType, metadata = {} }) {
  const { PutObjectCommand } = loadS3Commands('PutObjectCommand');
  const client = getS3Client();
  await client.send(new PutObjectCommand({
    Bucket: bucket,
    Key: key,
    Body: body,
    ContentType: contentType || 'application/octet-stream',
    Metadata: metadata
  }));
}
~~~

#### Code mẫu bắt đầu multipart upload

~~~js
async function createVideoMultipartUpload({ movieId, episodeName, originalName, contentType }) {
  const { CreateMultipartUploadCommand } = loadS3Commands('CreateMultipartUploadCommand');
  const key = 'movies/' + movieId + '/episodes/' + Date.now() + '-' + safeS3Name(originalName, 'video');
  const result = await client.send(new CreateMultipartUploadCommand({
    Bucket: awsConfig.s3InputBucket,
    Key: key,
    ContentType: contentType || 'application/octet-stream',
    Metadata: {
      movieId: String(movieId),
      episodeName: String(episodeName || '')
    }
  }));

  return { key, uploadId: result.UploadId, s3Uri: 's3://' + awsConfig.s3InputBucket + '/' + key };
}
~~~

#### Cách kiểm tra trên AWS Console

1. Vào S3 -> bucket <code>netflop-input-source</code>.
2. Kiểm tra video gốc ở prefix <code>movies/{movieId}/episodes/</code>.
3. Vào bucket <code>netflop-output-source</code>.
4. Kiểm tra HLS output, ảnh banner, avatar và phụ đề.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Cấu trúc S3 bucket

Netflop dùng hai bucket:

* <code>netflop-input-source</code>: chứa video gốc và SRT source.
* <code>netflop-output-source</code>: chứa HLS output, phụ đề VTT, avatar/banner nếu cấu hình upload media public.

#### Prefix đề xuất

~~~text
uploads/videos/{movieId}/{timestamp}-{filename}
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
movies/{movieId}/episodes/{episodeId}/hls/*.ts
subtitle-input/episodes/{episodeId}/{lang}/{file}.srt
subtitles/episodes/{episodeId}/{lang}/{file}.vtt
avatars/users/{userId}/{file}
episode-banners/{file}
~~~

#### Code upload object lên S3

~~~js
await s3.send(new PutObjectCommand({
  Bucket: bucket,
  Key: key,
  Body: file.buffer,
  ContentType: file.mimetype || 'application/octet-stream',
  Metadata: metadata
}));
~~~

#### Code multipart upload cho video lớn

~~~js
const part = await awsS3Service.uploadVideoPart({
  key,
  uploadId,
  partNumber,
  body: req.file.buffer
});

const uploaded = await awsS3Service.completeVideoMultipartUpload({
  key,
  uploadId,
  parts
});
~~~

#### Vì sao cần multipart

Upload phim 5GB qua một request dễ bị lỗi <code>413 Request Entity Too Large</code> hoặc timeout. Multipart upload chia file thành nhiều phần nhỏ, có thể retry từng part và hiển thị tiến trình chính xác hơn trên admin.


<!-- NETFLOP_IMPLEMENTATION_END -->
