---
title: "Test admin video upload"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

#### Upload flow

1. Admin signs in to the admin page.
2. Admin selects a movie/episode.
3. Admin selects an MP4/MKV video file.
4. Backend receives multipart upload request.
5. Video is uploaded to `netflop-input-source`.
6. Backend creates a MediaConvert job.
7. MediaConvert writes HLS output to `netflop-output-source`.
8. EventBridge receives COMPLETE.
9. Lambda `netflop-mediaconvert-notifier` calls backend webhook.
10. Backend updates episode `upload_status` to ready.

#### Backend values to check

* `jobId`
* `hls_url`
* `cloudfront_url`
* `upload_status`

{{% notice info %}}
Image needed: admin upload screen, S3 input object, MediaConvert Complete job, HLS output, and episode ready status.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminin1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminput1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup2.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup3.png)

<!-- NETFLOP_DETAIL_START -->
#### How admin video upload works

The admin UI uses multipart upload for large files. After upload completes, the backend creates an episode record and sends the source video to MediaConvert for HLS processing.

#### Sample frontend multipart upload call

~~~js
const payload = await uploadVideoInChunks({
  file,
  movieId,
  episodeName,
  thumbnailUrl,
  duration,
  onProgress: setUploadProgress
});

setUploadProgress(100);
setMessage('Video uploaded to S3 and sent to MediaConvert for HLS processing.');
~~~

#### Sample upload API wrapper

~~~js
export const uploadApi = {
  startVideoMultipart: (payload) => axiosClient.post('/uploads/videos/multipart/start', payload),
  uploadVideoMultipartPart: (formData, options = {}) => axiosClient.post(
    '/uploads/videos/multipart/parts',
    formData,
    { headers: { 'Content-Type': 'multipart/form-data' }, onUploadProgress: options.onUploadProgress }
  ),
  completeVideoMultipart: (payload) => axiosClient.post('/uploads/videos/multipart/complete', payload)
};
~~~

#### Sample backend after multipart complete

~~~js
const uploaded = await awsS3Service.completeVideoMultipartUpload({ key, uploadId, parts });
const pendingEpisode = await episodeModel.createUploadEpisode({
  movieId,
  name: episodeName,
  sourceUrl: uploaded.s3Uri,
  uploadStatus: 'uploaded'
});

const job = await mediaConvertService.createHlsJob({
  inputS3Uri: uploaded.s3Uri,
  movieId,
  episodeId: pendingEpisode.MaTap
});
~~~

#### Test scenario

1. Select a movie and enter episode name.
2. Select an MP4/MKV file.
3. Watch upload progress.
4. Confirm source file in S3 input.
5. Confirm MediaConvert job changes from SUBMITTED/PROGRESSING to COMPLETE.
6. Confirm episode status becomes ready.
7. Open the user website and play the uploaded episode.
<!-- NETFLOP_DETAIL_END -->
