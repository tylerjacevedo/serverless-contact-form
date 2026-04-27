# serverless-contact-form

Serverless contact form built with AWS S3, Lambda, API Gateway, SES, and DynamoDB

## Architecture
- S3: Static website hosting
- API Gateway: REST API endpoint
- Lambda: Form processing
- DynamoDB: Stores submissions
- SES: Email notifs
- Cloudwatch: Logging

## How it works
- User fills out form hosted on S3
- Form submits to API Gateway
- Lambda saves to DynamoDB and sends email notification via SES
