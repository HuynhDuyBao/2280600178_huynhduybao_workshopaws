---
title: "Configure S3 input/output buckets"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

#### Goal

Amazon S3 is used to store media files instead of storing them directly on EC2. This avoids large upload failures, reduces server load, and fits production deployment.

#### Buckets in use

| Bucket | Role |
| --- | --- |
| `netflop-input-source` | Stores original uploaded videos and `.srt` subtitle input |
| `netflop-output-source` | Stores HLS output, movie images, episode banners, avatars, and `.vtt` subtitles |

#### Directory conventions

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

#### Large-file upload

For videos around 5GB or larger, the frontend/backend should use direct S3 upload or multipart upload. Uploading through Nginx/backend like a normal request may cause:

```text
413 Request Entity Too Large
```

In Netflop, the preferred flow is:

1. The backend generates upload info, presigned URLs, or multipart upload credentials.
2. The frontend uploads the file directly to S3.
3. The backend stores episode metadata.
4. The backend creates a MediaConvert job after the file is available in S3.

#### Media types stored

* Original episode video files.
* HLS output after transcoding.
* Movie poster and banner images.
* Episode banner images.
* User avatars.
* Subtitle files in `.vtt` format.

#### Verification

```bash
aws s3 ls s3://netflop-input-source
aws s3 ls s3://netflop-output-source
```

![input](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.1-s3-buckets/s3input.png)
![output](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.1-s3-buckets/s3ouput.png)

<!-- NETFLOP_DETAIL_START -->
#### How to upload to S3

The backend uses the AWS SDK to write objects to S3. Source videos go to the input bucket, while HLS output, images, avatars, and subtitles are stored in the output bucket.

#### Sample object upload code

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

#### Sample multipart upload start

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

#### Console verification

1. Open S3 -> `netflop-input-source`.
2. Check source video under `movies/{movieId}/episodes/`.
3. Open S3 -> `netflop-output-source`.
4. Check HLS output, banners, avatars, and subtitles.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### S3 bucket structure

Netflop uses two buckets:

* <code>netflop-input-source</code>: stores source videos and SRT source files.
* <code>netflop-output-source</code>: stores HLS output, VTT subtitles, avatars, and episode banners depending on configuration.

#### Suggested prefixes

~~~text
uploads/videos/{movieId}/{timestamp}-{filename}
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
movies/{movieId}/episodes/{episodeId}/hls/*.ts
subtitle-input/episodes/{episodeId}/{lang}/{file}.srt
subtitles/episodes/{episodeId}/{lang}/{file}.vtt
avatars/users/{userId}/{file}
episode-banners/{file}
~~~

#### PutObject sample

~~~js
await s3.send(new PutObjectCommand({
  Bucket: bucket,
  Key: key,
  Body: file.buffer,
  ContentType: file.mimetype || 'application/octet-stream',
  Metadata: metadata
}));
~~~

#### Multipart upload sample

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

#### Why multipart is required

A 5GB movie upload in a single request can fail with <code>413 Request Entity Too Large</code> or timeout. Multipart upload splits the file into smaller parts, retries only failed parts, and allows the admin UI to show accurate progress.


<!-- NETFLOP_IMPLEMENTATION_END -->
