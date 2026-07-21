---
title: "Blog 1"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# An AI meeting summarization system with Amazon Bedrock and Amazon Transcribe

## Summary
An AWS serverless system that turns meeting recordings into structured summaries, built from several AWS services with per-use billing.

## Main content

![Meeting summarization architecture](/images/3-Blog/blog1_architecture.png)

### The problem
Organizations have a lot of audio data (meetings, interviews) but struggle to extract useful information: audio is unstructured and replaying it is time-consuming.

### The solution
The system processes in an automated flow:
1. **Amazon Transcribe** converts the speech in the recording into text.
2. **Amazon Bedrock** (a Claude model) reads the text and generates a structured summary: stakeholders, objectives, action items and technical requirements.
3. **AWS Step Functions** orchestrates the whole flow, **Lambda** handles each step, and data is stored in **S3** and **DynamoDB**.
4. The frontend is a React app, authenticated with **Cognito**, querying data via **AppSync** (GraphQL).

### Results
- A concise, clearly structured summary instead of replaying the whole meeting.
- A serverless architecture that scales automatically with load.
- The source article reports an average cost of about $0.98 per meeting.

### Takeaways
Asking the model for a summary in a fixed structure (stakeholders, actions, requirements) gives directly usable output rather than a generic paragraph. The post also shows how Transcribe, Bedrock, Step Functions, Lambda, S3 and DynamoDB fit together in one flow.

## Reference
[Build an AI-powered automated summarization system with Amazon Bedrock and Amazon Transcribe](https://aws.amazon.com/blogs/machine-learning/build-an-ai-powered-automated-summarization-system-with-amazon-bedrock-and-amazon-transcribe-using-terraform/) (AWS Machine Learning Blog)

## Post link
https://www.facebook.com/share/p/1DGNJvxwZz/

## Images
![AWS Study Group post screenshot](/images/3-Blog/blog1_post.png)
