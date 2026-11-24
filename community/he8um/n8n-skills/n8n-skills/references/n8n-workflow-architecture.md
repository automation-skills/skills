# n8n Workflow Architecture

## Purpose

Use this reference when designing a workflow from scratch or refactoring an existing one. Treat each workflow as a directed graph of nodes, data contracts, external systems, and operational controls.

## Architecture Principles

### 1. Design the workflow as a system, not a canvas

A production n8n workflow needs these layers:

- **Trigger layer**: webhook, schedule, app event, manual trigger, queue/event source, or sub-workflow trigger.
- **Ingress validation layer**: normalize, validate, and reject invalid payloads early.
- **Enrichment layer**: fetch missing context from APIs, databases, files, or internal systems.
- **Decision layer**: If/Switch branches based on explicit, typed fields.
- **Action layer**: create/update/delete/send operations.
- **Persistence layer**: logs, state, dedupe keys, checkpoints, dead-letter records.
- **Response layer**: webhook response, email/message, task update, or final item output.
- **Observability layer**: summary object, execution ID, correlation ID, error category, retries, and operator notes.

### 2. Split workflows by lifecycle and blast radius

Prefer separate workflows for:

- public webhook intake,
- validation and normalization,
- business action execution,
- error handling and alerts,
- scheduled reconciliation,
- backfill/migration,
- reusable sub-workflows such as `normalize-contact`, `upsert-record`, `send-alert`, `http-with-retry`.

Large workflows become fragile when every branch shares the same state. Use sub-workflows when a graph becomes too wide, when a piece must be reused, or when a workflow needs an explicit input/output contract.

### 3. Build an explicit data contract

For every external boundary, define:

```json
{
  "correlation_id": "string",
  "source": "string",
  "event_type": "string",
  "entity_id": "string",
  "occurred_at": "ISO-8601 string",
  "payload": {},
  "metadata": {}
}
```

A good n8n workflow should not depend on ambiguous previous-node payloads. Normalize early and pass stable keys forward.

### 4. Separate transient and permanent failures

- **Transient**: 429, 5xx, network timeout, temporary auth refresh issue.
- **Permanent**: validation failure, 400/404 due bad payload, missing required data, permission denied, unsupported event type.
- **Unknown**: unexpected schema, unknown API behavior, malformed response.

Transient errors get retry/backoff. Permanent errors get dead-letter and human-readable failure messages. Unknown errors get alerting plus sample payload.

### 5. Every write needs idempotency

For create/update/sync workflows, define:

- stable external key,
- dedupe lookup step,
- upsert branch,
- safe replay behavior,
- conflict policy,
- log record per attempted write.

Never design an action workflow that can create duplicates when the same event is replayed.

## Recommended Workflow Naming

Use a predictable pattern:

```text
<domain> / <trigger> -> <outcome> / <env>
```

Examples:

```text
CRM / Webhook -> Upsert Lead / prod
Finance / Schedule -> Reconcile Invoices / prod
Ops / Manual -> Backfill Missing Owners / one-shot
Shared / Subworkflow -> HTTP Request With Retry
```

## Node Naming Standard

Use verb-first names:

- `Trigger: Receive Lead Webhook`
- `Validate: Required Lead Fields`
- `Normalize: Lead Payload`
- `Lookup: Existing Lead`
- `Branch: Create Or Update`
- `Action: Create Lead`
- `Action: Update Lead`
- `Log: Write Execution Summary`
- `Respond: Accepted`
- `Error: Stop With Validation Message`

## Canvas Layout Standard

- Left to right for primary path.
- Put validation and rejection branches near the trigger.
- Put happy path in the center.
- Put error/dead-letter branches below.
- Put alert/reporting branches above or below, not in the main line.
- Use sticky notes to mark credential setup, manual configuration, and intentional placeholders.

## Production Readiness Criteria

A workflow is production-ready only when it has:

- importable JSON or explicit manual setup steps,
- test payloads,
- credential map,
- validation path,
- error path,
- idempotency strategy,
- observability summary,
- activation/deactivation notes,
- ownership and change log.
