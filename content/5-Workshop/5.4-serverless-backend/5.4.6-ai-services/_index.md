---
title: "Add AI: Rekognition, Textract, Bedrock"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.4.6 </b> "
---

#### Goal

Add the AI layer so InsightShare understands file content instead of just storing it:

- **Amazon Rekognition**: label images.
- **Amazon Textract**: extract text from PDFs and scanned images.
- **Amazon Bedrock**: ask questions about a document and get a summary, answered in the same language as the question. This is the main AI feature.

Rekognition and Textract are ready-to-call, no model training. Bedrock runs a hosted Claude model, so nothing is trained either.

#### Step 1: Image → Rekognition labels

`analyze` branches on file type: images go to Rekognition, documents to text extraction. For an image, Rekognition reads the object straight from S3 and returns content labels. `MaxLabels=10` caps how many labels are kept, and `MinConfidence=55` drops any label whose confidence is below 55%:

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

On a photo taken inside a warehouse, Rekognition returned ten labels: `Architecture`, `Building`, `Warehouse`, `Box`, `Cardboard`, `Carton`, `Factory`, `Package Delivery`, `Person`, `Indoors`. They are stored in DynamoDB, so searching `warehouse` or `carton` returns this file even though neither word appears in its filename.

![The uploaded image with its Rekognition labels, shown in the app](/images/5-Workshop/5.4-serverless-backend/rekognition-labels.png)

{{% notice note %}}
`MinConfidence` is the minimum confidence percentage a label must reach to be returned, and `MaxLabels=10` is why exactly ten came back here. The threshold started at 70, which returned nothing for a screenshot of a diagram: there are few real-world objects in it for Rekognition to be confident about. Lowering it to 55 let the accurate-but-less-certain labels through. The photo above is the opposite case, full of physical objects, so it fills the ten-label cap easily.
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

#### Step 3: Bedrock answers questions about the document

This step uses the text the previous two steps stored in DynamoDB as the context for the model. The main feature is a document Q&A endpoint, `POST /files/{id}/ask`. It reads the extracted text from DynamoDB, wraps it with the question in a prompt that instructs the model to answer only from that document and in the same language as the question, and calls a Claude model on Amazon Bedrock. With no `question` in the body, the same handler summarizes the document instead.

There is also a library-wide endpoint, `POST /ask`. It scans every file in DynamoDB, ranks them by keyword overlap with the question, joins the text of the relevant files, and calls Bedrock. The response includes the numbered list of files the answer was drawn from.

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

The model id lives in the `BEDROCK_MODEL_ID` environment variable, so the model can change without editing code. The document text is capped at 20,000 characters before it goes into the prompt, which bounds the token count and cost per call and keeps the request within the model's context window.

{{% notice note %}}
**Design note.** The Bedrock integration uses the IAM `bedrock:InvokeModel` permission and an inference-profile model id. The `ask` handler wraps the `invoke_model` call: on success it returns the Claude answer, on error it returns HTTP 200 with a short English message instead of a 500, so a transient service error does not crash the demo.
{{% /notice %}}

#### Step 4: Store labels/text in DynamoDB

The AI output is written back once, so search and Q&A later read DynamoDB instead of re-calling Rekognition, Textract or Bedrock. The results are written back to the metadata item; the `search_blob` attribute powers content search, and the stored `text` is what the Bedrock Q&A reads. `size` also needs the `#sz` alias, because it is a DynamoDB reserved word just like `text`:

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

curl "$API/files/search?q=warehouse"

curl -X POST "$API/files/<id>/ask" -d '{"question":"Tai lieu noi ve gi?"}'
```

Search returned the image from the label `Warehouse`. On a `.txt` document, `ask` returned an answer taken from the document text, generated by Amazon Bedrock.

> Reference FCAJ lab on AI services: https://000056.awsstudygroup.com