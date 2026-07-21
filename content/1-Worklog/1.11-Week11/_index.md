---
title: "Week 11 Worklog"
date: 2026-08-10
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 objectives

* Test end-to-end, tune performance and finish the system.
* Script the teardown and clean up all resources.

### Tasks during the week (10/08 - 14/08/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Write a deploy script (zip the function and run `aws lambda update-function-code`) so re-deploying after a code change is one command. | 10/08/2026 | 10/08/2026 |  |
| Tue | Run end-to-end and failure testing: large files, expired presigned links, wrong permissions, and concurrent uploads. | 11/08/2026 | 11/08/2026 | Postman |
| Wed | Measure AI processing time and search latency in CloudWatch and tune Lambda memory to bring the durations down. | 12/08/2026 | 12/08/2026 |  |
| Thu | Write the teardown script (cleanup-aws.ps1) to remove the whole stack; note Step Functions as a way to orchestrate the multi-step AI pipeline later. | 13/08/2026 | 13/08/2026 | [Modernize](https://cloudjourney.awsstudygroup.com/4-modernize/) |
| Fri | Record the demo video and finalize the bilingual report. | 14/08/2026 | 14/08/2026 |  |

### Results achieved

1. Deploy and teardown are scripted (zip + update-function-code; cleanup-aws.ps1), so re-deploying and cleaning up are single commands.
2. The system was tested end-to-end and across failure cases; Lambda memory was tuned from the CloudWatch durations to lower latency.
3. The system is complete, with a demo video and a bilingual report; Step Functions is noted as a direction to orchestrate the AI pipeline later.
