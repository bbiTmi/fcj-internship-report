---
title: "Worklog Week 4"
date: ""
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---
### Week 4 Objectives:

* Understand and practice deploying static website hosting using Amazon S3 combined with CloudFront.  
* Become familiar with using AWS CLI via VSCode to interact with AWS services more efficiently and professionally.  
* Practice creating and managing Amazon RDS (MySQL).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Understand the basic concepts of S3 and CloudFront | 29/09/2025 | 29/09/2025 | |
|   | Practice: \- Turn an S3 bucket into a static website \- Configure access for the website \- Create CloudFront distribution \- Practice Bucket Versioning | 29/09/2025 | 30/09/2025 | [with Amazon S3](https://000057.awsstudygroup.com/) |
| 3 | Connect AWS with VSCode and use CLI to work with AWS services | 30/09/2025 | 30/09/2025 | |
| 4 | Practice: Create RDS instances and work with MySQL on AWS | 01/10/2025 | 01/10/2025 | [Amazon Relational Database Service (Amazon RDS)](https://000005.awsstudygroup.com/) |

### Week 4 Achievements:

#### Basic understanding of S3 and static website hosting mechanism

* S3 is an object storage service suitable for hosting static websites (HTML/CSS/JS).  
* Static Website Hosting requires enabling the feature and specifying `index.html` & `error.html`.  
* The S3 bucket must be configured for public access or served through CloudFront for secure distribution.

#### Practicing static website deployment on S3

* Upload web source files to S3  
* Enable static hosting  
* Add bucket policy to allow public read access  
* Test access via the S3 endpoint URL

#### Creating and configuring CloudFront Distribution

* Origin = S3 bucket  
* Behavior: caching settings, TTL, HTTP/HTTPS  
* Distribution domain name  
* Use Invalidations when updating website content  

CloudFront provides:

* Improved content delivery speed  
* Hiding direct S3 access from end users  
* HTTPS support via ACM Certificate

#### Connecting AWS with VSCode + AWS CLI

Setup steps:

* Install AWS Toolkit for VSCode  
* Configure credentials (`aws configure`)  
* Operate S3, EC2, RDS using CLI commands

#### Practicing Amazon RDS (MySQL)

* Create an RDS MySQL instance  
* Configure subnet group and security group  
* Connect using MySQL Workbench  
* Create databases & run basic SQL commands
