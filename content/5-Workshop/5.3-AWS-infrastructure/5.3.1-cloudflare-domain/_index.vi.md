---
title: "Cấu hình Cloudflare domain"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

#### Mục tiêu

Domain `netflop.win` được quản lý bằng Cloudflare để người dùng truy cập website bằng tên miền thay vì IP EC2. Cloudflare cũng hỗ trợ HTTPS phía người dùng, giúp tránh lỗi mixed content khi frontend gọi API và tải media.

#### Cấu hình DNS

Bản ghi chính:

| Type | Name | Value |
| --- | --- | --- |
| A | `@` | Public IP hoặc Elastic IP của EC2 |
| CNAME | `www` | `netflop.win` |

Trong môi trường production nên gắn **Elastic IP** cho EC2 để IP không thay đổi sau khi stop/start instance.

#### Cấu hình SSL/TLS

Trong Cloudflare:

* Bật SSL/TLS cho domain.
* Với triển khai đơn giản có thể dùng chế độ Flexible hoặc Full tùy cấu hình Nginx.
* Khi dùng HTTPS public, tất cả API và media URL trên frontend phải dùng `https://netflop.win` hoặc CloudFront HTTPS.

#### Đồng bộ OAuth callback

Các nhà cung cấp đăng nhập như Google/Cognito phải dùng domain production:

```text
https://netflop.win/auth/callback
```

Không nên để production redirect sang `localhost` hoặc URL EC2 dạng IP vì trình duyệt và OAuth provider có thể chặn.

#### Kiểm tra

1. Truy cập `https://netflop.win`.
2. Kiểm tra frontend load được.
3. Kiểm tra API `/api/...` không bị lỗi CORS.
4. Kiểm tra console browser không còn lỗi mixed content do gọi `http://`.

{{% notice info %}}
Cần thêm ảnh: Cloudflare DNS record, SSL/TLS mode, website `https://netflop.win` truy cập thành công và console không còn lỗi CORS/mixed content.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.1-cloudflare-domain/DNSrecord.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách thực hiện DNS trên Cloudflare

1. Vào Cloudflare -> chọn domain <code>netflop.win</code>.
2. Vào DNS -> Records.
3. Tạo record A cho root domain:

| Type | Name | Content |
| --- | --- | --- |
| A | @ | Public IP hoặc Elastic IP của EC2 |

4. Nếu cần dùng www, tạo thêm:

| Type | Name | Content |
| --- | --- | --- |
| CNAME | www | netflop.win |

5. Vào SSL/TLS và bật chế độ SSL phù hợp.

#### Kiểm tra DNS và HTTPS

~~~bash
nslookup netflop.win
curl -I https://netflop.win
~~~

Trong trình duyệt, mở DevTools -> Console. Nếu không còn lỗi CORS hoặc mixed content, domain production đã được cấu hình đúng.

#### Lưu ý cho OAuth

Nếu dùng Google/Cognito, callback URL phải đổi sang domain thật:

~~~text
https://netflop.win/auth/callback
~~~

Không dùng IP EC2 trong OAuth production vì nhiều provider không chấp nhận IP public làm origin/redirect ổn định.
<!-- NETFLOP_DETAIL_END -->
