# GEO measurement pipeline (n8n)

1. **Prompt set** in Data Table or CSV (intent strata).
2. **Runner** calls measurement API/CLI/MCP (geo-scope or custom).
3. **Normalize** model outputs → brand hit, rank position, citations.
4. **Diff** vs previous run; alert on drop.
5. **Evidence pack** links stored for client reporting.
6. **Action tickets** (not auto-rewrite entire site).

Never treat a single-run SoM number as universal truth — document sample size and date.
