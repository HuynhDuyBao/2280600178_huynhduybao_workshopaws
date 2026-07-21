---
title: "Clean up resources"
date: 2026-07-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

This section explains how to review and clean up AWS resources after the workshop to avoid unnecessary costs.


![budget](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/budget.png)
![budget](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.6-Cleanup/5.6.3-cost-check/cost.png)


#### Contents

1. [Clean up application resources](5.6.1-application-resources/)
2. [Clean up media and automation resources](5.6.2-media-resources/)
3. [Check cost after cleanup](5.6.3-cost-check/)

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Cleanup goal

Cleanup reduces cost after a demo or when the environment is no longer used. Important data should be backed up before deleting resources.

#### Cleanup principles

1. Back up the database before deleting RDS.
2. Export or keep important videos before deleting S3 objects.
3. Stop EC2 for temporary breaks; terminate only when no longer needed.
4. Delete CloudFront last because it must be disabled before deletion.
5. Check Cost Explorer after cleanup.

#### Suggested cleanup order

~~~text
Back up RDS
-> Stop PM2/Nginx if needed
-> Empty S3 test prefixes
-> Disable CloudFront test distribution
-> Delete Lambda/EventBridge test rules
-> Stop or terminate EC2
-> Delete RDS snapshot or database only if no longer needed
~~~
<!-- NETFLOP_IMPLEMENTATION_END -->
