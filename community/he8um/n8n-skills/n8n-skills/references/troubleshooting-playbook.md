# Troubleshooting Playbook

## Import Fails

Check:

- JSON parses correctly.
- Top-level `nodes` is an array.
- Top-level `connections` is an object.
- Node names are unique.
- Connections reference existing node names.
- Node `type` values exist in the user's n8n version.
- Node `typeVersion` is supported.
- Credentials are placeholders or valid local credentials.
- Pinned data does not contain incompatible binary payloads.

## Webhook Does Not Fire

Check:

- workflow is active for production webhook URL,
- correct test vs production URL,
- HTTP method matches,
- path matches,
- reverse proxy forwards headers/body,
- public URL configured correctly,
- response mode is appropriate,
- request body is valid JSON when expected.

## Expression Error

Check:

- expected previous node emitted items,
- item path exists,
- array vs object mismatch,
- branch item counts changed,
- Code node preserved item links,
- expression is evaluated in the intended node context.

## Merge Produces Wrong Items

Check:

- branch item counts,
- merge mode,
- stable join keys,
- item order assumptions,
- missing items from filtered branches.

## API Node Fails

Check:

- credential is selected,
- base URL and path are correct,
- query parameters are encoded,
- body content type is correct,
- rate limit response handling,
- pagination cursor,
- response format expected by downstream nodes.

## Workflow Creates Duplicates

Check:

- stable upsert key,
- lookup-before-create branch,
- source event replay behavior,
- duplicate webhook delivery,
- race condition between parallel executions,
- target API idempotency key support.

## Slow Workflow

Check:

- too many sequential API calls,
- no batching,
- huge pinned data,
- heavy Code node,
- large binary payloads,
- external API rate limit waits,
- worker concurrency,
- database bottleneck.

## AI Workflow Misbehaves

Check:

- untrusted content reached instruction area,
- missing structured output validation,
- tool parameters not allowlisted,
- no human approval gate,
- model output directly used in write node,
- prompt contains secrets,
- no fallback route.
