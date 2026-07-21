---
title: "Prerequisite"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Prerequisites

Building and deploying InsightShare requires the following tools, accounts and permissions.

#### Step 1. AWS account and credentials

Every command and deploy in the workshop runs against one AWS account in one region, so the identity and region are fixed before anything else. Work from a dedicated IAM user with MFA, not the root account, so a leaked key cannot touch billing or delete the account. Configure the CLI to `ap-southeast-1`, the region all InsightShare resources live in, and keep access keys out of the repository (this project gitignores them under `_private-keys/`).

![Console: IAM user with MFA enabled](/images/5-Workshop/5.2-Prerequiste/iam-user-mfa.png)

Configure the CLI, then confirm the identity resolves to the IAM user. `aws sts get-caller-identity` returns the ARN the CLI is acting as, which must be the IAM user and not the root account:

```bash
aws configure

aws sts get-caller-identity
```

#### Step 2. Developer Environment
- **Python 3** (3.10 or higher): the language used for the FastAPI local app and the Lambda function.
- **boto3**: the AWS SDK for Python (`pip install boto3`), used to generate presigned URLs and call S3, DynamoDB, Rekognition, Textract and Bedrock.
- **AWS CLI**: configured with `aws configure` (access key, region `ap-southeast-1`) to manage resources from the command line.
- **Node.js (v18 or higher)**: optional, to build the static frontend if you use a bundler.
- **Git**: to clone and manage the codebase.
- **Visual Studio Code** (or any editor): for backend and frontend development.

#### Step 3. Cloud Account & Region
- **An AWS account** with permission to create and delete the resources used in this workshop.
- Deployment region: **Asia Pacific (Singapore), `ap-southeast-1`**.
- An **S3 bucket** for uploaded files, and a second use of S3 to host the static frontend.
- Access configured for the AWS CLI, verified with `aws sts get-caller-identity` pointing to your IAM user (not the root account).

#### Step 4. Required IAM permissions
The account used for deployment needs permission to create and delete the following services. Follow the least-privilege principle and do not grant broader access than needed:

- **Amazon S3**: create bucket, configure, read/write objects
- **AWS Lambda**: create, update, invoke functions
- **Amazon API Gateway**: create and deploy APIs
- **Amazon DynamoDB**: create tables, read/write items
- **Amazon Rekognition**: `DetectLabels`
- **Amazon Textract**: `DetectDocumentText`
- **Amazon Bedrock**: `InvokeModel` (a Claude model)
- **Amazon CloudFront**: create distributions
- **Amazon CloudWatch and CloudWatch Logs**: view logs, create alarms
- **AWS IAM**: create the Lambda execution role

The account permissions above are for the person deploying the stack. Separately, the Lambda function assumes its own execution role at runtime, granted only the actions the code calls. The two are distinct: the deployer creates resources, the role lets the running function reach them:

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
        "textract:DetectDocumentText",
        "bedrock:InvokeModel"
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
