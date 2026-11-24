---
name: n8n-ai-seo-geo-automation
description: Use when building n8n workflows for SEO, AEO, GEO, AI visibility monitoring, content pipelines, schema/JSON-LD checks, rank/citation tracking, or integrating Molavi GEO tools (geo-scope, mcp-geo-server) with n8n.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
  keywords: [seo, aeo, geo, ai-visibility, n8n, molavi]
---

# n8n for SEO / AEO / GEO Automation

Author: [Taghi Molavi](https://molavi.pro) — Senior SEO & GEO Systems Architect

## Why this skill exists

AI coding agents often build brittle "SEO bots". This skill encodes **safe, measurable** automation patterns that pair n8n with modern AI-visibility work.

## Core workflow families

| Family | Example trigger | Outputs |
|---|---|---|
| **AI visibility pulse** | Schedule daily | Brand mention/citation checks → sheet + alert |
| **Content ops** | CMS webhook | Brief → draft QA → schema suggest → Slack approve |
| **Technical watch** | Crawl finish | Broken schema, indexation anomalies |
| **Entity / source monitoring** | RSS + Search | Competitor citations, Reddit/UGC spikes |
| **Lead + twin** | Form submit | Enrich → CRM → first response (human gate) |

## Pair with Molavi open source

- [geo-scope](https://github.com/tmolavi/geo-scope) — benchmark prompts & SoM-style experiments
- [mcp-geo-server](https://github.com/tmolavi/mcp-geo-server) — GEO/RAG readiness audits via MCP
- Site: [molavi.pro](https://molavi.pro)

Pattern: **n8n orchestrates**; GEO MCP/CLI measures; humans approve irreversible publishes.

## Non-negotiables for SEO automations
1. No mass-generated thin content pushed live without human review.
2. Log every prompt/model/version used for GEO experiments (reproducibility).
3. Separate measurement workflows from mutation workflows.
4. Respect robots/ToS/rate limits of sources.
5. Store evidence links for any "we are cited" claim.

## Suggested n8n building blocks
- Schedule / Webhook triggers
- HTTP Request + credentials toward Search Console, CMS, analytics
- Data Tables for prompt sets & run history
- AI Agent only behind approval gates for publish actions
- Error workflow → Telegram/Slack

## References
- `references/GEO_PIPELINE.md`
- `references/CONTENT_GUARDRAILS.md`
