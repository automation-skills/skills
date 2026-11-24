---
name: n8n-error-handling
description: Use for production n8n workflows, webhook APIs, retries, error outputs, error workflows, dead-letter patterns, and correct HTTP response status shapes for callers.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Error Handling

Author: [Taghi Molavi](https://molavi.pro)

## Defaults

1. Fallible nodes (HTTP, DB, payments, mail) get explicit error strategy.
2. Webhook-facing workflows: **4xx** for caller mistakes, **5xx** for internal failures — never 200 on hard fail unless intentionally accepting then async-processing (document it).
3. Central **Error Workflow** for uncaught failures + alert channel.
4. Retries: exponential backoff only on idempotent ops; otherwise dead-letter.
5. Do not swallow errors in Code with empty catch.

## Node error output
- Enable continue on error output when you need a branch
- Wire the error branch to normalize → notify → optionally respond

## References
- `references/WEBHOOK_API_SHAPES.md`
- `references/ERROR_WORKFLOWS.md`
