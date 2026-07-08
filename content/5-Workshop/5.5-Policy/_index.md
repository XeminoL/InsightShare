---
title: "Monitoring & Security (CloudWatch + IAM)"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Overview

Two final pieces: **monitoring** (CloudWatch) and **security** (IAM least-privilege).

#### Monitoring with CloudWatch

- **CloudWatch Logs**: the Lambda automatically writes to the log group `/aws/lambda/insightshare-api`. This is where the runtime errors during development showed up (the `Decimal`, presigned-URL and IAM issues were all diagnosed from these logs).
- **CloudWatch Metrics**: Lambda emits Invocations, Errors and Duration; API Gateway emits request count and 4xx/5xx counts.
- **CloudWatch Alarms**: two alarms are created on the `insightshare-api` function. `insightshare-lambda-errors` fires when the Lambda `Errors` metric reaches the threshold; `insightshare-lambda-throttles` fires when the function is throttled.

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name insightshare-lambda-errors \
  --namespace AWS/Lambda --metric-name Errors \
  --dimensions Name=FunctionName,Value=insightshare-api \
  --statistic Sum --period 300 --evaluation-periods 1 \
  --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold

aws cloudwatch put-metric-alarm \
  --alarm-name insightshare-lambda-throttles \
  --namespace AWS/Lambda --metric-name Throttles \
  --dimensions Name=FunctionName,Value=insightshare-api \
  --statistic Sum --period 300 --evaluation-periods 1 \
  --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold
```

- **CloudWatch Dashboard**: a dashboard `insightshare-monitoring` collects the operational views in one place. It has three widgets: Lambda invocations/errors, Lambda duration, and API Gateway request count.

```bash
aws cloudwatch put-dashboard \
  --dashboard-name insightshare-monitoring \
  --dashboard-body file://dashboard.json
```

#### Security with IAM (least-privilege)

The Lambda uses a dedicated least-privilege execution role, `insightshare-lambda-role`, confirmed active (its "Last activity" updates whenever the function runs):

![IAM execution role](/images/5-Workshop/5.5-Policy/iam-role.png)

The attached policy grants only what each service needs. S3 and DynamoDB are scoped to the specific bucket and table ARNs (not `"Resource": "*"`); the AI actions use `"*"` because Rekognition, Textract and Bedrock do not support resource-level permissions (Bedrock can optionally be scoped to the Claude foundation-model ARN):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ObjectAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::insightshare-files-khang-2352464/*"
    },
    {
      "Sid": "S3ListBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::insightshare-files-khang-2352464"
    },
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:UpdateItem",
                 "dynamodb:Query", "dynamodb:Scan", "dynamodb:DeleteItem"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/insightshare-files"
    },
    {
      "Sid": "AIServices",
      "Effect": "Allow",
      "Action": ["rekognition:DetectLabels",
                 "textract:DetectDocumentText", "bedrock:InvokeModel"],
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

{{% notice note %}}
The permission set was tuned during real testing: `dynamodb:UpdateItem` and `s3:ListBucket` were added after `analyze` failed with `AccessDeniedException`. This is least-privilege in practice: start narrow, then grant exactly the missing action rather than opening broad access.
{{% /notice %}}

#### Summary

InsightShare is monitored with CloudWatch (the `/aws/lambda/insightshare-api` log group, the `insightshare-lambda-errors` and `insightshare-lambda-throttles` alarms, and the `insightshare-monitoring` dashboard) and secured with an IAM role scoped to exactly the resources it needs.
