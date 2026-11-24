# Error Handling and Observability

## Error Strategy

A reliable n8n workflow classifies failures and routes them intentionally.

### Failure Classes

```json
{
  "error_category": "validation | auth | rate_limit | upstream_5xx | upstream_4xx | timeout | transform | duplicate | unknown",
  "retryable": true,
  "correlation_id": "evt_123",
  "source_node": "Action: Call API",
  "message": "safe human-readable message"
}
```

### Retry Policy

Use retries for:

- 429 rate limit,
- 408/timeout,
- 502/503/504,
- temporary DNS/network failures.

Do not retry blindly for:

- 400 validation failures,
- 401/403 auth/permission failures,
- 404 missing resource when the ID came from a trusted lookup,
- duplicate constraint errors without conflict handling.

## Error Trigger Workflow Pattern

Create a separate workflow for production-wide errors:

1. Error Trigger receives failed execution context.
2. Normalize error details.
3. Classify severity.
4. Redact sensitive data.
5. Write to logging target.
6. Notify owner/channel.
7. Optionally create incident/task.

## Dead-Letter Pattern

Use a dead-letter store when data cannot be processed automatically.

Dead-letter item shape:

```json
{
  "correlation_id": "evt_123",
  "workflow_name": "CRM / Webhook -> Upsert Lead / prod",
  "failed_at": "2026-01-01T00:00:00.000Z",
  "error_category": "validation",
  "retryable": false,
  "safe_payload": {},
  "operator_action": "fix source email and replay"
}
```

## Observability Summary

Every workflow should produce or log a summary object:

```json
{
  "workflow": "name",
  "execution_id": "={{ $execution.id }}",
  "correlation_id": "string",
  "status": "success | rejected | failed | partial",
  "started_at": "ISO-8601",
  "finished_at": "ISO-8601",
  "items_in": 0,
  "items_processed": 0,
  "items_failed": 0,
  "external_calls": 0,
  "warnings": [],
  "next_action": "none | retry | manual_review"
}
```

## Alerting Rules

Alert only when humans can act. Avoid noisy alerts for every minor branch.

Good alert criteria:

- auth failure,
- repeated 5xx from external API,
- dead-letter item created,
- unexpected payload schema,
- workflow deactivated unexpectedly,
- high failure rate,
- missing required credential.

## Troubleshooting Data to Preserve

- correlation ID,
- source system event ID,
- execution ID,
- sanitized input payload,
- node that failed,
- HTTP status and response body after redaction,
- retry count,
- branch taken.
