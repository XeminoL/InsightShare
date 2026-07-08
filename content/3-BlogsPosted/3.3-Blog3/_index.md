---
title: "Blog 3"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---
# AWS Lambda MicroVMs: isolated, stateful sandboxes for running untrusted code

## Summary
AWS Lambda MicroVMs, a serverless compute primitive announced in June 2026, runs user-supplied or AI-generated code in isolated, stateful environments: VM-level isolation, near-instant launch from a snapshot, and up to 8 hours of preserved state, without managing servers.

## Main content

![AWS Lambda MicroVMs architecture](/images/3-Blog/blog3_architecture.png)

### The problem
Applications such as AI coding assistants, interactive code interpreters and vulnerability scanners need to run code they do not fully trust. Existing options each force a tradeoff: a full VM isolates strongly but takes minutes to boot; a container starts fast but needs much hardening before it is safe for untrusted code; a normal Lambda function starts quickly but is built for short, stateless, event-driven work, not long stateful sessions. Lambda MicroVMs fills this gap.

### How it works
1. A Dockerfile plus the application code is packaged as a zip in Amazon S3. Lambda runs the Dockerfile, starts the application, and takes a Firecracker snapshot of the running memory and disk. This becomes a MicroVM Image.
2. A session starts from the image ARN and an idle policy. Lambda assigns a unique ID and returns a dedicated HTTPS endpoint, with no networking to configure. The MicroVM resumes from the snapshot instead of booting cold, so launch and resume are near-instant.
3. Each MicroVM keeps its memory, disk and running processes across sessions. When idle it is suspended automatically to save cost, and restores full state on the next request.

### Notable points
- Isolation is at the virtual machine level, powered by Firecracker, the same technology behind trillions of Lambda invocations each month. There is no shared kernel between sessions.
- State is preserved for up to 8 hours per session. Installed packages, loaded models and working files survive suspend and resume.
- Each MicroVM can use up to 16 vCPUs, 32 GB memory and 32 GB disk (ARM64). It is available in five regions at launch, including Asia Pacific (Tokyo).
- It complements Lambda functions rather than replacing them: a function can invoke a MicroVM when it needs isolated, stateful execution.

### Takeaways
MicroVMs close the gap between Lambda and EC2: VM-level isolation without minute-long boots or managing servers. For running AI-generated or user-supplied code safely, isolation, fast resume and preserved state in one managed primitive is a usable building block.

## Reference
[Run isolated sandboxes with full lifecycle control: AWS Lambda introduces MicroVMs](https://aws.amazon.com/blogs/aws/run-isolated-sandboxes-with-full-lifecycle-control-aws-lambda-introduces-microvms/) (AWS News Blog)

## Post link
https://www.facebook.com/share/p/18fFbiYK9m/

## Images
![AWS Study Group post screenshot](/images/3-Blog/blog3_post-v2.png)
