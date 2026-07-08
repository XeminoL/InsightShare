---
title: "Frontend S3 + CloudFront"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

#### Mục tiêu

Phục vụ giao diện web tĩnh của InsightShare từ **Amazon S3** và phân phối qua **Amazon CloudFront** (HTTPS, CDN).

#### Frontend

Giao diện là một file `index.html` tĩnh (HTML/CSS/JS thuần): upload file, hiển thị danh sách kèm nhãn AI, có ô tìm kiếm theo nội dung, link tải cho từng file (một presigned GET URL), và ô đặt câu hỏi về một tài liệu. Nó chỉ nói chuyện với endpoint API Gateway, nên cùng một trang chạy được cả ở local lẫn trên CloudFront.

Luồng upload trên trình duyệt gồm hai lời gọi: xin presigned URL từ API, rồi PUT file thẳng lên S3.

```javascript
// 1) xin API một presigned upload URL + id metadata
const r = await fetch(`${API}/files`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ filename: file.name, content_type: file.type }),
});
const { id, upload_url } = await r.json();

// 2) upload file trực tiếp lên S3 qua presigned URL
await fetch(upload_url, { method: "PUT",
  headers: { "Content-Type": file.type }, body: file });

// 3) kích hoạt phân tích AI
await fetch(`${API}/files/${id}/analyze`, { method: "POST" });
```

Tìm kiếm theo nội dung là một lời gọi tới route search:

```javascript
const res = await fetch(`${API}/files/search?q=` + encodeURIComponent(query));
render(await res.json());   // khớp theo nhãn AI + văn bản trích xuất
```

Hỏi về một tài liệu là thêm một lời gọi tới route `ask` (Bedrock/Claude trả lời bằng tiếng Việt; câu hỏi rỗng thì trả về bản tóm tắt):

```javascript
const res = await fetch(`${API}/files/${id}/ask`, {
  method: "POST", headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ question }),      // câu hỏi rỗng -> tóm tắt tài liệu
});
const { answer } = await res.json();       // hiển thị dưới file
```

#### Host frontend trên S3

Web tĩnh được host trên một S3 bucket riêng có bật website hosting:

```bash
aws s3api create-bucket --bucket insightshare-web-khang-2352464 --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3 website s3://insightshare-web-khang-2352464/ --index-document index.html
aws s3 cp index.html s3://insightshare-web-khang-2352464/index.html --content-type text/html
```

Chỉ bucket web này áp bucket policy public-read (bucket file ở 5.3 vẫn private). Trang chạy tại:

`http://insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com`

#### Phân phối qua CloudFront

Một **CloudFront distribution** đã được tạo với origin là endpoint website S3 và `ViewerProtocolPolicy` đặt `redirect-to-https`, nên trang được phân phối qua HTTPS và CDN edge cache.

```bash
aws cloudfront create-distribution \
  --origin-domain-name insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com \
  --default-root-object index.html
```

Distribution đã ở trạng thái `Deployed` và phục vụ trang qua HTTPS tại:

`https://d1to3a23j7tkdp.cloudfront.net`

Trang chạy được qua cả hai lối vào: endpoint website S3 qua HTTP, và domain CloudFront qua HTTPS.

Trang web đang chạy, có dải thống kê, nhãn AI, ảnh thu nhỏ và bộ lọc theo nhãn:

![Trang web InsightShare đang chạy](/images/5-Workshop/5.4-serverless-backend/web-live-v2.png)

#### Test end-to-end

Toàn bộ luồng được kiểm chứng từ trang web đã deploy qua API Gateway:

- `POST /files` trả về presigned URL, và `PUT` lên URL đó trả về **HTTP 200** (file vào S3).
- `POST /files/{id}/analyze` trả về nhãn Rekognition thật.
- `GET /files/search?q=diagram` trả về ảnh theo nhãn AI của nó, không phải theo tên file.
- `POST /files/{id}/ask` được nối sẵn để trả về câu trả lời tiếng Việt trên một tài liệu `.txt` (hoặc câu dự phòng khi hết hạn mức token ở HTTP 200, trong lúc inference quota Bedrock của tài khoản còn là 0).
- Các bước này chạy trực tiếp từ trình duyệt trên trang web đang hoạt động, nên upload, phân tích AI, tìm kiếm theo nội dung và hỏi đáp tài liệu đều chạy đầu-cuối trên AWS.

#### Tóm tắt

InsightShare chạy đầu-cuối: frontend tĩnh gọi API Gateway → Lambda → S3 (presigned URL) + DynamoDB (metadata) + lớp AI. CloudFront là lớp phân phối đặt trước web tĩnh, đã deploy tại `https://d1to3a23j7tkdp.cloudfront.net`.
