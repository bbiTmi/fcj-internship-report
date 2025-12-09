---
title: "Worklog Week 2"
date: ""
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---
### Week 2 Objectives:

* Understand core concepts of Amazon VPC to build a secure and best-practice network foundation.  
* Practice creating VPC, Subnets, Route Tables, and Gateways on AWS to understand how networking components operate in practice.  
* Learn how to draw network architecture diagrams using Draw.io.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 4 | Learn VPC concepts \- Understand Public Cloud/Private Cloud \- Distinguish Public Subnet, Private Subnet, and VPN-only Subnet \- Route table, Destination/Target \- Internet Gateway vs NAT Gateway | 16/09/2025 | 17/09/2025 | [YouTube](https://www.youtube.com/watch?v=O9Ac_vGHquM&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=25) |
| 4 | Practice creating a VPC on AWS | 17/09/2025 | 17/09/2025 | [Amazon VPC and AWS Site-to-Site VPN Workshop](https://000003.awsstudygroup.com/) |
| 6 | Draw architecture diagrams with Draw.io | 19/09/2025 | 19/09/2025 | [How to draw AWS architecture on Draw.io](https://www.youtube.com/watch?v=l8isyDe-GwY&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=2) |

### Week 2 Achievements:

#### VPC and the difference between Public Cloud and Private Cloud

* Public Cloud refers to infrastructure owned and operated by AWS; Private Cloud relates to isolated and secure network environments tailored for enterprise needs.  
* AWS VPC is a *Virtual Private Cloud* — logically private but still runs on AWS’s public infrastructure.

#### Distinguishing Public Subnet \- Private Subnet \- VPN-only Subnet

* **Public Subnet:** Route Table points to an Internet Gateway (IGW) → resources inside subnet can access the internet directly.  
* **Private Subnet:** No IGW connection → outbound internet access requires a NAT Gateway or NAT Instance.  
* **VPN-only Subnet:** Used only for connecting to on-premises environments via a VPN Gateway, without internet access.

#### Route Table \- Destination \- Target

* Route Tables define how traffic flows within a VPC.  
* **Destination:** the network address traffic is directed to.  
* **Target:** the gateway or component responsible for forwarding traffic to the Destination.

Route Table for **Public Subnet**

| Destination | Target |
| :---- | :---- |
| 0.0.0.0/0 | igw-12345 |

**Explanation:**  
→ All outbound internet traffic (0.0.0.0/0) must pass through the Internet Gateway.

Route Table for **Private Subnet**

| Destination | Target |
| :---- | :---- |
| 0.0.0.0/0 | nat-67890 |

**Explanation:**  
→ Instances in the Private Subnet must route outbound internet traffic through a NAT Gateway (which does not accept inbound connections).

#### Internet Gateway vs NAT Gateway

| Component | Purpose | Used in |
| :---- | :---- | :---- |
| Internet Gateway | Allows instances in a public subnet to **access** the internet and **receive inbound traffic** | Public subnet |
| NAT Gateway | Allows instances in a private subnet to **initiate outbound internet traffic** but **blocks inbound traffic** | Private subnet |

#### Drawing architecture with Draw.io

* Learned how to represent VPC, subnet, AZ, IGW, NAT Gateway, and EC2 using AWS icons.  
* Understood AWS architectural diagram standards.  
* Clear diagrams help convey architecture more effectively in reports and presentations.
