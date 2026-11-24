# Scaling and Self-Hosting

## Purpose

Use this reference when the user mentions self-hosting, high volume, queue mode, workers, concurrency, binary data, webhook processors, scaling, performance, or production infrastructure.

## Deployment Modes

Document whether the workflow is intended for:

- n8n Cloud,
- local/self-hosted single instance,
- self-hosted queue mode,
- enterprise source control/environments,
- multi-main high availability.

## Queue Mode Design Implications

Queue mode changes operational assumptions:

- executions run on workers,
- Redis is used as broker,
- all workers need the same encryption key to access credentials,
- filesystem-local binary data is not appropriate for queue mode persistence,
- workers need unique event log paths if sharing filesystem,
- webhook processing can be separated from main process,
- concurrency must be tuned based on external API and database constraints.

## Binary Data

If a workflow handles files:

- document binary size limits,
- prefer external storage for scalable deployments,
- avoid keeping large binary payloads in execution logs,
- redact file content from notifications,
- make cleanup/retention explicit.

## Performance Checklist

- Avoid unnecessary item fan-out.
- Batch external API calls when supported.
- Use pagination deliberately.
- Limit loops.
- Move reusable heavy logic to sub-workflows.
- Avoid giant pinned data in production exports.
- Avoid Code nodes for simple mappings.
- Add wait/backoff to protect external APIs.
- Use queue mode for high execution volume.

## Environment Variables

Document variable names only:

```text
N8N_ENCRYPTION_KEY
N8N_HOST
N8N_PROTOCOL
WEBHOOK_URL
QUEUE_BULL_REDIS_HOST
EXECUTIONS_MODE
```

Do not include values in generated docs or public workflow JSON.

## Source Control and Environments

For mature setups:

- keep dev/staging/prod separate,
- do not edit production directly,
- export workflows before major changes,
- version workflow JSON in a private repo if it contains sensitive names,
- publish sanitized templates separately,
- use release notes and rollback steps.
