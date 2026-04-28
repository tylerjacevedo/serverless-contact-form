# Serverless Contact Form — AWS Cloud Project

## Overview
A fully serverless contact form built on AWS that captures user submissions, 
stores them in DynamoDB, and sends email notifications via SES. This project 
demonstrates how to connect multiple AWS services into a working cloud-native 
application without managing any servers.

## Architecture
![Architecture Diagram](Serverless_Contact_Form)

## AWS Services Used

### S3 — Static Website Hosting
- Created an S3 bucket named `contact-form-frontend-001`
- Disabled "Block Public Access" settings
- Enabled static website hosting with `index.html` as the root document
- Attached a bucket policy to allow public read access to host the frontend

### API Gateway — REST API
- Created a REST API with a POST route `/contact`
- Enabled CORS to allow requests from the S3 website URL
- Deployed to a stage and integrated with Lambda as the backend

### Lambda — Form Processing
- Created function `contactFormHandler` using Node.js runtime
- Parses incoming form data from API Gateway
- Saves submission to DynamoDB
- Triggers SES to send an email notification
- IAM Role attached with the following permissions:
  - `AmazonDynamoDBFullAccess`
  - `AmazonSESFullAccess`
  - `AWSLambdaBasicExecutionRole`

### DynamoDB — Database
- Created table `ContactSubmissions`
- Set `email` as the partition key (String type)
- Used on-demand billing mode (pay per request)
- Stores name, email, message, and timestamp for every submission

### SES — Email Notifications
- Verified sender email identity in `us-east-2`
- Lambda sends an HTML formatted email notification on every form submission
- Account in sandbox mode (sends to verified emails only)

### CloudWatch — Logging & Monitoring
- Automatically provisioned with Lambda
- Used Log Insights to monitor Lambda executions
- Used to trace and debug errors during development

## How It Works
1. User visits the contact form hosted on S3
2. User fills out name, email, and message and clicks submit
3. Form sends a POST request to API Gateway
4. API Gateway triggers the Lambda function
5. Lambda validates the form data
6. Lambda saves the submission to DynamoDB
7. Lambda sends an email notification via SES
8. User receives a success message on the form

## Lambda Function
```javascript
