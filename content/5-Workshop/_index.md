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


