# Integrations, API, Webhooks, and Sync Patterns

## HTTP/API Node Design

For each external API call document:

- method,
- URL template,
- authentication type,
- headers,
- query params,
- body schema,
- pagination,
- rate limits,
- success response shape,
- error response shape,
- retry policy,
- idempotency key.

## API Sync Pattern

Use this flow for most source-to-target syncs:

1. Trigger: schedule/manual/webhook.
2. Load cursor/checkpoint.
3. Fetch page from source API.
4. Normalize source items.
5. Validate required fields.
6. Lookup target by stable external ID.
7. Branch create/update/no-change.
8. Write target record.
9. Log per-item result.
10. Save checkpoint.
11. Emit summary.

## Upsert Contract

```json
{
  "source_system": "string",
  "source_id": "string",
  "dedupe_key": "source_system:source_id",
  "operation": "create | update | skip | conflict",
  "target_id": "string|null",
  "changes": {},
  "observed_at": "ISO-8601"
}
```

## Pagination Pattern

- Prefer cursor pagination over page numbers when available.
- Save cursor only after successful processing of a page.
- Add max page guard.
- Add wait/backoff between pages if API is rate-limited.
- If using page numbers, capture last successful page and total pages if available.

## Webhook Intake Pattern

1. Receive request.
2. Verify method and signature.
3. Parse payload.
4. Normalize into canonical event object.
5. Respond quickly if long processing is needed.
6. Process asynchronously or call sub-workflow.
7. Log final result.

## cURL Import Risk

When users import cURL into HTTP Request nodes, auth headers may be carried into workflow JSON. Generated public workflows must avoid this; use credential placeholders instead.

## n8n API Usage

When using the n8n public API:

- use API key authentication documented for the instance,
- use pagination when listing large collections,
- do not hardcode API keys,
- prefer workflow export/import through supported UI/CLI/API paths,
- maintain environment-specific base URLs separately.

## Integration Spec Template

```markdown
### Integration: <name>

- Source system:
- Target system:
- Trigger:
- Auth method:
- Required credentials:
- Input schema:
- Output schema:
- Upsert key:
- Pagination:
- Rate limit:
- Retry policy:
- Error policy:
- Test payload:
```
