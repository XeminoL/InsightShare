---
title: "Add AI: Rekognition, Textract, Bedrock (Claude)"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Goal

Add the AI layer so InsightShare understands file content instead of just storing it:

- **Amazon Rekognition**: label images (`DetectLabels`).
- **Amazon Textract**: extract text from PDFs and scanned images (`DetectDocumentText`).
- **Amazon Bedrock (Claude)**: ask questions about a document and get a summary, answered in Vietnamese (`InvokeModel`). This is the main AI feature.

Rekognition and Textract are ready-to-call, no model training. Bedrock runs a hosted Claude model, so nothing is trained either. The IAM permissions Lambda needs are in section 5.5.

#### Step 1: Image → Rekognition labels

After an image is uploaded, `analyze` calls Rekognition on the S3 object:

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

For a test image, Rekognition returned labels such as `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. These are stored in DynamoDB and become searchable, so a query for `diagram` finds the image even though the word is not in its filename.

{{% notice note %}}
`MinConfidence` started at 70 and returned no labels for a diagram-style image (few real-world objects). Lowering it to 55 produced accurate labels, so the threshold is worth tuning to the kind of content you expect.
{{% /notice %}}

#### Step 2: Document → text extraction

For a `.txt` file the content is read directly from S3 (no OCR needed). For a PDF or scanned image, `analyze` calls Textract:

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
            text = rec["filename"]      # fail soft if Textract is not available
            labels = ["Document"]
            notice = "Textract chua duoc bat tren tai khoan nay."
```

{{% notice warning %}}
**Limitation.** Textract returns `SubscriptionRequiredException: The AWS Access Key Id needs a subscription for the service`, including when called directly from the CLI. This is an account-level limit (Textract is not enabled on this credit-based account), not a code error. The code fails soft: `.txt` files are read directly and the flow keeps working, while the Textract call is wrapped so an unavailable service does not break analysis. When the account gains access, the same code path activates automatically.
{{% /notice %}}

#### Step 3: Bedrock (Claude) answers questions about the document

The flagship feature is a document Q&A endpoint, `POST /files/{id}/ask`. It takes the text already extracted into DynamoDB, wraps it with the question in a prompt, and calls a Claude model on Amazon Bedrock. Answers come back in Vietnamese. With no `question` in the body, the same handler summarizes the document instead.

```python
MODEL_ID = os.environ.get("BEDROCK_MODEL_ID",
                          "global.anthropic.claude-haiku-4-5-20251001-v1:0")
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

The model id lives in the `BEDROCK_MODEL_ID` environment variable (default `global.anthropic.claude-haiku-4-5-20251001-v1:0`), so the model can change without editing code. The document text is capped at 20,000 characters before it goes into the prompt.

{{% notice warning %}}
**Limitation.** Like Textract, Bedrock can fail on a credit-based account, here with `AccessDeniedException` when Model access for the Claude model is not enabled in the Bedrock console. The `ask` handler catches this and returns HTTP 200 with a short Vietnamese message ("Bedrock chua duoc bat tren tai khoan nay, can Model access") instead of a 500, so a demo never crashes. Once Model access is granted the same call returns a real answer.
{{% /notice %}}

#### Step 4: Store labels/text in DynamoDB

The AI results are written back to the metadata item (section 5.4.3); the `search_blob` attribute (labels + text, lowercased) powers content search, and the stored `text` is what the Bedrock Q&A reads:

```python
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s, #sz=:sz",
    ExpressionAttributeNames={"#t": "text", "#sz": "size"},
    ExpressionAttributeValues={":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower(), ":sz": size},
)
```

#### Test

```bash
# Upload an image, then analyze -> real Rekognition labels
curl -X POST "$API/files/<id>/analyze"
# -> {"labels": ["Diagram","Text","Network","Chart", ...], "notice": ""}

# Search by an AI label (not in the filename)
curl "$API/files/search?q=diagram"
# -> the uploaded image is returned

# Ask a question about a .txt document (Bedrock/Claude, Vietnamese)
curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
# -> {"answer": "...", "is_summary": false}   (or a Model-access notice, HTTP 200)
```

Search returned the image by an AI label that is not in its filename. On a `.txt` document with Bedrock Model access enabled, `ask` returned a Vietnamese answer grounded in the document text; without it, the endpoint returned the fallback message at HTTP 200.

> Reference FCJ lab on AI services (Rekognition/Textract): https://000056.awsstudygroup.com
