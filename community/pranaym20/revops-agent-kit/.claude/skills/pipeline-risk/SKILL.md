---
name: pipeline-risk
description: Review an opportunity pipeline, flag at-risk deals with plain-English reasons, and write a manager-ready deal review. Use for pipeline reviews, forecast calls, deal inspection, "which deals are slipping", or any opportunities CSV/CRM export.
---

# Pipeline risk review

1. Expect a CSV with `id,account,owner,stage,amount,close_date,stage_entered,last_activity,next_step,decision_maker,forecast_category`. Map other column names to these first and tell the user the mapping.
2. Run `python revops.py risk <opportunities.csv>` (add `--today YYYY-MM-DD` to replay a past review, `--ai` for a CRO summary).
3. Lead with the headline line (high-risk count and dollars at risk), then the table, then the by-owner list.
4. Stage-age norms live in `STAGE_MAX_DAYS` in `revops.py`. If the user's sales cycle differs, say so and offer to adjust.

Every flag must trace to a rule. Don't add opinions the data doesn't support.
