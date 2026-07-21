---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}}

![FFmpeg on EC2 compared with AWS Elemental MediaConvert](/2280600178_huynhduybao_workshopaws/images/3-BlogsPosted/3.1-Blog1/ffmpeg-vs-mediaconvert.png)

# Why Our Team Chose AWS Elemental MediaConvert Instead of FFmpeg

Hello everyone,

While working on **Netflop - a movie streaming website built on AWS**, our team explored and implemented several AWS services to build a complete media pipeline. One of the decisions we spent quite a lot of time considering was how to encode uploaded videos.

At first, we planned to use **FFmpeg running directly on Amazon EC2** to convert videos after upload. However, after testing with multiple large video files, the EC2 instance CPU often stayed at a high usage level, processing took a long time, and managing multiple concurrent encoding tasks became complicated.

After further research, our team decided to switch to **AWS Elemental MediaConvert**, combined with **Amazon S3, Amazon CloudFront, Amazon EventBridge, and AWS Lambda** to build an automated video processing workflow.

After implementation, we noticed several clear benefits:

- Automatically converts videos to **HLS** with multiple quality levels such as 360p, 480p, 720p, and 1080p.
- Significantly reduces the load on EC2 because the encoding process is handled by MediaConvert.
- Automatically updates movie status after encoding is completed through **EventBridge + Lambda**.
- Makes the architecture easier to scale as the number of videos or users increases.

Through this project, our team realized that using AWS **Managed Services** not only reduces operational effort, but also helps the system become more stable and scalable compared with managing everything directly on servers.

This was a very valuable experience for our team during the learning and project implementation process.

Have you ever used **AWS Elemental MediaConvert** or another solution for a video streaming system? We would love to hear your experiences and suggestions so our team can continue improving the project.

Thank you for reading!

## References

- [AWS Elemental MediaConvert](https://aws.amazon.com/mediaconvert/)
- [What is AWS Elemental MediaConvert?](https://docs.aws.amazon.com/mediaconvert/latest/ug/what-is.html)
- [Working with jobs in AWS Elemental MediaConvert](https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-jobs.html)
