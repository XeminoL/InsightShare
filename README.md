# InsightShare

A serverless document assistant on AWS, built as the capstone for the First Cloud AI Journey (FCAJ) internship at AWS Vietnam.

## What it does

You upload an image or PDF, and InsightShare turns it into something you can search and question:

- Detects labels on images with Amazon Rekognition, and pulls text out of documents with Amazon Textract.
- Stores the extracted metadata (text, labels, file info) in DynamoDB.
- Lets you search your files by a word that appears inside them.
- Lets you ask questions about a document in plain language, answered by Amazon Bedrock (Claude).

The original files stay in a private S3 bucket. The browser never gets public URLs; it uploads and downloads through short-lived presigned URLs.

## How it works

End-to-end flow:

1. The frontend (a single static page) signs in through Cognito, then calls API Gateway with the JWT.
2. API Gateway validates the token with a JWT authorizer and routes everything to one Python Lambda.
3. Upload: Lambda returns a presigned S3 URL, and the browser puts the file straight into the private bucket.
4. Analyze: the browser calls the analyze endpoint; Rekognition handles images, Textract handles documents, and the labels and text go into DynamoDB.
5. Search: Lambda scans the stored labels and text and returns matching files.
6. Ask: Lambda sends the question to Bedrock (Claude) with the document's stored text as context, and returns the answer.

Design notes:

- Region: ap-southeast-1 (Singapore).
- Cost: roughly 1 USD/month at demo scale, since everything is pay-per-use and idle costs nothing.
- Auth: a Cognito user pool; the JWT `sub` claim scopes every file to its owner.
- IAM: one least-privilege execution role, scoped to the bucket and table the function touches.
- Monitoring: CloudWatch logs, metrics and two alarms on the function.