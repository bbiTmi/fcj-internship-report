---
title: "Worklog Week 10"
date: ""
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---
### Week 10 Objectives:

* Finalize the project dataset and add required attributes to support training and user authentication via AWS Cognito.  
* Use LocalStack to simulate AWS services for safe, cost-efficient testing and faster development.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 2 | Continue refining the dataset and adding additional features for AWS Cognito | 10/11/2025 | 12/11/2025 | |
| 5 | Test system functionalities using LocalStack to simulate the AWS environment | 13/11/2025 | 14/11/2025 | |

### Week 10 Achievements:

#### Testing AWS using LocalStack

LocalStack simulates AWS services locally without incurring cost, supporting many key AWS components.

**Installing and configuring LocalStack**

* Use Docker to run LocalStack.  
* Configure AWS CLI to point to LocalStack endpoints instead of real AWS services.

**Testing system functionalities**

* Create local DynamoDB tables  
* Create test S3 buckets  
* Test backend APIs without deploying to AWS  
* Reduce risk, save deployment time, and minimize testing costs
