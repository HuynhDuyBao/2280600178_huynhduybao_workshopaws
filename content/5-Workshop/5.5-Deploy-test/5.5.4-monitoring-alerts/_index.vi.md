---
title: "Kiểm tra monitoring và cảnh báo"
date: 2026-07-10
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

#### Mục tiêu

Kiểm tra CloudWatch và SNS để theo dõi trạng thái hệ thống Netflop sau khi deploy. Monitoring giúp phát hiện sớm lỗi backend, lỗi Lambda, thiếu tài nguyên EC2/RDS hoặc chi phí tăng bất thường.

#### CloudWatch alarms đang dùng

| Alarm | Mục đích |
| --- | --- |
| `Netflop-EC2-CPU-High` | Cảnh báo CPU EC2 cao |
| `Netflop-EC2-Memory-High` | Cảnh báo RAM EC2 cao |
| `Netflop-EC2-Disk-High` | Cảnh báo ổ đĩa EC2 gần đầy |
| `Netflop-EC2-StatusCheckFailed` | Cảnh báo instance lỗi status check |
| `Netflop-RDS-CPU-High` | Cảnh báo CPU RDS cao |
| `Netflop-RDS-Connections-High` | Cảnh báo số kết nối RDS cao |
| `Netflop-RDS-FreeStorage-Low` | Cảnh báo dung lượng RDS còn thấp |
| `Netflop-RDS-FreeableMemory-Low` | Cảnh báo bộ nhớ RDS còn thấp |
| `Netflop-Lambda-netflop-mediaconvert-notifier-Errors` | Cảnh báo Lambda notifier lỗi |
| `Netflop-Lambda-netflop-subtitle-converter-Errors` | Cảnh báo Lambda subtitle converter lỗi |

#### SNS topic

```text
netflop-alerts
```

Topic này dùng để gửi cảnh báo khi CloudWatch alarm chuyển trạng thái. Email nhận cảnh báo cần xác nhận subscription trước khi SNS gửi được thông báo.

#### Log cần kiểm tra

* PM2 logs của backend trên EC2.
* Nginx access/error logs.
* CloudWatch Logs của Lambda `netflop-mediaconvert-notifier`.
* CloudWatch Logs của Lambda `netflop-subtitle-converter`.
* CloudWatch metrics của RDS.

#### Các bước kiểm tra

1. Vào CloudWatch console.
2. Kiểm tra trạng thái các alarm.
3. Vào Lambda logs để kiểm tra lỗi MediaConvert notifier/subtitle converter.
4. Vào SNS console kiểm tra topic `netflop-alerts`.
5. Kiểm tra Billing Dashboard/AWS Budgets để theo dõi chi phí.

{{% notice info %}}
Cần thêm ảnh: CloudWatch alarms trạng thái OK, Lambda logs, SNS topic `netflop-alerts`, email subscription đã Confirmed và Billing Dashboard/Cost Explorer.
{{% /notice %}}

![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw1.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/cw2.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/sns.png)
![](/2280600178_huynhduybao_workshopaws/images/5-Workshop/5.5-Deploy-test/5.5.4-monitoring-alerts/console.png)

<!-- NETFLOP_DETAIL_START -->
#### Cách tạo và kiểm tra alarm

CloudWatch alarm được dùng để theo dõi các ngưỡng quan trọng. Ví dụ:

* CPU EC2 cao.
* Disk EC2 gần đầy.
* RDS CPU hoặc connection cao.
* Lambda lỗi.
* RDS còn ít dung lượng.

#### Lệnh kiểm tra alarm

~~~bash
aws cloudwatch describe-alarms --region ap-southeast-1
aws logs describe-log-groups --region ap-southeast-1
aws sns list-topics --region ap-southeast-1
~~~

#### Cách kiểm tra trong Console

1. Vào CloudWatch -> Alarms.
2. Lọc các alarm bắt đầu bằng <code>Netflop-</code>.
3. Kiểm tra trạng thái OK/ALARM/INSUFFICIENT_DATA.
4. Vào CloudWatch Logs để xem log Lambda và backend nếu đã cấu hình agent.
5. Vào SNS -> topic <code>netflop-alerts</code>, xác nhận email subscription đã Confirmed.

#### Kết quả cần ghi vào báo cáo

Nếu tất cả alarm ở trạng thái OK, hệ thống đang hoạt động bình thường tại thời điểm kiểm tra. Nếu có alarm ALARM, cần ghi rõ nguyên nhân và hướng xử lý.
<!-- NETFLOP_DETAIL_END -->
