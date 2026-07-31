---
title: "Bản đề xuất"
date: 2026-06-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# InsightShare
## Nền tảng chia sẻ ảnh & tài liệu thông minh trên AWS

### 1. Tóm tắt

InsightShare là một ứng dụng web để tải lên, phân tích và chia sẻ ảnh/tài liệu. Sau khi file được tải lên, các dịch vụ AI của AWS gắn nhãn ảnh, trích văn bản từ tài liệu, và trả lời câu hỏi hoặc tóm tắt tài liệu theo ngôn ngữ câu hỏi, nhờ đó một từ nằm bên trong file cũng tìm ra được file đó. Toàn bộ theo kiến trúc **serverless** trên AWS: không phải quản lý máy chủ, tính phí theo lượt gọi, tự mở rộng theo tải. Nền tảng dùng S3, Lambda, API Gateway, DynamoDB, CloudFront và Cognito, cùng ba dịch vụ AI Rekognition, Textract và Bedrock. Amazon Cognito lo phần đăng nhập, và claim `sub` trong JWT gán mỗi file cho đúng chủ nên người dùng chỉ thấy file của mình.

### 2. Tuyên bố vấn đề

*Vấn đề*
- Người dùng cần một cách nhanh để lưu và chia sẻ ảnh/tài liệu, nhưng giải pháp tự dựng máy chủ (EC2 chạy 24/7) tốn chi phí cố định kể cả khi nhàn rỗi và phải tự vận hành, vá lỗi, mở rộng.
- Khi số lượng file tăng, việc tìm lại đúng file rất khó vì chỉ tìm được theo tên.
- Tìm được tài liệu rồi thì vẫn phải mở ra đọc mới biết bên trong nói gì.

*Giải pháp*

InsightShare tập trung dữ liệu và xử lý trên một stack serverless thống nhất:
- **Lưu trữ & chia sẻ:** S3 lưu file, chia sẻ qua presigned URL có thời hạn; metadata lưu trong DynamoDB.
- **Xử lý nghiệp vụ:** Lambda + API Gateway sinh presigned URL, điều phối phân tích AI, ghi/đọc dữ liệu.
- **Hiểu nội dung bằng AI:** Rekognition gắn nhãn ảnh, Textract trích văn bản tài liệu, và Bedrock trả lời câu hỏi và tóm tắt tài liệu theo ngôn ngữ câu hỏi. Cả ba đều gọi qua API, không có bước huấn luyện nào.
- **Tìm kiếm thông minh:** nhãn và văn bản trích được lưu vào DynamoDB để tìm file theo nội dung.

*Lợi ích*
- Mô hình serverless trả theo lượng dùng; ở mức demo tổng chi phí dưới 1 USD/tháng.
- File người dùng không public và chỉ vào được qua link ký có thời hạn; bucket public duy nhất là bucket chứa trang web tĩnh. Quyền theo IAM least-privilege và CloudWatch giám sát hệ thống.
- Tìm kiếm chạy trên nhãn AI và văn bản trích, nên một từ nằm bên trong file cũng ra được file đó.

### 3. Kiến trúc giải pháp

*Tổng quan*

Trình duyệt tải giao diện tĩnh từ **S3 + CloudFront** qua HTTPS, đăng nhập qua **Cognito**, rồi gọi **API Gateway**, cổng này chuyển tới **Lambda**. API Gateway chạy một JWT authorizer kiểm tra token Cognito, và Lambda đọc claim `sub` để gán mỗi file cho đúng chủ sở hữu. Lambda sinh presigned URL để trình duyệt truyền file trực tiếp với S3. Sau khi upload, Lambda gọi Rekognition, Textract hoặc Bedrock và lưu kết quả vào DynamoDB phục vụ tìm kiếm. CloudWatch thu log và metric; IAM kiểm soát quyền theo least-privilege.

![Kiến trúc InsightShare](/images/2-Proposal/insightshare_architecture-v6.png)

*Dịch vụ AWS sử dụng*

| Dịch vụ | Vai trò |
|---|---|
| Amazon S3 | Lưu file người dùng; host giao diện web tĩnh |
| Amazon CloudFront | CDN phân phối web, HTTPS, tăng tốc |
| Amazon API Gateway | Cổng API công khai cho ứng dụng; một JWT authorizer kiểm tra token Cognito |
| Amazon Cognito | Đăng nhập người dùng; claim `sub` trong JWT gán mỗi file cho đúng chủ sở hữu để cô lập theo người dùng |
| AWS Lambda | Xử lý nghiệp vụ; một handler điều hướng request API Gateway HTTP API theo method và path |
| Amazon DynamoDB | Lưu metadata + nhãn AI + văn bản trích, phục vụ tìm kiếm |
| Amazon Rekognition | Gắn nhãn ảnh |
| Amazon Textract | Trích văn bản từ PDF và ảnh scan |
| Amazon Bedrock | Hỏi đáp và tóm tắt tài liệu, trả lời theo ngôn ngữ câu hỏi |
| Amazon CloudWatch | Log, metric, alarm giám sát hệ thống |
| AWS IAM | Phân quyền least-privilege cho Lambda và từng dịch vụ AI |

*Thiết kế thành phần*
- **Frontend:** trang web tĩnh chọn file, hiển thị danh sách kèm nhãn AI, ô tìm kiếm theo nội dung, ô đặt câu hỏi về một tài liệu.
- **API:** các endpoint yêu cầu URL upload, chạy phân tích AI, liệt kê/tìm kiếm file, lấy URL download, hỏi đáp về một tài liệu, và hỏi đáp trên toàn thư viện.

### 4. Triển khai kỹ thuật

*Các giai đoạn triển khai*

| Giai đoạn | Nội dung |
|---|---|
| 1. Nền tảng & thiết kế | Học nền tảng AWS, chốt đề tài, vẽ kiến trúc, thiết lập tài khoản an toàn, thiết kế schema DynamoDB. |
| 2. Ứng dụng cơ bản | Dựng web app chạy local, tách lớp lưu trữ và lớp AI. |
| 3. Đưa lên cloud | S3 + presigned URL, Lambda + API Gateway, tích hợp DynamoDB và link tải bằng presigned URL. |
| 4. Lớp AI & tìm kiếm | Tích hợp Rekognition/Textract/Bedrock, lưu kết quả vào DynamoDB, xây tìm kiếm thông minh. |
| 5. Hoàn thiện & vận hành | CloudFront + HTTPS, CloudWatch log/alarm, tối ưu chi phí/bảo mật, một script deploy/dọn dẹp lặp lại được. |

*Yêu cầu kỹ thuật*
- Tài khoản AWS, region `ap-southeast-1`.
- Công cụ: AWS CLI, Python 3, boto3.
- Kiến thức: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch và các dịch vụ AI.

### 5. Ước tính ngân sách

Ở mức demo, mô hình serverless tính theo lượt dùng cùng Free Tier giữ chi phí dưới 1 USD/tháng. Một AWS Budget `InsightShare-Monthly` giới hạn 5 USD/tháng theo dõi chi tiêu. Số liệu chi tiết tính bằng [AWS Pricing Calculator](https://calculator.aws/).

Ở quy mô thật, chi phí tăng theo mức dùng, chủ yếu đến từ các dịch vụ AI. Ước tính hàng tháng cho **1.000 người dùng**:

| Dịch vụ | Cơ sở tính | Ước tính/tháng |
| --- | --- | --- |
| AWS Lambda | ~120k lượt gọi | ~0,10 USD |
| Amazon API Gateway | ~120k request HTTP | ~0,12 USD |
| Amazon S3 | ~40 GB lưu + request | ~1,00 USD |
| Amazon DynamoDB | ~140k đọc/ghi | ~0,20 USD |
| Amazon Rekognition | 20k ảnh | ~20,00 USD |
| Amazon Bedrock | ~20k lượt tóm tắt + hỏi đáp | ~15,00 USD |
| Amazon CloudFront | ~30 GB ra | ~2,50 USD |
| **Tổng** | | **~40 USD/tháng** |

### 6. Đánh giá rủi ro

| Rủi ro | Tác động | Xác suất | Giảm thiểu |
|---|---|---|---|
| Cấu hình IAM/policy sai gây lỗi truy cập | Trung bình | Trung bình | Least-privilege, kiểm thử kỹ trước khi mở rộng quyền |
| Phát sinh chi phí ngoài dự kiến | Thấp | Thấp | Budget Alert, giới hạn kích thước/loại file gửi AI, dọn tài nguyên sau test |
| File lớn gây timeout Lambda/API Gateway | Trung bình | Thấp | Presigned URL upload trực tiếp lên S3, nội dung file không đi qua Lambda hay API Gateway |
| Dịch vụ AI chậm hoặc không sẵn sàng | Thấp | Trung bình | Phân tích là một lời gọi riêng tách khỏi upload và mỗi lời gọi AI đều fail-soft, nên file vẫn nằm trong danh sách và tải được dù phân tích không xong |

*Kế hoạch dự phòng:* giữ một script dọn dẹp để xóa toàn bộ tài nguyên nhanh chóng.

### 7. Kết quả kỳ vọng

*Cải tiến kỹ thuật*
- Ứng dụng web hoạt động end-to-end: upload → tự động phân tích nội dung bằng AI → liệt kê → tìm kiếm theo nội dung → hỏi đáp về một tài liệu → tải qua presigned link.
- Kiến trúc serverless kết hợp các dịch vụ AI managed trên AWS.
- Hỏi đáp tài liệu bằng Amazon Bedrock: endpoint `ask` nhận một tài liệu và một câu hỏi, được nối vào lệnh gọi `bedrock:InvokeModel` với model id theo inference-profile cùng phần xử lý request/response đầy đủ.

*Giá trị dài hạn*
- Nền tảng có thể mở rộng: điều phối pipeline AI nhiều bước, kích hoạt phân tích từ S3 event thay vì để client gọi, hỗ trợ thêm loại file.
- Tài liệu workshop chi tiết để người khác có thể làm theo và phát triển tiếp.