---
name: crm-copilot
description: Answer "which leads need follow-up today?" from routed leads, rank them by SLA breach, and create follow-up tasks only after explicit human approval. Use for follow-up queues, SLA breaches, speed-to-lead checks, or daily rep task lists.
---

# CRM copilot (human in the loop)

1. Needs `routed.csv` from the lead-routing skill. If missing, run that first.
2. Run `python revops.py followups routed.csv` and show the numbered list as-is.
3. Ask the user which numbers to action. Do not choose for them.
4. Only after they answer, run `python revops.py followups routed.csv --approve <numbers>`. That is the only write path; it appends to `audit.jsonl`.

Never pass `--approve` on your own initiative, even if the user said "handle it" earlier in the session. Ask for the numbers.
