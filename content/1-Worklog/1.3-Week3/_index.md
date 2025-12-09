---
title: "Worklog Week 3"
date: ""
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---
### Week 3 Objectives:

* Understand the concept of Amazon EC2 and the differences between instance types.  
* Configure basic network security using Security Groups to ensure safe connectivity.  
* Practice deploying EC2 on Linux & Windows, connecting via SSH and RDP.  
* Get familiar with installing Web Servers (LAMP/XAMPP) on EC2 to prepare for upcoming lessons on application architecture.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Learn EC2 concepts and instance types | 22/09/2025 | 22/09/2025 | [Module 02-Lab03-04.1 - Create EC2 Instances in Subnets](https://www.youtube.com/watch?v=duJEdF_g1To&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=41) |
|   | Configure Security Groups for the VPC — set proper inbound/outbound rules for SSH connectivity | 22/09/2025 | 22/09/2025 |   |
|   | Practice: Launch and connect to an EC2 instance on AWS — basic operations with EC2 | 22/09/2025 | 23/09/2025 | [Introduction to Amazon EC2](https://000004.awsstudygroup.com/) |
| 4 | Deploy an EC2 Linux instance: \- Connect via SSH \- Recover access when losing the Key Pair \- Install LAMP Web Server | 24/08/2025 | 24/08/2025 | |
| 5 | Deploy an EC2 Windows instance: \- Connect via Remote Desktop Protocol (RDP) \- Install XAMPP on the Windows instance | 24/08/2025 | 25/08/2025 | |

### Week 3 Achievements:

#### EC2 Concepts and Architecture

* EC2 is a compute service delivered under the IaaS model that allows creating virtual machines (instances) on demand.  
* The EC2 ecosystem includes:  
  * Instance Types  
  * AMI  
  * EBS Volume  
  * Key Pair  
  * Security Group  
  * Elastic IP  
  * User Data (bootstrapping scripts)

#### Security Group — the first firewall layer

* Security Group is a **stateful firewall**.  
* **Inbound rules:** control traffic entering the instance.  
* **Outbound rules:** control traffic leaving the instance.

#### Deploying EC2 Linux

1. Connect via SSH  
2. Recovering access when losing the key pair:  
   1. Create a new key pair  
   2. Use PuTTYgen to extract and copy the full Public Key from the newly created key pair  
   3. Stop the instance  
   4. Edit the instance’s User Data and paste the full cloud-config script with the copied Public Key into the `ssh-authorized-keys` section for the appropriate user (e.g., `ec2-user`). The Public Key must start with `ssh-rsa`  
   5. Restart the instance  
3. Install the LAMP stack (Apache, MariaDB, PHP)

#### Deploying EC2 Windows

* Connect via RDP (mstsc)  
* Retrieve the Administrator password using the key pair  
* Install and configure XAMPP  
* Access the web server through a browser

#### Difficulties Encountered

* SSH initially timed out because the Security Group did not allow the correct port.  
* When installing LAMP, Apache failed to start due to changes in Amazon Linux 2023, where “amazon-linux-extras” has been replaced by “distro-stream repositories”.

#### Solutions & Lessons Learned

* Double-check Security Group rules to ensure necessary ports are open.  
* Stop EC2 instances when not in use to avoid unnecessary charges.  
* Verify the Amazon Linux version and run appropriate commands.
