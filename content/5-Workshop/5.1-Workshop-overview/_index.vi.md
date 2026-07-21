---
title: "Tổng quan"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu InsightShare

**InsightShare** là một ứng dụng web serverless để tải lên, phân tích và chia sẻ ảnh và tài liệu trên AWS. Nó dùng các dịch vụ AI gọi sẵn của AWS để hiểu nội dung mỗi file khi tải lên, nhờ vậy người dùng tìm file theo nội dung và chia sẻ qua link có thời hạn.

Nền tảng được xây trên:
- **Amazon S3 + presigned URL**: lưu trữ file riêng tư; trình duyệt tải lên/tải xuống trực tiếp qua link ký có thời hạn ngắn.
- **AWS Lambda + Amazon API Gateway**: back-end Python không phải quản lý máy chủ, expose thành API HTTP.
- **Amazon Cognito**: đăng nhập người dùng qua Hosted UI, để mỗi người chỉ thấy file của mình, cô lập theo claim `sub` trong JWT.
- **Amazon DynamoDB**: metadata của file cùng nhãn AI và văn bản trích, phục vụ tìm kiếm theo nội dung.
- **Amazon Rekognition, Textract và Bedrock (Claude)**: lớp AI gắn nhãn ảnh, trích văn bản tài liệu, và trả lời câu hỏi hoặc tóm tắt tài liệu bằng tiếng Việt, không cần train model.
- **Amazon CloudFront + CloudWatch + IAM**: phân phối frontend tĩnh qua HTTPS, giám sát và kiểm soát quyền theo nguyên tắc tối thiểu.

#### Tổng quan workshop

InsightShare được xây dựng từ đầu tới cuối gồm các phần sau:

- Tầng lưu trữ file an toàn trên **Amazon S3** với **presigned URL**.
- Back-end trên **AWS Lambda** (Python) sau **Amazon API Gateway**, metadata lưu trong **Amazon DynamoDB**.
- **Lớp AI** đọc nội dung file tự động: gắn nhãn ảnh bằng **Amazon Rekognition**, trích văn bản bằng **Amazon Textract**, và hỏi đáp / tóm tắt tài liệu bằng tiếng Việt bằng **Amazon Bedrock** (Claude).
- **Tìm kiếm theo nội dung** dựa trên nhãn ảnh và văn bản trích từ tài liệu.
- Frontend tĩnh trên **Amazon S3 + CloudFront** (HTTPS).
- Giám sát và bảo mật bằng **Amazon CloudWatch** và **IAM Role** theo nguyên tắc tối thiểu quyền.

#### Tổng quan kiến trúc

InsightShare hoạt động theo luồng serverless hoàn toàn. Các bước đánh số khớp với các mũi tên trong sơ đồ kiến trúc, theo thứ tự:

1. **User → CloudFront**: trình duyệt tải giao diện web tĩnh từ **Amazon S3**, phân phối qua HTTPS bằng **Amazon CloudFront**.
2. **User → API Gateway → Lambda**: trình duyệt gọi **Amazon API Gateway**, cổng này chuyển yêu cầu đến **AWS Lambda** (Python) để xử lý nghiệp vụ.
3. **Lambda → S3 (presigned URL)**: Lambda trả về **presigned URL** để trình duyệt tải file trực tiếp lên **Amazon S3**.
4. **Lambda → dịch vụ AI**: sau khi upload, Lambda gọi lớp AI, **Rekognition** gắn nhãn ảnh và **Textract** trích văn bản tài liệu, và endpoint `ask` gửi văn bản đã lưu tới **Amazon Bedrock** (Claude) để trả lời câu hỏi hoặc tóm tắt tài liệu bằng tiếng Việt.
5. **Lambda → DynamoDB**: metadata của file, nhãn AI và văn bản trích được ghi vào **Amazon DynamoDB**, phục vụ tìm kiếm theo nội dung.
6. **Giám sát & bảo mật**: **Amazon CloudWatch** thu thập log và số liệu; **IAM Role** cấp quyền tối thiểu cho từng dịch vụ.

![Kiến trúc InsightShare](/images/5-Workshop/5.1-Workshop-overview/insightshare_architecture-v3.png)

#### Các dịch vụ AWS sử dụng

| Dịch vụ | Vai trò trong InsightShare | Lý do lựa chọn |
|---|---|---|
| Amazon S3 | Lưu file người dùng tải lên và host web tĩnh | Bền vững, chi phí thấp, hỗ trợ presigned URL nên trình duyệt truyền file trực tiếp không qua Lambda |
| Amazon CloudFront | CDN phân phối giao diện web tĩnh qua HTTPS | Phân phối nhanh trên toàn cầu và có HTTPS cho frontend mà không cần quản lý web server |
| Amazon API Gateway | Cổng API công khai để frontend gọi back-end; một JWT authorizer kiểm tra token Cognito | Endpoint HTTP được quản lý sẵn, có throttling và CORS, không phải chạy server |
| Amazon Cognito | Đăng nhập người dùng qua Hosted UI; cấp JWT có claim `sub` để gán file theo từng người | User directory và đăng nhập OAuth2 được quản lý sẵn, không phải tự dựng server xác thực |
| AWS Lambda | Xử lý nghiệp vụ back-end bằng Python | Không phải quản lý server, trả tiền theo lượt gọi, tự co giãn theo tải |
| Amazon DynamoDB | Lưu metadata của file kèm nhãn AI và văn bản trích xuất | NoSQL serverless đọc mili-giây, hợp với dạng metadata theo từng file và nhu cầu tìm kiếm |
| Amazon Rekognition | Gắn nhãn ảnh | AI gọi sẵn, không cần train model, trả về nhãn chỉ với một lời gọi API |
| Amazon Textract | Trích văn bản từ PDF và ảnh chữ | OCR gọi sẵn, giúp tài liệu tìm kiếm được theo nội dung |
| Amazon Bedrock (Claude) | Trả lời câu hỏi và tóm tắt tài liệu bằng tiếng Việt | Model Claude host sẵn, không train; biến văn bản đã trích thành câu trả lời trực tiếp chỉ với một lời gọi API |
| Amazon CloudWatch | Ghi log, số liệu và cảnh báo | Giám sát tập trung cho Lambda và API Gateway để gỡ lỗi và theo dõi chi phí/mức dùng |
| AWS IAM | Kiểm soát quyền theo nguyên tắc tối thiểu | Cấp cho mỗi thành phần đúng quyền cần thiết, giữ file không công khai |
