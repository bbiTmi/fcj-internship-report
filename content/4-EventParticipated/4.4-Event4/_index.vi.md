---
title: "Sự kiện 4"
date: 2025-11-29
weight: 1
chapter: false
pre: " <b> 4.4. </b> "
---
# **Bài Thu Hoạch: “AWS Cloud Mastery Series \#3 – Security Pillar Workshop”**

## **1\. Mục Tiêu Sự Kiện**

Workshop tập trung vào **Security Pillar** trong AWS Well-Architected Framework, bao gồm 5 nhóm nội dung chính:

1. **IAM (Identity & Access Management)**

2. **Detection & Continuous Monitoring**

3. **Infrastructure Protection**

4. **Data Protection**

5. **Incident Response**

Ngoài ra, sự kiện giới thiệu hệ sinh thái **AWS Cloud Clubs**, nơi hỗ trợ sinh viên phát triển kỹ năng cloud.

## **2\. Diễn Giả**

- **Lê Vũ Xuân An** - AWS Cloud Club Captain HCMUTE

- **Trần Đức Anh** - AWS Cloud Club Captain SGU

- **Trần Đoàn Công Lý** - AWS Cloud Club Captain PTIT

- **Danh Hoàng Hiếu Nghị** - AWS Cloud Club Captain HUFLIT

- **Huỳnh Hoàng Long** - AWS Community Builders

- **Đinh Lê Hoàng Anh** - AWS Community Builders

- **Nguyễn Tuấn Thịnh** - Cloud Engineer Trainee

- **Nguyễn Đỗ Thành Đạt** - Cloud Engineer Trainee

- **Văn Hoàng Kha** - Cloud Security Engineer, AWS Community Builder

- **Thịnh Lâm** - FCJ Member

- **Việt Nguyễn** - FCJ Member

- **Mendel Grabski (Long)** - Ex-Head of Security & DevOps, Cloud Security Solution Architect

- **Trịnh Trương** - Platform Engineer tại TymeX, AWS Community Builder

## **3\. Các Nội Dung Nổi Bật**

### **3.1 AWS Cloud Club**

* Xây dựng cộng đồng học cloud cho sinh viên.

* Cung cấp mentorship và cơ hội networking.

* Hỗ trợ học tập qua các tình huống thực tế.

### **3.2 IAM – Trụ cột quan trọng**

* Áp dụng nguyên tắc **Least Privilege**.

* Không dùng root access key, hạn chế wildcard (\*).

* Ưu tiên **AWS SSO** cho multi-account.

* Sử dụng **SCP**, **Permission Boundaries** và **MFA (TOTP & FIDO2)**.

* Secrets Manager \+ quy trình xoay vòng credential.

### **3.3 Detection & Monitoring**

* Quan sát đa lớp: Management events, Data events, VPC Flow Logs.

* EventBridge dùng để cảnh báo và tự động hóa.

* Triển khai Detection-as-Code qua CloudTrail Lake.

### **3.4 GuardDuty**

* Phát hiện mối đe dọa dựa trên CloudTrail, VPC Flow Logs và DNS queries.

* Tính năng nâng cao: Malware scan, EKS audit log, RDS anomaly detection, Lambda runtime monitoring.

* Tuân thủ AWS Security Best Practices & CIS Benchmarks.

### **3.5 Network Security**

* Phân biệt SG (stateful) vs NACL (stateless).

* Route 53 Resolver cho hybrid.

* AWS Network Firewall \+ threat intel từ GuardDuty.

### **3.6 Data Protection**

* KMS key hierarchy, IAM condition khi mã hóa dữ liệu.

* ACM cho chứng chỉ TLS miễn phí.

* Bắt buộc encryption khi truy cập S3, DynamoDB, RDS.

* Secrets Manager rotation.

### **3.7 Incident Response**

* Best practice: dùng temporary credentials, không public S3, tách service vào private subnet.

* Quy trình IR 5 bước: Chuẩn bị → Phát hiện → Cô lập → Khôi phục → Rút kinh nghiệm.

## **4\. Ứng Dụng Vào Dự Án**

Workshop giúp nhóm cải thiện dự án Chatbot AI qua:

* IAM chặt chẽ hơn cho backend & admin.

* Logging – monitoring rõ ràng để tránh misconfiguration.

* Áp dụng quy trình response để xử lý sự cố nhanh và nhất quán.

