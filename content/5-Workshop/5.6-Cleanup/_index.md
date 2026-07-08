---
title: "Clean up"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Clean up resources

Delete everything created in the workshop to stop any further charges (region `ap-southeast-1`). With the CLI:

```bash
# 1) API Gateway
aws apigatewayv2 delete-api --api-id <api-id>

# 2) Lambda function
aws lambda delete-function --function-name insightshare-api

# 3) DynamoDB table
aws dynamodb delete-table --table-name insightshare-files

# 4) S3: empty then delete the bucket(s)
aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464

# 5) IAM: delete the inline policy, then the role
aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role

# 6) CloudWatch: delete the alarm and (optional) the log group
aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api
```
