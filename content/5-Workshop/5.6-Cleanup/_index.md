---
title: "Clean up"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Clean up resources

Delete everything created in the workshop to stop any further charges (region `ap-southeast-1`). The teardown is scripted in `cleanup-aws.ps1`, which removes every resource in one run (with a `-Force` flag to skip the confirmation prompt). It deletes, in order: the API Gateway HTTP API, the Lambda function, the DynamoDB table, both S3 buckets (files and web), the CloudFront distribution, the CloudWatch alarms/dashboard/log group, and the IAM role. The order matters where resources depend on each other: the IAM role is removed last because the Lambda uses it, and each S3 bucket is emptied (`s3 rm --recursive`) before it can be deleted, since S3 refuses to delete a non-empty bucket. The equivalent CLI commands:

```bash
aws apigatewayv2 delete-api --api-id <api-id>

aws lambda delete-function --function-name insightshare-api

aws dynamodb delete-table --table-name insightshare-files

aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464
aws s3 rm s3://insightshare-web-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-web-khang-2352464

aws cloudfront delete-distribution --id <dist-id> --if-match <etag>

aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors insightshare-lambda-throttles
aws cloudwatch delete-dashboards --dashboard-names insightshare-monitoring
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api

aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role
```
