# Day 18 – QuickCart Serverless Order Processing Platform

---

## 📌 Project Overview

QuickCart is an end-to-end order processing platform built using AWS managed services and Infrastructure as Code with AWS CloudFormation.

The project demonstrates how a customer order can travel through an HTTP API, Lambda functions, AWS Step Functions and DynamoDB while using IAM least-privilege permissions.

The application also includes an EC2-based web interface running Nginx, allowing users to submit orders and monitor their processing status directly from a browser.

---
## 🏗️ Architecture

## Serverless Order Processing

![Architecture](images/architecture1.png)

---
## Cloudformation Lifecycle

![Architecture](images/architecture2.png)

---
## Key AWS Services Used

- AWS CloudFormation – Infrastructure as Code

- Amazon EC2 – QuickCart web UI

- Amazon API Gateway – HTTP API

- AWS Lambda – Serverless application logic

- AWS Step Functions – Workflow orchestration

- Amazon DynamoDB – Order persistence

- AWS IAM – Least-privilege access control

- Amazon CloudWatch Logs – Logging and observability

---
## Project Outcome

This project demonstrates how a complete order-processing application can be provisioned through CloudFormation and implemented using managed AWS services. It covers API integration, serverless compute, workflow orchestration, database persistence, error handling, retries, IAM security, logging, and an EC2-hosted customer interface.

- Project: QuickCart Serverless Order Processing Platform
- Day: 18
- Region: us-east-2 (Ohio)

---
### Part A — AWS Lambda

# 1. Lambda Function Configuration
   
Created quickcart-validate-order-dev-iac-day18 and verified its basic configuration.

![Architecture](images/1.jpg)

![Architecture](images/2.jpg)

---
## 2. Lambda Accepted Event

- Invoked the Lambda with an accepted order event and verified the successful result.

- Event name: test-accepted

Event JSON:
```json
{
  "orderId": "QC-1800",
  "amount": 2500
}
```
![Architecture](images/3.jpg)

![Architecture](images/4.jpg)

---
## 3. Lambda Rejected Event

- Invoked the Lambda with a rejected order event and verified the rejection result.
  
- Event name: test-rejected

Event JSON:
```json
{
  "orderId": "QC-1802",
  "amount": 0
}
```
![Architecture](images/5.jpg)

![Architecture](images/6.jpg)

---
## 4. Lambda Technical Failure Event

- Invoked the Lambda with a deliberate technical-failure event and verified the failure behavior.
  
- Event name: test-failure

Event JSON:
```json
{
  "orderId": "QC-1803",
  "amount": 5000,
  "simulateFailure": true
}
```
![Architecture](images/7.jpg)

![Architecture](images/8.jpg)

---
## Part B — Lambda Asynchronous Retry and SQS

## 5. Async Retry Configuration

- Configured asynchronous Lambda retries and verified the configured retry behavior.
  
- Event name: test-async-failure

Event JSON:
```json
{
  "orderId": "O-1804",
  "amount": 7500,
  "simulateFailure": true
}
```
![Architecture](images/9.jpg)

![Architecture](images/10.jpg)

---
## 6. SQS Failure Destination

- Configured quickcart-lambda-failure-day18 as the Lambda asynchronous failure destination and verified the failed event record.

![Architecture](images/11.jpg)

---
## Part C — API Gateway HTTP API

## 7. POST /orders Route

- Created the HTTP API POST /orders route and configured the Lambda integration.

![Architecture](images/12.jpg)

8. HTTP 200 Test
   
- Invoked POST /orders with a valid request (amount: 3500) and verified HTTP 200, status: ACCEPTED.

9. HTTP 400 Test
   
- Invoked POST /orders with an invalid request (amount: 0) and verified HTTP 400, status: REJECTED, reason: "Amount must be greater than zero".

---
## Part D — AWS Step Functions
 
## 10. Standard Workflow Definition
 
- Created the Standard Step Functions order workflow (quickcart-order-workflow-dev-iac-day18) using Task, Retry, Catch, Choice, Succeed, and Fail states.

![Architecture](images/15.jpg)

---
## 11. Accepted Workflow Execution

- Executed an accepted order and verified the successful workflow path.
```json
{
  "orderId": "QC-SFN-1801",
  "amount": 4500
}
```
![Architecture](images/16.jpg)

---
## 12. Business Rejection Workflow Execution

- Executed a business-rejection order and verified the expected rejection path.
```json
{
  "orderId": "O-SFN-1802",
  "amount": 0
}
```
![Architecture](images/17.jpg)

---
## 13. Technical Failure Workflow Execution

- Executed a technical-failure order and verified Retry/Catch behavior and the failure path.
```json
{
  "orderId": "O-SFN-1803",
  "simulateFailure": true
}
```
![Architecture](images/18.jpg)

---
## 14. Step Functions Execution History
 
- Reviewed execution histories for the accepted, business-rejection, and technical-failure workflow executions.

![Architecture](images/19.jpg)

---
## Part E — AWS CloudFormation

## 15. CloudFormation Stack

- Created quickcart-infrastructure-day18 using the CloudFormation template. (1.quickcart-day18-CloudFormation.yaml)

![Architecture](images/20.jpg)

---
## 16. CloudFormation Stack Events

- Reviewed CloudFormation stack events and verified successful resource creation and update activity.

![Architecture](images/21.jpg)

---
## 17. CloudFormation Stack Outputs

- Reviewed the CloudFormation stack outputs after successful stack creation.

![Architecture](images/22.jpg)

---
## Part F — CloudFormation Change Set

## 18. Visibility Timeout Change Set

- Created a Change Set to modify the SQS visibility timeout (30 → 60 seconds) and reviewed the proposed change before execution.

![Architecture](images/23.jpg)

---
## 19. Change Set Execution & No-Replacement Verification

- Executed the Change Set and confirmed the SQS visibility timeout update completed successfully with the resource modified in place (no replacement).

![Architecture](images/25.jpg)

![Architecture](images/24.jpg)

---
## Part G — CloudFormation Drift Detection

## 20. Safe SQS Drift

- Made a safe SQS configuration change outside CloudFormation (message retention period 4 Days → 1 Day) to intentionally create drift.

![Architecture](images/26.jpg)

---
## 21. Drift Detection — MODIFIED
 
- Ran CloudFormation drift detection and verified that the modified resource was detected as MODIFIED (MessageRetentionPeriod: 345600 expected vs. 86400 actual).

![Architecture](images/27.jpg)

---
##  22. Drift Reconciliation and Final IN_SYNC

- Reconciled the SQS configuration with the CloudFormation template and verified that the resource returned to IN_SYNC.

![Architecture](images/28.jpg)

---
## Part H — Retained S3 Archive Bucket

## 23. S3 Retention and Replacement Behavior

- Reviewed the deletion and replacement behavior of the retained S3 archive bucket managed through the CloudFormation lifecycle template.
  
- Deleted the stack and verified the bucket (quickcart-archive-dev-day18-<account-id>) survived deletion due to DeletionPolicy: Retain and UpdateReplacePolicy: Retain.

![Architecture](images/29.jpg)

---
## Part I — End-to-End Order Application

## 24. Template Review and Deployment

- Reviewed the parameters and security requirements of the end-to-end order application template before deploying, then created the stack successfully (24 resources, Environment: test). (2.quickcart-day18-CloudFormation.yaml)

![Architecture](images/30.jpg)  

---
## 25. Order Portal UI
 
- Opened the WebsiteUrl stack output in a browser and confirmed the QuickCart Order Portal loaded successfully.

![Architecture](images/31.jpg)  

---
## 26. Accepted Order

- Submitted an order (O-UI-1801, amount 4500) with no simulated failure, and verified it completed successfully end-to-end.
  
![Architecture](images/32.jpg) 

---
## 27. Accepted Order DynamoDB Record

- Reviewed the DynamoDB orders table (quickcart-orders-test-iac-day18) and verified the accepted order was persisted with status COMPLETED.

![Architecture](images/33.jpg) 

---
## 28. Accepted Order Step Functions Execution

- Reviewed the Step Functions execution for the accepted order and verified it followed the expected success path (ValidateOrder → CheckOrderStatus → ProcessOrder → NotifyCustomer → OrderCompleted).

![Architecture](images/34.jpg) 

---
## 29. Business-Rejected Order

- Submitted an order (O-UI-1802, amount 0) and verified it was rejected by business validation.

![Architecture](images/35.jpg) 

---
## 30. Business-Rejected Order DynamoDB Record

- Reviewed the DynamoDB orders table and verified the rejected order was persisted with status REJECTED.

![Architecture](images/36.jpg) 

---
## 31. Business-Rejected Order Step Functions Execution

- Reviewed the Step Functions execution for the rejected order and verified it followed the expected business-rejection path (ValidateOrder → CheckOrderStatus → MarkRejected → OrderRejected).

![Architecture](images/37.jpg) 

---
## 32. Technical-Failure Order

- Submitted an order (O-UI-1803, amount 500) with "Simulate a technical failure" enabled and verified the workflow retried and then failed as expected (status: FAILED, reason: "Lambda processing failed after retries").

![Architecture](images/38.jpg) 

---
## 33. Technical-Failure Order DynamoDB Record

- Reviewed the DynamoDB orders table and verified the technically failed order was persisted with status FAILED.

![Architecture](images/39.jpg) 

---
## 34. Technical-Failure Order Step Functions Execution

- Reviewed the Step Functions execution for the technical-failure order and verified it followed the expected retry-and-catch path (ValidateOrder → Catch → MarkTechnicalFailure → TechnicalFailure).

![Architecture](images/40.jpg) 

---
## 🧹 Cleanup

Day 18 cleanup should be performed only after all required evidence has been captured.

- Delete the Lambda function quickcart-validate-order-dev-iac-day18.
- Delete the created SQS failure-destination queue quickcart-lambda-failure-dev-day18.
- Delete the created HTTP API quickcart-orders-dev-iac-day18.
- Delete the created Standard Step Functions workflow quickcart-order-workflow-dev-iac-day18.
- Delete the CloudFormation stack quickcart-infrastructure-day18.
- Verify the retained S3 archive bucket quickcart-archive-dev-day18-<ACCOUNT-ID> survived stack deletion, then empty and delete it manually, since DeletionPolicy/UpdateReplacePolicy: Retain does not delete it automatically.
- Verify that no billable Day 18 resources remain in Ohio (us-east-2).

---
## 🎯 What I Learned

Through this project, I practiced:

- Creating AWS CloudFormation templates using YAML
- Managing AWS infrastructure as code (IaC)
- Defining CloudFormation parameters and outputs
- Creating and configuring Amazon DynamoDB tables
- Configuring DynamoDB encryption and Point-in-Time Recovery
- Creating IAM roles and policies for AWS services
- Building AWS Lambda functions with Python
- Implementing Lambda-based order validation and processing
- Handling Lambda failures with Retry and Catch logic
- Creating AWS Step Functions Standard workflows
- Using Task, Choice, Succeed, and Fail states
- Integrating Lambda functions with Step Functions
- Updating DynamoDB records through workflow execution
- Building HTTP APIs using Amazon API Gateway
- Integrating API Gateway with Lambda functions
- Creating an EC2-based web server for the QuickCart Order Portal
- Configuring security groups and HTTP access for EC2
- Deploying infrastructure using CloudFormation
- Using CloudFormation to manage multiple AWS resources together
- Understanding DeletionPolicy: Retain and UpdateReplacePolicy: Retain
- Retaining an S3 archive bucket during CloudFormation stack deletion
- Testing an end-to-end serverless order-processing workflow
- Verifying order status and records in DynamoDB
- Understanding infrastructure lifecycle and cleanup
- Building a practical cloud-based order-processing architecture using AWS IaC

---
## 👨‍💻 Author

Hardik Darji
> DevOps Engineer

---
## ⭐ Support

If you found this project useful, consider giving it a ⭐ on GitHub.
