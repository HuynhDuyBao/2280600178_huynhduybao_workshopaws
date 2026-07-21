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

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Admin episode upload test

This is the most important workshop feature because it combines frontend, backend, S3, MediaConvert, Lambda, and CloudFront.

#### UI test steps

1. Sign in as administrator.
2. Open episode management or the Video & Trailer step in add/edit movie.
3. Enter movie ID, episode name, and optional episode banner.
4. Select an MP4/MKV file.
5. Click upload video.
6. Watch the multipart upload progress bar.
7. After upload finishes, the status changes to MediaConvert processing.
8. When the job completes, the episode status becomes ready/completed.
9. Open the watch page and play the new episode.

#### Frontend chunk upload sample

~~~js
for (let offset = 0, partNumber = 1; offset < file.size; offset += CHUNK_SIZE, partNumber += 1) {
  const end = Math.min(offset + CHUNK_SIZE, file.size);
  const chunk = file.slice(offset, end);
  const formData = new FormData();
  formData.append('key', uploadSession.key);
  formData.append('uploadId', uploadSession.uploadId);
  formData.append('partNumber', String(partNumber));
  formData.append('chunk', chunk, file.name);

  const response = await uploadPartWithRetry(formData, {
    onUploadProgress: (event) => {
      const percent = ((uploadedBytes + event.loaded) / file.size) * 100;
      onProgress(Math.min(99, Math.max(1, percent)));
    }
  });
}
~~~

#### Backend complete multipart and create job

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

#### Why manual Sync is no longer required

MediaConvert sends job state changes to EventBridge. Lambda <code>netflop-mediaconvert-notifier</code> receives the event and calls backend webhook <code>/api/uploads/mediaconvert/events</code>. The backend uses jobId/episodeId to update the episode status.

#### Lambda notifier sample

~~~js
const response = await fetch(webhookUrl, {
  method: 'POST',
  headers: {
    'content-type': 'application/json',
    'x-netflop-event-secret': secret
  },
  body: JSON.stringify(event)
});
~~~

<!-- NETFLOP_IMPLEMENTATION_END -->
