---
title: "Test user features"
date: 2026-07-10
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

#### Goal

Verify Netflop user-facing functionality after deployment to the production domain `netflop.win`.

#### Test checklist

| Feature group | Expected result |
| --- | --- |
| Home page | Banner, new movies, popular movies, and series/movie categories display correctly |
| Search | Search finds movies by name or actor |
| Movie detail | Poster, banner, metadata, rating, and episode list display correctly |
| Watch movie | Player streams HLS, supports seek, fullscreen, and subtitle selection |
| Mobile responsive | Movie cards scroll horizontally, player controls remain visible, subtitle selector is touch-friendly |
| Sign in/register | Local account registration and login succeed |
| Continue watching | Progress is saved per account and isolated between accounts |
| Watch history | Previously watched movies display correctly for each user |
| Favorites/reviews/comments | Data is stored in RDS correctly |
| Avatar | User can change avatar and the image uploads to S3 successfully |

#### Mobile player checks

* Play/pause controls work with touch input.
* Seek bar is not blocked by any overlay or notification.
* Subtitle, settings, and fullscreen buttons do not overflow the player UI.
* Subtitles do not appear duplicated.
* Resume playback continues from saved progress.

#### Expected results

The user can watch movies smoothly on desktop and mobile without CORS or mixed-content errors. Raw stream URLs should not leak in error messages. User data must remain separated by account.

![home](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/home.png)
![watch](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/tieptucxem.png)
![watch](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/tieptucxem2.png)
![play](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/moviedetail.png)
![play](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/movie1.png)
![play](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.2-user-test/movierun.png)

<!-- NETFLOP_DETAIL_START -->
#### How to implement continue watching

The player periodically sends the current watch position to the backend. The backend stores it in RDS by user and episode. When the user opens the same episode again, the frontend resumes playback from the saved position.

#### Sample frontend progress tracking

~~~js
videoRef.current.addEventListener('timeupdate', () => {
  const currentTime = Math.floor(videoRef.current.currentTime);
  if (currentTime % 15 === 0) {
    watchHistoryApi.saveProgress({
      episodeId,
      currentTime,
      duration: Math.floor(videoRef.current.duration || 0)
    });
  }
});
~~~

#### Sample backend upsert

~~~js
async function saveProgress({ userId, episodeId, currentTime, duration }) {
  await db.query(
    `INSERT INTO watch_history (user_id, episode_id, current_time, duration)
     VALUES (?, ?, ?, ?)
     ON DUPLICATE KEY UPDATE current_time = VALUES(current_time), duration = VALUES(duration)`,
    [userId, episodeId, currentTime, duration]
  );
}
~~~

#### How to test

1. Sign in as user A.
2. Watch one episode for around one minute.
3. Leave the player and open it again.
4. Confirm playback resumes near the previous position.
5. Sign in as user B and confirm user A's progress does not appear.
<!-- NETFLOP_DETAIL_END -->
