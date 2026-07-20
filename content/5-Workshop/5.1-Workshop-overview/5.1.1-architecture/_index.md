---
title: "Overall architecture"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.1.1. </b> "
---

#### Architecture goal

Netflop separates the web application, database, media storage, video processing, content delivery, and monitoring into distinct layers. This design makes the system easier to operate, avoids storing large files on EC2, and provides a foundation for future scaling.

#### Main access flow

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

#### User flow

1. A user opens `https://netflop.win`.
2. Cloudflare handles DNS and HTTPS for the public domain.
3. The request reaches EC2.
4. Nginx serves the React frontend or proxies `/api/*` requests to the Node.js backend.
5. The backend reads/writes movie data, accounts, watch history, continue watching, favorites, ratings, and comments in RDS MySQL.
6. When the user plays a movie, the backend issues a stream session and the player fetches HLS from CloudFront.

#### Admin upload flow

1. The admin selects a movie, episode, and MP4/MKV file in the admin panel.
2. The backend creates an episode record in a processing state.
3. The original file is uploaded to the S3 bucket `netflop-input-source`.
4. The backend creates a MediaConvert job.
5. MediaConvert converts the video to HLS at 360p, 480p, 720p, and 1080p.
6. The HLS output is stored in the S3 bucket `netflop-output-source`.
7. EventBridge captures MediaConvert state changes and invokes Lambda.
8. Lambda calls the backend webhook to update the episode status in the database.
9. The website automatically shows the completed status without admin manual sync.

#### Automatic status update flow

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

#### Subtitle flow

```text
Admin upload .srt or .vtt
   |
   v
S3 input/output
   |
   v
Lambda converts .srt to .vtt if needed
   |
   v
Video Player loads subtitle track
```

![sodo](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.1-Workshop-overview/5.1.1-architecture/sodo.jpg)

<!-- NETFLOP_DETAIL_START -->
#### How to implement the architecture

The architecture is implemented in layers:

1. Domain layer: Cloudflare manages DNS for `netflop.win`.
2. Application layer: EC2 runs Nginx, React frontend build, and Node.js backend.
3. Database layer: RDS MySQL stores movie and user data.
4. Media layer: S3 input stores source videos, MediaConvert creates HLS, and S3 output stores the converted files.
5. Delivery layer: CloudFront delivers HLS and protects streams with signed cookies.
6. Automation layer: EventBridge and Lambda update MediaConvert job status.
7. Monitoring layer: CloudWatch and SNS monitor the system.

#### Related source files

| Component | Related file |
| --- | --- |
| RDS connection | `backend/src/config/database.js` |
| S3 upload | `backend/src/services/awsS3.service.js` |
| MediaConvert job | `backend/src/services/mediaConvert.service.js` |
| MediaConvert webhook | `backend/src/controllers/upload.controller.js` |
| Signed cookies | `backend/src/services/cloudFrontSignedCookie.service.js` |
| HLS player | `frontend/src/components/VideoPlayer.jsx` |
| Subtitle Lambda | `aws/lambda/subtitle-converter/index.mjs` |

#### Sample code: backend video processing flow

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
