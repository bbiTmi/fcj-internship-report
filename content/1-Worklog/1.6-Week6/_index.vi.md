---
title: "Worklog Tuần 6"
date: ""
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu tuần 6:

* Củng cố kiến thức về Secure Architecture và các cơ chế bảo mật cốt lõi của AWS.  
* Hiểu cơ chế hoạt động của Elastic Load Balancing và các phương pháp routing.  
* Nắm vững các chiến lược Disaster Recovery trong AWS và phân biệt RTO/RPO.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Ôn tập: Secure Architecture \- Nắm chắc kiến thức về IAM (User, Group, Policy, Role, Quyền truy cập tối thiểu) \- Các loại MFA để bảo mật tài khoản \- So sánh Security Group vs NACL | 13/10/2025 | 13/10/2025 |   |
| 3 | Học khái niệm về Elastic Load Balancing: \- Application Load Balancer (ALB) \- Network Load Balancer (NLB) \- Open Systems Interconnections \- Các loại routing của NLB và ALB \- Sticky Session/Session Affinity | 14/10/2025 | 14/10/2025 |   |
| 4 | Tìm hiểu về Key Management Service, AWS Certificate Management và cách mã hóa, lưu key | 15/10/2025 | 15/10/2025 |   |
| 5 | Nắm kiến thức các loại Disaster Recovery Strategies: \- Backup & Restore \- Pilot Light \- Warm Standby \- Multi-Site Active-Active Phân biệt RTO và PTO   | 16/10/2025 | 16/08/2025 |   |

###  Kết quả đạt được tuần 6:

#### Secure Architecture 

Nắm vững những điểm quan trọng:

* IAM gồm User, Group, Policy, Role; tuân thủ nguyên tắc quyền tối thiểu (Least Privilege).  
* MFA giúp bảo vệ tài khoản với các loại như: Virtual MFA, Hardware MFA, SMS MFA.  
* Security Group: cấp phép theo stateful firewall (tự động cho phép traffic phản hồi).  
* NACL: stateless, áp dụng ở cấp subnet, thích hợp cho các rule deny.  
* SG kiểm soát từng instance \-\> chi tiết hơn; NACL bảo vệ lớp subnet \-\> tổng quát hơn.

#### Elastic Load Balancing \- ALB, NLB và routing

* ALB hoạt động ở Layer 7, hỗ trợ HTTP/HTTPS, routing theo path, host, header, query string.  
* NLB hoạt động ở Layer 4, tối ưu tốc độ và xử lý hàng triệu request/giây, phù hợp TCP/UDP.  
* Model OSI giúp phân biệt vai trò của ALB (L7) và NLB (L4).  
* Hiểu các cơ chế routing: weighted, host-based, path-based (ALB) và TCP/UDP routing (NLB).  
* Sticky Session / Session Affinity hoạt động bằng cookies để giữ kết nối người dùng về cùng target.

#### Tìm hiểu về KMS & ACM \- mã hóa và quản lý khóa

* AWS KMS giúp tạo, lưu, và quản lý khóa mã hóa (CMK)  
* Hỗ trợ mã hóa S3, EBS, RDS và nhiều dịch vụ khác bằng envelope encryption.  
* Phân biệt AWS-managed keys và Customer-managed keys (CMK).  
* AWS Certificate Manager (ACM) cung cấp chứng chỉ SSL/TLS miễn phí và tự động gia hạn.  
* Kết hợp ACM \+ CloudFront/ALB để đảm bảo HTTPS end-to-end.

#### Disaster Recovery Strategies 

| Chiến lược | Mô tả | Mức độ phục hồi |
| :---- | :---- | :---- |
| Backup & Restore | Sao lưu dữ liệu và khôi phục khi sự cố | Thấp |
| Pilot Light | Giữ phần lõi hệ thống hoạt động ở mức tối thiểu | Trung bình |
| Warm Standby | Một phiên bản thu nhỏ chạy song song | Cao |
| Multi-Site Active-Active | Nhiều site hoạt động đồng thời | Rất cao |

* RTO (Recovery Time Objective): thời gian hệ thống có thể chấp nhận downtime.  
* RPO (Recovery Point Objective): lượng dữ liệu tối đa có thể mất trong sự cố.

   