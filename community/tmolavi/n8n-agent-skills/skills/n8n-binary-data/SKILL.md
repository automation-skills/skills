---
name: n8n-binary-data
description: Use when handling files, images, PDFs, attachments, binary properties, vision model inputs, or merging binary with JSON context in n8n.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Binary & Files

Author: [Taghi Molavi](https://molavi.pro)

## Basics
- Items can carry `binary` keys alongside `json`
- Downstream must reference the correct binary property name
- Some nodes clear binary — plan Merge/NoOp checkpoints

## Patterns
- Download → transform → upload/store
- Email attachments → extract → route by mime
- Vision: ensure model node receives binary or URL per node docs
- For agent tools needing files, document size limits

## Gotchas
- CDN/public URL requirements for some AI nodes
- Base64 in JSON blows tokens — keep binary binary
- Pair context fields via Merge before multi-input consumers
