---
title: "Blog 3"
date: ""
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---
# Enforcing organization-wide Amazon S3 bucket-tagging policies

In today’s complex cloud environments, maintaining consistent resource tagging is a critical challenge faced by organizations of all sizes. Proper resource tagging is essential for cost allocation, security compliance, operational management, and maintaining governance at scale. However, enforcing tagging standards across distributed teams and numerous resources can be difficult, especially when dealing with rapid deployment cycles and multiple stakeholder requirements. Organizations often struggle to maintain consistent tagging practices, which leads to poor resource visibility, inaccurate cost attribution, and challenges in maintaining compliance with internal policies.

AWS provides several capabilities to help organizations implement and maintain effective tagging governance for [Amazon S3](https://aws.amazon.com/s3/), which offers cloud storage that serves as the foundation for countless workloads across development, analytics, backup, content delivery use cases, and more. As storage deployments grow and become more distributed across teams and projects, they often become a prime example tagging challenges. Through a combination of proactive and reactive controls, organizations can establish automated mechanisms to make sure that resources meet tagging requirements. These controls can be implemented across an entire [AWS Organization](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html), providing centralized oversight while maintaining the flexibility needed for different business units and workloads.

In this post, I demonstrate how to implement a reactive automated tagging governance solution for S3 buckets using [AWS Config](https://aws.amazon.com/config/), [AWS Lambda](https://aws.amazon.com/lambda/), and [Amazon EventBridge](https://aws.amazon.com/eventbridge). You can learn how to deploy a solution that automatically restricts object uploads to non-compliant buckets and removes these restrictions when the necessary tags are applied. This approach enables organizations to enforce tagging standards without manual intervention, make sure of consistent resource categorization, and maintain accurate cost allocation—all while minimizing impact on development teams and existing workflows.


## **Solution overview**

This solution takes a reactive approach to tagging governance by using AWS Config compliance service and using the “required-tags” AWS Config managed rule. This rule monitors and identifies resources where necessary tags are missing. Then, this rule applies a resource policy on buckets that are non-compliant to block new object uploads. When the necessary tags are applied by the bucket owners, the solution automatically removes the policy, lifting the object upload restriction. With minimal adjustment to the configurations, this solution could be extended to be used to enforce tagging for other types of AWS resources.

The solution uses a hub and spoke model. The compliance monitoring is done at the account and [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) level. Then, compliance status changes are routed to a centralized account that is responsible for taking action when an S3 bucket goes from compliant to non-compliant state or the reverse. This solution uses AWS Config to evaluate the compliance status and to detect changes, EventBridge to communicate the changes, and Lambda to take automated remediation action.

The following diagram represents the components that are deployed as part of this solution:


![S3 Tagging Governance Solution Diagram](/images/3-Blog/3.3-Blog3/S3-Tagging-Governance-Solution-Diagram.png)


*Figure 1: Architecture diagram of the AWS resources deployed by the CloudFormation templates into management and linked AWS accounts to support this solution*

Each linked account where this solution is deployed has an AWS Config rule deployed that monitors the tagging compliance of the S3 buckets. Each linked account/Region also has an EventBridge rule that forwards the notification about compliance change to the solution management (or designated orchestration) account. The solution management account has the centralized EventBridge bus, a rule, and a Lambda function that update the S3 bucket policy according to the tagging compliance status. If you are looking to enforce tagging on the management (or designated orchestration) account along with using it for orchestration, then make sure to deploy both CFN templates in that account. In that scenario, all components represented in the preceding figure are deployed in that account.

This solution also recommends that you use our proactive governance capability with [AWS Organization tag policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html) (optional). Tag policies allow you to enforce consistency in how tag keys are named and which values are allowed to be assigned. [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates provided with this post do not deploy tag policies, and you must configure them separately.



1. Monitoring: An AWS Config rule in each account and AWS Region within your organization monitors S3 buckets for adherence to the tagging policy that you define in the rule. When the compliance status changes, the rule sends an event to the Default EventBridge bus in that account.
2. Event forwarding: The EventBridge rule in each individual account picks up that event and forwards it to a centralized custom EventBridge bus hosted in the management(or designated orchestration) account.
3. Processing: A rule set up on the custom EventBridge bus in the management (or designated orchestration) account picks up the event and sends it for processing to a Lambda function. The rule also records the event in [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) Logs for record keeping.
4. Policy application: The Lambda function checks if the S3 bucket compliance status is “Non-compliant”, and updates the bucket resource policy to prevent any objects from being uploaded to it with the following policy:


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "BlockFileUploadToBucket",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<bucket name>/*"
        }
    ]
}
```


The Lambda function in the management (or designated orchestration) account updates the bucket policy in the linked account. It does this by assuming a role that is deployed through StackSets to each target account.



5. Remediation: When the bucket becomes “Compliant” again, when the necessary tags are applied the S3 bucket, the Lambda function reverses the restriction on uploading objects by removing the resource bucket policy.


## **Deployment process**

To deploy the solution, use two [CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) templates provided in [this GitHub project](https://github.com/aws-samples/Creating-s3-tagging-governance-across-aws-organization). One of the templates is intended to be deployed in the management account or another account designated to act as a centralized account for the solution. The second CFN template is to be deployed in each account and AWS Region where tagging enforcement is needed. This could be done directly using CloudFormation Stacks or using CloudFormation StackSets from the Organization management account.

The s3-tagging-governance-mngt.yaml CloudFormation template deploys the components intended to run in your management (or designated orchestration) account.



* You can deploy this template in your management account or any other account that serves as the designated orchestration account and is responsible for managing non-compliance notifications and taking remediation actions.
* Deploy in a single Region where most of your resources are deployed.
* Use CloudFormation Stacks to deploy this template.
* As part of deployment, you must provide the OrgID for your Organization. Obtain this by going to the Organizations console -> Settings. The Organization ID is listed under Organization Detail.
* When it is successfully deployed, go to the Outputs tab of the deployed CFN Stack, and note the following values (you must specify them when deploying the s3-tagging-governance-linked.yaml template:
    a. CentralEventBusArn: contains the [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) of the deployed central event bus.
    b. LambdaAssumeRole: contains thename of the role that Lambda assumes while executing to deploy resource policy update in each linked account.
    c. LambdaMngtExecutionRoleArn: contains the ARN of the Lambda execution role deployed in the management (or designated orchestration) account.

The s3-tagging-governance-linked.yaml CloudFormation template is to be deployed into accounts and regions where you want to implement tagging governance.



* Deploy using CloudFormation StackSets or regular CFN Stacks.
* Deploy in each account and AWS Region where you want to enforce tagging.
* When deploying this template, use the values that you captured when deploying s3-tagging-governance-mngt.yaml template:
    a. ExecLambdaRoleName: use the output parameter value that contains the name of the role that Lambda assumes while executing to deploy the resource policy update in each linked account
    b. MngtEventBusArn: use the output parameter that contains the ARN of the event bus deployed in the management (or designated orchestration) account.
    c. MngtLambdaRoleArn: use the output parameter that contains the ARN of the Lambda execution role deployed in the management (or designated orchestration) account
* Make sure to deploy this template in the solution management (or designated orchestration) account (along with s3-tagging-governance-mngt.yaml) if you are also looking to enforce tagging compliance for S3 buckets in that account.

CloudFormation templates do not deploy tag policies, but you can define them by following the instructions here: [AWS Organizations – Tag Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies-getting-started.html).


## **A few important considerations**



* The solution as it is packaged uses tags to determine which objects are in scope for the AWS Config rule. The tag that the Config rule looks for has a “TagsRequired-PLEASE-UPDATE” key and a value of “Yes”. Prior to deploying the solution, make sure to update CloudFormation templates to change the tag to something that is used to determine if the bucket should be governed by this policy or use a different technique to [define scope for the AWS Config Rule](https://docs.aws.amazon.com/config/latest/APIReference/API_Scope.html) (resource for instance).
* CAUTION: When deploying this solution, carefully consider which S3 buckets to target (consider the preceding bullet point) and how this impacts existing workloads that are using those buckets. When the solution applies the resource policy on non-compliant buckets, the workloads can no longer upload files to those buckets, which might have a negative effect on your workloads.
* If your S3 buckets are already using bucket resource polices and a bucket that is non-compliant (does not have the necessary tags) already has the existing policy, then it is updated to include the statement to block the file uploads that this solution applies. Your existing statements aren’t impacted. Lambda code reviewsall statements using statement ID (SID) to determine if the statement with ID “BlockFileUploadToBucket” already exists and adds it if it is not there. It uses the same SID to remove the statement from the policy when the bucket is compliant.
* To prevent users from manually updating S3 bucket policies, use Service Control Policies (SCPs) or user [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) polices to lock down the user’s permissions by explicitly denying actions such as “s3:PutBucketPolicy” and “s3:DeleteBucketPolicy”.


## **Customer spotlight: Marriott International**

The [Marriott International](https://www.marriott.com/) Cloud FinOps team relies on resource tags for their financial reporting. These tags are used to identify resource owners and attribute the costs to respective workloads and business units. Although they had a policy in place that needed the teams to use tagging, the compliance rate was very low and a lot of costs were being reported as “uncategorized.” To enforce the policy, the Marriott FinOps team embarked on the project to create tagging governance across most used AWS services used across their teams.

They successfully employed Service Control Policies to block the creation of any [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instance or [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) volumes that did not have the necessary tags. However, when they tried to employ this approach for S3 buckets, it presented challenges. First, they started to come up against the character size limits described earlier in this post, and they discovered that the tag condition is not supported in SCP polices for Amazon S3.

At that point, the Marriott FinOps team partnered with their AWS Technical Account Manager and Solution Architect to develop the solution described in this post. After deploying this solution across their Organization, they were able to reduce the footprint of untagged S3 buckets by 80%. This allowed them to properly report the costs associated with Amazon S3 usage to respective resource owners and to the leadership.


## **Conclusion**

This solution demonstrates a comprehensive approach to enforcing Amazon S3 bucket tagging compliance using the AWS Config, Amazon EventBridge, and AWS Lambda functions in a hub-and-spoke model. The solution monitors S3 buckets across your AWS Organization, automatically restricts object uploads to non-compliant buckets through resource policies, and removes these restrictions when the necessary tags are applied. Key components of the solution include AWS Config rules for monitoring, EventBridge for event routing, and Lambda functions for automated policy enforcement. All of these are deployable through the provided AWS CloudFormation templates.

The benefits of implementing this solution extend beyond tag enforcement. Organizations can achieve better cost allocation and resource management through consistent tagging, as demonstrated by Marriott’s 80% reduction in untagged S3 buckets. This approach overcomes the limitations of SCPs, offering a scalable way to enforce tagging governance without running into SCP character limits or service-specific constraints. Furthermore, the solution’s automated remediation capabilities minimize administrative overhead while making sure of continuous compliance.

To get started with implementing this tagging governance solution, download the provided CloudFormation templates from the GitHub repository and follow the detailed deployment instructions. Remember to carefully consider your bucket targeting strategy and existing workload requirements before deployment. For more guidance or customization needs, consult the AWS documentation or reach out to your AWS account team to make sure of a successful implementation that aligns with your organization’s specific requirements.

TAGS: [Amazon EventBridge](https://aws.amazon.com/blogs/storage/tag/amazon-eventbridge/), [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/blogs/storage/tag/amazon-simple-storage-service-amazon-s3/), [AWS Cloud Storage](https://aws.amazon.com/blogs/storage/tag/aws-cloud-storage/), [AWS Config](https://aws.amazon.com/blogs/storage/tag/aws-config/), [AWS Lambd](https://aws.amazon.com/blogs/storage/tag/aws-lambda/)a


**Pavel Rabinovich**
<img src="/images/3-Blog/3.3-Blog3/Pavel-Rabinovich-Profile-Image-1.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Pavel Rabinovich is a Senior Technical Account Manager at AWS, based in Washington, D.C., with over 25 years of experience in IT and software engineering. He advises enterprise customers in the hospitality, healthcare, and insurance industries on cloud strategy and security. As a member of the AWS Security Technical Field Community, Pavel focuses on identity and application security.
<div style="clear: both;"></div>

**Avinash Pala**
<img src="/images/3-Blog/3.3-Blog3/Avinash-Pala-Profile-Image.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Avinash Pala is a Cloud Solution Architect and Data Architect. He is AWS Certified.
<div style="clear: both;"></div>

**Reema Bali**
<img src="/images/3-Blog/3.3-Blog3/Reema-Bali-Profile-Image.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Reema Bali is "Director - Enterprise FinOps" at Marriott International responsible for bridging the gap between Engineering and Finance, ensuring cloud spending is optimized without sacrificing speed or innovation. She is a seasoned IT leader with over 25 years of experience and a proven track record in driving large-scale, high-impact programs that align with strategic business goals.
<div style="clear: both;"></div>

**Ruby Siddiqui**
<img src="/images/3-Blog/3.3-Blog3/Ruby-Siddiqui-Profile-Image-1.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Ruby Siddiqui is Senior Director of FinOps Engineering and Technology Strategy at Marriott International, leading enterprise-wide cloud financial operations and optimization initiatives. With expertise in cloud economics, Application development, observability, and governance, Ruby drives scalable FinOps practices that deliver cost transparency and business value across multi-cloud environments.
<div style="clear: both;"></div>

**Sri Gudavalli**
<img src="/images/3-Blog/3.3-Blog3/srgudava.png" width="100" style="float: left; margin-right: 15px; margin-top:-1px;">

Sri Gudavalli is a Solutions Architect with AWS helping Enterprise customers with their cloud migration and modernization journey. He works with Enterprise customers from the US-East Region to build leading edge cloud applications and services on AWS.
