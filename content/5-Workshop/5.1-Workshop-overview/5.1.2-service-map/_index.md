---
title: "Service map"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.1.2. </b> "
---

#### Services in use

| Service | Status | Role in Netflop |
| --- | --- | --- |
| Cloudflare | In use | Manages domain `netflop.win`, DNS, and user-facing HTTPS |
| Amazon EC2 | Running | Hosts Nginx, React frontend build, and Node.js backend API |
| Nginx on EC2 | Running | Reverse proxy, serves frontend, forwards API requests |
| PM2 on EC2 | In use | Manages the Node.js backend process |
| Amazon RDS MySQL | Running | Main database `web_xem_phim_final` |
| Amazon S3 | In use | Stores video input, HLS output, movie images, banners, avatars, and subtitles |
| AWS Elemental MediaConvert | In use | Converts MP4/MKV to HLS 360p/480p/720p/1080p |
| Amazon CloudFront | In use | Distributes HLS stream, reduces S3 load, and protects stream with signed cookies |
| AWS Lambda | In use | Handles MediaConvert events and converts SRT subtitles to VTT |
| Amazon EventBridge | In use | Captures MediaConvert COMPLETE/ERROR/CANCELED events |
| Amazon CloudWatch | In use | Monitors EC2, RDS, Lambda, logs, and alarms |
| Amazon SNS | In use | Sends alerts through topic `netflop-alerts` |
| AWS IAM Role | In use | Grants permissions for EC2, MediaConvert, and Lambda by role |
| AWS Systems Manager | In use | Supports deploy/reload EC2 from AWS CLI |
| Amazon Cognito | Configured | Supports social login/OAuth and requires correct callback domain |

#### Services not used as main components

| Service | Status | Notes |
| --- | --- | --- |
| Amazon API Gateway | Not used | Backend runs directly on EC2 via Nginx; API Gateway would be suitable for serverless separation |
| AWS WAF | Not used | Could be added later to filter malicious requests at CloudFront or ALB |
| Application Load Balancer | Not used | Not needed for a single EC2 instance; required only when scaling multiple instances |
| AWS Certificate Manager | Not used directly | Public HTTPS currently goes through Cloudflare; ACM is needed if using ALB/CloudFront custom domains |
| Route 53 | Not used | Domain is managed in Cloudflare |

#### Architecture notes

* EC2 runs the main application, while Lambda is used for small automation tasks.
* S3 does not store business state; episode and movie state remain in RDS.
* CloudFront signed cookies help avoid publicly exposing raw HLS object URLs.
* A security improvement is to restrict the RDS security group so only the EC2 security group can access it.

<!-- NETFLOP_DETAIL_START -->

#### How to use this service map during implementation

Use this page as a checklist before building each function. Each application feature should be mapped to the AWS service that supports it, the source code area that calls it, and the result that must be verified.

| Function | Main AWS services | Application code to check | Expected result |
| --- | --- | --- | --- |
| User opens website | Cloudflare, EC2, Nginx | React build and Nginx site config | `https://netflop.win` loads the frontend |
| User signs in | Cognito/OAuth or backend auth, RDS | Auth controller, user model, frontend login page | User session is created and saved correctly |
| User watches movie | CloudFront, S3, backend API, RDS | Video stream API, player component | HLS video plays through CloudFront |
| Admin uploads video | EC2, S3, MediaConvert, RDS | Admin upload page, upload route, MediaConvert service | Video is uploaded and processing starts |
| Processing status updates | EventBridge, Lambda, backend API, RDS | Lambda notifier, processing webhook, episode update logic | Episode status changes to complete/error |
| Subtitle upload | S3, Lambda, backend API | Subtitle upload route, converter Lambda | `.srt` file is converted to `.vtt` |
| Monitoring | CloudWatch, SNS | Alarms and log groups | Errors or high resource usage trigger alerts |

#### Example implementation flow

When implementing a new feature, identify the full path from UI to AWS:

~~~text
Frontend action
-> Backend API on EC2
-> AWS service call
-> Database update in RDS
-> Frontend displays updated result
~~~

For example, the video upload function follows this path:

~~~text
Admin selects movie file
-> React admin page calls backend upload API
-> Backend uploads source file to S3 input bucket
-> Backend creates a MediaConvert job
-> EventBridge catches the job result
-> Lambda calls the backend webhook
-> Backend updates episode status and HLS URL in RDS
-> User watches HLS video through CloudFront
~~~

#### Report writing note

For each later workshop section, explain three points:

1. What service is configured.
2. Which project function uses that service.
3. What screenshot or output proves that the function works.

<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### AWS service map by website feature

| Netflop feature | AWS service | Reason |
| --- | --- | --- |
| Host frontend/backend | EC2 | Runs Node.js, Nginx, and PM2 on one server |
| Movie/user database | RDS MySQL | Managed database with backups and security groups |
| Store source videos | S3 input bucket | Durable storage for large MP4/MKV files |
| Store HLS output | S3 output bucket | Stores manifests, segments, subtitles, and media assets |
| Transcode video | MediaConvert | Generates adaptive HLS renditions from 360p to 1080p |
| Update job status | EventBridge + Lambda | Notifies the backend when a job is COMPLETE or ERROR |
| Fast video delivery | CloudFront | CDN for HLS manifests and segments |
| Protect streams | CloudFront signed cookies | Limits HLS access by time and policy |
| Operations | CloudWatch + SNS | Metrics, logs, alarms, and email alerts |

#### Services not required in the current main architecture

API Gateway, Application Load Balancer, AWS WAF, and Route 53 are not mandatory in the current deployment. The website currently uses Nginx reverse proxy on EC2 and Cloudflare for DNS/HTTPS. These services can be added later when the system needs multiple EC2 instances, managed rate limiting, or AWS-managed DNS.

#### How to prove the service map in the report

1. Capture a screenshot of each real AWS resource.
2. Include resource names such as <code>netflop-db</code>, <code>netflop-input-source</code>, and <code>netflop-output-source</code>.
3. Explain which website feature depends on each resource.
4. Add the related code samples in the following workshop sections.
<!-- NETFLOP_IMPLEMENTATION_END -->
