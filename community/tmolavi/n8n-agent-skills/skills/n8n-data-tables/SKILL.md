---
name: n8n-data-tables
description: Use when designing or querying n8n Data Tables for durable state, deduplication, checkpoints, lightweight CRM-like storage, or idempotency keys inside workflows.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Data Tables

Author: [Taghi Molavi](https://molavi.pro)

## When to use
- Dedup processed IDs
- Backfill checkpoints
- Small operational datasets without external DB
- Feature flags / allowlists for agents

## Schema design
- Stable primary key (external id)
- `processed_at`, `status`, `attempts`
- Avoid unbounded blob columns; store pointers

## Ops
- Index/filter by status for workers
- TTL/archive strategy for high-volume tables
- Do not treat as analytics warehouse
