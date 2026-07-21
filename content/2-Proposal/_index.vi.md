---
title: "Bản đề xuất"
date: 2026-06-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# InsightShare
## Nền tảng chia sẻ ảnh & tài liệu thông minh trên AWS (serverless + AI)

### 1. Tóm tắt

InsightShare là một ứng dụng web để tải lên, phân tích và chia sẻ ảnh/tài liệu. Khi file được tải lên, các dịch vụ AI của AWS hiểu nội dung: gắn nhãn ảnh, trích văn bản từ tài liệu, và trả lời câu hỏi hoặc tóm tắt tài liệu bằng tiếng Việt, nhờ đó người dùng tìm file theo nội dung thay vì chỉ theo tên. Toàn bộ theo kiến trúc **serverless** trên AWS (region `ap-southeast-1`): không phải quản lý máy chủ, tính phí theo lượt gọi nên chi phí lúc nhàn rỗi nhỏ, tự mở rộng theo tải. Nền tảng dùng S3, Lambda, API Gateway, DynamoDB, CloudFront và Cognito cùng ba dịch vụ AI Rekognition, Textract và Bedrock (Claude). Amazon Cognito lo phần đăng nhập người dùng để mỗi người chỉ thấy file của chính mình, cô lập theo claim `sub` trong JWT.

### 2. Tuyên bố vấn đề

*Vấn đề*
- Người dùng cần một cách nhanh để lưu và chia sẻ ảnh/tài liệu, nhưng giải pháp tự dựng máy chủ (EC2 chạy 24/7) tốn chi phí cố định kể cả khi nhàn rỗi và phải tự vận hành, vá lỗi, mở rộng.
- Khi số lượng file tăng, việc tìm lại đúng file rất khó vì chỉ tìm được theo tên.
- Tìm được tài liệu rồi thì vẫn phải mở ra đọc mới biết bên trong nói gì.

*Giải pháp*
InsightShare tập trung dữ liệu và xử lý trên một stack serverless thống nhất:
- **Lưu trữ & chia sẻ:** S3 lưu file (bucket private), chia sẻ qua presigned URL có thời hạn; metadata lưu trong DynamoDB.
- **Xử lý nghiệp vụ:** Lambda + API Gateway sinh presigned URL, điều phối phân tích AI, ghi/đọc dữ liệu.
- **Hiểu nội dung bằng AI:** Rekognition gắn nhãn ảnh, Textract trích văn bản tài liệu, và Bedrock (một model Claude) trả lời câu hỏi và tóm tắt tài liệu bằng tiếng Việt. Tất cả đều là dịch vụ gọi sẵn, không huấn luyện mô hình.
- **Tìm kiếm thông minh:** nhãn và văn bản trích được lưu vào DynamoDB để tìm file theo nội dung.

*Lợi ích*
- **Chi phí thấp:** mô hình serverless trả theo lượng dùng; ở mức demo tổng chi phí dưới 1 USD/tháng.
- **Tin cậy & bảo mật:** file không public, IAM least-privilege, giám sát bằng CloudWatch.
- **Tìm kiếm theo nội dung:** nhãn AI và văn bản trích giúp tìm file theo nội dung bên trong, không chỉ theo tên.

### 3. Kiến trúc giải pháp

*Tổng quan*
Trình duyệt tải giao diện tĩnh từ **S3 + CloudFront (HTTPS)** → đăng nhập qua **Amazon Cognito** → gọi **API Gateway** → **Lambda (Python)**. API Gateway chạy một JWT authorizer kiểm tra token Cognito, và Lambda đọc claim `sub` để gán mỗi file cho đúng chủ sở hữu. Lambda sinh presigned URL để trình duyệt upload/download trực tiếp với **S3**. Sau khi upload, Lambda gọi các dịch vụ AI (**Rekognition / Textract / Bedrock**) và lưu kết quả vào **DynamoDB** phục vụ tìm kiếm. **CloudWatch** giám sát log/metric; **IAM** kiểm soát quyền theo least-privilege.

![Kiến trúc InsightShare](/images/2-Proposal/insightshare_architecture-v4.png)

*Dịch vụ AWS sử dụng*

| Dịch vụ | Vai trò |
|---|---|
| Amazon S3 | Lưu file người dùng; host giao diện web tĩnh |
| Amazon CloudFront | CDN phân phối web, HTTPS, tăng tốc |
| Amazon API Gateway | Cổng API công khai cho ứng dụng; một JWT authorizer kiểm tra token Cognito |
| Amazon Cognito | Đăng nhập người dùng (Hosted UI); claim `sub` trong JWT gán mỗi file cho đúng chủ sở hữu để cô lập theo người dùng |
| AWS Lambda | Xử lý nghiệp vụ (Python/boto3); một handler điều hướng request API Gateway HTTP API theo method và path |
| Amazon DynamoDB | Lưu metadata + nhãn AI + văn bản trích, phục vụ tìm kiếm |
| Amazon Rekognition | Gắn nhãn ảnh (DetectLabels) |
| Amazon Textract | Trích văn bản từ PDF/ảnh chữ (DetectDocumentText) |
| Amazon Bedrock (Claude) | Hỏi đáp và tóm tắt tài liệu bằng tiếng Việt (InvokeModel) |
| Amazon CloudWatch | Log, metric, alarm giám sát hệ thống |
| AWS IAM | Phân quyền least-privilege cho Lambda và từng dịch vụ AI |

*Thiết kế thành phần*
- **Frontend:** trang web tĩnh (HTML/JS) chọn file, hiển thị danh sách kèm nhãn AI, ô tìm kiếm theo nội dung, ô đặt câu hỏi về một tài liệu.
- **API:** các endpoint yêu cầu URL upload, xác nhận upload (kích hoạt phân tích AI), liệt kê/tìm kiếm file, lấy URL download, và hỏi đáp về một tài liệu.

### 4. Triển khai kỹ thuật

*Các giai đoạn triển khai*

| Giai đoạn | Nội dung |
|---|---|
| 1. Nền tảng & thiết kế | Học nền tảng AWS, chốt đề tài, vẽ kiến trúc, thiết lập tài khoản an toàn (MFA, IAM user, CLI, Budgets), thiết kế schema DynamoDB. |
| 2. Ứng dụng cơ bản | Dựng web app (FastAPI + frontend) chạy local, tách lớp lưu trữ và lớp AI. |
| 3. Đưa lên cloud | S3 + presigned URL, Lambda + API Gateway, tích hợp DynamoDB và link tải bằng presigned URL. |
| 4. Lớp AI & tìm kiếm | Tích hợp Rekognition/Textract/Bedrock, lưu kết quả vào DynamoDB, xây tìm kiếm thông minh. |
| 5. Hoàn thiện & vận hành | CloudFront + HTTPS, CloudWatch log/alarm, tối ưu chi phí/bảo mật, một script deploy/dọn dẹp lặp lại được. |

*Yêu cầu kỹ thuật*
- Tài khoản AWS (Free Tier), region `ap-southeast-1` (Singapore).
- Công cụ: AWS CLI, Python 3, boto3.
- Kiến thức: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch và các dịch vụ AI (Rekognition, Textract, Bedrock).

### 5. Ước tính ngân sách

Ở mức demo, mô hình serverless tính theo lượt dùng cùng Free Tier giữ chi phí dưới 1 USD/tháng. Một AWS Budget `InsightShare-Monthly` giới hạn 5 USD/tháng theo dõi chi tiêu. Số liệu chi tiết tính bằng [AWS Pricing Calculator](https://calculator.aws/).

Ở quy mô thật, chi phí tăng theo mức dùng, chủ yếu đến từ các dịch vụ AI (đánh đổi cho việc không phải huấn luyện hay tự vận hành mô hình). Ước tính hàng tháng cho **1.000 người dùng** (khoảng 20.000 lượt upload, mỗi lượt phân tích một lần, cùng duyệt và tìm kiếm):

| Dịch vụ | Cơ sở tính | Ước tính/tháng |
| --- | --- | --- |
| AWS Lambda | ~120k lượt gọi | ~0,10 USD |
| Amazon API Gateway | ~120k request HTTP | ~0,12 USD |
| Amazon S3 | ~40 GB lưu + request | ~1,00 USD |
| Amazon DynamoDB (on-demand) | ~140k đọc/ghi | ~0,20 USD |
| Amazon Rekognition | 20k ảnh (DetectLabels) | ~20,00 USD |
| Amazon Bedrock (Claude Haiku) | ~20k lượt tóm tắt + hỏi đáp | ~15,00 USD |
| Amazon CloudFront | ~30 GB ra | ~2,50 USD |
| **Tổng** | | **~40 USD/tháng** |

### 6. Đánh giá rủi ro

| Rủi ro | Tác động | Xác suất | Giảm thiểu |
|---|---|---|---|
| Cấu hình IAM/policy sai gây lỗi truy cập | Trung bình | Trung bình | Least-privilege, kiểm thử kỹ trước khi mở rộng quyền |
| Phát sinh chi phí ngoài dự kiến (gọi AI nhiều) | Thấp | Thấp | Budget Alert, giới hạn kích thước/loại file gửi AI, dọn tài nguyên sau test |
| File lớn gây timeout Lambda/API Gateway | Trung bình | Thấp | Presigned URL upload trực tiếp S3; gọi AI bất đồng bộ qua S3 event |
| Dịch vụ AI chậm hoặc kết quả chưa chính xác | Thấp | Trung bình | Phân tích AI bất đồng bộ, file vẫn tải được kể cả khi chưa phân tích xong |

*Kế hoạch dự phòng:* giữ một script dọn dẹp (cleanup-aws.ps1) để xóa toàn bộ tài nguyên nhanh chóng.

### 7. Kết quả kỳ vọng

*Cải tiến kỹ thuật*
- Ứng dụng web hoạt động end-to-end: upload → tự động phân tích nội dung bằng AI → liệt kê → tìm kiếm theo nội dung → hỏi đáp về một tài liệu → tải qua presigned link.
- Kiến trúc serverless kết hợp các dịch vụ AI managed trên AWS.
- Hỏi đáp tài liệu bằng Amazon Bedrock (Claude): endpoint `ask` nhận một tài liệu và một câu hỏi, được nối vào lệnh gọi `bedrock:InvokeModel` với model id theo inference-profile cùng phần xử lý request/response đầy đủ.

*Giá trị dài hạn*
- Nền tảng có thể mở rộng: thêm đăng nhập người dùng (Cognito), điều phối pipeline AI nhiều bước (Step Functions), hỗ trợ thêm loại file.
- Tài liệu workshop chi tiết để người khác có thể làm theo và phát triển tiếp.
