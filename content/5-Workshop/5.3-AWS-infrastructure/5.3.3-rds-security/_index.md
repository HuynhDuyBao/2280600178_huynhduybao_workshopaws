---
title: "Configure RDS and Security Group"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

#### Goal

Amazon RDS MySQL is used as the main database for Netflop. The production database is named:

```text
web_xem_phim_final
```

#### RDS information

| Attribute | Value |
| --- | --- |
| DB identifier | `netflop-db` |
| Engine | MySQL |
| Instance class | `db.t4g.micro` |
| Endpoint | `netflop-db.c76we6m8scy0.ap-southeast-1.rds.amazonaws.com` |
| Port | `3306` |
| Database | `web_xem_phim_final` |

#### Importing the local database to RDS

The local source dump is:

```text
database/web_xem_phim_final_dump.sql
```

Import procedure:

1. Export the local database to an SQL file.
2. Clean any statements incompatible with RDS, such as `DEFINER` or triggers requiring `SUPER`.
3. Create the `web_xem_phim_final` database on RDS.
4. Import the dump into RDS.
5. Verify the number of tables and movie/episode/user data.

#### RDS Security Group

In production, use:

| Type | Port | Source |
| --- | --- | --- |
| MySQL/Aurora | 3306 | EC2 security group `netflop-ec2-sg` |

Do not leave RDS open to `0.0.0.0/0` long term. If importing from a local machine, allow the personal IP temporarily and close it after import.

#### Backend connection checks

```bash
pm2 logs netflop-api
curl https://netflop.win/api/catalog/genres
curl https://netflop.win/api/movies?limit=12
```

If the API returns movie/genre data correctly, the backend has successfully connected to RDS.

{{% notice info %}}
Add images: RDS `netflop-db` status Available, RDS endpoint, Security Group inbound rules restricted to EC2, database import results, and successful API data responses.
{{% /notice %}}

<!-- NETFLOP_DETAIL_START -->
#### How to create and import RDS

1. Go to RDS -> Create database.
2. Choose MySQL.
3. Set DB identifier to <code>netflop-db</code>.
4. Create the initial database as <code>web_xem_phim_final</code>.
5. Restrict the RDS Security Group to port 3306 from the EC2 security group.
6. Import the dump from local or from EC2.

#### Sample import command

~~~bash
mysql -h netflop-db.c76we6m8scy0.ap-southeast-1.rds.amazonaws.com \
  -P 3306 \
  -u <db-user> \
  -p web_xem_phim_final < database/web_xem_phim_final_dump_rds.sql
~~~

#### Sample backend RDS connection code

~~~js
const mysql = require('mysql2/promise');
const env = require('./env');

const pool = mysql.createPool({
  host: env.db.host,
  port: env.db.port,
  database: env.db.database,
  user: env.db.user,
  password: env.db.password,
  waitForConnections: true,
  connectionLimit: env.db.connectionLimit,
  charset: 'utf8mb4',
  namedPlaceholders: true
});
~~~

#### Connection verification

~~~bash
curl https://netflop.win/api/movies?limit=12
curl https://netflop.win/api/catalog/genres
~~~

If the API returns lists of movies and genres, the backend is reading data from RDS successfully.
<!-- NETFLOP_DETAIL_END -->
