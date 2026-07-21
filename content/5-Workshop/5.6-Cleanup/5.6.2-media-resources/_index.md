---
title: "Clean up media and automation resources"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

#### Resources to check

* S3 bucket `netflop-input-source`
* S3 bucket `netflop-output-source`
* Unused CloudFront distributions
* Unused MediaConvert jobs/templates
* Lambda `netflop-mediaconvert-notifier`
* Lambda `netflop-subtitle-converter`
* EventBridge rule `netflop-mediaconvert-job-state-change`
* IAM roles/policies for MediaConvert and Lambda

#### Cleanup steps

1. Back up videos/HLS/subtitles if needed.
2. Empty S3 buckets before deletion.
3. Disable/delete unused CloudFront distributions.
4. Delete unused Lambda functions.
5. Delete unused EventBridge rules.
6. Review IAM roles and policies.


<!-- NETFLOP_DETAIL_START -->

#### How to clean up S3 media files

S3 is the first place to check because Netflop stores uploaded source videos, converted HLS folders, images, avatars, banners, and subtitle files there. Before deleting anything, confirm whether the files are still needed for the final demo.

~~~bash
aws s3 ls s3://netflop-input-source
aws s3 ls s3://netflop-output-source
~~~

If the internship demo is already finished and media files are no longer needed, empty the buckets:

~~~bash
aws s3 rm s3://netflop-input-source --recursive
aws s3 rm s3://netflop-output-source --recursive
~~~

After the buckets are empty, delete unused buckets from the S3 console. If the buckets are kept for future testing, record that decision in the report.

#### How to clean up CloudFront

CloudFront distributions cannot be deleted while enabled. First identify the distribution that is used for HLS streaming.

~~~bash
aws cloudfront list-distributions
~~~

Then in the AWS Console:

1. Open CloudFront.
2. Select the unused distribution.
3. Disable the distribution.
4. Wait until the status changes from `Deployed` after disabling.
5. Delete the distribution if it is no longer required.

For the report, capture the distribution status screen after disabling or deleting.

#### How to clean up Lambda and EventBridge

The project uses Lambda mainly for automation, so check whether the functions are still required before deleting them.

~~~bash
aws lambda list-functions --region ap-southeast-1
aws events list-rules --region ap-southeast-1
~~~

If the MediaConvert notification flow is no longer needed, remove the EventBridge target first, then delete the rule, and finally delete the Lambda function.

~~~bash
aws events list-targets-by-rule --rule netflop-mediaconvert-job-state-change --region ap-southeast-1
aws events remove-targets --rule netflop-mediaconvert-job-state-change --ids TARGET_ID --region ap-southeast-1
aws events delete-rule --name netflop-mediaconvert-job-state-change --region ap-southeast-1
aws lambda delete-function --function-name netflop-mediaconvert-notifier --region ap-southeast-1
aws lambda delete-function --function-name netflop-subtitle-converter --region ap-southeast-1
~~~

Replace `TARGET_ID` with the actual target ID returned by the first command.

#### Cleanup result table

Use this table in the report after cleanup:

| Resource | Action | Result |
| --- | --- | --- |
| `netflop-input-source` | Emptied or kept | Record final status |
| `netflop-output-source` | Emptied or kept | Record final status |
| CloudFront HLS distribution | Disabled/deleted or kept | Record final status |
| `netflop-mediaconvert-notifier` | Deleted or kept | Record final status |
| `netflop-subtitle-converter` | Deleted or kept | Record final status |
| EventBridge MediaConvert rule | Deleted or kept | Record final status |

<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Clean up media resources

S3 and CloudFront can create cost when many videos are stored or watched. Separate real data from test data before deleting anything.

#### Check S3 usage

~~~bash
aws s3 ls s3://netflop-input-source --recursive --summarize
aws s3 ls s3://netflop-output-source --recursive --summarize
~~~

#### Delete test prefixes

~~~bash
aws s3 rm s3://netflop-input-source/uploads/test/ --recursive
aws s3 rm s3://netflop-output-source/movies/test/ --recursive
~~~

#### Recommended lifecycle rules

* Delete incomplete multipart uploads after a few days.
* Move rarely used source videos to a cheaper storage class if they must be retained.
* Do not automatically delete HLS output for movies that are still public.

#### CloudFront invalidation

If HLS output is changed or deleted while CloudFront still caches it, create an invalidation:

~~~bash
aws cloudfront create-invalidation --distribution-id <distribution-id> --paths "/movies/*"
~~~
<!-- NETFLOP_IMPLEMENTATION_END -->
