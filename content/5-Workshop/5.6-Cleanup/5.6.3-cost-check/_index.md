---
title: "Check cost after cleanup"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.6.3. </b> "
---

#### Items to check

* EC2 running instances
* Unattached EBS volumes
* Unused Elastic IPs
* RDS databases/snapshots
* S3 storage
* CloudFront distributions
* CloudWatch logs/alarms
* Lambda functions
* NAT Gateway if created

#### Checking steps

1. Open AWS Billing Dashboard.
2. Check cost by service.
3. Open Cost Explorer if detailed checking is needed.
4. Check AWS Budgets alerts.
5. Record cleanup results in the report.

{{% notice info %}}
Image needed: Billing Dashboard, Cost Explorer, or AWS Budgets after cleanup.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/budget.png)

<!-- NETFLOP_DETAIL_START -->

#### How to check remaining paid resources

After deleting or stopping project resources, check the services that can still create cost. The goal is to confirm that the account no longer keeps unnecessary compute, database, storage, CDN, log, or networking resources.

~~~bash
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" --region ap-southeast-1
aws ec2 describe-volumes --filters "Name=status,Values=available" --region ap-southeast-1
aws ec2 describe-addresses --region ap-southeast-1
aws rds describe-db-instances --region ap-southeast-1
aws s3 ls
aws cloudfront list-distributions
aws lambda list-functions --region ap-southeast-1
~~~

If any resource is still required for the demo, keep it and record the reason. If it is no longer required, stop or delete it from the AWS Console.

#### Console checking steps

1. Open AWS Billing Dashboard and check current month charges.
2. Open Cost Explorer and group cost by service.
3. Check whether EC2, RDS, S3, CloudFront, CloudWatch, Lambda, and NAT Gateway still appear.
4. Open AWS Budgets and confirm that budget alerts are configured if the account will stay active.
5. Take one screenshot for the report after cleanup.

#### Simple cost report table

| Service | Remaining resource | Estimated status |
| --- | --- | --- |
| EC2 | Running/stopped instance, EBS volume, Elastic IP | Check and record |
| RDS | Database instance or snapshot | Check and record |
| S3 | Media buckets and object storage | Check and record |
| CloudFront | Distribution and data transfer | Check and record |
| CloudWatch | Logs and alarms | Check and record |
| Lambda | Remaining functions | Check and record |
| NAT Gateway | Any active NAT Gateway | Delete if not used |

#### Final cleanup conclusion

In the report, conclude this section with a short sentence such as:

~~~text
After cleanup, the main remaining cost risks are EC2/RDS if they are still running, S3 storage if media files are kept, and CloudFront data transfer if the distribution remains active.
~~~

<!-- NETFLOP_DETAIL_END -->
