# Nodes, Data, and Expressions

## Node Categories

Use these categories when designing or auditing workflows:

- **Trigger nodes**: start the workflow. Examples: Webhook, Schedule Trigger, Manual Trigger, app triggers.
- **Action nodes**: call an app/API, create/update/delete records, send messages.
- **Flow-control nodes**: If, Switch, Merge, Wait, Loop Over Items, Stop and Error.
- **Data-shaping nodes**: Edit Fields/Set, Code, Aggregate, Split Out, Rename Keys, Filter, Sort.
- **Execution nodes**: Execute Workflow, Execute Sub-workflow Trigger, Error Trigger, Workflow Trigger.
- **AI nodes**: Chat Trigger, AI Agent, LLM/model nodes, vector store nodes, tool nodes, MCP Client/Server nodes.

## Item Model

n8n passes arrays of items. Design with this mental model:

```json
[
  {
    "json": {
      "id": "external-id",
      "status": "ready",
      "payload": {}
    },
    "binary": {}
  }
]
```

Rules:

- Normalize item shapes early.
- Do not depend on deeply nested vendor payloads across many nodes.
- Rename ambiguous keys.
- Preserve source payload under `raw` only when needed for audit.
- Make branch fields explicit, such as `operation`, `is_valid`, `error_category`, `should_retry`.

## Expression Rules

Use expressions for lightweight value mapping, not large business logic.

Good expression use:

```text
={{ $json.customer.email }}
={{ $now.toISO() }}
={{ $json.amount_cents / 100 }}
```

Risky expression use:

- long nested ternaries,
- business-critical validation hidden inside fields,
- secrets embedded in strings,
- complex loops better handled by Code or sub-workflow,
- expressions that assume a previous node always emitted exactly one item.

## Code Node Rules

Use Code nodes only when built-in nodes become unreadable. Keep them small, deterministic, and free of credentials.

Recommended Code node contract:

```javascript
return items.map((item) => {
  const input = item.json;
  return {
    json: {
      correlation_id: input.correlation_id,
      normalized: true,
      payload: input.payload,
      generated_at: new Date().toISOString()
    }
  };
});
```

Avoid:

- network calls inside Code nodes when HTTP Request can do it,
- reading local files unless the workflow is explicitly self-hosted and hardened,
- shell execution for business workflows,
- storing secrets in code,
- swallowing errors without a structured output.

## Branching

Use If for binary decisions and Switch for multi-route decisions.

Each branch should define:

- condition,
- expected input shape,
- output shape,
- error path,
- merge behavior.

## Merge Design

Before using Merge, define whether the workflow needs:

- append items,
- combine by position,
- combine by key,
- wait for both branches,
- preserve one branch as metadata.

Many n8n bugs come from merging arrays with mismatched item counts. When in doubt, normalize to a dictionary keyed by stable ID in a Code node or sub-workflow.

## Looping and Pagination

For paginated APIs:

- capture `nextCursor`, `nextPage`, or `hasMore`,
- set a max-page safety cap,
- log last cursor processed,
- use rate limit waits,
- design resume behavior.

Do not create unbounded loops.

## Webhook Response Design

For public webhook intake:

- validate payload before expensive operations,
- return 2xx only when accepted,
- return structured 4xx for invalid payloads when safe,
- avoid returning internal error details,
- include `correlation_id` in the response.

Example response:

```json
{
  "status": "accepted",
  "correlation_id": "evt_123",
  "message": "request accepted for processing"
}
```
