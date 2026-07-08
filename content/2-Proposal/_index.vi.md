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

InsightShare là một ứng dụng web để tải lên, phân tích và chia sẻ ảnh/tài liệu. Khi file được tải lên, các dịch vụ AI của AWS hiểu nội dung: gắn nhãn ảnh, kiểm duyệt nội dung nhạy cảm, trích văn bản từ tài liệu và đọc văn bản thành audio, nhờ đó người dùng tìm file theo nội dung thay vì chỉ theo tên. Toàn bộ theo kiến trúc **serverless** trên AWS (region `ap-southeast-1`): không phải quản lý máy chủ, tính phí theo lượt gọi nên chi phí lúc nhàn rỗi nhỏ, tự mở rộng theo tải. Nền tảng dùng S3, Lambda, API Gateway, DynamoDB, CloudFront cùng ba dịch vụ AI Rekognition, Textract và Polly.

### 2. Tuyên bố vấn đề

*Vấn đề*
- Người dùng cần một cách nhanh để lưu và chia sẻ ảnh/tài liệu, nhưng giải pháp tự dựng máy chủ (EC2 chạy 24/7) tốn chi phí cố định kể cả khi nhàn rỗi và phải tự vận hành, vá lỗi, mở rộng.
- Khi số lượng file tăng, việc tìm lại đúng file rất khó vì chỉ tìm được theo tên.
- Không có cơ chế kiểm duyệt nội dung tự động, dễ lọt file không phù hợp.

*Giải pháp*
InsightShare tập trung dữ liệu và xử lý trên một stack serverless thống nhất:
- **Lưu trữ & chia sẻ:** S3 lưu file (bucket private), chia sẻ qua presigned URL có thời hạn; metadata lưu trong DynamoDB.
- **Xử lý nghiệp vụ:** Lambda + API Gateway sinh presigned URL, điều phối phân tích AI, ghi/đọc dữ liệu.
- **Hiểu nội dung bằng AI:** Rekognition gắn nhãn + kiểm duyệt ảnh, Textract trích văn bản tài liệu, Polly đọc văn bản thành audio. Tất cả đều là dịch vụ gọi sẵn, không huấn luyện mô hình.
- **Tìm kiếm thông minh:** nhãn và văn bản trích được lưu vào DynamoDB để tìm file theo nội dung.

*Lợi ích & ROI*
- **Tập trung:** lưu trữ, chia sẻ và phân tích trên một nền tảng duy nhất, giảm thao tác thủ công.
- **Thông minh:** tự gắn nhãn, trích text, kiểm duyệt và tìm kiếm theo nội dung, bổ sung trên nền lưu trữ.
- **Chi phí thấp:** mô hình serverless trả theo lượng dùng; các dịch vụ AI nằm trong Free Tier ở mức demo, tổng chi phí dưới 1 USD/tháng.
- **Tin cậy & bảo mật:** file không public, IAM least-privilege, giám sát bằng CloudWatch.
- Là nền tảng học tập thực tế về kiến trúc serverless kết hợp AI, có thể mở rộng thành sản phẩm lớn hơn.

### 3. Kiến trúc giải pháp

*Tổng quan*
Trình duyệt tải giao diện tĩnh từ **S3 + CloudFront (HTTPS)** → gọi **API Gateway** → **Lambda (Python)**. Lambda sinh presigned URL để trình duyệt upload/download trực tiếp với **S3**. Sau khi upload, Lambda gọi các dịch vụ AI (**Rekognition / Textract / Polly**) và lưu kết quả vào **DynamoDB** phục vụ tìm kiếm. **CloudWatch** giám sát log/metric; **IAM** kiểm soát quyền theo least-privilege.

![Kiến trúc InsightShare](/images/2-Proposal/insightshare_architecture.png)

*Dịch vụ AWS sử dụng*

| Dịch vụ | Vai trò |
|---|---|
| Amazon S3 | Lưu file người dùng & audio do Polly tạo; host giao diện web tĩnh |
| Amazon CloudFront | CDN phân phối web, HTTPS, tăng tốc |
| Amazon API Gateway | Cổng API công khai cho ứng dụng |
| AWS Lambda | Xử lý nghiệp vụ (Python/boto3); một handler điều hướng request API Gateway HTTP API theo method và path |
| Amazon DynamoDB | Lưu metadata + nhãn AI + văn bản trích, phục vụ tìm kiếm |
| Amazon Rekognition | Gắn nhãn ảnh (DetectLabels) + kiểm duyệt (DetectModerationLabels) |
| Amazon Textract | Trích văn bản từ PDF/ảnh chữ (DetectDocumentText) |
| Amazon Polly | Đọc văn bản thành audio (SynthesizeSpeech) |
| Amazon CloudWatch | Log, metric, alarm giám sát hệ thống |
| AWS IAM | Phân quyền least-privilege cho Lambda và từng dịch vụ AI |

*Thiết kế thành phần*
- **Frontend:** trang web tĩnh (HTML/JS) chọn file, hiển thị danh sách kèm nhãn AI, ô tìm kiếm theo nội dung, lấy link chia sẻ.
- **API:** các endpoint yêu cầu URL upload, xác nhận upload (kích hoạt phân tích AI), liệt kê/tìm kiếm file, lấy URL download.
- **Lưu trữ:** file & audio trong S3; metadata + kết quả AI trong DynamoDB.
- **Bảo mật:** IAM Role least-privilege; file không public, chỉ truy cập qua presigned URL.

### 4. Triển khai kỹ thuật

*Các giai đoạn triển khai*

| Giai đoạn | Nội dung |
|---|---|
| 1. Nền tảng & thiết kế | Học nền tảng AWS, chốt đề tài, vẽ kiến trúc, thiết lập tài khoản an toàn (MFA, IAM user, CLI, Budgets), thiết kế schema DynamoDB. |
| 2. Ứng dụng cơ bản | Dựng web app (FastAPI + frontend) chạy local, tách lớp lưu trữ và lớp AI. |
| 3. Đưa lên cloud | S3 + presigned URL, Lambda + API Gateway, tích hợp DynamoDB và tính năng chia sẻ link. |
| 4. Lớp AI & tìm kiếm | Tích hợp Rekognition/Textract/Polly, lưu kết quả vào DynamoDB, xây tìm kiếm thông minh. |
| 5. Hoàn thiện & vận hành | CloudFront + HTTPS, CloudWatch log/alarm, tối ưu chi phí/bảo mật, IaC bằng AWS SAM, CI/CD. |

*Yêu cầu kỹ thuật*
- Tài khoản AWS (Free Tier), region `ap-southeast-1` (Singapore).
- Công cụ: AWS CLI, Python 3, boto3, (tùy chọn) AWS SAM.
- Kiến thức: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch và các dịch vụ AI (Rekognition, Textract, Polly).

### 5. Lộ trình & Mốc triển khai

| Mốc | Hoạt động |
|---|---|
| Chuẩn bị | Chốt kiến trúc, thiết kế DynamoDB, thiết lập tài khoản AWS, viết Proposal |
| Phát triển lõi | Lưu trữ S3 + presigned URL, back-end Lambda + API Gateway, DynamoDB + chia sẻ link |
| Tích hợp AI | Rekognition + Textract + Polly; tìm kiếm thông minh theo nội dung |
| Hoàn thiện | CloudFront, CloudWatch, tối ưu chi phí/bảo mật, SAM, CI/CD |
| Vận hành | Giám sát, kiểm thử, rà bảo mật và dọn dẹp tài nguyên |

Mốc chính: hoàn thành MVP chia sẻ (upload + link) trước; tích hợp lớp AI và tìm kiếm tiếp theo; sau đó hoàn thiện, tối ưu và vận hành.

### 6. Ước tính ngân sách

Chi phí ước tính thấp do mô hình serverless tính theo lượt dùng, và các dịch vụ AI nằm trong Free Tier ở mức demo. Số liệu chi tiết sẽ tính bằng [AWS Pricing Calculator](https://calculator.aws/).

| Dịch vụ | Ước tính/tháng |
| --- | --- |
| AWS Lambda | ~0,00 USD (trong Free Tier) |
| Amazon API Gateway | ~0,01 USD |
| Amazon S3 (lưu trữ + request) | ~0,10-0,20 USD |
| Amazon DynamoDB (on-demand) | ~0,00-0,05 USD |
| Amazon CloudFront | ~0,00-0,10 USD |
| Rekognition + Textract + Polly | ~0,00 USD (mức demo, trong Free Tier) |
| **Tổng** | **< 1 USD/tháng (mức demo)** |

Đo trên hạ tầng đang chạy, chi phí vận hành thực tế tháng 7/2026 về cơ bản là 0 USD: mọi dịch vụ đều nằm trong Free Tier hoặc theo mô hình pay-per-use, nên một stack nhàn rỗi gần như không tốn gì. Một AWS Budget `InsightShare-Monthly` giới hạn 5 USD/tháng đã được tạo để theo dõi chi tiêu, bên cạnh budget zero-spend có sẵn của tài khoản.

Ở quy mô thật, chi phí tăng theo mức dùng. Ước tính hàng tháng cho **1.000 người dùng** (khoảng 20.000 lượt upload, mỗi lượt phân tích một lần, cùng duyệt và tìm kiếm):

| Dịch vụ | Cơ sở tính | Ước tính/tháng |
| --- | --- | --- |
| AWS Lambda | ~120k lượt gọi | ~0,10 USD |
| Amazon API Gateway | ~120k request HTTP | ~0,12 USD |
| Amazon S3 | ~40 GB lưu + request | ~1,00 USD |
| Amazon DynamoDB (on-demand) | ~140k đọc/ghi | ~0,20 USD |
| Amazon Rekognition | 20k ảnh (DetectLabels) | ~20,00 USD |
| Amazon Polly | ~5M ký tự (neural) | ~80,00 USD |
| Amazon CloudFront | ~30 GB ra | ~2,50 USD |
| **Tổng** | | **~105 USD/tháng** |

Ở quy mô lớn, chi phí chủ yếu đến từ các dịch vụ AI (Rekognition, Polly), đây là đánh đổi cho việc không phải huấn luyện hay tự vận hành mô hình. Ở mức demo, chi phí gần 0 vì lượng dùng nằm trong Free Tier.

### 7. Đánh giá rủi ro

| Rủi ro | Tác động | Xác suất | Giảm thiểu |
|---|---|---|---|
| Cấu hình IAM/policy sai gây lỗi truy cập | Trung bình | Trung bình | Least-privilege, kiểm thử kỹ trước khi mở rộng quyền |
| Phát sinh chi phí ngoài dự kiến (gọi AI nhiều) | Thấp | Thấp | Budget Alert, giới hạn kích thước/loại file gửi AI, dọn tài nguyên sau test |
| File lớn gây timeout Lambda/API Gateway | Trung bình | Thấp | Presigned URL upload trực tiếp S3; gọi AI bất đồng bộ qua S3 event |
| Dịch vụ AI chậm hoặc kết quả chưa chính xác | Thấp | Trung bình | Phân tích AI bất đồng bộ, file vẫn chia sẻ được kể cả khi chưa phân tích xong |

*Kế hoạch dự phòng:* dùng Infrastructure-as-Code (AWS SAM) để tái tạo/xóa hạ tầng nhanh chóng.

### 8. Kết quả kỳ vọng

*Cải tiến kỹ thuật*
- Ứng dụng web hoạt động end-to-end: upload → tự động phân tích nội dung bằng AI → liệt kê → tìm kiếm theo nội dung → chia sẻ qua link.
- Làm chủ kiến trúc serverless kết hợp các dịch vụ AI managed trên AWS.

*Giá trị dài hạn*
- Nền tảng có thể mở rộng: thêm đăng nhập người dùng (Cognito), điều phối pipeline AI nhiều bước (Step Functions), hỗ trợ thêm loại file.
- Tài liệu workshop chi tiết để người khác có thể làm theo và phát triển tiếp.
