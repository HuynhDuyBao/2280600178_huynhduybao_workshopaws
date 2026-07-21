---
title: "Configure storage, database, and media processing"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This section focuses on the Netflop media pipeline. It is the most important part of the movie website because admins must upload large videos, the system automatically converts them to multi-quality HLS, stores output on S3, and serves playback through CloudFront.

![lambda](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/lamda.png)
![s3in](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/inputsub.png)
![s3out](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/outputsub.png)
![player](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/player1.png)

#### Contents

1. [Configure S3 input/output buckets](5.4.1-s3-buckets/)
2. [Configure MediaConvert](5.4.2-mediaconvert/)
3. [Configure CloudFront HLS streaming](5.4.3-cloudfront-hls/)
4. [Configure subtitle converter Lambda](5.4.4-subtitle-converter/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Storage and media pipeline

This is the core part of the streaming website. The backend no longer stores large videos in local upload folders. Instead, it uses S3 and MediaConvert to produce HLS playback sources.

#### Complete media flow

1. Admin selects an MP4/MKV file in the admin UI.
2. Frontend uploads the file with multipart upload to support large videos.
3. Backend receives chunks and uploads them to the S3 input bucket.
4. After multipart completion, backend creates an episode record with <code>processing</code> status.
5. Backend calls MediaConvert to create multi-quality HLS output.
6. MediaConvert writes output into the S3 output bucket.
7. EventBridge invokes Lambda when the job status changes.
8. Lambda notifies the backend and the backend changes the episode status to <code>ready</code>.
9. The player uses a CloudFront playback URL.

#### Metadata stored in database

| Data | Purpose |
| --- | --- |
| S3 source URL | Trace the original uploaded video |
| MediaConvert job id | Sync job status |
| HLS URL | Manifest path in S3 output |
| CloudFront URL | Optimized playback URL for users |
| Upload status | processing/ready/failed |
<!-- NETFLOP_IMPLEMENTATION_END -->
