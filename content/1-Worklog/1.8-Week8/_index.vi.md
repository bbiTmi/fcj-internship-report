---
title: "Worklog Tuần 8"
date: ""
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---
### Mục tiêu tuần 8:

* Củng cố kiến thức về High-Performing Architectures để tối ưu hiệu năng hệ thống.  
* Nắm chắc các kỹ thuật tối ưu chi phí Cost Optimization.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Ôn tập: High Performing Architectures \- Compute Scaling \- Storage Optimization \- Caching \- Network Optimization | 27/10/2025 | 27/10/2025 |   |
| 3 | Ôn tập: Cost Optimized Architecture \- Budget \- Saving plan \- Reserved instance | 28/10/2025 | 28/10/2025 |   |
| 4 | Tổng hợp kiến thức, giải đề do AI tạo  Giải thử đề SSA | 29/10/2025 | 30/10/2025 |   |
| 6 | Ôn tập và thi giữa kỳ | 31/10/2025 | 31/10/2025 |   |

###  Kết quả đạt được tuần 8:

#### High-Performing Architectures

**Compute Scaling:**

* Phân biệt vertical scaling và horizontal scaling.  
* Hiểu Auto Scaling Group, scaling policies và cơ chế tăng/giảm EC2 theo tải hệ thống.

**Storage Optimization**

* Cơ chế S3 tiering (S3 Standard \-\> IA \-\> Glacier) để tối ưu tốc độ & chi phí.

**Caching**

* Caching giúp giảm tải backend và tăng tốc ứng dụng.  
* Sử dụng Amazon CloudFront, ElastiCache (Redis/Memcached) để cải thiện độ trễ.

**Network Optimization**

* Hiểu tầm quan trọng của VPC endpoints, enhanced networking và tối ưu đường đi dữ liệu.  
* Sử dụng CloudFront để giảm độ trễ toàn cầu.

#### Cost-Optimized Architectures

**AWS Budget:** Thiết lập cảnh báo dựa trên mức chi tiêu hoặc dự đoán chi tiêu.  
**Saving Plans**

* Compute Saving Plans linh hoạt (hoạt động cho EC2, Fargate, Lambda).  
* EC2 Instance Saving Plans chi phí thấp hơn nhưng ít linh hoạt hơn.

**Reserved Instance (RI)**

* Cam kết 1-3 năm để tiết kiệm chi phí EC2/RDS.  
* Phù hợp workload ổn định, predictable.