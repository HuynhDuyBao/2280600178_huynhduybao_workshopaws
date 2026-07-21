---
title: "Cấu hình MediaConvert"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Mục tiêu

AWS Elemental MediaConvert được dùng để chuyển file video MP4/MKV do admin upload sang định dạng HLS để phát tốt trên web và mobile. HLS giúp trình phát tự chọn chất lượng phù hợp với tốc độ mạng.

#### Endpoint

Endpoint MediaConvert của hệ thống:

```text
https://mediaconvert.ap-southeast-1.amazonaws.com
```

#### Luồng tạo job

```text
Admin upload video
   |
   v
S3 input source
   |
   v
Backend CreateJob
   |
   v
MediaConvert
   |
   v
S3 output HLS
```

#### Output cần tạo

MediaConvert xuất HLS với nhiều chất lượng:

| Chất lượng | Mục đích |
| --- | --- |
| 360p | Kết nối yếu/mobile |
| 480p | Chất lượng trung bình |
| 720p | HD phổ biến |
| 1080p | Full HD |

Output chính là manifest:

```text
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
```

#### Trạng thái tập phim

Backend lưu trạng thái xử lý để admin theo dõi:

| Trạng thái | Ý nghĩa |
| --- | --- |
| `processing` | Video đã upload và đang convert |
| `ready` | Convert xong, có thể phát |
| `failed` | Convert lỗi hoặc webhook báo lỗi |

Frontend admin hiển thị thanh tiến trình theo các giai đoạn: upload lên S3, MediaConvert processing và hoàn thành.

#### Kiểm tra

1. Upload một tập phim từ admin.
2. Vào MediaConvert console.
3. Kiểm tra job đang chạy hoặc COMPLETE.
4. Vào S3 output kiểm tra file `index.m3u8` và segment.
5. Mở tập phim trên web để kiểm tra player.

{{% notice info %}}
Cần thêm ảnh: MediaConvert job detail, input S3 path, output HLS path, job status COMPLETE và thanh tiến trình upload/convert trong admin.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.2-mediaconvert/mediaconvert.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.2-mediaconvert/hls.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện MediaConvert job

Sau khi video gốc đã có trong S3 input, backend gọi MediaConvert <code>CreateJob</code>. Trong job, backend truyền:

* Role ARN cho MediaConvert.
* File input dạng <code>s3://bucket/key</code>.
* Output destination trong S3 output.
* Danh sách output HLS 360p, 480p, 720p và 1080p.
* Metadata gồm movieId, episodeId, outputPrefix và masterKey.

#### Code mẫu rút gọn

~~~js
async function createHlsJob({ inputS3Uri, movieId, episodeId }) {
  const outputPrefix = 'movies/' + movieId + '/episodes/' + episodeId + '/hls/';
  const masterKey = outputPrefix + 'index.m3u8';

  const command = new CreateJobCommand({
    Role: awsConfig.mediaConvertRoleArn,
    UserMetadata: { movieId: String(movieId), episodeId: String(episodeId), outputPrefix, masterKey },
    Settings: {
      Inputs: [{ FileInput: inputS3Uri }],
      OutputGroups: [{
        Name: 'Apple HLS',
        OutputGroupSettings: {
          Type: 'HLS_GROUP_SETTINGS',
          HlsGroupSettings: {
            Destination: 's3://' + awsConfig.s3OutputBucket + '/' + outputPrefix + 'index',
            SegmentLength: 6,
            OutputSelection: 'MANIFESTS_AND_SEGMENTS'
          }
        },
        Outputs: [
          createHlsOutput({ nameModifier: '_360p', width: 640, height: 360 }),
          createHlsOutput({ nameModifier: '_480p', width: 854, height: 480 }),
          createHlsOutput({ nameModifier: '_720p', width: 1280, height: 720 }),
          createHlsOutput({ nameModifier: '_1080p', width: 1920, height: 1080 })
        ]
      }]
    }
  });

  const result = await client.send(command);
  return { jobId: result.Job?.Id, outputPrefix, masterKey };
}
~~~

#### Kiểm tra kết quả

Sau khi job COMPLETE, S3 output phải có:

~~~text
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
movies/{movieId}/episodes/{episodeId}/hls/*.ts
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Các bước tạo MediaConvert job

1. Backend nhận <code>inputS3Uri</code> sau khi upload video xong.
2. Tạo <code>outputPrefix</code> theo movieId/episodeId.
3. Gọi AWS SDK <code>CreateJobCommand</code>.
4. Truyền MediaConvert role ARN.
5. Khai báo output group Apple HLS.
6. Tạo 4 rendition: 360p, 480p, 720p, 1080p.
7. Lưu jobId và CloudFront URL vào tập phim.

#### Code mẫu từ service MediaConvert

~~~js
const outputPrefix = 'movies/' + movieId + '/episodes/' + episodeId + '/hls/';
const masterKey = outputPrefix + 'index.m3u8';

const command = new CreateJobCommand({
  Role: awsConfig.mediaConvertRoleArn,
  UserMetadata: {
    movieId: String(movieId),
    episodeId: String(episodeId),
    outputPrefix,
    masterKey
  },
  Settings: {
    Inputs: [{ FileInput: inputS3Uri }],
    OutputGroups: [{
      Name: 'Apple HLS',
      OutputGroupSettings: {
        Type: 'HLS_GROUP_SETTINGS',
        HlsGroupSettings: {
          Destination: 's3://' + awsConfig.s3OutputBucket + '/' + outputPrefix + 'index',
          SegmentLength: 6,
          OutputSelection: 'MANIFESTS_AND_SEGMENTS'
        }
      },
      Outputs: [
        createHlsOutput({ nameModifier: '_360p', width: 640, height: 360 }),
        createHlsOutput({ nameModifier: '_480p', width: 854, height: 480 }),
        createHlsOutput({ nameModifier: '_720p', width: 1280, height: 720 }),
        createHlsOutput({ nameModifier: '_1080p', width: 1920, height: 1080 })
      ]
    }]
  }
});
~~~

#### Cập nhật database sau khi submit job

~~~js
const job = await mediaConvertService.createHlsJob({
  inputS3Uri: uploaded.s3Uri,
  movieId,
  episodeId: pendingEpisode.MaTap
});

await episodeModel.updateUploadProcessing(pendingEpisode.MaTap, {
  jobId: job.jobId,
  hlsUrl: playbackUrls.hlsUrl,
  cloudFrontUrl: playbackUrls.cloudFrontUrl,
  outputKey: job.masterKey
});
~~~


<!-- NETFLOP_IMPLEMENTATION_END -->
