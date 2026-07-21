---
title: "Giám sát & Bảo mật (CloudWatch + IAM)"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Tổng quan

Hai mảng cuối: **giám sát** (CloudWatch) và **bảo mật** (IAM least-privilege).

#### Bước 1: Giám sát với CloudWatch

- **CloudWatch Logs**: Lambda tự động ghi vào log group `/aws/lambda/insightshare-api`. Đây chính là nơi các lỗi runtime lúc phát triển hiện ra (các lỗi `Decimal`, presigned URL và IAM đều được chẩn đoán từ log này).
- **CloudWatch Metrics**: Lambda phát Invocations, Errors, Duration; API Gateway phát request count và số 4xx/5xx.
- **CloudWatch Alarm**: hai alarm được tạo trên function `insightshare-api`. `insightshare-lambda-errors` kích hoạt khi metric `Errors` của Lambda chạm ngưỡng; `insightshare-lambda-throttles` kích hoạt khi function bị throttle.

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

![Console: các alarm CloudWatch đã tạo](/images/5-Workshop/5.5-Policy/cloudwatch-alarms.png)

_Ảnh chụp Console của bạn cho thấy hai alarm `insightshare-lambda-errors` và `insightshare-lambda-throttles` (ảnh cần bổ sung)._

- **CloudWatch Dashboard**: dashboard `insightshare-monitoring` gom các khung theo dõi vào một chỗ. Nó có ba widget: Lambda invocations/errors, Lambda duration, và request count của API Gateway.

```bash
aws cloudwatch put-dashboard \
  --dashboard-name insightshare-monitoring \
  --dashboard-body file://dashboard.json
```

![Console: dashboard giám sát CloudWatch](/images/5-Workshop/5.5-Policy/cloudwatch-dashboard.png)

_Ảnh chụp Console của bạn cho thấy các widget của dashboard `insightshare-monitoring` (ảnh cần bổ sung)._

#### Bước 2: Bảo mật với IAM (least-privilege)

Lambda dùng một execution role riêng least-privilege, `insightshare-lambda-role`, được xác nhận đang hoạt động (mục "Last activity" cập nhật mỗi khi function chạy):

![Execution role IAM](/images/5-Workshop/5.5-Policy/iam-role.png)

Policy gắn kèm chỉ cấp đúng những gì từng dịch vụ cần. S3 và DynamoDB được giới hạn theo ARN bucket và bảng cụ thể (không dùng `"Resource": "*"`); các action AI dùng `"*"` vì Rekognition, Textract và Bedrock không hỗ trợ phân quyền theo tài nguyên (Bedrock có thể giới hạn tùy chọn theo ARN của foundation model Claude):

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
Tập quyền được tinh chỉnh trong lúc test thật: `dynamodb:UpdateItem` và `s3:ListBucket` được thêm sau khi `analyze` báo `AccessDeniedException`. Đây là least-privilege trong thực tế: bắt đầu hẹp, rồi cấp đúng action còn thiếu thay vì mở quyền rộng.
{{% /notice %}}
