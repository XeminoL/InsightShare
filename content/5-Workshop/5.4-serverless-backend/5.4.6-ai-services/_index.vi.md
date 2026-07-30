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
- **Amazon Bedrock (Claude)**: hỏi đáp về nội dung tài liệu và tóm tắt, trả lời theo ngôn ngữ câu hỏi (`InvokeModel`). Đây là tính năng AI chính.

Rekognition và Textract là dịch vụ gọi sẵn, không cần train model. Bedrock chạy một model Claude được host sẵn nên cũng không phải huấn luyện.

#### Bước 1: Ảnh → Rekognition gắn nhãn

`analyze` rẽ nhánh theo loại file: ảnh đưa sang Rekognition, tài liệu sang trích văn bản. Với ảnh, Rekognition đọc object thẳng từ S3 (không có bytes nào đi qua Lambda) và trả về nhãn nội dung. `MaxLabels=10` giới hạn số nhãn giữ lại, và `MinConfidence=55` loại mọi nhãn có độ tin cậy dưới 55%:

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

Với một ảnh chụp bên trong kho hàng, Rekognition trả về mười nhãn: `Architecture`, `Building`, `Warehouse`, `Box`, `Cardboard`, `Carton`, `Factory`, `Package Delivery`, `Person`, `Indoors`. Các nhãn này lưu trong DynamoDB, nên tra `warehouse` hay `carton` vẫn ra đúng file dù hai từ đó không nằm trong tên file (`20260619_092010.jpg`).

![Ảnh đã upload cùng các nhãn Rekognition, xem trên app](/images/5-Workshop/5.4-serverless-backend/rekognition-labels.png)

{{% notice note %}}
`MinConfidence` là phần trăm độ tin cậy tối thiểu để một nhãn được trả về, còn `MaxLabels=10` là lý do ở đây ra đúng mười nhãn. Ngưỡng ban đầu để 70 thì không ra nhãn nào với một ảnh chụp màn hình sơ đồ: trong đó có ít vật thể thực để Rekognition chắc chắn. Hạ xuống 55 thì lọt các nhãn chính xác nhưng kém chắc chắn hơn. Ảnh phía trên là trường hợp ngược lại, đầy vật thể thực, nên chạm luôn giới hạn mười nhãn.
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
**Ghi chú thiết kế.** Lời gọi Textract được bọc trong `try/except` để khi nó lỗi thì luồng vẫn đi tiếp: file `.txt` đọc thẳng từ S3, còn PDF hoặc ảnh scan lấy kết quả `DetectDocumentText` làm văn bản tài liệu. Nếu lời gọi ném `ClientError`, `analyze` lùi về dùng tên file thay vì trả 500.
{{% /notice %}}

#### Bước 3: Bedrock (Claude) hỏi đáp về tài liệu

Bước này dùng văn bản mà hai bước trước lưu trong DynamoDB làm ngữ cảnh cho model. Tính năng chính là endpoint hỏi đáp tài liệu, `POST /files/{id}/ask`. Nó đọc phần văn bản đã trích từ DynamoDB, ghép cùng câu hỏi trong một prompt yêu cầu model chỉ trả lời dựa trên tài liệu đó và theo ngôn ngữ câu hỏi, rồi gọi một model Claude trên Amazon Bedrock. Nếu body không có `question`, chính handler đó tóm tắt tài liệu.

Ngoài ra có endpoint hỏi đáp toàn thư viện, `POST /ask`. Nó quét mọi file trong DynamoDB, xếp hạng theo độ trùng từ khóa với câu hỏi, ghép văn bản các file liên quan (mỗi file đánh số để trích nguồn) rồi gọi Bedrock. Response trả kèm danh sách file đã đánh số mà câu trả lời lấy từ đó.

```python
MODEL_ID = os.environ.get("BEDROCK_MODEL_ID",
                          "global.anthropic.claude-haiku-4-5-20251001-v1:0")
bedrock = boto3.client("bedrock-runtime", region_name=REGION)

doc_text = (rec.get("text") or "").strip()[:20000]
prompt = (
    "You are a document assistant. Answer ONLY based on the document below. "
    "Answer in the SAME LANGUAGE as the question. If the question is empty, "
    "summarize the document in the same language as its content, defaulting to "
    "English if the language is unclear. If the document does not contain the "
    "answer, say so clearly in the language of the question.\n\n"
    "<document>\n" + doc_text + "\n</document>\n\n"
    "Question: " + question
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
**Ghi chú thiết kế.** Phần tích hợp Bedrock dùng quyền IAM `bedrock:InvokeModel` và một model id dạng inference profile. Handler `ask` bọc lời gọi `invoke_model`: khi thành công thì trả về câu trả lời của Claude, khi lỗi thì trả về HTTP 200 kèm một câu tiếng Anh ngắn thay vì 500, nên một lỗi dịch vụ tạm thời không làm sập bản demo.
{{% /notice %}}

#### Bước 4: Lưu nhãn/văn bản vào DynamoDB

Output AI được ghi lại một lần, nên về sau tìm kiếm và hỏi đáp đọc DynamoDB thay vì gọi lại Rekognition, Textract hay Bedrock. Kết quả ghi trở lại item metadata: thuộc tính `search_blob` (nhãn + văn bản, viết thường) phục vụ tìm kiếm theo nội dung, còn phần `text` được lưu là thứ mà hỏi đáp Bedrock đọc. `size` cũng phải đặt bí danh `#sz` vì nó là từ khóa dành riêng của DynamoDB giống `text`:

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

curl "$API/files/search?q=warehouse"

curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
```

Tìm kiếm trả về ảnh đó từ nhãn `Warehouse`. Với tài liệu `.txt`, `ask` trả về câu trả lời lấy từ nội dung tài liệu, do Amazon Bedrock (Claude) sinh ra.

> Tham khảo lab FCAJ về AI services (Rekognition/Textract): https://000056.awsstudygroup.com
