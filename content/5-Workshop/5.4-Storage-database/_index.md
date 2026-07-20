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
