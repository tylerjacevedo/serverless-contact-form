# Serverless Contact Form — AWS Cloud Project

## Overview
A fully serverless contact form built on AWS that captures user submissions, 
stores them in DynamoDB, and sends email notifications via SES. This project 
demonstrates how to connect multiple AWS services into a working cloud-native 
application without managing any servers.

## Architecture
![Architecture Diagram](architecture/diagram.drawio.png)

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
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";

const db = new DynamoDBClient({ region: "us-east-2" });
const ses = new SESClient({ region: "us-east-2" });

export const handler = async (event) => {
  const body = JSON.parse(event.body || "{}");
  const { name, email, message } = body;

  if (!name || !email || !message) {
    return {
      statusCode: 400,
      body: JSON.stringify({ error: "Missing required fields" })
    };
  }

  await db.send(new PutItemCommand({
    TableName: "ContactSubmissions",
    Item: {
      email: { S: email },
      name: { S: name },
      message: { S: message },
      timestamp: { S: new Date().toISOString() }
    }
  }));

  await ses.send(new SendEmailCommand({
    Source: "your-verified-email@example.com",
    Destination: { ToAddresses: [email] },
    Message: {
      Subject: { Data: "Thanks for contacting us!" },
      Body: {
        Text: { Data: `Hi ${name}, we received your message and will be in touch soon!` }
      }
    }
  }));

  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Form submitted successfully!" })
  };
};
```

## Challenges & How I Solved Them
- **IAM Permission Error** — Lambda was missing `ses:SendEmail` permission. 
  Fixed by attaching `AmazonSESFullAccess` to the Lambda IAM role. 
  Diagnosed using CloudWatch Log Insights.
- **SES Sender Not Verified** — The placeholder source email was still in 
  the Lambda code. Fixed by replacing it with a verified SES identity.
- **CORS Error** — Resolved by enabling CORS on the API Gateway POST route 
  to allow requests from the S3 hosted frontend.

## What I Learned
- How to connect multiple AWS services into a working serverless application
- How IAM roles control permissions between AWS services
- How to debug Lambda errors using CloudWatch Log Insights
- How serverless architecture eliminates the need for managing servers
- How API Gateway acts as the bridge between frontend and backend
- Real world troubleshooting of cloud infrastructure errors

## Deployment Guide
To deploy your own version:
1. Create an S3 bucket with static website hosting enabled
2. Deploy the Lambda function with the Node.js runtime
3. Set up API Gateway with a POST route pointing to Lambda
4. Create a DynamoDB table named `ContactSubmissions` with `email` as the key
5. Verify a sender email in SES
6. Attach `AmazonSESFullAccess` and `AmazonDynamoDBFullAccess` to the Lambda IAM role
7. Update the API Gateway URL in your `index.html` fetch call
8. Upload `index.html` to your S3 bucket

## Live Demo
Available upon request.

## Author
Tyler Acevedo
