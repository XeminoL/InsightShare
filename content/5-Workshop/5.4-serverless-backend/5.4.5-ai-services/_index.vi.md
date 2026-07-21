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
```

Với một ảnh test, Rekognition trả về các nhãn như `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. Các nhãn này lưu trong DynamoDB và tìm kiếm được, nên tra `diagram` vẫn ra ảnh dù từ đó không nằm trong tên file.

![Console: nhãn Rekognition trên ảnh đã upload](/images/5-Workshop/5.4-serverless-backend/rekognition-labels.png)

_Ảnh chụp Console của bạn (hoặc trang web đang chạy) cho thấy các nhãn Rekognition trả về cho một ảnh đã upload (ảnh cần bổ sung)._

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
            text = rec["filename"]
            labels = ["Document"]
```

{{% notice note %}}
**Ghi chú thiết kế.** Lời gọi Textract được bọc lại để phần trích văn bản xử lý mềm: file `.txt` đọc trực tiếp từ S3, còn PDF hoặc ảnh scan thì lấy kết quả `DetectDocumentText` làm văn bản tài liệu. Nếu lời gọi ném `ClientError`, `analyze` lùi về dùng tên file và giữ phần còn lại của luồng vẫn chạy thay vì trả về 500.
{{% /notice %}}

#### Bước 3: Bedrock (Claude) hỏi đáp về tài liệu

Tính năng AI chính là endpoint hỏi đáp tài liệu, `POST /files/{id}/ask`. Nó lấy phần văn bản đã trích sẵn trong DynamoDB, ghép cùng câu hỏi thành một prompt, rồi gọi một model Claude trên Amazon Bedrock. Câu trả lời trả về bằng tiếng Việt. Nếu body không có `question`, chính handler đó tóm tắt tài liệu.

Ngoài ra có endpoint hỏi đáp toàn thư viện, `POST /ask`. Nó quét mọi tệp trong DynamoDB, xếp hạng theo độ trùng từ khóa với câu hỏi, ghép văn bản các tệp liên quan (mỗi tệp đánh số để trích nguồn) rồi gọi Bedrock. Câu trả lời kèm danh sách tệp chứa thông tin, giúp tìm đúng tệp mà không phải mở từng cái.

```python
MODEL_ID = os.environ.get("BEDROCK_MODEL_ID",
                          "global.anthropic.claude-haiku-4-5-20251001-v1:0")
bedrock = boto3.client("bedrock-runtime", region_name=REGION)

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

Model id nằm trong biến môi trường `BEDROCK_MODEL_ID` (mặc định `global.anthropic.claude-haiku-4-5-20251001-v1:0`), nên đổi model mà không phải sửa code. Văn bản tài liệu được cắt còn 20.000 ký tự trước khi đưa vào prompt.

{{% notice note %}}
**Ghi chú thiết kế.** Phần tích hợp Bedrock dùng quyền IAM `bedrock:InvokeModel` và một model id dạng inference profile (`global.anthropic.claude-haiku-4-5-20251001-v1:0`) đọc từ biến môi trường, nên đổi model mà không phải sửa code. Handler `ask` bọc lời gọi `invoke_model`: khi thành công thì trả về câu trả lời của Claude, khi lỗi thì trả về HTTP 200 kèm một câu tiếng Việt ngắn thay vì 500, nên bản demo không sập vì một lỗi dịch vụ tạm thời.
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
curl -X POST "$API/files/<id>/analyze"

curl "$API/files/search?q=diagram"

curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
```

Tìm kiếm trả về đúng ảnh nhờ một nhãn AI không nằm trong tên file. Với tài liệu `.txt`, `ask` trả về câu trả lời tiếng Việt dựa trên nội dung tài liệu, do Amazon Bedrock (Claude) sinh ra.

> Tham khảo lab FCJ về AI services (Rekognition/Textract): https://000056.awsstudygroup.com
