---
title: "Deploy and test the application"
date: 2026-07-10
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

This section describes how to deploy or reload Netflop on EC2, test end-user functionality, validate admin video upload flow, and verify monitoring and alerts.

#### Contents

1. [Deploy/reload with EC2 and Systems Manager](5.5.1-deploy-ssm/)
2. [Test user features](5.5.2-user-test/)
3. [Test admin video upload](5.5.3-admin-upload-video/)
4. [Check monitoring and alerts](5.5.4-monitoring-alerts/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Post-deployment test plan

After deployment, the homepage is not enough. The main business flows must be tested:

1. Backend health check.
2. Movie list and movie detail page.
3. Normal user registration and sign-in.
4. Admin sign-in.
5. Upload video to S3 and MediaConvert.
6. Play HLS through CloudFront.
7. Upload SRT/VTT subtitles.
8. Save watch history per account.
9. Check mobile responsiveness.
10. Check logs and monitoring.

#### Quick test commands

~~~bash
curl -I https://netflop.win
curl https://netflop.win/api/health
pm2 status
sudo systemctl status nginx --no-pager
aws s3 ls s3://netflop-input-source
aws s3 ls s3://netflop-output-source
~~~


![cw](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw1.png)
![cw](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw2.png)

![cw](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/console.png)
![cw](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/sns.png)
<!-- NETFLOP_IMPLEMENTATION_END -->
