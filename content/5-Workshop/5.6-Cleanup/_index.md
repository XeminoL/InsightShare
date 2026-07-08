---
title: "Clean up"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Clean up resources

Delete everything created in the workshop to stop any further charges (region `ap-southeast-1`). The teardown is scripted in `cleanup-aws.ps1`, which removes every resource in one run (with a `-Force` flag to skip the confirmation prompt). It deletes, in order: the API Gateway HTTP API, the Lambda function, the DynamoDB table, both S3 buckets (files and web), the CloudFront distribution, the CloudWatch alarms/dashboard/log group, and the IAM role. The equivalent CLI commands:

```bash
# 1) API Gateway
aws apigatewayv2 delete-api --api-id <api-id>

# 2) Lambda function
aws lambda delete-function --function-name insightshare-api

# 3) DynamoDB table
aws dynamodb delete-table --table-name insightshare-files

# 4) S3: empty then delete BOTH buckets (files + web frontend)
aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464
aws s3 rm s3://insightshare-web-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-web-khang-2352464

# 5) CloudFront: a distribution must be DISABLED first (Enabled=false),
#    then deleted once its status is Deployed
#    aws cloudfront get-distribution-config --id <dist-id>   # edit Enabled=false
#    aws cloudfront update-distribution --id <dist-id> ...    # then wait
aws cloudfront delete-distribution --id <dist-id> --if-match <etag>

# 6) CloudWatch: delete BOTH alarms, the dashboard, and the log group
aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors insightshare-lambda-throttles
aws cloudwatch delete-dashboards --dashboard-names insightshare-monitoring
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api

# 7) IAM: delete the inline policy, then the role
aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role
```
