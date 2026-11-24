# Output Templates

## Workflow Design Template

```markdown
# n8n Workflow Design: <name>

## Intent

## Assumptions

## Workflow Graph

## Node Table

| # | Node name | Type | Purpose | Key parameters | Credentials | Failure behavior |
|---|---|---|---|---|---|---|

## Data Contract

### Input

```json
{}
```

### Output

```json
{}
```

## Credential Map

## Error and Retry Strategy

## Idempotency Strategy

## Import JSON

```json
{}
```

## Setup Steps

## Test Payloads

## Validation Checklist
```

## Import JSON Response Template

Use this when the user directly asks for JSON they can import.

```markdown
## What this workflow does

## Before import

1. Create these credentials:
2. Replace these placeholders:
3. Confirm n8n version/node support:

## Import-ready workflow JSON

```json
{
  "name": "...",
  "nodes": [],
  "connections": {},
  "settings": {"executionOrder": "v1"},
  "pinData": {},
  "meta": {"generatedBy": "n8n-skills"}
}
```

## Post-import setup

## Test payload

## Validation checklist
```

## Blueprint JSON Schema

Agents may first produce a blueprint before exact import JSON:

```json
{
  "workflow_name": "string",
  "intent": "string",
  "environment": "dev|staging|prod|template",
  "trigger": {
    "type": "webhook|schedule|manual|app_event|subworkflow",
    "description": "string"
  },
  "nodes": [
    {
      "name": "string",
      "kind": "trigger|validation|transform|branch|action|log|response|error",
      "n8n_type": "string",
      "purpose": "string",
      "inputs": [],
      "outputs": [],
      "credentials": []
    }
  ],
  "data_contracts": {
    "input": {},
    "output": {},
    "errors": {}
  },
  "credentials": [
    {
      "name": "string",
      "type": "string",
      "used_by": []
    }
  ],
  "reliability": {
    "idempotency_key": "string",
    "retry_policy": "string",
    "dead_letter": "string",
    "observability": "string"
  },
  "security": {
    "secret_handling": "string",
    "webhook_verification": "string",
    "approval_gates": []
  }
}
```

## Audit Report Template

```markdown
# n8n Workflow Audit: <name>

## Executive Summary

## Critical Issues

## High Priority Issues

## Medium Priority Improvements

## Import JSON Problems

## Security Findings

## Reliability Findings

## Performance Findings

## Refactor Plan

## Fixed JSON or Patch Guidance

## Validation Checklist
```
