---
title: "Thêm AI: Rekognition, Textract, Bedrock (Claude)"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Mục tiêu

Bổ sung lớp AI để InsightShare hiểu nội dung file thay vì chỉ lưu trữ:

- **Amazon Rekognition**: gắn nhãn ảnh (`DetectLabels`).
- **Amazon Textract**: trích văn bản từ PDF và ảnh scan (`DetectDocumentText`).
- **Amazon Bedrock (Claude)**: hỏi đáp về nội dung tài liệu và tóm tắt, trả lời bằng tiếng Việt (`InvokeModel`). Đây là tính năng AI chính.

Rekognition và Textract là dịch vụ gọi sẵn, không cần train model. Bedrock chạy một model Claude được host sẵn nên cũng không phải huấn luyện. Quyền IAM Lambda cần nằm ở phần 5.5.

#### Bước 1: Ảnh → Rekognition gắn nhãn

Sau khi ảnh được upload, `analyze` gọi Rekognition trên object trong S3:

```python
if fname.endswith(IMAGE_EXTS):
    try:
        det = rekognition.detect_labels(
            Image={"S3Object": {"Bucket": BUCKET, "Name": key}},
            MaxLabels=10, MinConfidence=55,
        )
        labels = [l["Name"] for l in det.get("Labels", [])]
        text = "image containing " + ", ".join(labels).lower()
    except ClientError:
        labels = ["Image"]
        text = rec["filename"]
        notice = "Rekognition chua duoc bat tren tai khoan nay."
```

Với một ảnh test, Rekognition trả về các nhãn như `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. Các nhãn này lưu trong DynamoDB và tìm kiếm được, nên tra `diagram` vẫn ra ảnh dù từ đó không nằm trong tên file.

{{% notice note %}}
`MinConfidence` ban đầu để 70 và không ra nhãn nào cho ảnh dạng sơ đồ (ít vật thể thực). Hạ xuống 55 thì ra nhãn chính xác, nên ngưỡng này đáng tinh chỉnh theo loại nội dung bạn dự kiến.
{{% /notice %}}

#### Bước 2: Tài liệu → trích văn bản

Với file `.txt`, nội dung được đọc thẳng từ S3 (không cần OCR). Với PDF hoặc ảnh scan, `analyze` gọi Textract:

```python
elif fname.endswith(DOC_EXTS):
    labels = ["Document", "Text"]
    if fname.endswith(".txt"):
        obj = s3.get_object(Bucket=BUCKET, Key=key)
        text = obj["Body"].read().decode("utf-8", errors="ignore").strip()
    else:
        try:
            det = textract.detect_document_text(
                Document={"S3Object": {"Bucket": BUCKET, "Name": key}}
            )
            text = "\n".join(b["Text"] for b in det.get("Blocks", [])
                             if b["BlockType"] == "LINE")
        except ClientError:
            text = rec["filename"]      # fail soft nếu Textract chưa dùng được
            labels = ["Document"]
            notice = "Textract chua duoc bat tren tai khoan nay."
```

{{% notice warning %}}
**Giới hạn.** Textract trả về `SubscriptionRequiredException: The AWS Access Key Id needs a subscription for the service`, kể cả khi gọi trực tiếp bằng CLI. Đây là giới hạn ở mức tài khoản (Textract chưa được kích hoạt trên tài khoản dạng credit này), không phải lỗi code. Code xử lý mềm: file `.txt` đọc trực tiếp và luồng vẫn chạy, còn lời gọi Textract được bọc lại để một dịch vụ chưa khả dụng không làm hỏng phần phân tích. Khi tài khoản được cấp quyền, chính đường code đó tự kích hoạt.
{{% /notice %}}

#### Bước 3: Bedrock (Claude) hỏi đáp về tài liệu

Tính năng AI chính là endpoint hỏi đáp tài liệu, `POST /files/{id}/ask`. Nó lấy phần văn bản đã trích sẵn trong DynamoDB, ghép cùng câu hỏi thành một prompt, rồi gọi một model Claude trên Amazon Bedrock. Câu trả lời trả về bằng tiếng Việt. Nếu body không có `question`, chính handler đó tóm tắt tài liệu.

```python
MODEL_ID = os.environ.get("BEDROCK_MODEL_ID",
                          "anthropic.claude-3-5-haiku-20241022-v1:0")
bedrock = boto3.client("bedrock-runtime", region_name=REGION)

# ask_document(event, file_id):
doc_text = (rec.get("text") or "").strip()[:20000]
prompt = (
    "Ban la tro ly tai lieu. Chi tra loi DUA TREN noi dung tai lieu duoi day, "
    "bang tieng Viet.\n\n<document>\n" + doc_text + "\n</document>\n\n"
    "Cau hoi: " + question
)
payload = {
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": prompt}],
}
out = bedrock.invoke_model(modelId=MODEL_ID, body=json.dumps(payload))
answer = json.loads(out["body"].read())["content"][0]["text"]
```

Model id nằm trong biến môi trường `BEDROCK_MODEL_ID` (mặc định `anthropic.claude-3-5-haiku-20241022-v1:0`), nên đổi model mà không phải sửa code. Văn bản tài liệu được cắt còn 20.000 ký tự trước khi đưa vào prompt.

{{% notice warning %}}
**Giới hạn.** Giống Textract, Bedrock có thể lỗi trên tài khoản dạng credit, ở đây là `AccessDeniedException` khi Model access cho model Claude chưa được bật trong console Bedrock. Handler `ask` bắt lỗi này và trả về HTTP 200 kèm một câu tiếng Việt ngắn ("Bedrock chua duoc bat tren tai khoan nay, can Model access") thay vì 500, nên bản demo không sập. Khi được cấp Model access, chính lời gọi đó trả về câu trả lời thật.
{{% /notice %}}

#### Bước 4: Lưu nhãn/văn bản vào DynamoDB

Kết quả AI được ghi trở lại item metadata (phần 5.4.3); thuộc tính `search_blob` (nhãn + văn bản, viết thường) phục vụ tìm kiếm theo nội dung, và phần `text` được lưu chính là thứ mà hỏi đáp Bedrock đọc:

```python
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s, #sz=:sz",
    ExpressionAttributeNames={"#t": "text", "#sz": "size"},
    ExpressionAttributeValues={":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower(), ":sz": size},
)
```

#### Kiểm thử

```bash
# Upload một ảnh, rồi analyze -> nhãn Rekognition thật
curl -X POST "$API/files/<id>/analyze"
# -> {"labels": ["Diagram","Text","Network","Chart", ...], "notice": ""}

# Tìm theo một nhãn AI (không nằm trong tên file)
curl "$API/files/search?q=diagram"
# -> trả về đúng ảnh vừa upload

# Hỏi đáp về một tài liệu .txt (Bedrock/Claude, tiếng Việt)
curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
# -> {"answer": "...", "is_summary": false}   (hoặc thông báo cần Model access, HTTP 200)
```

Tìm kiếm trả về đúng ảnh nhờ một nhãn AI không nằm trong tên file. Với tài liệu `.txt` khi đã bật Model access cho Bedrock, `ask` trả về câu trả lời tiếng Việt dựa trên nội dung tài liệu; khi chưa bật, endpoint trả về câu thông báo dự phòng ở HTTP 200.

> Tham khảo lab FCJ về AI services (Rekognition/Textract): https://000056.awsstudygroup.com
