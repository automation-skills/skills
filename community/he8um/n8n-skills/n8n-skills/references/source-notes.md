# Source Notes

## Official Sources Used for This Skill

This skill is grounded in the public n8n documentation and common practitioner patterns. Important official source areas:

- Workflows, workflow components, nodes, connections, executions, tags, export/import.
- Nodes and built-in integrations.
- Expressions, item data, data transformation, referencing previous nodes.
- Flow logic: conditionals, merge, loops, waiting, sub-workflows, error handling.
- Credentials, credential sharing, users, access, RBAC, projects.
- API authentication, pagination, CLI import/export.
- Hosting, queue mode, scaling, task runners, security, logging and monitoring.
- Advanced AI: AI workflows, human fallback, human-in-the-loop, MCP server/client, evaluations.

## Import/Export Grounding

n8n stores workflows in JSON and supports export/import through copy-paste, UI import from file/URL, and command-line import/export. Exported workflow JSON can include credential names and IDs, and HTTP Request nodes imported from cURL may contain authentication headers, so public templates must be sanitized.

## Community-Derived Patterns

The community patterns in this skill are synthesized from repeated practitioner experience rather than copied from individual forum posts:

- keep workflows modular,
- avoid hidden business logic in expressions,
- explicitly design retry and dead-letter behavior,
- use stable IDs for upsert,
- avoid embedding secrets,
- prefer sub-workflows for shared logic,
- separate public intake from heavy processing,
- add logging and human-readable summaries,
- use queue mode and external binary storage for scalable self-hosted deployments.

## Maintenance Policy

n8n node parameters and type versions can change. When generating import JSON for a specific node, verify against the user's installed n8n version if possible. If the exact parameter schema is uncertain, generate a safer skeleton plus manual setup instructions rather than claiming guaranteed import.
