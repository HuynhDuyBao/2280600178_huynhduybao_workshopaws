---
title: "Workshop"
date: 2026-07-10
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying the Netflop Movie Website on AWS

This workshop describes the process of deploying the **Netflop** movie streaming website to AWS. The content follows the current system flow: users access the `netflop.win` domain, the frontend and backend run on EC2, data is stored in RDS MySQL, media is stored on S3, videos are transcoded with AWS Elemental MediaConvert, and playback is delivered through CloudFront.

The goal of this chapter is to present how to build a movie streaming system capable of uploading large videos, automatically converting them to HLS at multiple resolutions, delivering video through a CDN, storing watch history by account, supporting subtitles, and monitoring operational status with CloudWatch.

#### Contents

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisite/)
3. [Create AWS infrastructure](5.3-AWS-infrastructure/)
4. [Configure storage, database, and media processing](5.4-Storage-database/)
5. [Deploy and test the application](5.5-Deploy-test/)
6. [Clean up resources](5.6-Cleanup/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Practical implementation scope

This workshop section documents how the Netflop movie streaming website was moved from local development to AWS. The content follows a hands-on structure: infrastructure setup, application deployment, video processing, stream protection, subtitle handling, monitoring, and cleanup.

The sample code snippets are shortened from the current Netflop source code. Sensitive values such as database passwords, AWS access keys, OAuth client secrets, JWT secrets, and CloudFront private keys must not be copied into the report.

#### Recommended structure for each feature

1. State the purpose of the feature.
2. List the AWS services or application modules involved.
3. Describe the implementation steps in AWS Console or CLI.
4. Explain the processing flow in the source code.
5. Include a short code sample from the project.
6. Add test results and screenshots required for evidence.

#### Main features covered in the workshop

| Group | Implemented feature | Evidence to include |
| --- | --- | --- |
| Infrastructure | EC2, Nginx, PM2, RDS, Security Groups | Website loads and API health check works |
| Media storage | S3 input/output, CloudFront | Source video in input bucket, HLS in output bucket |
| Video processing | MediaConvert, EventBridge, Lambda notifier | Job COMPLETE and episode status updates automatically |
| Subtitles | VTT/SRT upload, Lambda SRT to VTT converter | Subtitle track is visible in the player |
| Stream security | CloudFront signed cookies | HLS plays through CloudFront with temporary cookies |
| Operations | CloudWatch, SNS, Cost Explorer | Alarm, logs, dashboard, or cost screenshots |

#### Repository

[Netflop source code](https://github.com/Danh0104/Web-xem-phim-Netflop.git)

### Demo Stream

<div class="netflop-demo-player">
  <video id="netflop-demo-video-en" controls playsinline preload="metadata" style="width:100%;max-width:960px;background:#000;border-radius:8px;"></video>
  <p id="netflop-demo-status-en">
    <a href="https://customer-mq3bsojkqgoa0nyg.cloudflarestream.com/8bc4e084d7f19b8303da087366e0fe91/manifest/video.m3u8">Open HLS demo stream</a>
  </p>
</div>

<script src="https://cdn.jsdelivr.net/npm/hls.js@1/dist/hls.min.js"></script>
<script>
(function () {
  const source = "https://customer-mq3bsojkqgoa0nyg.cloudflarestream.com/8bc4e084d7f19b8303da087366e0fe91/manifest/video.m3u8";
  const video = document.getElementById("netflop-demo-video-en");
  const status = document.getElementById("netflop-demo-status-en");
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
      if (status) status.textContent = "The HLS demo stream could not be loaded in this browser.";
    });
    return;
  }

  if (status) status.textContent = "This browser does not support HLS playback.";
})();
</script>
<!-- NETFLOP_IMPLEMENTATION_END -->
