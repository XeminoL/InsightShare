---
title: "Dọn dẹp tài nguyên"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên

Xóa toàn bộ tài nguyên đã tạo trong workshop để dừng phát sinh chi phí (region `ap-southeast-1`). Bằng CLI:

```bash
# 1) API Gateway
aws apigatewayv2 delete-api --api-id <api-id>

# 2) Lambda function
aws lambda delete-function --function-name insightshare-api

# 3) Bảng DynamoDB
aws dynamodb delete-table --table-name insightshare-files

# 4) S3: làm rỗng rồi xóa bucket
aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464

# 5) IAM: xóa inline policy, rồi xóa role
aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role

# 6) CloudWatch: xóa alarm và (tùy chọn) log group
aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api
```
