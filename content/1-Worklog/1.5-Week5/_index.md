---
title: "Worklog Week 5"
date: ""
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---
### Week 5 Objectives:

* Deepen understanding of AWS services through blog translation.  
* Define the group project topic and assign tasks to team members.  
* Learn how to use the AWS Pricing Calculator to estimate project costs.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Translate Blog 1: **Building geolocation verification for iGaming and sports betting on AWS** Translate Blog 2: **How to run AI model inference with GPUs on Amazon EKS Auto Mode** Translate Blog 3: **Enforcing organization-wide Amazon S3 bucket-tagging policies** | 06/10/2025 | 08/10/2025 | [Geolocation Verification for iGaming & Sports Betting](https://docs.google.com/document/d/16V56RoubCuY4XGh_B0TClIuMR0ldb7YG8EpPRcY2VaA/edit?tab=t.0#heading=h.8lvjg3ekz0as)  
[Running AI Model Inference with GPUs on Amazon EKS Auto Mode](https://docs.google.com/document/d/1Fhfr6sTk9h5dAqC5BY_d9d4Tn4bmWUqNyMwKjPz4fuo/edit?tab=t.0#heading=h.bvk366iqrbm5)  
[Organization-wide S3 Tagging Governance](https://docs.google.com/document/d/1AaMq6i-3POxvIc3Z0xdkTH4uvw53R7zWy2ElzqbiCAY/edit?tab=t.0#heading=h.rmy7wqry9du5) |
| 3 | Finalize project topic and assign tasks to team members | 07/10/2025 | 07/10/2025 | |
| 5 | Use AWS Pricing Calculator to estimate project cost | 09/10/2025 | 09/10/2025 | |
| 7 | Write the project Proposal | 11/10/2025 | 13/10/2025 | |

### Week 5 Achievements:

#### Blog 1 – Geolocation Verification for iGaming & Sports Betting on AWS

Key knowledge learned:

* Understood how to enforce location-based access control using Route 53 Geolocation and CloudFront geo-restriction.  
* Learned about risks such as VPN or proxy spoofing and mitigation methods using SDKs and device-level location verification.  
* Recognized the importance of compliance requirements for industries operating under strict legal geographic boundaries.

#### Blog 2 – How to run AI model inference with GPUs on Amazon EKS Auto Mode

* Understood how EKS Auto Mode automatically provisions GPU nodes based on workload demand (powered by Karpenter).  
* EKS Auto Mode supports optimized GPU AMIs, automatic node repair, NMA monitoring, and DCGM.  
* The workflow for deploying AI inference workloads (e.g., vLLM) on GPU nodes becomes significantly simpler compared to self-managed EKS clusters.

#### Blog 3 – Organization-wide S3 Tagging Governance

* Learned how to use the AWS Config “required-tags” rule for organization-wide compliance enforcement.  
* When a bucket is non-compliant, Lambda automatically applies a deny-upload policy; once compliant, the restriction is removed.  
* Understood EventBridge forwarding from member accounts to the management account for centralized governance.

#### AWS Pricing Calculator

* Estimated the cost of EC2, Lambda, API Gateway, RDS, CloudFront, and S3.  
* Identified the services contributing the highest costs within the architecture.  
* Learned how to adjust instance types, regions, and storage options to optimize spending.
