---
title: "Worklog Week 11"
date: ""
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---
### Week 11 Objectives:

* Refine the complete dataset for the project to standardize input for backend services.  
* Understand the data flow from Cognito into the database and evaluate suitable storage options (S3 vs RDS vs DynamoDB).

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :---- | :---- | :---- | :---- | :---- |
| 3 | Refine and generate the complete dataset for the project | 18/11/2025 | 20/11/2025 | |
| 6 | \- Understand how Cognito writes data to the database in real time \- Consider switching to S3 or RDS for data storage | 21/11/2025 | 21/11/2025 | |

### Week 11 Achievements:

#### Cognito User Pool does not automatically write data to a database

→ However, real-time data syncing can be enabled through:

* **Post Confirmation Trigger** → Lambda → write to DynamoDB / RDS.  
* **Pre Token Generation Trigger** → add custom claims.  
* Sync user information via **Cognito Identity Pool** (if used).

#### Comparison: S3 vs DynamoDB vs RDS

| Service | Advantages | Disadvantages | Best use cases |
| :---- | :---- | :---- | :---- |
| **S3** | Low cost, stores large files/datasets, easy backup | Not suitable for fast queries, non-transactional | Storing logs, history records, datasets |
| **RDS** | Powerful SQL queries, supports complex relationships | Higher cost | Relational information, reporting systems |
| **DynamoDB** | Fast, highly scalable, event-driven architecture | No joins, flat schema | Real-time user profiles, session data |
