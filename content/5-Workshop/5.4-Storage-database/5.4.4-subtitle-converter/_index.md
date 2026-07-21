---
title: "Configure subtitle converter Lambda"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

#### Goal

The video player requires subtitles in WebVTT format (`.vtt`). Because admins can upload `.srt` subtitles, the system uses Lambda `netflop-subtitle-converter` to automatically convert `.srt` files to `.vtt`.

#### Subtitle processing flow

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
Backend stores subtitle URL
  |
  v
Video Player displays subtitle track
```

#### Upload handling

| File type | Handling |
| --- | --- |
| `.vtt` | Uploaded directly to S3 output and saved in database |
| `.srt` | Uploaded to S3 input, Lambda converts it to `.vtt`, then the backend/player uses the `.vtt` file |

#### S3 trigger

Lambda is triggered by new objects:

```text
Bucket: netflop-input-source
Prefix: subtitle-input/
Suffix: .srt
Event: s3:ObjectCreated:*
```

#### Admin UI notes

The admin interface should clearly display:

* The episode associated with the subtitle.
* Subtitle language, e.g. Vietnamese.
* Whether the uploaded file is `.srt` or `.vtt`.
* Conversion status when uploading `.srt`.
* The final `.vtt` URL after conversion.

#### Verification

1. Upload an `.srt` file from the admin panel.
2. Confirm the file appears under `netflop-input-source/subtitle-input/`.
3. Check CloudWatch Logs for Lambda `netflop-subtitle-converter`.
4. Confirm a `.vtt` file is created in the S3 output bucket.
5. Open the video player and enable subtitles.
6. Verify that subtitles display correctly without duplicate lines on mobile.

{{% notice info %}}
Image needed: admin subtitle upload, `.srt` input object, Lambda log output, `.vtt` output object, and subtitle display in the player.
{{% /notice %}}

<!-- NETFLOP_DETAIL_START -->
#### How subtitle upload and conversion works

When the admin uploads subtitles:

1. If the file is `.vtt`, the backend uploads it directly to S3 output.
2. If the file is `.srt`, the backend uploads the source to S3 input with metadata for the output bucket/key.
3. The backend may also convert SRT to VTT immediately to return a URL to the web UI.
4. The `netflop-subtitle-converter` Lambda still processes `.srt` objects created under `subtitle-input/`.

#### Sample backend code for receiving SRT

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

#### Sample Lambda SRT-to-VTT code

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
#### Subtitle feature

Administrators can add subtitles to each episode by URL or file upload. The website supports <code>.vtt</code> directly and supports <code>.srt</code> by converting it to WebVTT.

#### Subtitle upload flow

1. Admin selects an episode.
2. Admin enters a language code, for example <code>vi</code>.
3. Admin uploads a <code>.vtt</code> or <code>.srt</code> file.
4. Backend uploads the file to S3.
5. If the file is SRT, backend creates a VTT output immediately and Lambda can also process the SRT object serverlessly.
6. Backend stores the subtitle URL in the database.
7. The player receives subtitles and displays them in the CC selector.

#### Backend SRT handling sample

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

#### Lambda SRT to VTT sample

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

#### Player subtitle mapping

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
Screenshots needed: admin subtitle upload UI, SRT file in S3 input, VTT file in S3 output, Lambda invocation log, and CC selector in the player.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
