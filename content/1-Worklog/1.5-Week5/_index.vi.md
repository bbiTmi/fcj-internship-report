---
title: "Worklog Tuần 5"
date: ""
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---
### Mục tiêu tuần 5:

* Hiểu sâu hơn về các dịch vụ AWS thông qua việc dịch blog  
* Xác định đề tài dự án nhóm và phân chia nhiệm vụ.  
* Học cách sử dụng AWS Pricing Calculator để ước tính chi phí dự án.

### Các công việc cần triển khai trong tuần này:

 

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Dịch Blog 1: **Building geolocation verification for iGaming and sports betting on AWS** Dịch Blog 2: **How to run AI model inference with GPUs on Amazon EKS Auto Mode** Dịch Blog 3: **Enforcing organization-wide Amazon S3 bucket-tagging policies**   | 06/10/2025 | 08/10/2025 | [Xây dựng hệ thống xác minh vị trí địa lý cho iGaming và cá cược thể thao trên AWS](https://docs.google.com/document/d/16V56RoubCuY4XGh_B0TClIuMR0ldb7YG8EpPRcY2VaA/edit?tab=t.0#heading=h.8lvjg3ekz0as)[Cách thực thi AI model inference với GPUs trên Amazon EKS Auto Mode](https://docs.google.com/document/d/1Fhfr6sTk9h5dAqC5BY_d9d4Tn4bmWUqNyMwKjPz4fuo/edit?tab=t.0#heading=h.bvk366iqrbm5) [Thực thi chính sách gắn thẻ (tagging) trên toàn tổ chức cho Amazon S3 Buckets](https://docs.google.com/document/d/1AaMq6i-3POxvIc3Z0xdkTH4uvw53R7zWy2ElzqbiCAY/edit?tab=t.0#heading=h.rmy7wqry9du5)   |
| 3 | Chốt đề tài dự án nhóm, phân chia công việc cho các thành viên | 07/10/2025 | 07/10/2025 |   |
| 5 | Sử dụng Pricing Calculator để tính toán chi phí (ước tính) cho dự án | 09/10/2025 | 09/10/2025 |   |
| 7 | Viết Proposal cho dự án | 11/10/2025 | 13/10/2025 |   |

###  Kết quả đạt được tuần 5:

#### Blog 1 \- Geolocation Verification for iGaming & Sports Betting on AWS

Các kiến thức đã nắm được:

* Hiểu cách kiểm soát truy cập dựa trên vị trí địa lý bằng Route 53 Geolocation, CloudFront geo-restriction.  
* Hiểu các rủi ro như VPN, proxy spoofing và cách giảm thiểu bằng SDK \+ thiết bị xác minh vị trí.  
* Nhận thức rõ tầm quan trọng của compliance đối với các ngành yêu cầu giới hạn vùng pháp lý.

#### Blog 2 \- How to run AI model inference with GPUs on Amazon EKS Auto Mode

* Hiểu cơ chế EKS Auto Mode tự động provision node GPU theo nhu cầu workload (dựa vào Karpenter).  
* EKS Auto Mode hỗ trợ GPU AMI tối ưu, auto-repair node, monitoring NMA, DCGM.  
* Quy trình deploy mô hình AI với vLLM trên GPU node \-\> đơn giản hơn nhiều so với EKS tự quản lý.

#### Blog 3 \- Organization-wide S3 Tagging Governance

* Sử dụng AWS Config “required-tags” rule để kiểm soát compliance toàn tổ chức.  
* Khi bucket non-compliant, Lambda tự động áp policy deny upload; khi compliant thì tự mở khóa trở lại.  
* Sử dụng EventBridge forwarding từ các account con về management account.

#### AWS Pricing Calculator

* Ước tính chi phí EC2, Lambda, API Gateway, RDS, CloudFront, S3.  
* Xác định dịch vụ nào tốn chi phí cao nhất trong kiến trúc.  
* Biết cách điều chỉnh instance type, region, storage để tối ưu hóa chi phí.

 