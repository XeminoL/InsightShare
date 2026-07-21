---
title: "Prerequisite"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Prerequisites

Building and deploying InsightShare requires the following tools, accounts and permissions.

#### Step 1. Create the AWS account and secure it

- **Create an AWS account** at aws.amazon.com and complete billing/verification.
- **Enable MFA on the root account**, then stop using root for daily work.
- **Create a dedicated IAM user** for everyday work (not root), with programmatic access, and enable MFA on that user too.
- **Create an access key** for the IAM user under Security credentials.

![Console: IAM user created with MFA enabled](/images/5-Workshop/5.2-Prerequiste/iam-user-mfa.png)

_Screenshot: your AWS Console showing the IAM user with an MFA device enabled (screenshot to add)._

{{% notice warning %}}
**Security.** Each person uses **their own AWS account and their own credentials**. Never share or commit access keys or any credential. Keep keys in a local, gitignored folder (this project keeps them under `_private-keys/`) and never in the repository. If a key ever leaks, **rotate it immediately** in the IAM console and delete the old one.
{{% /notice %}}

#### Step 2. Configure the AWS CLI with the IAM user's keys

Configure the CLI with the IAM user's own access keys and the deployment region, then confirm the identity resolves to the IAM user (not root):

```bash
aws configure

aws sts get-caller-identity
```

#### Step 3. Developer Environment
- **Python 3** (3.10 or higher): the language used for the FastAPI local app and the Lambda function.
- **boto3**: the AWS SDK for Python (`pip install boto3`), used to generate presigned URLs and call S3, DynamoDB, Rekognition, Textract and Bedrock.
- **AWS CLI**: configured with `aws configure` (access key, region `ap-southeast-1`) to manage resources from the command line.
- **Node.js (v18 or higher)**: optional, to build the static frontend if you use a bundler.
- **Git**: to clone and manage the codebase.
- **Visual Studio Code** (or any editor): for backend and frontend development.

#### Step 4. Cloud Account & Region
Because InsightShare runs on AWS managed services, you need:
- **An AWS account** with permission to create and delete the resources used in this workshop.
- Deployment region: **Asia Pacific (Singapore), `ap-southeast-1`**.
- An **S3 bucket** for uploaded files, and a second use of S3 to host the static frontend.
- Access configured for the AWS CLI, verified with `aws sts get-caller-identity` pointing to your IAM user (not the root account).

#### Step 5. Required IAM permissions
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
