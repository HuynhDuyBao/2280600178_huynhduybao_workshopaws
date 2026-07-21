---
title: "Configure Cloudflare domain"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

Cloudflare manages `netflop.win`, DNS, and user-facing HTTPS.

```text
User -> Cloudflare DNS + HTTPS -> netflop.win -> EC2 public IP -> Nginx
```

#### Checking steps

1. Open Cloudflare Dashboard.
2. Select `netflop.win`.
3. Check the DNS record pointing to EC2 public IP `18.143.150.109`.
4. Check proxy/HTTPS status.
5. Open `https://netflop.win`.



![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.3-AWS-infrastructure/5.3.1-cloudflare-domain/DNSrecord.png)

<!-- NETFLOP_DETAIL_START -->
#### How to configure DNS in Cloudflare

In Cloudflare, create DNS records for the main domain and `www` subdomain:

| Type | Name | Value |
| --- | --- | --- |
| A | `@` | `18.143.150.109` |
| CNAME | `www` | `netflop.win` |

After DNS is configured, open:

```text
https://netflop.win
```

#### Verification commands

~~~bash
nslookup netflop.win
curl -I https://netflop.win
~~~

Expected result:

* DNS resolves correctly.
* HTTPS is active through Cloudflare.
* Browser can open the Netflop website.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Configure netflop.win

The website uses Cloudflare to manage DNS and HTTPS for <code>netflop.win</code>. Without an Application Load Balancer, DNS records point directly to the EC2 public IP or Elastic IP.

#### Implementation steps

1. Open Cloudflare -> DNS.
2. Create an <code>A</code> record for <code>@</code> pointing to the EC2 IP.
3. Create a <code>CNAME</code> record for <code>www</code> pointing to <code>netflop.win</code>.
4. Enable proxy if Cloudflare SSL/CDN is used.
5. In SSL/TLS, choose the correct mode. If EC2 does not have an origin certificate, Flexible can work temporarily. With a valid origin certificate, use Full or Full strict.
6. Update OAuth/Cognito callback URLs to <code>https://netflop.win/auth/callback</code>.

#### DNS and HTTPS checks

~~~bash
nslookup netflop.win
curl -I https://netflop.win
curl -I https://netflop.win/api/health
~~~

#### Common issues

| Issue | Cause | Fix |
| --- | --- | --- |
| Mixed content | HTTPS page loads HTTP or localhost media/API | Use HTTPS production domain for API and media URLs |
| OAuth redirect mismatch | Google/Cognito callback still points to localhost or EC2 DNS | Add <code>https://netflop.win/auth/callback</code> |
| API timeout | Nginx/PM2/backend or security group problem | Check PM2, Nginx, and inbound ports 80/443 |

{{% notice info %}}
Screenshots needed: Cloudflare DNS records, SSL/TLS mode, website through https://netflop.win, and OAuth callback using the domain.
{{% /notice %}}
<!-- NETFLOP_IMPLEMENTATION_END -->
