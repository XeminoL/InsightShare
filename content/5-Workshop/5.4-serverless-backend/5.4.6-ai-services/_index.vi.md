---
title: "Thêm AI: Rekognition, Textract, Bedrock (Claude)"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.4.6 </b> "
---

#### Mục tiêu

Bổ sung lớp AI để InsightShare hiểu nội dung file thay vì chỉ lưu trữ:

- **Amazon Rekognition**: gắn nhãn ảnh (`DetectLabels`).
- **Amazon Textract**: trích văn bản từ PDF và ảnh scan (`DetectDocumentText`).
- **Amazon Bedrock (Claude)**: hỏi đáp về nội dung tài liệu và tóm tắt, trả lời bằng tiếng Việt (`InvokeModel`). Đây là tính năng AI chính.

Rekognition và Textract là dịch vụ gọi sẵn, không cần train model. Bedrock chạy một model Claude được host sẵn nên cũng không phải huấn luyện. Quyền IAM Lambda cần nằm ở phần 5.5.

#### Bước 1: Ảnh → Rekognition gắn nhãn

`analyze` rẽ nhánh theo loại file: ảnh đưa sang Rekognition, tài liệu sang trích văn bản. Với ảnh, Rekognition đọc object thẳng từ S3 (không có bytes nào đi qua Lambda) và trả về nhãn nội dung, chính là thứ giúp tìm ảnh theo cái nó thể hiện thay vì theo tên file. `MaxLabels=10` giới hạn số nhãn giữ lại, và `MinConfidence=55` loại mọi nhãn mà Rekognition chắc dưới 55%:

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

{{% notice note %}}
`MinConfidence` là phần trăm độ tin cậy tối thiểu để một nhãn được trả về. Ban đầu để 70 và không ra nhãn nào cho ảnh dạng sơ đồ (ít vật thể thực để nhận với độ tin cậy cao). Hạ xuống 55 thì lọt các nhãn chính xác nhưng kém chắc chắn hơn. Tinh chỉnh ngưỡng theo loại nội dung: cao hơn cho ảnh chụp vật thể, thấp hơn cho sơ đồ và ảnh chụp màn hình.
{{% /notice %}}

#### Bước 2: Tài liệu → trích văn bản

Với tài liệu, mục tiêu giống ảnh: biến file thành văn bản tìm kiếm được và hỏi đáp được. File `.txt` đã là văn bản nên đọc thẳng từ S3, không cần OCR. PDF hoặc ảnh scan thì không, nên `analyze` gọi Textract, chỉ giữ các block `LINE` và ghép lại thành văn bản tài liệu:

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

Bước này dùng văn bản mà hai bước trước lưu trong DynamoDB làm ngữ cảnh cho model. Tính năng chính là endpoint hỏi đáp tài liệu, `POST /files/{id}/ask`. Nó đọc phần văn bản đã trích từ DynamoDB, ghép cùng câu hỏi trong một prompt yêu cầu model chỉ trả lời dựa trên tài liệu đó và bằng tiếng Việt, rồi gọi một model Claude trên Amazon Bedrock. Nếu body không có `question`, chính handler đó tóm tắt tài liệu.

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

Model id nằm trong biến môi trường `BEDROCK_MODEL_ID` (mặc định `global.anthropic.claude-haiku-4-5-20251001-v1:0`), nên đổi model mà không phải sửa code. Văn bản tài liệu được cắt còn 20.000 ký tự trước khi đưa vào prompt, để chặn số token và chi phí mỗi lần gọi, đồng thời giữ request trong context window của model.

{{% notice note %}}
**Ghi chú thiết kế.** Phần tích hợp Bedrock dùng quyền IAM `bedrock:InvokeModel` và một model id dạng inference profile. Handler `ask` bọc lời gọi `invoke_model`: khi thành công thì trả về câu trả lời của Claude, khi lỗi thì trả về HTTP 200 kèm một câu tiếng Việt ngắn thay vì 500, nên một lỗi dịch vụ tạm thời không làm sập bản demo.
{{% /notice %}}

#### Bước 4: Lưu nhãn/văn bản vào DynamoDB

Lưu lại output AI chính là điều khiến `analyze` chỉ tốn một lần: về sau tìm kiếm và hỏi đáp đọc từ DynamoDB thay vì gọi lại Rekognition, Textract hay Bedrock. Kết quả được ghi trở lại item metadata (phần 5.4.3); thuộc tính `search_blob` (nhãn + văn bản, viết thường) phục vụ tìm kiếm theo nội dung, và phần `text` được lưu chính là thứ mà hỏi đáp Bedrock đọc. `size` được đặt bí danh `#sz` cùng lý do từ khóa dành riêng như `text`:

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
