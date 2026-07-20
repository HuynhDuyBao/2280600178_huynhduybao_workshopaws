---
title: "Worklog Week 10"
date: 2026-06-22
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 objectives:

* Integrate Amazon S3 into Netflop to store images, posters, video input, and HLS output.
* Learn the video processing flow with MediaConvert and video delivery through CloudFront.
* Test the media upload flow from the admin page.

### Tasks implemented this week:

| Day | Task | Start date | Completion date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Check buckets `netflop-input-source` and `netflop-output-source` <br> - Document each bucket role in the video upload flow | 22/06/2026 | 22/06/2026 | S3 |
| Tuesday | - Configure IAM roles/policies for EC2, S3, and MediaConvert <br> - Limit permissions based on required actions only | 23/06/2026 | 23/06/2026 | IAM, S3 Policy |
| Wednesday | - Test poster, thumbnail, and video upload from the admin page <br> - Store URLs or object keys in the database | 24/06/2026 | 24/06/2026 | Backend, S3 |
| Thursday | - Learn MediaConvert jobs for MP4/MKV to HLS conversion <br> - Check `.m3u8` output and segments in S3 output | 25/06/2026 | 25/06/2026 | MediaConvert |
| Friday | - Check CloudFront distribution for HLS streaming <br> - Document signed cookies through `/api/stream/session` | 26/06/2026 | 26/06/2026 | CloudFront |

### Week 10 outcomes:

* Integrated S3 input/output buckets into the Netflop media management flow.
* Uploaded and displayed posters, thumbnails, or sample media files.
* Understood how to store object keys/URLs in the database so both backend and frontend can use them.
* Understood the role of MediaConvert in converting videos to multi-quality HLS.
* Understood the role of CloudFront in HLS streaming and signed-cookie video protection.
