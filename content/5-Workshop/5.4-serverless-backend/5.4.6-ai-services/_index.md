---
title: "Add AI: Rekognition, Textract, Bedrock (Claude)"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.4.6 </b> "
---

#### Goal

Add the AI layer so InsightShare understands file content instead of just storing it:

- **Amazon Rekognition**: label images (`DetectLabels`).
- **Amazon Textract**: extract text from PDFs and scanned images (`DetectDocumentText`).
- **Amazon Bedrock (Claude)**: ask questions about a document and get a summary, answered in the same language as the question (`InvokeModel`). This is the main AI feature.

Rekognition and Textract are ready-to-call, no model training. Bedrock runs a hosted Claude model, so nothing is trained either.

#### Step 1: Image → Rekognition labels

`analyze` branches on file type: images go to Rekognition, documents to text extraction. For an image, Rekognition reads the object straight from S3 (no bytes pass through Lambda) and returns content labels, which are what makes an image findable by what it shows rather than its filename. `MaxLabels=10` caps how many labels are kept, and `MinConfidence=55` drops any label Rekognition is less than 55% sure of:

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

For a test image, Rekognition returned labels such as `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. These are stored in DynamoDB and become searchable, so a query for `diagram` finds the image even though the word is not in its filename.

![Console: Rekognition labels on the uploaded image](/images/5-Workshop/5.4-serverless-backend/rekognition-labels.png)

{{% notice note %}}
`MinConfidence` is the minimum confidence percentage a label must reach to be returned. It started at 70 and returned no labels for a diagram-style image (few real-world objects to recognize with high confidence). Lowering it to 55 let the accurate-but-less-certain labels through. Tune the threshold to the content: higher for photos of objects, lower for diagrams and screenshots.
{{% /notice %}}

#### Step 2: Document → text extraction

For documents the goal is the same as for images: turn the file into searchable, question-answerable text. A `.txt` file already is text, so it is read directly from S3 with no OCR. A PDF or scanned image is not, so `analyze` calls Textract, which keeps only the `LINE` blocks and joins them into the document text:

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
**Design note.** The Textract call is wrapped so text extraction fails soft: `.txt` files are read directly from S3, and for PDFs or scanned images the `DetectDocumentText` result is stored as the document text. If the call raises a `ClientError`, `analyze` falls back to the filename and keeps the rest of the flow working instead of returning a 500.
{{% /notice %}}

#### Step 3: Bedrock (Claude) answers questions about the document

This step uses the text the previous two steps stored in DynamoDB as the context for the model. The main feature is a document Q&A endpoint, `POST /files/{id}/ask`. It reads the extracted text from DynamoDB, wraps it with the question in a prompt that instructs the model to answer only from that document and in the same language as the question, and calls a Claude model on Amazon Bedrock. With no `question` in the body, the same handler summarizes the document instead.

There is also a library-wide endpoint, `POST /ask`. It scans every file in DynamoDB, ranks them by keyword overlap with the question, joins the text of the relevant files (each numbered so it can be cited), and calls Bedrock. The answer comes back with the list of files it came from, so you find the right file without opening each one.

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

The model id lives in the `BEDROCK_MODEL_ID` environment variable (default `global.anthropic.claude-haiku-4-5-20251001-v1:0`), so the model can change without editing code. The document text is capped at 20,000 characters before it goes into the prompt, which bounds the token count and cost per call and keeps the request within the model's context window.

{{% notice note %}}
**Design note.** The Bedrock integration uses the IAM `bedrock:InvokeModel` permission and an inference-profile model id. The `ask` handler wraps the `invoke_model` call: on success it returns the Claude answer, on error it returns HTTP 200 with a short English message instead of a 500, so a transient service error does not crash the demo.
{{% /notice %}}

#### Step 4: Store labels/text in DynamoDB

Persisting the AI output is what makes `analyze` a one-time cost: search and Q&A later read from DynamoDB instead of re-calling Rekognition, Textract or Bedrock. The results are written back to the metadata item; the `search_blob` attribute (labels + text, lowercased) powers content search, and the stored `text` is what the Bedrock Q&A reads. `size` is aliased with `#sz` for the same reserved-word reason as `text`:

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
curl -X POST "$API/files/<id>/analyze"

curl "$API/files/search?q=diagram"

curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
```

Search returned the image by an AI label that is not in its filename. On a `.txt` document, `ask` returns an answer grounded in the document text, generated by Amazon Bedrock (Claude).

> Reference FCJ lab on AI services (Rekognition/Textract): https://000056.awsstudygroup.com
