---
title: "Clean up application resources"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

#### Goal

Clean up or review application-layer resources such as EC2, Nginx, PM2, domain settings, IAM, and RDS. If the production system is still required, do not remove core resources; only remove test or unused resources.

#### Resources to check

| Resource | Action |
| --- | --- |
| EC2 `netflop-web` | Keep if the website is still running; stop if only demo/testing |
| EBS volume | Check for volumes not attached to instances |
| Elastic IP | Keep if the domain still points to it; release if unused |
| Security Group | Remove redundant rules, especially wide SSH/RDS access |
| IAM user/access key | Disable unused keys; prefer IAM roles |
| RDS `netflop-db` | Keep production; snapshot before deleting if needed |
| Cloudflare DNS | Keep production records; remove test records |

#### Notes for RDS

Do not delete the production database until a backup exists. Before cleanup:

1. Export the database or create a snapshot.
2. Verify the backend no longer depends on that database.
3. Record the endpoint and snapshot information in the report.

#### Expected results

* No test instances, volumes, or keys remain causing cost.
* Security Groups are smaller and more secure.
* The production database has a backup before any major change.



<!-- NETFLOP_DETAIL_START -->
#### How to clean up application resources

If the environment is only being stopped for a demo:

~~~bash
pm2 stop netflop-api
sudo systemctl stop nginx
~~~

If you are deleting infrastructure completely, do it in order:

1. Backup the RDS database or create a snapshot.
2. Stop EC2 if no longer needed.
3. Verify there are no attached EBS volumes you still need.
4. Release Elastic IP if it is no longer used by the domain.
5. Delete unused Security Groups.
6. Disable unused access keys.

#### Check RDS before deleting

~~~bash
aws rds describe-db-instances --region ap-southeast-1
aws rds create-db-snapshot \
  --db-instance-identifier netflop-db \
  --db-snapshot-identifier netflop-db-final-snapshot \
  --region ap-southeast-1
~~~

Do not delete RDS until you have a snapshot or exported data you want to keep.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Clean up application resources

If this is only a temporary break, stop local backend/frontend and optionally stop EC2. Terminate EC2 only when the demo environment is no longer needed.

#### Commands on EC2

~~~bash
pm2 status
pm2 stop netflop-api
sudo systemctl stop nginx
~~~

#### Stop EC2 with CLI

~~~bash
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxxxx --region ap-southeast-1
~~~

#### RDS backup

Before deleting RDS, create a snapshot:

~~~bash
aws rds create-db-snapshot \
  --db-instance-identifier netflop-db \
  --db-snapshot-identifier netflop-db-before-cleanup
~~~


<!-- NETFLOP_IMPLEMENTATION_END -->
