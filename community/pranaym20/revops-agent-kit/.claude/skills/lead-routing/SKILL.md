---
name: lead-routing
description: Score inbound leads against the ICP, route each to the right owner by tier and region, and write a sales handoff note. Use when someone asks to route, score, triage, qualify or assign leads, or pastes a lead list or CRM export.
---

# Lead routing

1. Find the lead file (CSV with `email,first_name,last_name,title,source,created_at,last_touch`). If the user pasted leads, save them to `leads.csv` first.
2. Run `python revops.py route <leads.csv> > routed.csv` from the kit root. Add `--ai` only if `ANTHROPIC_API_KEY` is set and the user wants rewritten notes.
3. Report: count per tier, every `HOLD` lead with its `missing` field, and the top 5 Tier A handoff notes verbatim.
4. If scores look wrong, change `icp.json` (weights, tiers, owners), never the code, and re-run. Say which weights you changed.

Never invent firmographics. A lead with no company match stays on HOLD.
