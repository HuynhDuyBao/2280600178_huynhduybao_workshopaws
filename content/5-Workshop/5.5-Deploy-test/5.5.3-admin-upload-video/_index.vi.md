---
title: "Kiểm thử admin upload video"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

#### Mục tiêu

Kiểm thử toàn bộ pipeline admin upload video, từ lúc chọn file đến khi video được convert sang HLS và tập phim chuyển sang trạng thái hoàn thành.

#### Luồng kiểm thử

1. Admin đăng nhập vào trang quản trị.
2. Vào phần quản lý phim hoặc quản lý tập phim.
3. Chọn phim/tập cần upload.
4. Chọn video MP4/MKV.
5. Chọn banner tập nếu có.
6. Bấm tải video.
7. Frontend hiển thị thanh tiến trình upload.
8. File gốc được upload lên `netflop-input-source`.
9. Backend tạo MediaConvert job.
10. Giao diện chuyển sang trạng thái đang xử lý.
11. MediaConvert xuất HLS sang `netflop-output-source`.
12. EventBridge nhận trạng thái COMPLETE.
13. Lambda `netflop-mediaconvert-notifier` gọi webhook backend.
14. Backend cập nhật tập phim sang `ready`.
15. Trang admin tự cập nhật trạng thái hoàn thành, không cần bấm sync thủ công.

#### Dữ liệu backend cần lưu

| Trường | Ý nghĩa |
| --- | --- |
| `jobId` | ID job MediaConvert |
| `source_url` | Vị trí video gốc trên S3 input |
| `hls_url` | URL manifest HLS output |
| `cloudfront_url` | URL phát qua CloudFront |
| `upload_status` | Trạng thái xử lý |
| `duration` | Thời lượng video nếu có |
| `episode_banner` | Banner riêng của tập |

#### Kiểm thử file lớn

Với video khoảng 5GB, cần xác nhận:

* Không upload qua request backend thông thường gây lỗi `413 Request Entity Too Large`.
* Upload dùng S3/multipart hoặc cơ chế phù hợp cho file lớn.
* Thanh tiến trình phản ánh được upload progress.
* Sau upload, trạng thái chuyển sang MediaConvert processing.

#### Kết quả mong đợi

* S3 input có video gốc.
* MediaConvert job hoàn tất.
* S3 output có file `.m3u8` và segment.
* Database cập nhật trạng thái tập phim.
* Người dùng phát được video qua CloudFront.
* Admin nhìn thấy tiến trình và kết quả hoàn thành ngay trên UI.

{{% notice info %}}
Cần thêm ảnh: admin upload video, thanh tiến trình upload/processing/completed, object S3 input, MediaConvert job Complete, HLS output, bảng tập phim trạng thái hoàn thành và player phát tập vừa upload.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminin1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminput1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup2.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.3-admin-upload-video/adminup3.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện upload tập phim trong admin

Frontend admin dùng multipart upload để xử lý file lớn. Khi upload xong, backend tạo bản ghi tập phim và gửi MediaConvert xử lý HLS.

#### Code mẫu frontend gọi upload multipart

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
setMessage('Đã upload video lên S3 và gửi MediaConvert xử lý HLS.');
~~~

#### Code mẫu API upload

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

#### Code mẫu backend sau khi complete multipart

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

#### Kịch bản test admin

1. Chọn phim và nhập tên tập.
2. Chọn video MP4/MKV.
3. Theo dõi thanh upload progress.
4. Kiểm tra S3 input có file gốc.
5. Kiểm tra MediaConvert job SUBMITTED/PROGRESSING/COMPLETE.
6. Kiểm tra bảng tập phim chuyển sang ready.
7. Mở web người dùng và phát tập vừa upload.
<!-- NETFLOP_DETAIL_END -->
