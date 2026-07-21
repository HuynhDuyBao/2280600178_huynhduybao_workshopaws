---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---



This section lists and introduces the blogs our team posted while working on **Netflop - a movie streaming website built on AWS**.

### [Blog 1 - Why Our Team Chose AWS Elemental MediaConvert Instead of FFmpeg](3.1-Blog1/)

![Blog 1 Facebook post](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/blog1-facebook-post.png)

This blog explains why our team chose **AWS Elemental MediaConvert** instead of running **FFmpeg on Amazon EC2** for video encoding. It focuses on reducing EC2 workload, automatically converting videos to multi-quality HLS, and making the media pipeline easier to scale.

### [Blog 2 - Why We Used Amazon CloudFront Instead of Serving Videos Directly from S3](3.2-Blog2/)

![Blog 2 Facebook post](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/blog2-facebook-post.png)

This blog shares our experience using **Amazon CloudFront** to deliver HLS videos for a movie streaming website. It highlights benefits such as lower latency, caching at Edge Locations, fewer direct S3 requests, HTTPS support, and content protection with CloudFront Signed Cookies.

### [Blog 3 - Building an Event-Driven Media Pipeline on AWS](3.3-Blog3/)

![AWS media workflow with S3, MediaConvert, CloudFront, Lambda, EventBridge, CloudWatch, and SNS](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/3.3-Blog3/mediaconvert-workflow.png)

This blog describes how our team used an **Event-Driven Architecture** with **MediaConvert -> EventBridge -> Lambda -> Backend Webhook** to automatically update episode status after encoding is completed, instead of making the Backend poll continuously.
Facebook source: https://www.facebook.com/groups/awsstudygroupfcj/?multi_permalinks=2218428612255432


