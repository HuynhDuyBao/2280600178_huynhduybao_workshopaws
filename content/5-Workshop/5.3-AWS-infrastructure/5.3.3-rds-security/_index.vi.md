---
title: "Cấu hình RDS và Security Group"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

#### Mục tiêu

Amazon RDS MySQL được dùng làm database chính cho Netflop. Database production sử dụng tên:

```text
web_xem_phim_final
```

#### Thông tin RDS

| Thuộc tính | Giá trị |
| --- | --- |
| DB identifier | `netflop-db` |
| Engine | MySQL |
| Instance class | `db.t4g.micro` |
| Endpoint | `netflop-db.c76we6m8scy0.ap-southeast-1.rds.amazonaws.com` |
| Port | `3306` |
| Database | `web_xem_phim_final` |

#### Import database local lên RDS

Source local có file dump:

```text
database/web_xem_phim_final_dump.sql
```

Quy trình import:

1. Export database local thành file SQL.
2. Làm sạch các câu lệnh không phù hợp với RDS nếu có, ví dụ `DEFINER` hoặc trigger yêu cầu `SUPER`.
3. Tạo database `web_xem_phim_final` trên RDS.
4. Import dump lên RDS.
5. Kiểm tra số bảng và dữ liệu phim/tập phim/người dùng.

#### Security Group RDS

Hướng production nên dùng:

| Type | Port | Source |
| --- | --- | --- |
| MySQL/Aurora | 3306 | Security Group của EC2 `netflop-ec2-sg` |

Không nên mở RDS cho `0.0.0.0/0` lâu dài. Nếu cần import từ máy local, chỉ mở tạm IP cá nhân rồi đóng lại sau khi import xong.

#### Kiểm tra backend kết nối RDS

```bash
pm2 logs netflop-api
curl https://netflop.win/api/catalog/genres
curl https://netflop.win/api/movies?limit=12
```

Nếu API trả dữ liệu phim/thể loại bình thường, backend đã kết nối RDS thành công.

<!-- NETFLOP_DETAIL_START -->
#### Cách tạo và import RDS

1. Vào RDS -> Create database.
2. Chọn MySQL.
3. Đặt DB identifier là <code>netflop-db</code>.
4. Tạo database ban đầu là <code>web_xem_phim_final</code>.
5. Security Group của RDS chỉ nên mở port 3306 cho Security Group EC2.
6. Import file dump từ local hoặc từ EC2.

#### Lệnh import mẫu

~~~bash
mysql -h netflop-db.c76we6m8scy0.ap-southeast-1.rds.amazonaws.com \
  -P 3306 \
  -u <db-user> \
  -p web_xem_phim_final < database/web_xem_phim_final_dump_rds.sql
~~~

#### Code mẫu kết nối RDS trong backend

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

#### Kiểm tra kết nối

~~~bash
curl https://netflop.win/api/movies?limit=12
curl https://netflop.win/api/catalog/genres
~~~

Nếu API trả danh sách phim/thể loại, backend đã đọc dữ liệu từ RDS thành công.
<!-- NETFLOP_DETAIL_END -->

<!-- NETFLOP_IMPLEMENTATION_START -->
#### Tạo RDS MySQL cho Netflop

Database production dùng RDS MySQL, database name là <code>web_xem_phim_final</code>. RDS lưu dữ liệu người dùng, phim, tập phim, phụ đề, lịch sử xem, bình luận và đánh giá.

#### Các bước tạo và bảo vệ RDS

1. Chọn engine MySQL.
2. Chọn instance nhỏ phù hợp giai đoạn demo, ví dụ <code>db.t4g.micro</code>.
3. Đặt DB identifier <code>netflop-db</code>.
4. Tạo database <code>web_xem_phim_final</code>.
5. Chọn VPC cùng EC2.
6. Security Group của RDS chỉ mở port 3306 cho Security Group của EC2.
7. Không mở 3306 public cho mọi IP khi chạy production.

#### Import database local lên RDS

~~~bash
mysql -h netflop-db.xxxxxx.ap-southeast-1.rds.amazonaws.com \
  -u admin -p web_xem_phim_final < database/web_xem_phim_final_dump.sql
~~~

#### Code backend kết nối RDS

~~~js
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

#### Kiểm tra sau import

1. Backend khởi động không báo lỗi database.
2. Trang chủ trả danh sách phim.
3. Đăng nhập/đăng ký hoạt động.
4. Admin thấy danh sách phim/tập phim.

<!-- NETFLOP_IMPLEMENTATION_END -->
