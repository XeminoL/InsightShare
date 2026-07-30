---
title: "Dọn dẹp tài nguyên"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Xóa những gì đã tạo

Các tài nguyên này vẫn phát sinh chi phí, nên xóa chúng sau khi đồ án đã được chấm. `cleanup-aws.ps1 -Force` chạy mọi thứ bên dưới trừ CloudFront distribution, vì cái đó phải disable trước.

```bash
aws apigatewayv2 delete-api --api-id <api-id>

aws lambda delete-function --function-name insightshare-api

aws dynamodb delete-table --table-name insightshare-files

aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464
aws s3 rm s3://insightshare-web-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-web-khang-2352464

aws cognito-idp delete-user-pool-client --user-pool-id <user-pool-id> --client-id <app-client-id>
aws cognito-idp delete-user-pool --user-pool-id <user-pool-id>

aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors insightshare-lambda-throttles
aws cloudwatch delete-dashboards --dashboard-names insightshare-monitoring
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api

aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role
```

Cognito user pool giữ các tài khoản đã đăng ký, nên xóa nó là xóa luôn phần dữ liệu đó cùng với cả stack.

Bucket file bật versioning, nên `aws s3 rm --recursive` vẫn để lại các version cũ và delete marker, rồi `delete-bucket` báo `BucketNotEmpty`. `cleanup-aws.ps1` xóa hết version và delete marker trước, nên dùng script đỡ việc hơn ở chỗ này.

{{% notice warning %}}
**CloudFront cần hai bước.** Không xóa được distribution khi nó còn enabled: đặt `Enabled` thành `false`, đợi trạng thái quay lại `Deployed` (khoảng 15 phút), rồi xóa bằng ETag hiện tại. Đây là tài nguyên duy nhất script chỉ báo chứ không tự xóa.
{{% /notice %}}

```bash
aws cloudfront get-distribution-config --id <dist-id> > dist.json
# giữ lại đúng object DistributionConfig trong dist.json, đặt "Enabled": false,
# lưu thành config.json, rồi:
aws cloudfront update-distribution --id <dist-id> \
  --distribution-config file://config.json --if-match <etag>
# đợi tới trạng thái Deployed, rồi:
aws cloudfront delete-distribution --id <dist-id> --if-match <new-etag>
```
