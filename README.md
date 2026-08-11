A serverless document assistant on AWS, built as the capstone for the First Cloud AI Journey (FCAJ) internship at AWS Vietnam.

Upload an image or a PDF and InsightShare reads it for you: it labels images, pulls text out of documents, and lets you find a file by a word that appears **inside** it rather than in its name. You can also ask a question in plain language and get an answer traced back to the file it came from.

**[Full report](https://xeminol.github.io/InsightShare/)** · **[Demo video](https://xeminol.github.io/InsightShare/videos/insightshare-demo.mp4)**

> **Note on the live demo.** The AWS resources were torn down after recording the demo, so the hosted page now shows the interface only — uploads will not work. Everything below is from the deployed system while it was running. `infrastructure.yaml` recreates the whole stack.

---

## Architecture

![InsightShare architecture on AWS](static/images/readme/architecture.png)

The browser loads a static page from S3 through CloudFront, signs in with Cognito, and calls API Gateway with the JWT. A single Python Lambda handles every route.

The part worth pointing at: **the file bytes never pass through Lambda.** Lambda only signs a presigned S3 URL and hands it back, so the browser transfers the file straight to the private bucket. Lambda then calls the AI services and writes labels and text into DynamoDB.

---

## The app, running

### 1. Pick a file and upload

![Upload screen with a file selected and the progress bar running](static/images/readme/app-01-upload.png)

The three cards state what the app does. Below them, a file is selected and `Reading content ...` shows while the browser PUTs it to S3 with the presigned URL.

### 2. AI reads it, and the file becomes searchable

![Library showing Rekognition labels and text extracted from a file](static/images/readme/app-02-ai-labels.png)

Rekognition returned nine labels for the screenshot — `Page`, `Text`, `File`, `Electronics`, `Mobile Phone`, `Phone`, `Webpage`, `Advertisement`, `Paper` — and they are stored, so searching `webpage` finds this file even though the word is nowhere in its filename.

Below it, `demo.txt` shows its extracted content: *"Hoa don thang 3 nam 2026. Tien dien 850000 dong."* Searching `dien` returns this file. The chips across the top count how many files carry each label.

### 3. Ask across the whole library

![Ask-the-library box and a PDF selected for upload](static/images/readme/app-03-library.png)

`ASK THE WHOLE LIBRARY` sends the question to Bedrock with the stored text as context and names the source file in the answer. Each file also has its own ask box for a single-document question.

---

## How it works

1. The static frontend signs in through Cognito and calls API Gateway with the JWT.
2. API Gateway validates the token with a JWT authorizer on the `$default` route, then routes to one Lambda. `OPTIONS` is left unauthenticated so the CORS preflight is not blocked.
3. **Upload:** Lambda returns a presigned S3 URL; the browser puts the file straight into the private bucket.
4. **Analyze:** triggered two ways — an `s3:ObjectCreated` event fires it automatically, and the browser can also call the analyze endpoint directly when it wants the result immediately. Both run the same code.
5. **Search:** Lambda matches the keyword against the stored labels and text in DynamoDB.
6. **Ask:** Lambda sends the question plus the stored text to Bedrock (Claude) and returns the answer with its source.

## Design notes

- **Region:** ap-southeast-1 (Singapore).
- **Cost:** about 1 USD/month at demo scale — everything is pay-per-use and idle costs nothing.
- **Auth:** a Cognito user pool; the JWT `sub` claim scopes every file to its owner, so a user only ever sees their own files.
- **IAM:** one least-privilege execution role. S3 and DynamoDB are scoped to the exact bucket and table ARNs; the AI actions use `*` because Rekognition, Textract and Bedrock have no resource-level permissions.
- **Fail-soft:** every AI call is wrapped, so a service being unavailable returns a short message instead of a 500 — upload, listing and search keep working.
- **Monitoring:** CloudWatch logs and metrics, two alarms on the function, and a three-widget dashboard.
- **Infrastructure as code:** `insightshare-aws/infrastructure.yaml` defines all 17 resources, so the stack is reproducible instead of clicked together.

## Known limits

Stated plainly rather than left for someone to discover:

- **Textract is not enabled on this account** (`SubscriptionRequiredException`). The PDF path is written and scoped in IAM but never ran end to end; plain text files are read directly from S3, which is what the demo shows.
- **Bedrock is credit-limited** on this account, so a day of testing hits the daily token quota and `InvokeModel` throttles until it resets.
- **Search uses `scan`, not `query`** — there is no GSI on the table. Fine at demo scale, wrong at real scale.
- No automated tests, no CI/CD for the infrastructure, no container build.

## Repository layout

| Path | What it is |
|---|---|
| `content/` | The report itself — worklog, proposal, blogs, events, workshop, self-evaluation, feedback, references |
| `static/images/` | Console screenshots and diagrams used in the report |
| `static/videos/` | The demo recording |
| `layouts/` | Hugo overrides on top of the theme |

The application source lives in the parent project: `insightshare-aws/` for the AWS version, `insightshare-app/` for the local FastAPI stage that came first.
