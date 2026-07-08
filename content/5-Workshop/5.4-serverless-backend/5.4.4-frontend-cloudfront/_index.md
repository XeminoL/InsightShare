---
title: "Frontend S3 + CloudFront"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 5.4.4 </b> "
---

#### Goal

Serve InsightShare's static web interface from **Amazon S3** and deliver it over **Amazon CloudFront** (HTTPS, CDN).

#### The frontend

The interface is a single static `index.html` (vanilla HTML/CSS/JS): it uploads a file, shows the list with AI labels, offers a content-search box, a per-file download link (a presigned GET URL), and a box to ask a question about a document. It talks only to the API Gateway endpoint, so the same page works locally or on CloudFront.

The upload flow in the browser is a two-step call: ask the API for a presigned URL, then PUT the file straight to S3.

```javascript
// 1) ask the API for a presigned upload URL + metadata id
const r = await fetch(`${API}/files`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ filename: file.name, content_type: file.type }),
});
const { id, upload_url } = await r.json();

// 2) upload the file directly to S3 through the presigned URL
await fetch(upload_url, { method: "PUT",
  headers: { "Content-Type": file.type }, body: file });

// 3) trigger AI analysis
await fetch(`${API}/files/${id}/analyze`, { method: "POST" });
```

Content search is a single call to the search route:

```javascript
const res = await fetch(`${API}/files/search?q=` + encodeURIComponent(query));
render(await res.json());   // matches on AI labels + extracted text
```

Asking a question about a document is one more call to the `ask` route (Bedrock/Claude answers in Vietnamese; an empty question returns a summary):

```javascript
const res = await fetch(`${API}/files/${id}/ask`, {
  method: "POST", headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ question }),      // empty question -> document summary
});
const { answer } = await res.json();       // shown under the file
```

#### Host the frontend on S3

The static site is hosted on a separate S3 bucket with website hosting enabled:

```bash
aws s3api create-bucket --bucket insightshare-web-khang-2352464 --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3 website s3://insightshare-web-khang-2352464/ --index-document index.html
aws s3 cp index.html s3://insightshare-web-khang-2352464/index.html --content-type text/html
```

A public-read bucket policy is applied to this web bucket only (the file bucket in 5.3 stays private). The site is live at:

`http://insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com`

#### Distribute through CloudFront

A **CloudFront distribution** was created with the S3 website endpoint as the origin and `ViewerProtocolPolicy` set to `redirect-to-https`, so the site is delivered over HTTPS and the CDN edge cache.

```bash
aws cloudfront create-distribution \
  --origin-domain-name insightshare-web-khang-2352464.s3-website-ap-southeast-1.amazonaws.com \
  --default-root-object index.html
```

The distribution reached the `Deployed` state and serves the page over HTTPS at:

`https://<distribution>.cloudfront.net`

CloudFront serves the site over HTTPS in front of the S3 origin.

The live site, showing the stats bar, AI labels, thumbnails and label filter:

![InsightShare live site](/images/5-Workshop/5.4-serverless-backend/web-live-v3.png)

#### End-to-end test

The full flow was verified from the deployed web page through API Gateway:

- `POST /files` returned a presigned URL, and `PUT` to it returned **HTTP 200** (file landed in S3).
- `POST /files/{id}/analyze` returned real Rekognition labels.
- `GET /files/search?q=diagram` returned the image by its AI label, not its filename.
- `POST /files/{id}/ask` is wired to return a Vietnamese answer over a `.txt` document (or the token-quota fallback at HTTP 200 while the account's Bedrock inference quota is 0).
- The same steps run from the browser on the live site, so upload, AI analysis, content search and document Q&A all work end to end on AWS.

#### Summary

InsightShare runs end to end: the static frontend calls API Gateway → Lambda → S3 (presigned URL) + DynamoDB (metadata) + the AI layer. CloudFront is the delivery layer in front of the static site, deployed at `https://<distribution>.cloudfront.net`.
