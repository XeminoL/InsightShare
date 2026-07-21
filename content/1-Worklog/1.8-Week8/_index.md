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
| Mon | Study the FCJ AI services workshop; settle how to call Rekognition and Textract from boto3 and where to trigger them in the upload flow. | 20/07/2026 | 20/07/2026 | [AI services](https://000056.awsstudygroup.com) |
| Tue | Call Rekognition `detect_labels` (MaxLabels=10, MinConfidence=70) to tag images on upload; store the labels for search. | 21/07/2026 | 21/07/2026 |  |
| Wed | Call Textract `detect_document_text` to extract text from PDFs and scanned images; read `.txt` files straight from the S3 object. | 22/07/2026 | 22/07/2026 |  |
| Thu | Add the `POST /files/{id}/ask` endpoint: send the extracted text plus a question to a Claude model on Amazon Bedrock (`invoke_model`) and return the answer in the same language as the question; with no question, summarize instead. | 23/07/2026 | 23/07/2026 | [Bedrock](https://docs.aws.amazon.com/bedrock/) |
| Fri | Extend the IAM Role with rekognition/textract/bedrock permissions; save the labels and extracted text to DynamoDB; test the whole pipeline. | 24/07/2026 | 24/07/2026 |  |

### Results achieved

1. Uploaded images are auto-labeled with Rekognition; documents have text extracted by Textract (`.txt` read directly).
2. A document Q&A endpoint answers questions and summarizes in the same language as the question via Bedrock/Claude, wrapped to fail soft so an unavailable AI service does not break the analysis flow.
3. Labels and text are saved to DynamoDB for search; AI permissions are least-privilege.
