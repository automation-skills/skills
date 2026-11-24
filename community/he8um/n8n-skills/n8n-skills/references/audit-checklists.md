# Audit Checklists

## Workflow Design Audit

- [ ] Workflow has a clear owner and purpose.
- [ ] Trigger is appropriate and documented.
- [ ] Workflow name follows naming convention.
- [ ] Nodes are named by responsibility.
- [ ] Data contract is documented.
- [ ] Required fields are validated near ingress.
- [ ] Branch logic is explicit.
- [ ] Merge logic is safe.
- [ ] Large repeated logic is moved to sub-workflows.
- [ ] Workflow has manual test payloads.

## Import JSON Audit

- [ ] JSON parses.
- [ ] `nodes` exists and is an array.
- [ ] `connections` exists and is an object.
- [ ] Node names are unique.
- [ ] Node IDs are unique.
- [ ] Connections reference existing nodes.
- [ ] No real credentials or tokens.
- [ ] `active` is false or omitted.
- [ ] Node positions exist.
- [ ] `settings.executionOrder` exists when appropriate.

## Security Audit

- [ ] Credentials are placeholders only.
- [ ] No Authorization header values are embedded.
- [ ] Webhook path contains no secrets.
- [ ] Pinned data is sanitized.
- [ ] Code nodes are justified.
- [ ] Execute Command is absent or explicitly reviewed.
- [ ] AI nodes have prompt-injection controls.
- [ ] Public workflow JSON does not leak private URLs.
- [ ] Production activation requires manual review.

## Reliability Audit

- [ ] Transient errors are retryable.
- [ ] Permanent errors are not blindly retried.
- [ ] Dead-letter path exists for failed items.
- [ ] Error Trigger workflow is specified for production.
- [ ] Upsert operations are idempotent.
- [ ] Pagination has a max guard.
- [ ] Rate limits are documented.
- [ ] Summary/logging object exists.
- [ ] Alerts are actionable.

## AI Agent Audit

- [ ] Inputs are treated as untrusted.
- [ ] Model output is structured JSON.
- [ ] Output schema is validated.
- [ ] Tool calls are constrained.
- [ ] Write actions require approval where needed.
- [ ] Secrets are not passed to prompts.
- [ ] Fallback path exists.
- [ ] Evaluation samples exist.
