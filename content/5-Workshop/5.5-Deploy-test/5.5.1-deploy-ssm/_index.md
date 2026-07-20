---
title: "Deploy/reload with EC2 and Systems Manager"
date: 2026-07-10
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

#### Goal

Deploy or reload Netflop on EC2 `netflop-web` without requiring manual server intervention. Use AWS Systems Manager to execute deployment commands from your local AWS CLI.

#### Deployment process

1. Build the React frontend.
2. Copy built frontend files to the Nginx served directory.
3. Update backend source on EC2.
4. Install backend packages if dependencies changed.
5. Reload the backend using PM2.
6. Reload Nginx if reverse proxy or static asset configuration changed.
7. Verify website and API responses.

#### Systems Manager role

AWS Systems Manager enables command execution on EC2 without broad SSH access or a local key pair. This is preferable for production because it reduces attack surface and centralizes operational control.

#### Expected results

* Frontend loads successfully at `https://netflop.win`.
* Backend API responds correctly under `/api`.
* PM2 shows the backend process online.
* Nginx passes configuration tests.

#### Commands to verify

```bash
pm2 status
pm2 logs netflop-api
sudo nginx -t
sudo systemctl reload nginx
curl -I https://netflop.win
curl https://netflop.win/api/health
```

![nginx](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.1-deploy-ssm/ngix%20status.png)
![pm2](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.1-deploy-ssm/pm2%20status.png)

<!-- NETFLOP_DETAIL_START -->
#### How to deploy the frontend

1. Build the frontend locally.
2. Package the `frontend/dist` folder.
3. Upload the artifact to a staging S3 path.
4. Use Systems Manager to tell EC2 to download the artifact.
5. Extract it to `/var/www/netflop`.
6. Reload Nginx.

#### Sample commands

~~~bash
npm --prefix frontend run build
tar -czf netflop-frontend.tgz -C frontend/dist .
aws s3 cp netflop-frontend.tgz s3://netflop-input-source/deploy/frontend/netflop-frontend.tgz
~~~

On EC2, the SSM command runs a flow like:

~~~bash
curl -fL '<presigned-url>' -o /tmp/netflop-deploy/frontend.tgz
mkdir -p /tmp/netflop-frontend-build
tar -xzf /tmp/netflop-deploy/frontend.tgz -C /tmp/netflop-frontend-build
sudo cp -a /tmp/netflop-frontend-build/. /var/www/netflop/
sudo nginx -t
sudo systemctl reload nginx
~~~

#### How to deploy the backend

~~~bash
cd /home/ubuntu/netflop/backend
npm install --omit=dev
pm2 restart netflop-api --update-env
pm2 logs netflop-api
~~~
<!-- NETFLOP_DETAIL_END -->
