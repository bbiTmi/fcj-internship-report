---
title: "Worklog Week 6"
date: ""
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
### Week 6 Objectives:

* Strengthen understanding of Secure Architecture and AWS core security mechanisms.  
* Understand how Elastic Load Balancing works and the different routing methods.  
* Master Disaster Recovery strategies in AWS and distinguish RTO/RPO.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Review: Secure Architecture \- Understand IAM (User, Group, Policy, Role, Least Privilege) \- Types of MFA for account protection \- Compare Security Group vs NACL | 13/10/2025 | 13/10/2025 | |
| 3 | Learn Elastic Load Balancing concepts: \- Application Load Balancer (ALB) \- Network Load Balancer (NLB) \- Open Systems Interconnections \- Routing types of ALB and NLB \- Sticky Session / Session Affinity | 14/10/2025 | 14/10/2025 | |
| 4 | Study Key Management Service, AWS Certificate Management, and encryption/key storage | 15/10/2025 | 15/10/2025 | |
| 5 | Learn Disaster Recovery Strategies: \- Backup & Restore \- Pilot Light \- Warm Standby \- Multi-Site Active-Active Distinguish RTO and RPO | 16/10/2025 | 16/10/2025 | |

### Week 6 Achievements:

#### Secure Architecture

Key points consolidated:

* IAM includes User, Group, Policy, Role; apply the Least Privilege principle.  
* MFA protects accounts with methods like Virtual MFA, Hardware MFA, and SMS MFA.  
* Security Group: stateful firewall (automatically allows response traffic).  
* NACL: stateless, applied at subnet level, suitable for deny rules.  
* SG controls instance-level traffic → more granular; NACL protects at subnet layer → more general.

#### Elastic Load Balancing – ALB, NLB, and routing

* ALB operates at Layer 7, supports HTTP/HTTPS, and routing by path, host, header, or query string.  
* NLB operates at Layer 4, optimized for extremely high throughput and millions of requests per second, suitable for TCP/UDP.  
* OSI model clarifies ALB (L7) vs NLB (L4).  
* Understood routing mechanisms: weighted, host-based, path-based (ALB) and TCP/UDP routing (NLB).  
* Sticky Session / Session Affinity uses cookies to maintain user connections to the same target.

#### KMS & ACM – Encryption and key management

* AWS KMS manages creation, storage, and lifecycle of encryption keys (CMK).  
* Supports encryption across S3, EBS, RDS, and many services via envelope encryption.  
* Distinguish AWS-managed keys vs Customer-managed keys (CMK).  
* AWS Certificate Manager (ACM) provides free SSL/TLS certificates with automatic renewal.  
* Combining ACM with CloudFront/ALB ensures end-to-end HTTPS.

#### Disaster Recovery Strategies

| Strategy | Description | Recovery Level |
| :---- | :---- | :---- |
| Backup & Restore | Backup data and restore during disaster | Low |
| Pilot Light | Keep the minimal core system running | Medium |
| Warm Standby | A smaller-scale version running in parallel | High |
| Multi-Site Active-Active | Multiple active sites simultaneously | Very high |

* **RTO (Recovery Time Objective):** acceptable downtime duration.  
* **RPO (Recovery Point Objective):** maximum acceptable data loss during an incident.
