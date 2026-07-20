---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

# A Small Experience from Building a Movie Streaming Website on AWS

Hello everyone,

During the development of **Netflop**, our team once asked an important question:

**Why not serve videos directly from Amazon S3 instead of using Amazon CloudFront?**

After researching and implementing the solution in practice, we found that CloudFront provides many useful benefits.

Instead of allowing users to access S3 directly, all **HLS** video content is delivered through **Amazon CloudFront**.

Some benefits our team observed include:

- Lower latency when users watch videos.
- Faster loading thanks to caching at Edge Locations.
- Fewer direct requests to S3.
- Built-in HTTPS support.
- The ability to protect content with **CloudFront Signed Cookies**, so only authenticated users can watch videos.

For websites with a large amount of media content, such as movie streaming platforms or online learning systems, CloudFront is definitely a service worth exploring.

When building streaming systems, do you usually use **CloudFront**, direct **S3 access**, or another CDN? We would love to learn from your experience.

## References

- [Amazon CloudFront](https://aws.amazon.com/cloudfront/)
- [What is Amazon CloudFront?](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Serving private content with signed URLs and signed cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/PrivateContent.html)
- [Use signed cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html)
