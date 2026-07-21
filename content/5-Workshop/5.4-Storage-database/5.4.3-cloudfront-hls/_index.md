---
title: "Configure CloudFront HLS streaming"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

#### Goal

CloudFront is used to distribute HLS streams from the S3 output bucket to viewers. The stream distribution is configured as protected HLS and uses signed cookies to restrict direct video access.

#### Stream distribution

```text
d11jdx7259stge.cloudfront.net
```

The origin points to:

```text
netflop-output-source.s3.ap-southeast-1.amazonaws.com
```

#### Playback flow

```text
Frontend -> Backend retrieves episode info
Backend -> issues stream session / signed cookies
Video Player -> CloudFront -> S3 output HLS
```

#### Related API

The backend issues a stream session through:

```text
/api/stream/session
```

This API creates CloudFront signed cookies so the browser can download the HLS manifest and segments for a limited time. As a result, the frontend does not need to expose the raw HLS object URLs publicly.

#### Media proxy fallback

The backend also provides a proxy API:

```text
/api/media-proxy
```

This endpoint handles cases where media or subtitle requests need proxying or when CORS/mixed content issues occur. In production, avoid exposing raw stream URLs in player error messages.

#### Verification

1. Open the CloudFront console.
2. Verify the distribution is Enabled.
3. Verify the origin points to the S3 output bucket.
4. Open an episode in Netflop.
5. Confirm the player loads HLS segments through CloudFront.

![distribution](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.3-cloudfront-hls/cloudfront1.png)
![distribution](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.3-cloudfront-hls/cf2.png)
![phim](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.4-Storage-database/5.4.3-cloudfront-hls/player1.png)

<!-- NETFLOP_DETAIL_START -->
#### How signed cookies protect streams

When a user opens an episode, the frontend requests a stream session from the backend. The backend creates CloudFront signed cookies and sets them in the response. The player then loads HLS through CloudFront.

#### Sample backend stream session

~~~js
async function createStreamSession(req, res, next) {
  const sourceUrl = req.body?.sourceUrl || req.query?.sourceUrl;
  const signedSession = cloudFrontSignedCookieService.buildSignedCookies(sourceUrl);

  cloudFrontSignedCookieService.setSignedCookies(res, signedSession);

  res.json({
    success: true,
    data: {
      enabled: signedSession.enabled,
      playbackUrl: signedSession.playbackUrl,
      expiresAt: signedSession.expiresAt || null
    }
  });
}
~~~

#### Sample cookie object

~~~js
return {
  enabled: true,
  playbackUrl,
  expiresAt,
  cookies: {
    'CloudFront-Policy': toCloudFrontBase64(policy),
    'CloudFront-Signature': signPolicy(policy),
    'CloudFront-Key-Pair-Id': awsConfig.cloudFrontKeyPairId
  }
};
~~~

#### Sample frontend player logic

~~~js
const useCredentialedPlayback = useMemo(
  () => shouldUseCredentialedPlayback(mediaUrl, signedPlaybackUrl),
  [mediaUrl, signedPlaybackUrl]
);

const hls = new Hls({
  xhrSetup: (xhr) => {
    xhr.withCredentials = useCredentialedPlayback;
  }
});
~~~
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### CloudFront purpose

CloudFront is placed in front of the S3 output bucket so users can stream HLS faster and more reliably. When stream protection is required, the backend issues signed cookies with a short TTL so the player can request the HLS manifest and segments.

#### Signed cookie flow

1. The player prepares to play an HLS source.
2. Frontend calls <code>/api/stream/session</code> with the source URL.
3. Backend builds a policy that limits the HLS resource and expiration time.
4. Backend signs the policy with the CloudFront private key.
5. Browser receives <code>CloudFront-Policy</code>, <code>CloudFront-Signature</code>, and <code>CloudFront-Key-Pair-Id</code>.
6. Player plays the CloudFront URL and sends the cookies with requests.

#### Backend stream session code

~~~js
async function createStreamSession(req, res, next) {
  const sourceUrl = req.body?.sourceUrl || req.query?.sourceUrl;
  const signedSession = cloudFrontSignedCookieService.buildSignedCookies(sourceUrl);

  cloudFrontSignedCookieService.setSignedCookies(res, signedSession);

  res.json({
    success: true,
    data: {
      enabled: signedSession.enabled,
      playbackUrl: signedSession.playbackUrl,
      expiresAt: signedSession.expiresAt || null
    }
  });
}
~~~

#### Cookie creation sample

~~~js
return {
  enabled: true,
  playbackUrl,
  expiresAt,
  cookies: {
    'CloudFront-Policy': toCloudFrontBase64(policy),
    'CloudFront-Signature': signPolicy(policy),
    'CloudFront-Key-Pair-Id': awsConfig.cloudFrontKeyPairId
  }
};
~~~

#### Player credentials sample

~~~js
const useCredentialedPlayback = shouldUseCredentialedPlayback(mediaUrl, signedPlaybackUrl);

const hls = new Hls({
  xhrSetup: (xhr) => {
    xhr.withCredentials = useCredentialedPlayback;
  }
});
~~~

{{% notice warning %}}
The player should not send credentials for non-signed public sources such as Cloudflare Stream, otherwise CORS can fail because wildcard origin is not valid with credentials.
{{% /notice %}}

{{% notice info %}}
Screenshots needed: CloudFront distribution, behavior/cache policy, signed cookies in DevTools Application tab, and HLS requests returning 200.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
