---
title: "Week 3 Worklog"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 objectives

* Build the InsightShare web app running locally.
* Separate the storage and AI layers for easy swapping later.
* Test the basic flow before moving to the cloud.

### Tasks during the week (15/06 - 19/06/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Initialize the FastAPI project with a virtualenv and `uvicorn`; design the upload and list endpoints (POST /files, GET /files) and the request/response schemas. | 15/06/2026 | 15/06/2026 | [FastAPI](https://fastapi.tiangolo.com/) |
| Tue | Write the upload and list APIs, storing files in a local folder for now and returning file id, name and size. | 16/06/2026 | 16/06/2026 |  |
| Wed | Build the static frontend (HTML/JS, fetch API): file picker, file list and an upload progress bar. | 17/06/2026 | 17/06/2026 |  |
| Thu | Extract storage and AI into Python interfaces (abstract base classes) so the cloud versions can drop in later; mock the AI results for now. | 18/06/2026 | 18/06/2026 |  |
| Fri | Refactor into layers (API / service / storage) and write unit tests with pytest for the file-handling code. | 19/06/2026 | 19/06/2026 |  |

### Results achieved

1. Have a working local app: upload and list files through the UI.
2. Storage and AI layers are cleanly separated, ready to be replaced later.
3. Code is organized by layers with unit tests for the core.
