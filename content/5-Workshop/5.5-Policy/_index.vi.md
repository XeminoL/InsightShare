---
title: "Giám sát & Bảo mật"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Tổng quan

Hai mảng cuối: **giám sát** và **bảo mật**.

#### Bước 1: Giám sát với CloudWatch

- **CloudWatch Logs**: Lambda tự động ghi vào log group `/aws/lambda/insightshare-api`. Đây chính là nơi các lỗi runtime lúc phát triển hiện ra.
- **CloudWatch Metrics**: Lambda phát Invocations, Errors, Duration; API Gateway phát request count và số 4xx/5xx.
- **CloudWatch Alarm**: hai alarm theo dõi function `insightshare-api` để phát hiện sự cố mà không phải đọc log. `insightshare-lambda-errors` bật khi metric `Errors` chạm ngưỡng: `--threshold 1` với `--period 300` nghĩa là một lần gọi lỗi trong năm phút là alarm kêu. Nó bắt lỗi code và lỗi quyền. `insightshare-lambda-throttles` bật trên metric `Throttles`, bắt lúc chạm giới hạn concurrency khi tải cao.

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

- **CloudWatch Dashboard**: dashboard `insightshare-monitoring` gom các khung theo dõi vào một chỗ. Nó có ba widget: Lambda invocations/errors, Lambda duration, và request count của API Gateway.

```bash
aws cloudwatch put-dashboard \
  --dashboard-name insightshare-monitoring \
  --dashboard-body file://dashboard.json
```

![Console: dashboard CloudWatch, thấy hai trong ba widget](/images/5-Workshop/5.5-Policy/cloudwatch-dashboard.png)

#### Bước 2: Bảo mật với IAM

Lambda dùng một execution role riêng least-privilege, `insightshare-lambda-role`. Mục "Last activity" cập nhật mỗi khi function chạy:

![Execution role IAM](/images/5-Workshop/5.5-Policy/iam-role.png)

Policy gắn kèm chỉ cấp đúng những gì từng dịch vụ cần. S3 và DynamoDB được giới hạn theo ARN bucket và bảng cụ thể; các action AI dùng `"*"` vì Rekognition, Textract và Bedrock không hỗ trợ phân quyền theo tài nguyên:

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

{{% notice info %}}
Đăng nhập Cognito không cần thêm quyền nào cho role Lambda này. JWT authorizer trên API Gateway kiểm tra token và truyền claim `sub` vào hàm, nên việc gán dữ liệu theo người dùng chạy trên đúng các quyền DynamoDB ở trên.
{{% /notice %}}