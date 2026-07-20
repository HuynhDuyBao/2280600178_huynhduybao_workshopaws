---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# Building an Event-Driven Media Pipeline on AWS

Hello everyone,
In the Netflop project, after the video was processed by AWS Elemental MediaConvert, the team needed to update the movie state so users could watch it immediately.
Initially, the team considered having the Backend continuously check the Job (Polling) state, but this method was both resource-intensive and inefficient.
Then, the team switched to an Event-Driven Architecture model with:
MediaConvert → EventBridge → Lambda → Backend Webhook
The workflow is as follows:
📌 MediaConvert completes the Job.

📌 EventBridge automatically receives the event.

📌 Lambda is triggered and calls the Webhook back to the Backend.

📌 The Backend updates the episode state from Processing to Ready.

As a result:
✅ No need for continuous Polling.

✅ The Backend is lighter.

✅ The system responds almost immediately after encoding is complete.

Through this project, our team found EventBridge to be a very useful service for building automated workflows between AWS services.

## References

- [MediaConvert events with EventBridge](https://docs.aws.amazon.com/mediaconvert/latest/ug/eventbridge_events.html)
- [Amazon EventBridge events for MediaConvert](https://docs.aws.amazon.com/eventbridge/latest/ref/events-ref-mediaconvert.html)
- [MediaConvert event list](https://docs.aws.amazon.com/mediaconvert/latest/ug/mediaconvert_event_list.html)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
