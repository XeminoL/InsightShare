---
title: "Dọn dẹp tài nguyên"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Dọn dẹp tài nguyên

Xóa toàn bộ tài nguyên đã tạo trong workshop để dừng phát sinh chi phí (region `ap-southeast-1`). Việc dọn dẹp được script hóa trong `cleanup-aws.ps1`, xóa mọi tài nguyên trong một lần chạy (có cờ `-Force` để bỏ qua bước hỏi xác nhận). Script xóa lần lượt: API Gateway HTTP API, hàm Lambda, bảng DynamoDB, cả hai bucket S3 (files và web), CloudFront distribution, các alarm/dashboard/log group của CloudWatch, và IAM role. Thứ tự quan trọng ở chỗ tài nguyên phụ thuộc nhau: IAM role xóa cuối vì Lambda dùng nó, và mỗi bucket S3 phải được làm rỗng (`s3 rm --recursive`) trước khi xóa, vì S3 từ chối xóa bucket chưa rỗng. Các lệnh CLI tương đương:

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
