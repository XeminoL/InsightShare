---
title: "Thêm AI: Rekognition, Textract, Polly"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Mục tiêu

Bổ sung **lớp AI** để InsightShare hiểu nội dung file thay vì chỉ lưu trữ:

- **Amazon Rekognition**: gắn nhãn ảnh (`DetectLabels`) và kiểm duyệt nội dung (`DetectModerationLabels`).
- **Amazon Textract**: trích văn bản từ PDF / ảnh scan (`DetectDocumentText`).
- **Amazon Polly**: đọc văn bản thành audio (`SynthesizeSpeech`).

Cả ba đều là dịch vụ gọi sẵn, không cần train model. Quyền IAM Lambda cần nằm ở phần 5.5.

#### Bước 1: Ảnh → Rekognition gắn nhãn + kiểm duyệt

Sau khi ảnh được upload, Lambda gọi Rekognition trên object trong S3:

```python
det = rekognition.detect_labels(
    Image={"S3Object": {"Bucket": BUCKET, "Name": key}},
    MaxLabels=10, MinConfidence=55,
)
labels = [l["Name"] for l in det["Labels"]]

mod = rekognition.detect_moderation_labels(
    Image={"S3Object": {"Bucket": BUCKET, "Name": key}}, MinConfidence=60,
)
if mod["ModerationLabels"]:
    labels.append("Flagged")
```

Với một ảnh test, Rekognition trả về các nhãn như `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. Các nhãn này lưu trong DynamoDB và tìm kiếm được, nên tra `diagram` vẫn ra ảnh dù từ đó không nằm trong tên file.

{{% notice note %}}
`MinConfidence` ban đầu để 70 và không ra nhãn nào cho ảnh dạng sơ đồ (ít vật thể thực). Hạ xuống 55 thì ra nhãn chính xác, nên ngưỡng này đáng tinh chỉnh theo loại nội dung bạn dự kiến.
{{% /notice %}}

#### Bước 2: Tài liệu → trích văn bản

Với file `.txt`, nội dung được đọc thẳng từ S3 (không cần OCR). Với PDF hoặc ảnh scan, Lambda gọi Textract:

```python
if fname.endswith(".txt"):
    obj = s3.get_object(Bucket=BUCKET, Key=key)
    text = obj["Body"].read().decode("utf-8", errors="ignore").strip()
else:
    try:
        det = textract.detect_document_text(
            Document={"S3Object": {"Bucket": BUCKET, "Name": key}}
        )
        text = "\n".join(b["Text"] for b in det["Blocks"] if b["BlockType"] == "LINE")
    except textract.exceptions.ClientError:
        text = fname            # fail soft nếu Textract chưa dùng được
        labels = ["Document", "Textract unavailable"]
```

{{% notice warning %}}
**Giới hạn.** Textract trả về `SubscriptionRequiredException: The AWS Access Key Id needs a subscription for the service`, kể cả khi gọi trực tiếp bằng CLI. Đây là giới hạn ở mức tài khoản (Textract chưa được kích hoạt trên tài khoản dạng credit này), không phải lỗi code. Code xử lý mềm: file `.txt` đọc trực tiếp và luồng vẫn chạy, còn lời gọi Textract được bọc lại để một dịch vụ chưa khả dụng không làm hỏng phần phân tích. Khi tài khoản được cấp quyền, chính đường code đó tự kích hoạt.
{{% /notice %}}

#### Bước 3: Polly đọc văn bản thành audio

Khi có văn bản, bản audio được sinh theo yêu cầu bằng giọng neural `Joanna`:

```python
audio = polly.synthesize_speech(
    Text=text[:3000], OutputFormat="mp3", VoiceId="Joanna", Engine="neural")
# audio["AudioStream"].read() -> bytes mp3, lưu vào S3 và trả qua /files/{id}/audio
```

`SynthesizeSpeech` tạo ra một file mp3 lưu vào S3 và trả về qua `/files/{id}/audio`. Rekognition và Polly chạy thật trên tài khoản; chỉ Textract bị giới hạn (xem phía trên).

#### Bước 4: Lưu nhãn/văn bản vào DynamoDB

Kết quả AI được ghi trở lại item metadata (phần 5.4.3); thuộc tính `search_blob` (nhãn + văn bản, viết thường) giúp tìm kiếm theo nội dung:

```python
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s",
    ExpressionAttributeNames={"#t": "text"},
    ExpressionAttributeValues={":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower()},
)
```

#### Kiểm thử

```bash
# Upload một ảnh, rồi analyze -> nhãn Rekognition thật
curl -X POST "$API/files/<id>/analyze"
# -> {"labels": ["Diagram","Text","Network","Chart", ...], "audio_available": true}

# Tìm theo một nhãn AI (không nằm trong tên file)
curl "$API/files/search?q=diagram"
# -> trả về đúng ảnh vừa upload
```

Tìm kiếm trả về đúng ảnh nhờ một nhãn AI không nằm trong tên file.

> Tham khảo lab FCJ về AI services (Rekognition/Polly/Lex): https://000056.awsstudygroup.com
