---
title: "Dọn dẹp tài nguyên"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên

Các tài nguyên này vẫn phát sinh chi phí, nên xóa chúng sau khi đồ án đã được chấm. Chạy `cleanup-aws.ps1 -Force` để xóa toàn bộ, hoặc dùng các lệnh dưới đây.

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
