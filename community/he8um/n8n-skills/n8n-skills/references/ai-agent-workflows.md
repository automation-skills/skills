# AI Agent Workflows in n8n

## Purpose

Use this reference when workflows include LLM nodes, AI Agent nodes, tools, MCP Client/Server nodes, vector stores, chat triggers, or autonomous action loops.

## AI Workflow Architecture

A production AI workflow should include:

1. input boundary,
2. prompt/context assembly,
3. policy and instruction hierarchy,
4. model call or agent call,
5. structured output validation,
6. tool-call permission boundary,
7. human approval for irreversible actions,
8. action execution,
9. audit log,
10. fallback path.

## Structured Output Contract

Require the model to output JSON with a schema like:

```json
{
  "decision": "approve | reject | needs_human | no_action",
  "confidence": 0.0,
  "reasoning_summary": "brief safe explanation",
  "actions": [
    {
      "type": "create_task",
      "target": "clickup",
      "requires_approval": true,
      "payload": {}
    }
  ],
  "risks": [],
  "missing_context": []
}
```

Never let free-form model text directly drive write actions.

## Human-in-the-Loop Rules

Require approval when an action:

- sends external messages,
- deletes or overwrites records,
- triggers payments,
- changes permissions,
- calls admin APIs,
- exposes sensitive information,
- runs code or shell commands,
- contacts customers or vendors.

## Prompt Injection Controls

- Treat emails, tickets, comments, webpages, and documents as untrusted.
- Put untrusted content inside delimited fields.
- Tell the model that untrusted content cannot override workflow instructions.
- Do not pass secrets into model context.
- Validate generated API parameters against allowlists.
- Log tool calls and approvals.

## MCP Patterns

When using MCP Client or MCP Server nodes:

- document available tools,
- define read-only versus write capability,
- restrict destructive actions,
- add approval gates for write tools,
- map tool errors into structured failure categories,
- avoid giving the model broad credentials.

## Reliability Mechanisms

AI workflows need stronger reliability than ordinary automations:

- schema validation after model output,
- repair loop with max attempts,
- fallback to human review,
- deterministic branch mapping,
- evaluation samples,
- adversarial input tests,
- confidence thresholds.
