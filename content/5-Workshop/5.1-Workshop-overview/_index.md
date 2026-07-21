---
title: "Workshop overview"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

This section introduces the overall AWS architecture of **Netflop** and the role of each service group. It helps explain the web access flow, HLS video processing flow, and automatic episode status update flow.

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.1-Workshop-overview/sodo.jpg)

#### Contents

1. [Overall architecture](5.1.1-architecture/)
2. [Service map](5.1.2-service-map/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Workshop scenario

The workshop is based on a real movie streaming website with two user groups:

* Normal users can register, sign in, watch movies, choose episodes, enable subtitles, resume watching, add favorites, and rate movies.
* Administrators can add movies, add episodes, upload large video files to S3, track MediaConvert progress, and manage episode subtitles and banners.

#### Demo flow

1. Open <code>https://netflop.win</code>.
2. Check the movie list, movie detail page, and watch page.
3. Sign in as an administrator.
4. Upload an MP4/MKV episode from the admin page.
5. Verify that the source file appears in the S3 input bucket.
6. Verify that MediaConvert creates HLS output in the S3 output bucket.
7. Play the episode from the user website through CloudFront.
8. Upload SRT/VTT subtitles and test the subtitle selector.
9. Verify watch history is stored per account.

#### Related project folders

| Component | Source path |
| --- | --- |
| Backend API | <code>backend/src</code> |
| React frontend | <code>frontend/src</code> |
| Admin upload UI | <code>frontend/src/admin</code> |
| MediaConvert notifier Lambda | <code>aws/lambda/mediaconvert-notifier</code> |
| Subtitle converter Lambda | <code>aws/lambda/subtitle-converter</code> |
| Database dump | <code>database/web_xem_phim_final_dump.sql</code> |

<!-- NETFLOP_IMPLEMENTATION_END -->
