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

{{% notice info %}}
Image needed: CloudWatch alarms, Lambda logs, SNS topic `netflop-alerts`, and Billing Dashboard.
{{% /notice %}}

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
