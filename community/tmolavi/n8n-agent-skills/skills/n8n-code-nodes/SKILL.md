---
name: n8n-code-nodes
description: Use when considering a Code node, writing JavaScript/Python in n8n, fixing Code return shape errors, or deciding between expression, Edit Fields, and Code.
license: MIT
metadata:
  author: Taghi Molavi
  homepage: https://molavi.pro
  version: "1.0.0"
---

# n8n Code Nodes

Author: [Taghi Molavi](https://molavi.pro)

## Decision tree

1. Can an **expression** on the consumer node do it? → do that.
2. Can **Edit Fields** + arrow function cover it? → prefer that.
3. Need multi-source aggregation, npm-less algorithms, or complex branching on items? → **Code**.
4. Need external libs / long-running jobs? → sub-workflow, custom node, or external service — not giant Code.

**Default language: JavaScript.** Use Python only if user insists or ecosystem requires it.

## Return contract (JS)

Code must return items in n8n shape:

```js
return items.map(item => ({
  json: { ...item.json, score: 1 },
  pairedItem: item.pairedItem, // preserve when possible
}));
```

Or:

```js
return [{ json: { ok: true, count: items.length } }];
```

### Common breaks
- Returning raw objects/arrays without `{ json: ... }`
- Forgetting to handle empty `items`
- Mutating shared state across runs
- Trying to `require` disallowed modules
- Hiding secrets in code

## Run once for all items vs per item
- Per item: simpler expressions pairing
- All items: aggregation, dedup, sort — then output one or many

## Pair with
- `n8n-expressions` for lighter transforms
- `n8n-debugging` when Code "fails silently" (often empty return)
