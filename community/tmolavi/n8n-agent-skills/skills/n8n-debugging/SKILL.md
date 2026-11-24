---
name: n8n-debugging
description: Use when n8n workflows fail, return empty data, drop items silently, show expression errors, or when diagnosing executions with MCP explain/timeline tools.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Debugging

Author: [Taghi Molavi](https://molavi.pro)

## #1 failure mode: silent item loss

Items become 0 between nodes with **no hard error** (IF route, empty HTTP body, Code return shape, Merge wait forever, wrong expression → filter).

### Protocol
1. Open execution; table of nodes with items in/out + errors.
2. Prefer `execution_explain` / timeline tools when available.
3. Inspect first node where out < in (or out = 0).
4. Check expressions against **actual** incoming JSON (not assumed schema).
5. Verify connections (wrong branch, missing error wire).
6. Fix → pin data → retest that node → retest flow.

## Unresolved expressions
`={{ ... }}` left unresolved often means expression disabled or wrong field mode — lint/explain should flag.

## Parameter verification
- typeVersion mismatch after n8n upgrade
- Operation-dependent fields left empty
- Credential attached but wrong environment

## References
- `references/DIAGNOSTIC_TREE.md`
