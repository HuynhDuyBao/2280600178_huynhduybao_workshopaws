---
title: "Configure MediaConvert"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

#### Goal

AWS Elemental MediaConvert is used to convert MP4/MKV files uploaded by the admin into HLS format for web and mobile playback. HLS allows the player to select the best quality based on network conditions.

#### Endpoint

The MediaConvert endpoint for this deployment is:

```text
https://mediaconvert.ap-southeast-1.amazonaws.com
```

#### Job flow

```text
Admin upload video
   |
   v
S3 input source
   |
   v
Backend CreateJob
   |
   v
MediaConvert
   |
   v
S3 output HLS
```

#### Required output

MediaConvert must produce HLS output at several quality levels:

| Quality | Purpose |
| --- | --- |
| 360p | Weak connections / mobile |
| 480p | Medium quality |
| 720p | Common HD |
| 1080p | Full HD |

The main output manifest is:

```text
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
```

#### Episode status

The backend stores processing status so admins can monitor progress:

| Status | Meaning |
| --- | --- |
| `processing` | Video uploaded and conversion is in progress |
| `ready` | Conversion complete and the episode is playable |
| `failed` | Conversion failed or webhook reported an error |

The admin frontend shows progress stages: upload to S3, MediaConvert processing, and completion.

#### Verification

1. Upload an episode from the admin page.
2. Open the MediaConvert console.
3. Check the job is running or complete.
4. Inspect the `index.m3u8` file and segments in S3 output.
5. Open the episode in the web player.

![mediaconvert](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.2-mediaconvert/mediaconvert.png)
![streamm3u8](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.2-mediaconvert/hls.png)

<!-- NETFLOP_DETAIL_START -->
#### How to create a MediaConvert job

After the source video exists in S3 input, the backend calls MediaConvert `CreateJob`. The job includes:

* MediaConvert role ARN.
* Input file as `s3://bucket/key`.
* Output destination in S3 output.
* HLS outputs for 360p, 480p, 720p, and 1080p.
* Metadata such as movieId, episodeId, outputPrefix, and masterKey.

#### Simplified sample code

~~~js
async function createHlsJob({ inputS3Uri, movieId, episodeId }) {
  const outputPrefix = 'movies/' + movieId + '/episodes/' + episodeId + '/hls/';
  const masterKey = outputPrefix + 'index.m3u8';

  const command = new CreateJobCommand({
    Role: awsConfig.mediaConvertRoleArn,
    UserMetadata: { movieId: String(movieId), episodeId: String(episodeId), outputPrefix, masterKey },
    Settings: {
      Inputs: [{ FileInput: inputS3Uri }],
      OutputGroups: [{
        Name: 'Apple HLS',
        OutputGroupSettings: {
          Type: 'HLS_GROUP_SETTINGS',
          HlsGroupSettings: {
            Destination: 's3://' + awsConfig.s3OutputBucket + '/' + outputPrefix + 'index',
            SegmentLength: 6,
            OutputSelection: 'MANIFESTS_AND_SEGMENTS'
          }
        },
        Outputs: [
          createHlsOutput({ nameModifier: '_360p', width: 640, height: 360 }),
          createHlsOutput({ nameModifier: '_480p', width: 854, height: 480 }),
          createHlsOutput({ nameModifier: '_720p', width: 1280, height: 720 }),
          createHlsOutput({ nameModifier: '_1080p', width: 1920, height: 1080 })
        ]
      }]
    }
  });

  const result = await client.send(command);
  return { jobId: result.Job?.Id, outputPrefix, masterKey };
}
~~~

#### Expected output

~~~text
movies/{movieId}/episodes/{episodeId}/hls/index.m3u8
movies/{movieId}/episodes/{episodeId}/hls/*.ts
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### MediaConvert job steps

1. Backend receives <code>inputS3Uri</code> after video upload completes.
2. Backend creates <code>outputPrefix</code> from movieId and episodeId.
3. Backend calls AWS SDK <code>CreateJobCommand</code>.
4. Backend passes the MediaConvert role ARN.
5. The job uses Apple HLS output group.
6. The job creates 360p, 480p, 720p, and 1080p renditions.
7. Backend stores jobId and CloudFront playback URL in the episode record.

#### MediaConvert service code sample

~~~js
const outputPrefix = 'movies/' + movieId + '/episodes/' + episodeId + '/hls/';
const masterKey = outputPrefix + 'index.m3u8';

const command = new CreateJobCommand({
  Role: awsConfig.mediaConvertRoleArn,
  UserMetadata: {
    movieId: String(movieId),
    episodeId: String(episodeId),
    outputPrefix,
    masterKey
  },
  Settings: {
    Inputs: [{ FileInput: inputS3Uri }],
    OutputGroups: [{
      Name: 'Apple HLS',
      OutputGroupSettings: {
        Type: 'HLS_GROUP_SETTINGS',
        HlsGroupSettings: {
          Destination: 's3://' + awsConfig.s3OutputBucket + '/' + outputPrefix + 'index',
          SegmentLength: 6,
          OutputSelection: 'MANIFESTS_AND_SEGMENTS'
        }
      },
      Outputs: [
        createHlsOutput({ nameModifier: '_360p', width: 640, height: 360 }),
        createHlsOutput({ nameModifier: '_480p', width: 854, height: 480 }),
        createHlsOutput({ nameModifier: '_720p', width: 1280, height: 720 }),
        createHlsOutput({ nameModifier: '_1080p', width: 1920, height: 1080 })
      ]
    }]
  }
});
~~~

#### Update database after job submission

~~~js
const job = await mediaConvertService.createHlsJob({
  inputS3Uri: uploaded.s3Uri,
  movieId,
  episodeId: pendingEpisode.MaTap
});

await episodeModel.updateUploadProcessing(pendingEpisode.MaTap, {
  jobId: job.jobId,
  hlsUrl: playbackUrls.hlsUrl,
  cloudFrontUrl: playbackUrls.cloudFrontUrl,
  outputKey: job.masterKey
});
~~~


<!-- NETFLOP_IMPLEMENTATION_END -->
