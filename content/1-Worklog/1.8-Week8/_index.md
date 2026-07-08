---
title: "Week 8 Worklog"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 objectives

* Add the AI layer so the system understands file content.
* Use AWS managed AI services, no model training.

### Tasks during the week (20/07 - 24/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Study the FCJ AI services workshop; settle how to call Rekognition, Textract and Polly from boto3 and where to trigger them in the upload flow. | 20/07/2026 | 20/07/2026 | [AI services](https://000056.awsstudygroup.com) |
| Tue | Call Rekognition `detect_labels` (MaxLabels=10, MinConfidence=70) to tag images and `detect_moderation_labels` to flag sensitive content on upload. | 21/07/2026 | 21/07/2026 |  |
| Wed | Call Textract `detect_document_text` to extract text from PDFs and scanned images, reading the file straight from the S3 object. | 22/07/2026 | 22/07/2026 |  |
| Thu | Call Polly `synthesize_speech` (OutputFormat=mp3, a chosen Voice) to read the extracted text into an mp3 and store it on S3. | 23/07/2026 | 23/07/2026 |  |
| Fri | Extend the IAM Role with rekognition/textract/polly permissions; save the labels and extracted text to DynamoDB; test the whole pipeline. | 24/07/2026 | 24/07/2026 |  |

### Results achieved

1. Uploaded images are auto-labeled and moderated with Rekognition.
2. Documents have text extracted by Textract and read aloud by Polly.
3. Labels and text are saved to DynamoDB for search; AI permissions are least-privilege and within Free Tier.
