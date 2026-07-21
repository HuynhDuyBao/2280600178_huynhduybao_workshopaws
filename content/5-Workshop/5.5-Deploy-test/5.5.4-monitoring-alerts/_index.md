---
title: "Check monitoring and alerts"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

#### CloudWatch alarms

Check these alarms:

* EC2 CPU high
* EC2 memory high
* EC2 disk high
* EC2 status check failed
* RDS CPU high
* RDS connections high
* RDS free storage low
* RDS freeable memory low
* Lambda errors for `netflop-mediaconvert-notifier`
* Lambda errors for `netflop-subtitle-converter`

SNS topic:

```text
netflop-alerts
```

#### Checking steps

1. Open CloudWatch console.
2. Check alarm states.
3. Open Lambda logs.
4. Check SNS topic `netflop-alerts`.
5. Check Billing Dashboard/AWS Budgets.

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw2.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/sns.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/console.png)

<!-- NETFLOP_DETAIL_START -->
#### How to create and check alarms

CloudWatch alarms track important thresholds such as:

* EC2 high CPU.
* EC2 disk almost full.
* RDS high CPU or connection count.
* Lambda errors.
* Low RDS free storage.

#### CLI checks

~~~bash
aws cloudwatch describe-alarms --region ap-southeast-1
aws logs describe-log-groups --region ap-southeast-1
aws sns list-topics --region ap-southeast-1
~~~

#### Console checks

1. Open CloudWatch -> Alarms.
2. Filter alarms beginning with `Netflop-`.
3. Check states: OK, ALARM, or INSUFFICIENT_DATA.
4. Open CloudWatch Logs for Lambda logs.
5. Open SNS -> topic `netflop-alerts` and confirm email subscription is Confirmed.

If all alarms are OK, the system is healthy at the checking time. If an alarm is ALARM, record the reason and fix direction in the report.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Monitoring goal

Monitoring helps detect production issues after deployment: high EC2 CPU, backend downtime, RDS load, S3/CloudFront cost growth, or MediaConvert job failures.

#### Components to monitor

| Component | Metrics/logs |
| --- | --- |
| EC2 | CPUUtilization, NetworkIn/Out, StatusCheckFailed |
| PM2/backend | error logs, API 500, API health check |
| Nginx | access log and error log |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections |
| S3 | bucket size and request count |
| CloudFront | 4xx/5xx error rate and data transfer |
| Lambda | Errors, Duration, Invocations |
| MediaConvert | Job ERROR/CANCELED |

#### Create SNS topic for alerts

~~~bash
aws sns create-topic --name netflop-alerts
aws sns subscribe --topic-arn arn:aws:sns:ap-southeast-1:<account-id>:netflop-alerts --protocol email --notification-endpoint <email>
~~~

#### EC2 high CPU alarm example

~~~bash
aws cloudwatch put-metric-alarm \
  --alarm-name netflop-ec2-high-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-xxxxxxxxxxxxxxxxx
~~~

#### Verification

1. Open CloudWatch -> Alarms.
2. Confirm the alarm is OK.
3. Open Log groups if CloudWatch Agent is configured.
4. Confirm the SNS email subscription.

<!-- NETFLOP_IMPLEMENTATION_END -->
