---
title: "Clean up"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Clean up resources

These resources keep incurring charges, so delete them once the project has been graded. Run `cleanup-aws.ps1 -Force` to remove everything, or use the commands below.

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
