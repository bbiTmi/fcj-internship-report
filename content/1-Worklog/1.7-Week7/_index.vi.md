---
title: "Worklog Tuần 7"
date: ""
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---
### Mục tiêu tuần 7:

* Hiểu và thực hành với DynamoDB  
* Học cách sử dụng Route 53 để host website và quản lý DNS.  
* Nắm sâu kiến thức về kiến trúc chịu lỗi (Resilient Architecture) và các chiến lược phục hồi sau thảm họa.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Tìm hiểu về DynamoDB và các thành phần | 20/10/2025 | 20/10/2025 | [Amazon DynamoDB](https://000060.awsstudygroup.com/)   |
| 3 | Thực hành tạo DynamoDB trên AWS Tìm hiểu cách host một website với Route53 | 21/10/2025 | 21/10/2025 |  |
| 4 | Ôn tập: Resilient Architecture \- Multi AZ, Multi Region \- Disaster Recovery Strategies (Backup and Restore, Pilot Light, Warm Standby, Multi Site active active) \- Health Check  \- Load Balancing \- Các dịch vụ giúp khôi phục hệ thống nhanh chóng | 22/10/2025 | 25/10/2025 |   |
| 7 | Vẽ kiến trúc dự án và ước tính lưu lượng sẽ dùng cho dự án | 25/10/2025 | 26/10/2025 |   |

###  Kết quả đạt được tuần 7:

#### DynamoDB 

* DynamoDB là cơ sở dữ liệu NoSQL key-value với khả năng scale gần như vô hạn.  
* Thành phần chính:  
  * Partition Key / Sort Key  
  * GSI (Global Secondary Index)  
  * LSI (Local Secondary Index)  
  * Provisioned / On-Demand Capacity

#### Hosting website với Route 53

* Cách mua/gắn domain vào Route 53  
* Tạo Hosted Zone và Record: A, CNAME  
* Trỏ domain về:  
  * S3 Static Website  
  * CloudFront Distribution  
  * EC2 Elastic IP  
* Hiểu cơ chế DNS propagation và TTL

#### Resilient Architecture Concepts

* Multi-AZ: Tự động chuyển đổi khi có sự cố trong một Availability Zone.  
* Multi-Region: Giải pháp dự phòng cho các hệ thống yêu cầu uptime cực cao hoặc phân phối toàn cầu.  
* Disaster Recovery Strategies

| Chiến lược | Độ phức tạp | Chi phí | Thời gian phục hồi |
| :---- | :---- | :---- | :---- |
| Backup & Restore | Thấp | Thấp | Lâu nhất |
| Pilot Light | TB | TB | Vừa |
| Warm Standby | Cao | Cao | Nhanh |
| Multi-Site Active/Active | Rất cao | Rất cao | Gần như không downtime |

RTO & RPO:

* RTO: Thời gian tối đa hệ thống có thể downtime.  
* RPO: Lượng dữ liệu tối đa có thể mất.

Health Check: Route 53 theo dõi tình trạng endpoint \-\> failover sang site dự phòng nếu có lỗi.  
Load Balancing: ALB/NLB giúp phân phối lưu lượng và tăng khả năng chịu lỗi.