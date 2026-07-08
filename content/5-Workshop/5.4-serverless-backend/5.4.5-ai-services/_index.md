---
title: "Add AI: Rekognition, Textract, Polly"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Goal

Add the **AI layer** so InsightShare understands file content instead of just storing it:

- **Amazon Rekognition**: label images (`DetectLabels`) and moderate content (`DetectModerationLabels`).
- **Amazon Textract**: extract text from PDFs / scanned images (`DetectDocumentText`).
- **Amazon Polly**: turn text into audio (`SynthesizeSpeech`).

All three are ready-to-call services, no model training. The IAM permissions Lambda needs are in section 5.5.

#### Step 1: Image → Rekognition labels + moderation

After an image is uploaded, Lambda calls Rekognition on the S3 object:

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

For a test image, Rekognition returned labels such as `Diagram`, `Text`, `Network`, `Chart`, `Webpage`. These are stored in DynamoDB and become searchable, so a query for `diagram` finds the image even though the word is not in its filename.

{{% notice note %}}
`MinConfidence` started at 70 and returned no labels for a diagram-style image (few real-world objects). Lowering it to 55 produced accurate labels, so the threshold is worth tuning to the kind of content you expect.
{{% /notice %}}

#### Step 2: Document → text extraction

For a `.txt` file the content is read directly from S3 (no OCR needed). For a PDF or scanned image, Lambda calls Textract:

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
        text = fname            # fail soft if Textract is not available
        labels = ["Document", "Textract unavailable"]
```

{{% notice warning %}}
**Limitation.** Textract returns `SubscriptionRequiredException: The AWS Access Key Id needs a subscription for the service`, including when called directly from the CLI. This is an account-level limit (Textract is not enabled on this credit-based account), not a code error. The code fails soft: `.txt` files are read directly and the flow keeps working, while the Textract call is wrapped so an unavailable service does not break analysis. When the account gains access, the same code path activates automatically.
{{% /notice %}}

#### Step 3: Polly turns text into audio

When there is text, an audio version is generated on demand with the `Joanna` neural voice:

```python
audio = polly.synthesize_speech(
    Text=text[:3000], OutputFormat="mp3", VoiceId="Joanna", Engine="neural")
# audio["AudioStream"].read() -> mp3 bytes, saved to S3 and served via /files/{id}/audio
```

`SynthesizeSpeech` produced an mp3 stored in S3 and returned through `/files/{id}/audio`. Rekognition and Polly run for real on the account; only Textract is limited (see above).

#### Step 4: Store labels/text in DynamoDB

The AI results are written back to the metadata item (section 5.4.3); the `search_blob` attribute (labels + text, lowercased) powers content search:

```python
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s",
    ExpressionAttributeNames={"#t": "text"},
    ExpressionAttributeValues={":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower()},
)
```

#### Test

```bash
# Upload an image, then analyze -> real Rekognition labels
curl -X POST "$API/files/<id>/analyze"
# -> {"labels": ["Diagram","Text","Network","Chart", ...], "audio_available": true}

# Search by an AI label (not in the filename)
curl "$API/files/search?q=diagram"
# -> the uploaded image is returned
```

The search returned the image by an AI label that is not in its filename.

> Reference FCJ lab on AI services (Rekognition/Polly/Lex): https://000056.awsstudygroup.com
