---
title: "Worklog Week 7"
date: ""
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---
### Week 7 Objectives:

* Understand and practice using DynamoDB.  
* Learn how to use Route 53 to host websites and manage DNS.  
* Deepen knowledge of Resilient Architecture and disaster recovery strategies.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Learn about DynamoDB and its components | 20/10/2025 | 20/10/2025 | [Amazon DynamoDB](https://000060.awsstudygroup.com/) |
| 3 | Practice creating DynamoDB on AWS and learn how to host a website using Route 53 | 21/10/2025 | 21/10/2025 | |
| 4 | Review: Resilient Architecture \- Multi AZ, Multi Region \- Disaster Recovery Strategies (Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active) \- Health Check \- Load Balancing \- AWS services that help recover systems quickly | 22/10/2025 | 25/10/2025 | |
| 7 | Draw system architecture for the project and estimate expected traffic usage | 25/10/2025 | 26/10/2025 | |

### Week 7 Achievements:

#### DynamoDB

* DynamoDB is a NoSQL key-value database with near-infinite scalability.  
* Key components include:  
  * Partition Key / Sort Key  
  * GSI (Global Secondary Index)  
  * LSI (Local Secondary Index)  
  * Provisioned / On-Demand Capacity

#### Hosting a website with Route 53

* How to purchase and configure a domain with Route 53.  
* Create Hosted Zones and Records: A, CNAME.  
* Pointing domain to:  
  * S3 Static Website  
  * CloudFront Distribution  
  * EC2 Elastic IP  
* Understanding DNS propagation and TTL.

#### Resilient Architecture Concepts

* **Multi-AZ:** Automatically fails over when an AZ encounters issues.  
* **Multi-Region:** Backup strategy for systems requiring extremely high uptime or global distribution.  
* **Disaster Recovery Strategies**

| Strategy | Complexity | Cost | Recovery Time |
| :---- | :---- | :---- | :---- |
| Backup & Restore | Low | Low | Longest |
| Pilot Light | Medium | Medium | Moderate |
| Warm Standby | High | High | Fast |
| Multi-Site Active/Active | Very high | Very high | Near-zero downtime |

**RTO & RPO:**

* **RTO:** Maximum allowable downtime.  
* **RPO:** Maximum acceptable data loss.

**Health Check:** Route 53 monitors endpoint health → fails over to backup site if failures occur.  
**Load Balancing:** ALB/NLB distribute traffic and improve fault tolerance.
