---
title: "Prerequisite"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Prerequisites

Building and deploying InsightShare requires the following tools, accounts and permissions.

#### 1. Developer Environment
- **Python 3** (3.10 or higher): the language used for the FastAPI local app and the Lambda function.
- **boto3**: the AWS SDK for Python (`pip install boto3`), used to generate presigned URLs and call S3, DynamoDB, Rekognition, Textract and Polly.
- **AWS CLI**: configured with `aws configure` (access key, region `ap-southeast-1`) to manage resources from the command line.
- **Node.js (v18 or higher)**: optional, to build the static frontend if you use a bundler.
- **Git**: to clone and manage the codebase.
- **Visual Studio Code** (or any editor): for backend and frontend development.

#### 2. Cloud Account & Region
Because InsightShare runs on AWS managed services, you need:
- **An AWS account** with permission to create and delete the resources used in this workshop.
- Deployment region: **Asia Pacific (Singapore), `ap-southeast-1`**.
- An **S3 bucket** for uploaded files and Polly-generated audio, and a second use of S3 to host the static frontend.
- Access configured for the AWS CLI, verified with `aws sts get-caller-identity` pointing to your IAM user (not the root account).

#### 3. Required IAM permissions
The account used for deployment needs permission to create and delete the following services. Follow the least-privilege principle and do not grant broader access than needed:

- **Amazon S3**: create bucket, configure, read/write objects
- **AWS Lambda**: create, update, invoke functions
- **Amazon API Gateway**: create and deploy APIs
- **Amazon DynamoDB**: create tables, read/write items
- **Amazon Rekognition**: `DetectLabels`, `DetectModerationLabels`
- **Amazon Textract**: `DetectDocumentText`
- **Amazon Polly**: `SynthesizeSpeech`
- **Amazon CloudFront**: create distributions
- **Amazon CloudWatch and CloudWatch Logs**: view logs, create alarms
- **AWS IAM**: create the Lambda execution role

The Lambda execution role is granted only the runtime actions it needs (detailed in section 5.5):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3FilesAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::insightshare-files-*/*"
    },
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:Query", "dynamodb:Scan"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/insightshare-files"
    },
    {
      "Sid": "AIServices",
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels",
        "rekognition:DetectModerationLabels",
        "textract:DetectDocumentText",
        "polly:SynthesizeSpeech"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Logging",
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:*:*"
    }
  ]
}
```
