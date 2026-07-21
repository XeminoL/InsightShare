---
title: "Event 1"
date: 2026-06-06
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

| Field | Detail |
| --- | --- |
| Event name | FCAJ Community Day - June 2026 |
| Date & time | [DD/MM/2026, 09:00-12:00] |
| Location | Floor 26 & 36, Bitexco Financial Tower, 2 Hai Trieu St, Saigon Ward, HCMC |
| Role | Attendee |

### Main content
A monthly community event where speakers from several companies shared hands-on experience building AI on AWS. Five talks:

- **Career in the AI era (Cloud Thinker).** A founder traced his path from on-prem server operations to AWS Solution Architect, then to running a startup. His point for students: the job market for plain developer and cloud roles is tightening as companies adopt AI, so build real experience early and learn to work well with AI tools. He then walked through an agentic platform that assists engineers on incident investigation, code review before production, cost optimization (FinOps) and security testing, and explained why they landed on a well-designed single agent over multi-agent for most tasks.
- **Voice AI (Renova Cloud, RhoAI).** Two architectures were compared: direct speech-to-speech, and speech-to-text then LLM then text-to-speech. The three-stage design is preferred for Vietnamese because Vietnamese is a low-resource language and because routing through text lets you control what the model says and use tool calling reliably. Handling turn-taking, interruption, gender detection and regional accents were shown as the hard parts of a production banking assistant.
- **DevOps Agent / AIOps (Cloud Kinetic).** An agent that investigates incidents automatically: it learns a topology of the system, then classifies, investigates root cause, and proposes (not executes) a fix. A live demo simulated a DDoS attack that pushed latency from 6s to 12s; the agent traced it to 1000 requests/second on the load balancer and suggested the mitigation steps. Reported results across cases: 75-77% lower mean time to resolution.
- **Amazon Quick for HR (Noventiq).** An agentic assistant that screens CVs against a job description, builds a reusable review skill, drafts job descriptions, and produces a scored talent report. The takeaway for candidates: CVs are increasingly screened by AI, so write them to match the job description.
- **Amazon Quick with private MCP (Renova Cloud).** How to connect Quick to a third-party MCP server without going over the public internet: place the MCP server in a private subnet, reach it through a VPC connection with an internal DNS resolver and an ALB doing TLS, and authenticate with Cognito. Cost drivers (Route 53 resolver, ALB, data transfer) were discussed frankly.

### What I learned
The recurring message across every talk was that AI assists engineers, it does not replace them: the DevOps agent only recommends, the voice assistant hands off to a human, and the HR tool still leaves the decision to people. That matched the direction of my own project, where the AI layer analyzes and answers but the user stays in control. The private-MCP talk was the most useful technically. It showed that "connect the AI to a third-party service" is not the whole job in an enterprise: keeping that connection off the public internet, with its own DNS, TLS and authentication, is what makes it acceptable to a bank. It also gave me a clearer sense of career direction beyond code, since two of the speakers work in solution and sales roles built on top of the same cloud knowledge.

### Photos
[Insert 1-2 photos or a video link as evidence.]
