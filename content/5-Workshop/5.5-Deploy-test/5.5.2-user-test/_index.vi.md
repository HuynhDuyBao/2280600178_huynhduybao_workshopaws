---
title: "Kiểm thử chức năng người dùng"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Mục tiêu

Kiểm thử các chức năng người dùng cuối của Netflop sau khi deploy lên production domain `netflop.win`.

#### Các nhóm chức năng cần kiểm thử

| Nhóm chức năng | Kết quả mong đợi |
| --- | --- |
| Trang chủ | Hiển thị banner, danh sách phim mới cập nhật, phim xem nhiều, phim bộ/phim lẻ |
| Tìm kiếm | Tìm phim theo tên hoặc diễn viên |
| Chi tiết phim | Hiển thị poster, banner, thông tin phim, đánh giá và danh sách tập |
| Xem phim | Player phát HLS, có tua, fullscreen, chọn phụ đề |
| Mobile responsive | Movie card kéo ngang được, player không mất nút, subtitle selector dễ bấm |
| Đăng nhập/đăng ký | Tạo tài khoản và đăng nhập local thành công |
| Tiếp tục xem | Lưu tiến độ theo từng tài khoản, tài khoản khác không bị lẫn dữ liệu |
| Lịch sử xem | Hiển thị phim đã xem đúng theo user |
| Yêu thích/đánh giá/bình luận | Dữ liệu được lưu vào RDS |
| Avatar | Người dùng đổi avatar, ảnh upload lên S3 và UI cập nhật |

#### Kiểm thử player trên mobile

Các điểm cần kiểm tra riêng:

* Nút play/pause bấm được bằng cảm ứng.
* Thanh tua không bị kẹt khi có thông báo tiếp tục xem.
* Nút phụ đề, cài đặt và fullscreen không bị tràn ra ngoài khung.
* Phụ đề không bị hiển thị trùng hai lớp.
* Khi chọn tiếp tục xem, video chạy từ thời điểm đã lưu.

#### Kết quả mong đợi

Người dùng có thể xem phim ổn định trên desktop và điện thoại, không gặp lỗi CORS/mixed content, không lộ raw stream URL trong thông báo lỗi, và dữ liệu cá nhân được tách theo từng tài khoản.

{{% notice info %}}
Cần thêm ảnh: homepage desktop, homepage mobile, movie card dạng slide ngang, trang chi tiết phim, player desktop/mobile, phụ đề đang bật, tiếp tục xem theo tài khoản và trang hồ sơ/avatar.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/home.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/tieptucxem.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/tieptucxem2.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/moviedetail.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/movie1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/movierun.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện chức năng tiếp tục xem

Frontend lưu tiến độ xem theo tài khoản bằng cách gọi API history trong khi người dùng xem video. Không lưu bằng localStorage cho chức năng này, vì localStorage sẽ bị lẫn giữa các tài khoản trên cùng trình duyệt.

#### Code mẫu frontend lưu tiến độ

~~~js
const persistWatchProgress = useCallback((progress, force = false) => {
  if (!user || !movie?.id) return;

  const watchedSeconds = Math.max(0, Math.floor(Number(progress?.currentTime || 0)));
  if (!force && watchedSeconds < 1) return;

  movieApi.saveHistory(movie.id, {
    episodeId: selectedEpisode?.id || null,
    watchedSeconds
  }).catch(() => {});
}, [movie?.id, selectedEpisode?.id, user]);
~~~

#### Code mẫu backend upsert lịch sử xem

~~~js
await pool.execute(
  `INSERT INTO lichsu (UserID, TenDN, MaPhim, MaTap, ThoiGianXem, ThoiGian)
   VALUES (:userId, :username, :movieId, :episodeId, :watchedSeconds, NOW())
   ON DUPLICATE KEY UPDATE
     MaTap = VALUES(MaTap),
     ThoiGianXem = VALUES(ThoiGianXem),
     ThoiGian = VALUES(ThoiGian)`,
  { userId: user.id, username: user.ten_dang_nhap, movieId, episodeId, watchedSeconds }
);
~~~

#### Kịch bản test

1. Đăng nhập tài khoản A, xem phim đến phút 1.
2. Thoát trang, quay lại trang chủ.
3. Kiểm tra mục Tiếp tục xem có phim đó.
4. Đăng xuất, đăng nhập tài khoản B.
5. Kiểm tra tài khoản B không thấy lịch sử của tài khoản A.
6. Bấm Tiếp tục xem ở tài khoản A và xác nhận player nhảy đúng thời điểm đã lưu.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Kiểm thử người dùng

Luồng user cần chứng minh website không chỉ hiển thị phim mà còn lưu trạng thái theo từng tài khoản:

1. Đăng ký hoặc đăng nhập.
2. Mở một phim có nhiều tập.
3. Xem vài phút rồi thoát.
4. Vào Hồ sơ cá nhân -> Tiếp tục xem hoặc Lịch sử xem.
5. Đăng nhập tài khoản khác để kiểm tra dữ liệu không bị dùng chung.

#### Code frontend lưu tiến trình xem

~~~js
const watchedSeconds = Math.max(0, Math.floor(Number(progress?.currentTime || 0)));

movieApi.saveHistory(movie.id, {
  episodeId: selectedEpisode?.id || null,
  watchedSeconds
}).catch(() => {});
~~~

#### Code backend lưu theo user

~~~js
await pool.execute(
  'INSERT INTO lichsu (UserID, TenDN, MaPhim, MaTap, ThoiGianXem, ThoiGian) VALUES (:userId, :username, :movieId, :episodeId, :watchedSeconds, NOW()) ON DUPLICATE KEY UPDATE MaTap = VALUES(MaTap), ThoiGianXem = VALUES(ThoiGianXem), ThoiGian = VALUES(ThoiGian)',
  {
    userId: user.id,
    username: user.ten_dang_nhap,
    movieId,
    episodeId,
    watchedSeconds
  }
);
~~~

#### Kết quả mong đợi

* Tài khoản A xem phim thì lịch sử chỉ hiện ở tài khoản A.
* Tài khoản B không thấy tiếp tục xem của tài khoản A.
* Khi bấm tiếp tục xem, player seek về thời điểm đã lưu.


<!-- NETFLOP_IMPLEMENTATION_END -->
