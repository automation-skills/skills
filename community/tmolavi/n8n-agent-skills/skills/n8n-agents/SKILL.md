---
name: n8n-agents
description: Use when building n8n AI Agent / LangChain clusters — tools, memory, RAG, structured output, system prompts, sub-workflow-as-tool, or fixing agent connection topology (ai_languageModel, ai_tool, ai_memory).
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n AI Agents

Author: [Taghi Molavi](https://molavi.pro)

## Topology (critical)

AI Agent is a **cluster**, not a single node:

- Chat model → connection type `ai_languageModel`
- Tools → `ai_tool` (HTTP tools, Code tools, Workflow tools, Vector retrieve, …)
- Memory → `ai_memory`
- Output parser → `ai_outputParser` when structured JSON required

Missing language model = agent that cannot run. Lint should catch this.

## Tool quality = prompt quality

Tool **names** and **descriptions** are part of the system prompt. Vague tools → tool thrash.

Good description:
> `create_draft_invoice` — Create a draft invoice in Stripe for an existing customer_id. Input: customer_id (string), line_items (array of {price_id, quantity}). Does not finalize or charge.

Bad: `helper` — does stuff with billing.

## System prompt design
- Role, tools available, refusal rules, output format
- Keep business policies explicit (PII, refunds, legal)
- Prefer structured output parser over "please return JSON"

## Patterns
| Pattern | When |
|---|---|
| Router agent + specialist sub-agents | Multi-domain support |
| Sub-workflow as tool | Reuse audited business logic |
| RAG retrieve → agent | Docs/help centers |
| Human review gate | Irreversible actions |
| Memory window | Multi-turn chat widgets |

## Safety
- Destructive tools require confirmation tool or human-in-the-loop
- Separate staging tools/credentials
- Log tool calls for audit

## References
- `references/TOOLS.md`
- `references/RAG_AND_MEMORY.md`
- `references/STRUCTURED_OUTPUT.md`
