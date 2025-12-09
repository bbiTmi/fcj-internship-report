---
title: "Event 4"
date: 2025-11-29
weight: 1
chapter: false
pre: " <b> 4.4. </b> "
---
# Summary Report: “AWS Cloud Mastery Series #3 – Security Pillar Workshop”

## 1. Event Objectives
This workshop focused on the Security Pillar of the AWS Well-Architected Framework, covering five core domains:
1. Identity & Access Management (IAM)
2. Detection & Continuous Monitoring
3. Infrastructure Protection
4. Data Protection
5. Incident Response

The event also introduced AWS Cloud Clubs, a community initiative supporting students in developing cloud skills.

## 2. Speakers
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

## 3. Key Highlights

### 3.1 AWS Cloud Club
- Supports students in learning cloud technologies through real-world scenarios.
- Provides mentorship and career development opportunities.
- Strengthens professional networking among participants.

### 3.2 IAM – Core Focus Area
- Apply Least Privilege.
- Remove root access keys and avoid wildcard (*).
- Use AWS SSO for multi-account access control.
- Implement SCPs, Permission Boundaries, and MFA (TOTP & FIDO2).
- Use Secrets Manager with a credential rotation workflow.

### 3.3 Detection & Monitoring
- Multi-layer observability: management events, data events, VPC Flow Logs.
- EventBridge used for automation and alert routing.
- Detection-as-Code implemented via CloudTrail Lake.

### 3.4 GuardDuty
- Detects threats using CloudTrail logs, VPC Flow Logs, and DNS queries.
- Advanced features: malware scanning, EKS audit logs, RDS anomaly detection, Lambda runtime monitoring.
- Aligns with AWS Security Best Practices and CIS Benchmarks.

### 3.5 Network Security
- Differentiates SG (stateful) vs NACL (stateless).
- Uses Route 53 Resolver for hybrid environments.
- AWS Network Firewall integrates GuardDuty threat intelligence.

### 3.6 Data Protection
- Uses KMS key hierarchy and IAM conditions for encryption control.
- ACM provides free TLS certificates.
- Enforces encryption for S3, DynamoDB, and RDS.
- Uses Secrets Manager rotation.

### 3.7 Incident Response
- Best practices: temporary credentials, avoid public S3, private subnet segmentation.
- Five-step IR workflow: Prepare → Detect & Analyze → Contain → Recover → Lessons Learned.

## 4. Application to Project
The workshop improved the team's AI Chatbot project by:
- Strengthening IAM for backend and admin access.
- Enhancing logging and monitoring to prevent misconfigurations.
- Establishing a consistent incident response process.

## 5. Event Experience
The workshop was well-organized, and complex concepts were explained clearly. The Q&A sessions helped reinforce understanding and resolve participant questions.
