---
title: "Blog 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
# Building multi-step applications with AWS Lambda Durable Functions

## Summary
AWS Lambda Durable Functions build multi-step applications and complex AI workflows with built-in state management and automatic failure recovery, without your own infrastructure.

## Main content

![Lambda Durable Functions diagram](/images/3-Blog/blog2_architecture.png)

### The problem
Building a long-running flow (an approval chain, a multi-step AI pipeline) forced developers to write a lot of code to store state, handle errors and wire services together, laborious and error-prone.

### The solution
Durable Functions bring that capability into Lambda with two main primitives:
1. **Steps:** each step is automatically checkpointed and retried on failure, using a "checkpoint and replay" mechanism.
2. **Waits:** you can pause execution for up to a year without paying for compute while waiting, ideal for waiting on human approval or an external API response.

It also has built-in idempotency (calling the same execution name returns the existing result, avoiding duplicate runs) and integrates with EventBridge to emit execution state.

### Notable points
- No need to build your own state storage or retry infrastructure.
- Long pauses incur no compute cost while waiting.
- Supports recent Node.js and Python versions; deployable with AWS SAM.

### Takeaways
For long business flows or AI workflows that need to wait, Durable Functions handle state and retries instead of a separate state store, and a human-in-the-loop approval step does not keep paying for idle compute.

## Reference
[Build multi-step applications and AI workflows with AWS Lambda durable functions](https://aws.amazon.com/blogs/aws/build-multi-step-applications-and-ai-workflows-with-aws-lambda-durable-functions/) (AWS News Blog)

## Post link
https://www.facebook.com/share/p/1D53oYGQmP/

## Images
![AWS Study Group post screenshot](/images/3-Blog/blog2_post.png)
