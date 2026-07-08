---
title: "Dọn dẹp tài nguyên"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên

Xóa toàn bộ tài nguyên đã tạo trong workshop để dừng phát sinh chi phí (region `ap-southeast-1`). Việc dọn dẹp được script hóa trong `cleanup-aws.ps1`, xóa mọi tài nguyên trong một lần chạy (có cờ `-Force` để bỏ qua bước hỏi xác nhận). Script xóa lần lượt: API Gateway HTTP API, hàm Lambda, bảng DynamoDB, cả hai bucket S3 (files và web), CloudFront distribution, các alarm/dashboard/log group của CloudWatch, và IAM role. Các lệnh CLI tương đương:

```bash
# 1) API Gateway
aws apigatewayv2 delete-api --api-id <api-id>

# 2) Lambda function
aws lambda delete-function --function-name insightshare-api

# 3) Bảng DynamoDB
aws dynamodb delete-table --table-name insightshare-files

# 4) S3: làm rỗng rồi xóa CẢ HAI bucket (files + web frontend)
aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464
aws s3 rm s3://insightshare-web-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-web-khang-2352464

# 5) CloudFront: phải DISABLE distribution trước (Enabled=false),
#    đợi trạng thái Deployed rồi mới xóa được
#    aws cloudfront get-distribution-config --id <dist-id>   # sửa Enabled=false
#    aws cloudfront update-distribution --id <dist-id> ...    # rồi đợi
aws cloudfront delete-distribution --id <dist-id> --if-match <etag>

# 6) CloudWatch: xóa CẢ HAI alarm, dashboard và log group
aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors insightshare-lambda-throttles
aws cloudwatch delete-dashboards --dashboard-names insightshare-monitoring
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api

# 7) IAM: xóa inline policy, rồi xóa role
aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role
```
