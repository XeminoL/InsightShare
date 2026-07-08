---
title: "Chuẩn bị"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Chuẩn bị

Việc xây dựng và triển khai InsightShare cần các công cụ, tài khoản và quyền sau.

#### 1. Môi trường phát triển
- **Python 3** (3.10 trở lên): ngôn ngữ dùng cho app FastAPI chạy local và cho Lambda function.
- **boto3**: thư viện AWS SDK cho Python (`pip install boto3`), dùng để sinh presigned URL và gọi S3, DynamoDB, Rekognition, Textract, Bedrock.
- **AWS CLI**: đã cấu hình bằng `aws configure` (access key, region `ap-southeast-1`) để thao tác tài nguyên từ dòng lệnh.
- **Node.js (v18 trở lên)**: tùy chọn, để build frontend tĩnh nếu dùng bundler.
- **Git**: để clone và quản lý mã nguồn.
- **Visual Studio Code** (hoặc trình soạn thảo bất kỳ): để phát triển back-end và frontend.

#### 2. Tài khoản Cloud & Region
Vì InsightShare chạy trên các dịch vụ được quản lý của AWS, bạn cần:
- **Một tài khoản AWS** có quyền tạo và xóa các tài nguyên dùng trong workshop.
- Region triển khai: **Asia Pacific (Singapore), `ap-southeast-1`**.
- Một **S3 bucket** cho file người dùng tải lên, cùng một bucket S3 để host frontend tĩnh.
- Đã cấu hình AWS CLI, kiểm tra bằng `aws sts get-caller-identity` trỏ đúng IAM user (không dùng tài khoản root).

#### 3. Quyền IAM cần thiết
Tài khoản dùng để triển khai cần quyền tạo và xóa các dịch vụ sau. Tuân theo nguyên tắc tối thiểu quyền (least-privilege), không cấp quyền rộng hơn mức cần:

- **Amazon S3**: tạo bucket, cấu hình, đọc/ghi object
- **AWS Lambda**: tạo, cập nhật, gọi function
- **Amazon API Gateway**: tạo và deploy API
- **Amazon DynamoDB**: tạo bảng, đọc/ghi item
- **Amazon Rekognition**: `DetectLabels`
- **Amazon Textract**: `DetectDocumentText`
- **Amazon Bedrock**: `InvokeModel` (một model Claude)
- **Amazon CloudFront**: tạo distribution
- **Amazon CloudWatch và CloudWatch Logs**: xem log, tạo alarm
- **AWS IAM**: tạo execution role cho Lambda

Execution role của Lambda chỉ được cấp đúng các action cần khi chạy (chi tiết ở phần 5.5):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3FilesAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::insightshare-files-*/*"
    },
    {
      "Sid": "DynamoDBAccess",
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:Query", "dynamodb:Scan"],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/insightshare-files"
    },
    {
      "Sid": "AIServices",
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels",
        "textract:DetectDocumentText",
        "bedrock:InvokeModel"
      ],
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
