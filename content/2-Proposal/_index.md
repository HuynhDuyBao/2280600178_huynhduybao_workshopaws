---
title: "Proposal"
date: 2026-04-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Netflop Movie Website on AWS
## A movie streaming architecture using EC2, RDS, S3, MediaConvert, CloudFront, and Lambda

### 1. Executive Summary

**Netflop** is a movie streaming website deployed on AWS. The system allows users to browse movies, search for movies, view movie details, stream HLS videos, and view subtitles. Administrators can upload movies, upload subtitles, manage episodes, posters, avatars, and movie metadata.

The current architecture uses **Cloudflare** for the `netflop.win` domain, **Amazon EC2** for Nginx, React frontend, and Node.js API, **Amazon RDS MySQL** for the main database, **Amazon S3** for video/images/subtitles, **AWS Elemental MediaConvert** to convert MP4/MKV videos to HLS, **Amazon CloudFront** to distribute video streams, **Lambda** and **EventBridge** to automate video/subtitle status updates, and **CloudWatch** with **SNS** for monitoring and alerts.

The goal is to build a realistic movie website architecture and learn how to deploy a web application, process media, deliver content, and monitor systems on AWS.

### 2. Problem Statement

A movie website needs to manage not only movie data but also uploaded videos, HLS outputs, posters, avatars, and subtitles. If everything is stored directly on one EC2 server, the system may face storage, scalability, automation, and monitoring issues.

The proposed solution separates the application, database, media storage, video processing, content delivery, and monitoring layers:

* **EC2/Nginx** runs the React frontend and Node.js backend API.
* **RDS MySQL** stores the main system data.
* **S3 input bucket** stores uploaded videos and subtitle inputs.
* **MediaConvert** converts MP4/MKV videos into multi-quality HLS.
* **S3 output bucket** stores HLS output, public images, avatars, and VTT subtitles.
* **CloudFront** delivers HLS streams and protects streams with signed cookies.
* **EventBridge + Lambda** receives MediaConvert events and calls the backend webhook to update episode status.
* **Lambda subtitle converter** converts `.srt` subtitles into `.vtt`.
* **CloudWatch + SNS** provides monitoring and alerting.

### 3. Solution Architecture

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
   |-- React frontend
   |-- Node.js backend API
   |
   |---- RDS MySQL netflop-db
   |
   |---- S3 netflop-input-source
   |          |
   |          v
   |      MediaConvert
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

Automation flow:

```text
MediaConvert Job State Change
   |
   v
EventBridge
   |
   v
Lambda netflop-mediaconvert-notifier
   |
   v
Backend webhook /api/uploads/mediaconvert/events
   |
   v
Update episode status in RDS
```

Subtitle flow:

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
Video Player subtitles
```

### 4. AWS Services Used

| Service | Role |
| --- | --- |
| Amazon EC2 | Hosts Node.js backend, React frontend build, and Nginx reverse proxy |
| Amazon RDS MySQL | Main database `web_xem_phim_final` |
| Amazon S3 | Stores video input, HLS output, images, avatars, and subtitles |
| AWS Elemental MediaConvert | Converts MP4/MKV to HLS 360p/480p/720p/1080p |
| Amazon CloudFront | Delivers HLS streams and protects them with signed cookies |
| AWS Lambda | Handles MediaConvert events and converts SRT subtitles to VTT |
| Amazon EventBridge | Captures MediaConvert COMPLETE/ERROR/CANCELED events |
| Amazon CloudWatch | Monitors EC2, RDS, and Lambda |
| Amazon SNS | Alert topic `netflop-alerts` |
| IAM Role | Grants permissions for EC2, MediaConvert, and Lambda |
| AWS Systems Manager | Supports EC2 deployment/reload from local AWS CLI |
| Amazon Cognito | Authentication is configured, but account/region confirmation is needed |

### 5. Timeline and Milestones

| Phase | Time | Main content |
| --- | --- | --- |
| Phase 1 | Week 1-2 | Learn AWS fundamentals, IAM, EC2, AWS CLI, and account security |
| Phase 2 | Week 3-4 | Practice VPC, EC2, S3, and basic app deployment |
| Phase 3 | Week 5-6 | Learn RDS, CloudWatch, Budgets, and prepare database |
| Phase 4 | Week 7-8 | Analyze Netflop, design frontend/backend and AWS architecture |
| Phase 5 | Week 9-10 | Integrate RDS, S3, media upload, and backend APIs |
| Phase 6 | Week 11-12 | Integrate MediaConvert, CloudFront, Lambda/EventBridge, test, and finish report |

### 6. Budget Estimation

This estimate is for a small demo deployment, **without Free Tier/credits**, in **Asia Pacific (Singapore) - ap-southeast-1**.

| Service | Estimated configuration | Estimated cost/month |
| --- | --- | --- |
| Amazon EC2 | 01 t3.micro Linux for Netflop | 10 USD |
| Amazon EBS | 20 GB gp3 for EC2 | 2 USD |
| Amazon RDS MySQL | 01 db.t4g.micro Single-AZ | 18 USD |
| RDS Storage/Backup | 20 GB database and small backup usage | 3 USD |
| Amazon S3 | 2 input/output buckets, around 20-30 GB media | 2 USD |
| AWS Elemental MediaConvert | Demo video conversion to multi-quality HLS | 5 USD |
| Amazon CloudFront | Demo-level HLS streaming | 8 USD |
| Data Transfer Out | Around 100 GB/month | 12 USD |
| AWS Lambda | 2 Lambda functions for events/subtitles | 1 USD |
| EventBridge + SNS | Demo-level events and alerts | 1 USD |
| Amazon CloudWatch | Basic logs, metrics, and alarms | 3 USD |
| AWS Systems Manager | Basic EC2 deploy/reload usage | 0 USD |

**Estimated total:** around **65 USD/month**.  
**Estimated total for three internship months:** around **195 USD**.

### 7. Risk Assessment

| Risk | Impact | Probability | Mitigation |
| --- | --- | --- | --- |
| Cost increase from video/data transfer | High | Medium | Use small sample videos and monitor Billing/Budgets |
| RDS public accessible | High | Medium | Improvement direction: restrict Security Group to EC2 only |
| Wrong S3 public access | High | Low | Use Block Public Access and IAM/CloudFront access control |
| MediaConvert job failure | Medium | Medium | Monitor job status, EventBridge, and CloudWatch logs |
| Lambda notifier/webhook failure | Medium | Medium | Check Lambda logs and `/api/uploads/mediaconvert/events` |
| CloudFront signed cookies misconfiguration | Medium | Medium | Test `/api/stream/session` and stream access |
| Cognito in another account/region | Low | Medium | Document that Cognito integration needs confirmation |

### 8. Expected Outcomes

* Netflop runs through the `netflop.win` domain.
* EC2 serves React frontend, Node.js API, and Nginx reverse proxy.
* RDS MySQL stores movies, episodes, accounts, comments, watch history, and favorites.
* Admin uploads videos to S3 input and the system creates MediaConvert jobs.
* Videos are converted to HLS and delivered through CloudFront.
* EventBridge/Lambda automatically updates episode status after MediaConvert completes.
* `.srt` subtitles are converted to `.vtt` by Lambda.
* CloudWatch and SNS monitor and alert EC2, RDS, and Lambda.
