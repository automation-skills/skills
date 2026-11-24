# Importable n8n JSON Standard

## Purpose

Use this reference when producing workflow JSON intended for n8n import. The goal is not merely valid JSON; the goal is JSON that imports cleanly, is credential-safe, and gives the user a strong starting workflow.

## Top-Level Shape

Use this top-level workflow object:

```json
{
  "name": "Example / Webhook -> Process / draft",
  "nodes": [],
  "connections": {},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null,
  "pinData": {},
  "versionId": "",
  "triggerCount": 0,
  "tags": [],
  "meta": {
    "generatedBy": "n8n-skills",
    "importNotes": []
  }
}
```

Keep `active` false or omit it unless the user explicitly asks for activation.

## Node Shape

Every node should have:

```json
{
  "parameters": {},
  "id": "stable-unique-id",
  "name": "Action: Do Something",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [600, 240]
}
```

Optional node keys:

- `credentials` when the credential type and name are known.
- `notes`, `notesInFlow`, `color` for operator documentation.
- `retryOnFail`, `maxTries`, `waitBetweenTries`, `alwaysOutputData`, `continueOnFail`, `executeOnce`, depending on node support and safety.

## Connections Shape

Connections are keyed by source node name:

```json
{
  "Source Node": {
    "main": [
      [
        {
          "node": "Target Node",
          "type": "main",
          "index": 0
        }
      ]
    ]
  }
}
```

Rules:

- Every referenced target node must exist.
- Every source node key must match a node name exactly.
- Use output index 0 for the primary path.
- Use separate output indexes for If/Switch branches when applicable.
- Never leave stale connection references after renaming nodes.

## Import-Safe Credential Handling

Generated workflows must not contain secret values. Use credential names only:

```json
"credentials": {
  "httpHeaderAuth": {
    "id": "PLACEHOLDER_CREDENTIAL_ID",
    "name": "REPLACE_WITH_HTTP_HEADER_AUTH_CREDENTIAL"
  }
}
```

Prefer documenting credentials separately when exact n8n credential type is uncertain. Never embed Authorization headers, cookies, API keys, password fields, webhook signing secrets, or private URLs.

## AI Agent Output Package

When the user asks for an importable workflow, produce this package:

1. **Workflow Summary** — what it does.
2. **Credential Setup** — credential names/types, no values.
3. **Environment Variables** — names only.
4. **Import JSON** — one valid JSON object.
5. **Manual Post-Import Steps** — set credentials, URLs, node-specific options.
6. **Test Payloads** — sample input JSON.
7. **Validation Checklist** — import, manual execution, production activation.
8. **Known Limitations** — node version assumptions and user-specific replacements.

## JSON Generation Checklist

Before returning JSON:

- parse it with a JSON parser,
- ensure all node names are unique,
- ensure all connection references are valid,
- ensure no credential values are present,
- ensure webhook paths do not contain secrets,
- ensure there is a trigger node,
- ensure irreversible actions are not active by default,
- include setup instructions for credentials and instance-specific node versions.

## When Exact Node Parameters Are Uncertain

n8n node parameter schemas can differ by n8n version and node typeVersion. If you cannot verify exact schema, provide:

- an importable skeleton using stable core nodes, and
- a manual configuration table for the uncertain node parameters.

Do not pretend uncertain JSON is guaranteed to import.
