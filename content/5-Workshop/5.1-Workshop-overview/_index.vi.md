---
title: "Tổng quan"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu InsightShare

**InsightShare** là một ứng dụng web serverless để tải lên, phân tích và chia sẻ ảnh và tài liệu trên AWS. Mỗi lượt upload nó gọi các dịch vụ AI của AWS rồi lưu kết quả cạnh metadata của file, còn việc chia sẻ đi qua link có thời hạn.

Nền tảng được xây trên:
- **Amazon S3 + presigned URL**: lưu trữ file riêng tư; trình duyệt tải lên và tải xuống trực tiếp qua link ký có thời hạn ngắn.
- **AWS Lambda + Amazon API Gateway**: back-end Python không phải quản lý máy chủ, mở ra thành API HTTP.
- **Amazon Cognito**: đăng nhập người dùng, cô lập theo claim `sub` trong JWT để mỗi người chỉ thấy file của mình.
- **Amazon DynamoDB**: metadata của file cùng nhãn AI và văn bản trích, làm nền cho tìm kiếm theo nội dung.
- **Amazon Rekognition, Textract và Bedrock (Claude)**: lớp AI gắn nhãn ảnh, trích văn bản tài liệu, và trả lời câu hỏi hoặc tóm tắt tài liệu theo ngôn ngữ câu hỏi.
- **Amazon CloudFront + CloudWatch + IAM**: phân phối frontend tĩnh qua HTTPS, giám sát và kiểm soát quyền theo nguyên tắc tối thiểu.

#### Tổng quan kiến trúc

Các bước đánh số khớp với các mũi tên trong sơ đồ kiến trúc, theo thứ tự:

1. **User → CloudFront**: trình duyệt tải giao diện web tĩnh từ S3, phân phối qua HTTPS bằng CloudFront.
2. **User → API Gateway → Lambda**: trình duyệt gọi API Gateway, cổng này chuyển yêu cầu đến Lambda (Python) để xử lý nghiệp vụ.
3. **Lambda → URL đã ký**: Lambda trả về presigned URL, trình duyệt dùng nó để truyền file trực tiếp với S3.
4. **Lambda → dịch vụ AI**: Lambda gọi Rekognition để gắn nhãn ảnh hoặc Textract để trích văn bản tài liệu, còn endpoint `ask` gửi văn bản đã lưu tới Bedrock (Claude) để trả lời câu hỏi hoặc tóm tắt tài liệu theo ngôn ngữ câu hỏi.
5. **Lambda → DynamoDB**: metadata của file, nhãn AI và văn bản trích được ghi vào DynamoDB, làm nền cho tìm kiếm theo nội dung.
6. **Giám sát và bảo mật**: CloudWatch thu log và số liệu; IAM role cấp quyền tối thiểu cho từng dịch vụ.

![Kiến trúc InsightShare](/images/5-Workshop/5.1-Workshop-overview/insightshare_architecture-v6.png)

#### Các dịch vụ AWS sử dụng

| Dịch vụ | Vai trò trong InsightShare | Lý do lựa chọn |
|---|---|---|
| Amazon S3 | Lưu file người dùng tải lên và host web tĩnh | Bền vững, chi phí thấp, hỗ trợ presigned URL nên trình duyệt truyền file trực tiếp không qua Lambda |
| Amazon CloudFront | CDN phân phối giao diện web tĩnh qua HTTPS | Phân phối nhanh trên toàn cầu và có HTTPS cho frontend mà không cần quản lý web server |
| Amazon API Gateway | Cổng API công khai để frontend gọi back-end; một JWT authorizer kiểm tra token Cognito | Endpoint HTTP được quản lý sẵn, có throttling và CORS, không phải chạy server |
| Amazon Cognito | Đăng nhập người dùng; cấp JWT có claim `sub` để gán file theo từng người | Danh bạ người dùng được quản lý sẵn, gọi được thẳng từ trang tĩnh, không phải tự dựng server xác thực |
| AWS Lambda | Xử lý nghiệp vụ back-end bằng Python | Không phải quản lý server, trả tiền theo lượt gọi, tự co giãn theo tải |
| Amazon DynamoDB | Lưu metadata của file kèm nhãn AI và văn bản trích xuất | NoSQL serverless đọc mili-giây, hợp với dạng metadata theo từng file và nhu cầu tìm kiếm |
| Amazon Rekognition | Gắn nhãn ảnh | Trả về nhãn chỉ với một lời gọi API, không có bước huấn luyện |
| Amazon Textract | Trích văn bản từ PDF và ảnh scan | OCR gọi qua API, biến tài liệu thành văn bản tìm kiếm được |
| Amazon Bedrock (Claude) | Trả lời câu hỏi và tóm tắt tài liệu theo ngôn ngữ câu hỏi | Model Claude host sẵn, không train; biến văn bản đã trích thành câu trả lời trực tiếp chỉ với một lời gọi API |
| Amazon CloudWatch | Ghi log, số liệu và cảnh báo | Giám sát tập trung cho Lambda và API Gateway để gỡ lỗi và theo dõi chi phí/mức dùng |
| AWS IAM | Kiểm soát quyền theo nguyên tắc tối thiểu | Cấp cho mỗi thành phần đúng quyền cần thiết, giữ file không công khai |
