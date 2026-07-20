---
title: "Worklog Week 11"
date: 2026-06-29
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 objectives:

* Test the complete Netflop system after AWS integration.
* Test the MediaConvert, EventBridge, Lambda, and subtitle automation flows.
* Add monitoring, logging, alerting, and cost control.
* Complete the main features before the final summary phase.

### Tasks implemented this week:

| Day | Task | Start date | Completion date | Reference |
| --- | --- | --- | --- | --- |
| Monday | - Test user flows: browse movie list, search, view details, and stream HLS video <br> - Record UI and API issues | 29/06/2026 | 29/06/2026 | Project test cases |
| Tuesday | - Test admin flows: upload video, create MediaConvert job, update poster/subtitle <br> - Check data stored in RDS and S3 | 30/06/2026 | 30/06/2026 | RDS, S3, MediaConvert |
| Wednesday | - Test EventBridge invoking Lambda when MediaConvert is COMPLETE/ERROR/CANCELED <br> - Check webhook `/api/uploads/mediaconvert/events` | 01/07/2026 | 01/07/2026 | EventBridge, Lambda |
| Thursday | - Test `.srt` subtitle upload and Lambda conversion to `.vtt` <br> - Review Security Groups, IAM policies, and bucket permissions | 02/07/2026 | 02/07/2026 | Lambda, IAM |
| Friday | - Check CloudWatch alarms, SNS topic `netflop-alerts`, Billing Dashboard, and AWS Budgets <br> - Summarize fixed issues and remaining limitations | 03/07/2026 | 03/07/2026 | CloudWatch, SNS |

### Week 11 outcomes:

* Completed the main features of Netflop at an internship demo level.
* Tested user flows and admin flows.
* Tested the upload video pipeline, MediaConvert, CloudFront HLS, and status update through Lambda/EventBridge.
* Learned how to monitor logs, metrics, and alerts with CloudWatch/SNS.
* Reviewed important security points such as Security Groups, IAM policies, S3 bucket permissions, and RDS public access.
* Cleaned up unnecessary resources and improved cost control in the AWS account.
