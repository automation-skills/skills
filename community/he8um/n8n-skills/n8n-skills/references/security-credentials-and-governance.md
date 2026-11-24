# Security, Credentials, and Governance

## Credential Safety

Never put secret values in generated workflow JSON. Generated artifacts may include credential names and placeholder IDs only.

Sensitive values include:

- API keys,
- bearer tokens,
- OAuth client secrets,
- HTTP Authorization headers,
- cookies,
- webhook signing secrets,
- SSH keys,
- database passwords,
- private internal hostnames when the repo is public.

## Safe Credential Map Format

```markdown
| Credential name | n8n credential type | Used by nodes | Setup owner | Notes |
|---|---|---|---|---|
| REPLACE_WITH_CRM_API | HTTP Header Auth | Action: Upsert CRM Lead | Admin | Scope to lead write only |
```

## Webhook Security

For production webhooks:

- avoid secrets in path names,
- require signature verification when source supports it,
- validate method and content type,
- reject oversized payloads when possible,
- rate-limit at the edge for public endpoints,
- log only sanitized payloads,
- return generic errors.

## AI Workflow Security

AI agent workflows need stronger controls:

- treat all external text as untrusted input,
- never allow user-provided text to override system/developer instructions,
- require structured output schemas,
- add human approval for irreversible actions,
- restrict tool nodes to explicit scopes,
- log prompt, tool call, and approval metadata where policy allows,
- block secrets from entering prompts.

## Code and Execute Command Risk

Code nodes and Execute Command nodes increase risk. Use them only when justified. For generated public templates:

- prefer built-in nodes,
- do not generate shell commands that access credentials,
- document self-hosted-only assumptions,
- warn when a workflow requires filesystem access,
- require review before enabling.

## Sharing Workflow JSON

Workflow exports may contain credential names and IDs. Names can be sensitive if they reveal vendors, tenants, or environments. Before publishing:

- replace credential names with placeholders,
- remove headers imported from cURL,
- remove production URLs if sensitive,
- remove pinned data containing personal or business data,
- remove execution data,
- set `active` to false.

## Governance Checklist

- Owner assigned.
- Credential owner assigned.
- Environment documented: local, dev, staging, prod.
- Activation rule documented.
- Rollback path documented.
- Change log maintained.
- Production workflow tagged.
- Error workflow configured or planned.
- Public URLs reviewed.
- Data retention policy documented.
