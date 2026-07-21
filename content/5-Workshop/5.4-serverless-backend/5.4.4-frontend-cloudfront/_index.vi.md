---
title: "Frontend S3 + CloudFront"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

#### Mục tiêu

Phục vụ giao diện web tĩnh của InsightShare từ **Amazon S3** và phân phối qua **Amazon CloudFront** (HTTPS, CDN).

#### Bước 1: Frontend

Frontend không có logic riêng ngoài việc gọi API và render JSON. Giao diện là một file `index.html` tĩnh (HTML/CSS/JS thuần): upload file, hiển thị danh sách kèm nhãn AI, có ô tìm kiếm theo nội dung, link tải cho từng file (một presigned GET URL), và ô đặt câu hỏi về một tài liệu. Nó chỉ nói chuyện với endpoint API Gateway, nên cùng một trang chạy được cả ở local lẫn trên CloudFront mà không cần build lại.

Luồng upload trên trình duyệt là hai lời gọi từ 5.3.2: xin presigned URL từ API, PUT file thẳng lên S3, rồi kích hoạt `analyze` để lớp AI xử lý object vừa upload.

```javascript
const r = await fetch(`${API}/files`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ filename: file.name, content_type: file.type }),
});
const { id, upload_url } = await r.json();

await fetch(upload_url, { method: "PUT",
  headers: { "Content-Type": file.type }, body: file });

await fetch(`${API}/files/${id}/analyze`, { method: "POST" });
```

Tìm kiếm theo nội dung là một lời gọi tới route search:

```javascript
const res = await fetch(`${API}/files/search?q=` + encodeURIComponent(query));
render(await res.json());
```

Hỏi về một tài liệu là thêm một lời gọi tới route `ask` (Bedrock/Claude trả lời theo ngôn ngữ câu hỏi; câu hỏi rỗng thì trả về bản tóm tắt):

```javascript
const res = await fetch(`${API}/files/${id}/ask`, {
  method: "POST", headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ question }),
});
const { answer } = await res.json();
```

#### Bước 2: Host frontend trên S3

Một bucket thứ hai phục vụ web tĩnh, tách khỏi bucket file để hai bên có policy truy cập ngược nhau: bucket file luôn hoàn toàn private còn bucket web này cho cả thế giới đọc. Bật website hosting để S3 phục vụ `index.html` tại website endpoint của bucket:

```bash
aws s3api create-bucket --bucket insightshare-web-khang-2352464 --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3 website s3://insightshare-web-khang-2352464/ --index-document index.html
aws s3 cp index.html s3://insightshare-web-khang-2352464/index.html --content-type text/html
```

Chỉ bucket web này áp bucket policy public-read, vì trang phải tải được cho bất kỳ ai có URL; bucket file ở 5.3 vẫn bật hết Block Public Access và không bao giờ public. Trang chạy tại:

`http://insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com`

#### Bước 3: Phân phối qua CloudFront

Website endpoint của S3 chỉ có HTTP và phục vụ từ một region; CloudFront đặt một lớp HTTPS, cache toàn cầu đứng trước nó. Một **CloudFront distribution** đã được tạo với origin là endpoint website S3 và `ViewerProtocolPolicy` đặt `redirect-to-https`, nên request HTTP thuần bị chuyển sang HTTPS và trang được cache tại edge CDN gần khách nhất.

```bash
aws cloudfront create-distribution \
  --origin-domain-name insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com \
  --default-root-object index.html
```

Distribution đã ở trạng thái `Deployed` và phục vụ trang qua HTTPS.

![Console: CloudFront distribution Deployed](/images/5-Workshop/5.4-serverless-backend/cloudfront-distribution.png)

Trang web đang chạy, có dải thống kê, nhãn AI, ảnh thu nhỏ và bộ lọc theo nhãn:

![Trang web InsightShare đang chạy](/images/5-Workshop/5.4-serverless-backend/web-live-v4.png)

#### Bước 4: Test end-to-end

Toàn bộ luồng được kiểm chứng từ trang web đã deploy qua API Gateway:

- `POST /files` trả về presigned URL, và `PUT` lên URL đó trả về **HTTP 200** (file vào S3).
- `POST /files/{id}/analyze` trả về nhãn Rekognition thật.
- `GET /files/search?q=diagram` trả về ảnh theo nhãn AI của nó, không phải theo tên file.
- `POST /files/{id}/ask` trả về câu trả lời trên một tài liệu `.txt`, do Amazon Bedrock (Claude) sinh ra từ văn bản đã lưu.
- Các bước này chạy trực tiếp từ trình duyệt trên trang web đang hoạt động, nên upload, phân tích AI, tìm kiếm theo nội dung và hỏi đáp tài liệu đều chạy đầu-cuối trên AWS.
