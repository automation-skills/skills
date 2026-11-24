---
name: zapier-to-n8n
description: "Migrate Zapier workflows to n8n. Triggers on: any uploaded Zapier JSON file with a request to convert, migrate, rebuild, or transfer it to n8n. Also triggers on 'update migration skill', 'rebuild migration skill', or 'sync skill'. Handles node mapping, replaces all AI steps with Claude Haiku HTTP Request nodes, runs multi-pass self-verification, and applies a known error playbook. Integrates with the n8n-mcp MCP server when available (node lookup, workflow validation, optional direct deploy). After each migration, saves new error patterns to memory and auto-rebuilds the skill file every 3 migrations."
---

# Zapier → n8n Migration Skill

## Trigger

**Migration mode:** User uploads a Zapier workflow JSON and asks to convert, migrate, rebuild, or transfer it to n8n.

**Skill update mode:** User says "update migration skill", "rebuild the skill", "sync the skill", or similar. Skip to Phase 9 immediately.

**Auto-update check (run at START of every migration):** Read the memory file `project_migration_count.md`. If it doesn't exist, the count is 0. If the count is a multiple of 3 (and > 0), run Phase 9 before Phase 1 to sync accumulated learnings into the skill file first.

---

## Companion Resources (subordinate to this skill)

If the **n8n-mcp MCP server** (tools like `search_nodes`, `get_node`, `validate_workflow`, `n8n_create_workflow`) and/or the **n8n-mcp-skills pack** (n8n-validation-expert, n8n-node-configuration, n8n-expression-syntax, etc.) are installed, use them as subordinate resources at the points marked in the phases below — node lookup for unmapped apps, extra validation passes, and optional direct deployment.

**This skill remains the authority for the migration flow.** Companion skills never override its conversion policies. In particular:
- AI steps are ALWAYS converted to Claude Haiku HTTP Request nodes — never LangChain/`@n8n/n8n-nodes-langchain` or dedicated Anthropic/OpenAI nodes, even though companion skills document them.
- The phase order, verification loop, memory capture, and self-rebuild process are unchanged.
- If no n8n-mcp tools are available, every phase works exactly as written — the MCP steps are optional enhancements, not dependencies.

---

## Scheduled / polling trigger constraints

Applies to any workflow with a Schedule Trigger, Cron, Interval, or other polling trigger.

- A polling trigger fires as soon as the workflow is saved on the instance. Publish/Active
  state does not gate it. Verified on 2026-07-28: an unpublished workflow (editor showing
  "0/1", only ever run via the manual Execute button) fired every 2 minutes continuously and
  kept firing after being unpublished, until it OOM-killed a shared cluster.
- Archive is the reliable stop. Unpublish/deactivate may not halt a running schedule.
- The trigger does not wait for the previous execution to finish; there is no built-in
  concurrency guard. If run time can exceed the interval, executions stack and run
  concurrently, each holding its full dataset in memory. Estimate worst-case run time and
  per-run volume (rows × columns × number of source sheets) before choosing an interval, and
  leave headroom. Add an explicit skip-if-already-running guard when run time is unpredictable.
- Read volume matters independently of logic correctness. Narrow the ranges rather than
  pulling whole sheets or an entire multi-thousand-row log every cycle.
- Deliver polling triggers with `"disabled": true`, or at a conservative interval (15 min
  floor, longer for heavy reads). Do not mirror the Zapier zap's polling frequency — Zapier
  absorbed that load on its own infrastructure.
- When debugging, disable the trigger node or archive first. Disabling only downstream nodes
  leaves the trigger and all upstream reads executing every interval, which reads as "nothing
  is running" while it still is.
- If a user reports something running that shouldn't be, ask for the Executions list before
  theorizing. Run durations and repeated/overlapping start timestamps are what diagnose it.

---

## Phase 1 — Environment Intake (ask BEFORE touching the JSON)

Before writing any n8n JSON, ask the user these questions using AskUserQuestion. Do this in a single call with multiple questions:

**Questions to ask:**

1. "Are you the owner/admin of your n8n instance, or a regular member?"
   - Options: Owner/Admin | Regular member (not admin)
   - This determines whether to use dedicated Anthropic nodes or HTTP Request nodes (non-admins typically can't install community packages).

2. "Which n8n version are you running?" (only if they said they're the admin — members may not know)
   - Options: n8n Cloud (managed by n8n.io) | Self-hosted (I manage it) | I don't know

3. "Does your n8n instance have the LangChain/AI nodes installed?" (look for: can you see an 'AI' section in the node panel, or nodes like 'Anthropic', 'OpenAI', 'Basic LLM Chain'?)
   - Options: Yes, I see AI/LangChain nodes | No, I don't see those | Not sure

4. "Do you have an Anthropic API key to use for the AI steps?"
   - Options: Yes | No, I'll need to get one | I want to use a different AI provider

5. "Should the workflow be built pointing at the genuine production targets (sheets, folders,
   spaces), or at testing targets if a testing sheet/folder exists for this system?"
   - Options: Genuine production targets | Testing targets (I have a test sheet/folder)
   - If testing targets: collect the test sheet/folder IDs before building. Either way the
     delivered file ships in the "testing phase" state — see Phase 4. Notification nodes
     disabled until the flow is proven.

**If user is not admin AND does not see LangChain nodes:** Use HTTP Request nodes for ALL AI steps (see Phase 3). This is the safe universal fallback.
**If user is admin AND has LangChain nodes:** Still prefer HTTP Request nodes for portability unless user explicitly requests the dedicated node.

Ask one additional free-form question:
- "Are there any steps in this workflow that should behave differently from the original Zapier version? For example: different recipients, different file destinations, new logic, or steps to skip?"

---

## Phase 1.5 — Node Availability Check (MANDATORY GATE — before any JSON is written)

List every distinct app in the source workflow (`selected_api` on each step). For EACH one,
establish whether a real n8n node already exists before deciding how to build it. Building a
generic Webhook / HTTP Request node for an app that has a first-party node is a serious error:
it hands the user manual configuration work the real node does automatically.

**Do NOT use web search to decide whether a node exists.** It has confidently returned "no
dedicated node exists, use HTTP Request" for an app that had an official vendor-published,
n8n-verified node. Web search is not authoritative here. Probe these instead:

1. **n8n's integration directory** — fetch `https://n8n.io/integrations/<app-slug>/` and
   `https://n8n.io/integrations/<app-slug>-trigger/`. A real page, especially one reading
   "built and maintained by our partners ... and verified by n8n", means the node exists.
2. **npm registry** — catches community nodes:
   `https://registry.npmjs.org/-/v1/search?text=n8n-nodes-<app>&size=20`
   Check the scope: a vendor-owned one (e.g. `@zohoforms/`) means it is official.
3. **n8n-mcp**, if connected: `search_nodes({query: "<app>"})`.
4. **Ask the user to look in their own node panel.** They see installed and verified community
   nodes no external source lists reliably. Fastest and most authoritative check available.

**If a node is found, read its source before wiring it** (GitHub link is on the integration
page). Extract and verify:
- exact node type string: `<npm package name>.<node name>`, e.g.
  `@zohoforms/n8n-nodes-zohoforms.zohoFormsTrigger`. A wrong type string breaks the import.
- `version` -> `typeVersion`; exact parameter names; exact credential name.
- **the OUTPUT SHAPE.** Trigger nodes wrapping a webhook often return
  `returnJsonArray(this.getBodyData())`, putting fields at the TOP LEVEL of `$json`, whereas a
  raw `n8n-nodes-base.webhook` nests them under `$json.body`. Guessing wrong silently blanks
  every downstream field instead of erroring.
- whether it self-registers its webhook (a `webhookMethods.default.create()` POSTing the n8n URL
  to the vendor API). If so the user needs to configure NOTHING by hand — say so explicitly,
  because the manual alternative is a real cost to them.

Only fall back to Webhook / HTTP Request after all four checks come back empty, and say that you
checked.

## Phase 2 — Zapier JSON Analysis

Read the uploaded Zapier JSON. Build an internal node map:

For each Zapier step, record:
- Step number (Zapier's `order` or position)
- Step name (verbatim)
- App/API used (`selected_api` field)
- Action type
- All configuration fields: prompts, queries, IDs, email bodies, credentials referenced
- Whether this step is an AI/LLM call

**Identify AI steps** — look for `selected_api` containing any of: `ChatGPT`, `OpenAI`, `GPT`, `Anthropic`, `Claude`, `LLM`, or action names containing `conversation_responses_api`, `extract_structured_data_responses_api`, `analyze_text_responses_api`.

**Identify trigger type** — first step, `selected_api` tells you the trigger app (Google Drive, webhook, schedule, Gmail, etc.).

**Flag anything that can't be automatically mapped** and list it for the user.

---

## Phase 3 — Conversion Rules

### General Rules

1. **Node IDs:** Generate a UUID v4 for every node `id` (e.g. `python3 -c "import uuid; print(uuid.uuid4())"` or `uuidgen`). n8n's frontend binds form state and the credential selector to the node ID — human-readable IDs like `node_1` cause subtle UI breakage. Never reuse IDs.
2. **Node names:** Copy verbatim from Zapier step names. If a Zapier step has no name, use the step number + action description.
3. **Positions:** Space nodes 200px apart horizontally: `[0,300]`, `[200,300]`, `[400,300]`, etc. Branch paths offset vertically by 150–200px.
4. **Connections:** Wire every node explicitly in the `connections` object. Branches (IF nodes) wire `true`/`false` outputs separately.
5. **typeVersion:** Use the versions known to work on n8n 2.26.7:
   - `googleDriveTrigger`: 1
   - `googleSheets`: 4.4
   - `gmail`: 2
   - `httpRequest`: 4.2
   - `code`: 2
   - `if`: 1
   - `wait`: 1
   - `postgres`: 2.3
   - `set`: 3.3
   - `merge`: 3
   - `splitInBatches`: 3
   - `slack`: 2.1
   - `googleDrive`: 3

   If the n8n-mcp MCP server is available, verify the current typeVersion with `get_node({nodeType: "nodes-base.<name>"})` instead of trusting this list — the list is the offline fallback validated on n8n 2.26.7.

6. **Retries:** Zapier retries failed steps automatically; n8n does not. Set `"retryOnFail": true, "maxTries": 3, "waitBetweenTries": 2000` (top-level node properties) on every network-calling node — Claude HTTP, Gmail, Sheets, Drive, Slack, Postgres, PDF.co — so transient 429s and network blips don't hard-fail the run.
7. **Branch reconvergence:** downstream of a point where IF branches reconverge, never use bare `$json.field` — whichever branch fired last wins, non-deterministically. Reference the source node explicitly: `$('Node Name').item.json.field` (or `.first()` when the node emits exactly one item).
8. **No `$env`:** `$env.X` does not work in n8n expressions (throws at runtime). Use `$vars.X`, or move secrets into credentials.

### Unmapped Zapier Apps

Phase 1.5 is the authority here — never conclude from memory or web search that an app lacks a node. If a Zapier step's app has no row in the conversion tables below AND the n8n-mcp server is available: run `search_nodes({query: "<app name>"})`, then `get_node({nodeType: "nodes-base.<result>"})` to get its operations and current typeVersion before building the node. If no dedicated n8n node exists (or no MCP tools are available), fall back to an `httpRequest` node against the service's public API — same pattern as the PDF.co rows below. Record the new mapping in memory per Phase 8 Step 2.

### AI / LLM Step Conversion

Replace every AI step with an `n8n-nodes-base.httpRequest` node (typeVersion 4.2):

```json
{
  "parameters": {
    "method": "POST",
    "url": "https://api.anthropic.com/v1/messages",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "anthropic-version", "value": "2023-06-01" }
      ]
    },
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ ({ model: 'claude-haiku-4-5', max_tokens: <N>, temperature: <T>, system: `<VERBATIM SYSTEM PROMPT>`, messages: [ { role: 'user', content: <USER_CONTENT_EXPRESSION> } ] }) }}",
    "options": {}
  },
  "id": "node_<N>",
  "name": "<verbatim step name>",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [<X>, <Y>],
  "credentials": {
    "httpHeaderAuth": { "id": "anthropic-header-auth", "name": "Anthropic API" }
  }
}
```

**Key rules for AI nodes:**
- Model: always `claude-haiku-4-5` (not claude-3-5-haiku-20241022 — retired Feb 2026)
- Copy system prompts and instructions **verbatim** from the Zapier step
- `max_tokens`: match Zapier step if specified; default 1024 for short extractions, 2000 for formatting/generation
- `temperature`: match Zapier step if specified; default 0 for structured extraction, 0.2 for formatting
- Downstream nodes read Claude's reply as: `$json.content[0].text`
- **NULL guard:** if the user content expression can be null/empty (e.g., from a `json_agg()` Postgres result), wrap it: `JSON.stringify($('NodeName').item.json.field ?? [])`

#### Zapier `extract_structured_data_responses_api` → Claude HTTP + Parse Code node

Zapier's structured extraction returns pre-parsed fields; Claude returns raw text. Two nodes are required:

1. Claude HTTP node — instruct via system prompt to return ONLY a raw JSON object with the exact field names from the Zapier `arguments` list. No markdown, no explanation.
2. Immediately follow with a **Parse Code node**: `const parsed = JSON.parse($input.first().json.content[0].text.trim()); return [{ json: parsed }];`

Downstream nodes reference parsed fields as `$('Parse <Name>').first().json['Field Name']`.

#### Zapier `create_conversation_with_assistant` (OpenAI Assistant) → Claude HTTP node

OpenAI assistant IDs are not portable and their internal system prompts are NOT exported in Zapier JSON.
- Use the `assistant_instructions` param (if present) verbatim as part of the Claude system prompt
- Map the `message` param to the user message `content`
- Reconstruct multi-part user messages via string concatenation: `'<Start...>\n' + upstreamText + '\n<End...>'`
- Warn the user that the assistant's full internal training is unavailable from the Zapier export and results may need prompt tuning

### Trigger Conversion

| Zapier Trigger | n8n Equivalent |
|---|---|
| Google Drive — new file in folder | `n8n-nodes-base.googleDriveTrigger` (triggerOn: specificFolder) |
| Gmail — new email | `n8n-nodes-base.gmailTrigger` |
| Schedule — every X | `n8n-nodes-base.scheduleTrigger` |
| Webhook | `n8n-nodes-base.webhook` |
| Typeform | `n8n-nodes-base.typeformTrigger` |
| Airtable — new record | `n8n-nodes-base.airtableTrigger` |
| Zoho Forms — new form entry (push) | `n8n-nodes-base.webhook` (typeVersion 2) — fields under `$json.body`; file fields are arrays `body['ImageUpload1']?.[0]?.file`/`.extension`; confirm payload shape with user |

### Common Action Conversion

| Zapier Action | n8n Equivalent |
|---|---|
| Gmail — send email | `n8n-nodes-base.gmail` (operation: send) — **always set emailType: html if body contains HTML tags** |
| Gmail — reply to thread | `n8n-nodes-base.gmail` (operation: send) + `options.threadId: "={{ $('NodeName').first().json.threadId }}"` — omitting threadId creates a new thread instead of a reply |
| Google Sheets — append row | `n8n-nodes-base.googleSheets` (operation: append, mappingMode: autoMapInputData) — sheet headers must exactly match the Code node output field names; if Zapier used COL$A/B/C column mapping, verify sheet headers match |
| Google Sheets — get rows | `n8n-nodes-base.googleSheets` (operation: read, "Get Row(s)") — **use "By Name" or "From list" for sheet tab, never "By ID" with a name string**; if Zapier specified a range (e.g. A:B) add `options.range` to limit columns |
| Google Drive — upload file (text/CSV) | `n8n-nodes-base.googleDrive` (operation: upload, binaryPropertyName: "data") — requires binary input; add a Code node before: `return [{ json: { fileName }, binary: { data: { data: Buffer.from(content,'utf-8').toString('base64'), mimeType:'text/csv', fileName, fileExtension:'csv' } } }]` |
| Google Drive — download file | `n8n-nodes-base.googleDrive` (operation: download) |
| Delay / Wait | `n8n-nodes-base.wait` (resume: "timeInterval", unit: minutes/seconds, amount: N) — add `webhookId` string at node top level |
| Code / JavaScript | `n8n-nodes-base.code` (typeVersion 2, jsCode field) — output must be `return [{ json: {...} }]` or `return items.map(...)` |
| Code / Python | Convert to `n8n-nodes-base.code` JavaScript — `requests.get(url)` → HTTP Request GET node; `ast.literal_eval` → `JSON.parse`; `csv.DictWriter` → manual CSV string; `return {'k':v}` → `return [{ json: {k:v} }]` |
| Filter / Condition | `n8n-nodes-base.if` — for case-insensitive string contains, use `.toLowerCase()` in value1 expression and lowercase value2 |
| Formatter — text Replace | `n8n-nodes-base.code`: `input.replace(/pattern/g, '')` — first in chain reads `$input.first().json.content[0].text`; subsequent ones read `$input.first().json.output` |
| Formatter — date Format | `n8n-nodes-base.code`: `new Date().toISOString().replace(/\\.\\d{3}Z$/, '+00:00')` for YYYY-MM-DDTHH:mm:ssZ; `{{zap_meta_human_now}}` → `new Date()` |
| Postgres — custom query | `n8n-nodes-base.postgres` (operation: executeQuery, query field supports `{{ }}` expressions) — **never interpolate user/webhook-supplied values into SQL via `{{ }}`** (n8n substitutes into the SQL string before the driver binds — injection vector); use `$1, $2` placeholders + `options.queryReplacement` for dynamic values |
| Slack — post message | `n8n-nodes-base.slack` (typeVersion 2.1, resource: "message", operation: "post") — channel as `{ "__rl": true, "value": "CHANNEL_ID", "mode": "id" }`; bot name in `otherOptions.botName`; credential type: `slackApi` |
| PDF.co — Make Searchable | `n8n-nodes-base.httpRequest` POST `https://api.pdf.co/v1/pdf/make/searchable` body: `{ url, lang }` — use separate Header Auth credential (x-api-key) for PDF.co, not the Anthropic one |
| PDF.co — Rotate Pages | `n8n-nodes-base.httpRequest` POST `https://api.pdf.co/v1/pdf/edit/rotate` body: `{ url, angle, lang }` — response.url is output file URL |
| PDF.co — Convert to Text | `n8n-nodes-base.httpRequest` POST `https://api.pdf.co/v1/pdf/convert/to/text` body: `{ url, lang, lineGrouping, unwrap, inline }` — follow with HTTP GET node to fetch the text file from response.url |
| Google Sheets — add_row / update_row / lookup mapped BY COLUMN LETTER (COL$A…) when headers are unknown or must not be edited | raw Google Sheets REST API via `n8n-nodes-base.httpRequest` + predefined `googleSheetsOAuth2Api`. Append: `POST /v4/spreadsheets/{id}/values/{{ encodeURIComponent("'Tab'!A:AB") }}:append` qs `valueInputOption=USER_ENTERED&insertDataOption=INSERT_ROWS`, body `={{ JSON.stringify({values:[positionalArray]}) }}` ('' for every blank column so letters line up). Update one cell: `PUT .../values/{{ encodeURIComponent("'Tab'!U"+row) }}`. Lookup: `GET .../values/{range}` then match by positional index, row=index+1. First sheet accepts unqualified `A:X`; other tabs need the title in the range |
| Google Chat — create_message (card with title/text/button) | `n8n-nodes-base.googleChat` (typeVersion 1) with a `googleApi` SERVICE-ACCOUNT credential, `resource:message`, `operation:create`, `spaceId` verbatim (`spaces/XXXX`), `jsonParameters:true` + `messageJson` = `={{ JSON.stringify({cardsV2:[{cardId,card:{header:{title},sections:[{widgets:[{textParagraph:{text}},{buttonList:{buttons:[{text,onClick:{openLink:{url}}}]}}]}]}}]}) }}`. Human OAuth (`googleChatOAuth2Api`) CANNOT send cards — text only |

#### Multi-row Google Sheets insert — loop pattern

When a Code node outputs multiple items that must each be appended to Sheets (and a Slack/email notification should fire only once after all rows), use `splitInBatches`:

1. Code node outputs multiple items (one per row)
2. `splitInBatches` (batchSize 1): output-0 → Sheets append → loops back; output-1 (done) → Slack/Gmail
3. In the Slack/email node, get row count via `{{ $('Code Node Name').all().length }}`

Without this pattern, Slack/email fires once per row.

### Credential Placeholder Convention

Use placeholder credential objects in the JSON — the user will select real credentials in the n8n UI:
```json
"credentials": {
  "googleDriveOAuth2Api": { "id": "google-drive-main", "name": "Google Drive" },
  "gmailOAuth2": { "id": "gmail-main", "name": "Gmail" },
  "googleSheetsOAuth2Api": { "id": "google-sheets-main", "name": "Google Sheets" },
  "postgresApi": { "id": "postgres-main", "name": "Postgres" },
  "slackApi": { "id": "slack-main", "name": "Slack" },
  "httpHeaderAuth": { "id": "anthropic-header-auth", "name": "Anthropic API" }
}
```

If a workflow has multiple `httpHeaderAuth` services (e.g. Anthropic + PDF.co), use **distinct credential IDs and names** per service to avoid conflicts:
```json
{ "id": "anthropic-header-auth", "name": "Anthropic API" }   // x-api-key = sk-ant-...
{ "id": "pdfco-header-auth",     "name": "PDF.co API" }      // x-api-key = PDF.co key
```

**Placeholder credentials are for file-import delivery ONLY.** If deploying directly through the n8n API or n8n-mcp (`n8n_create_workflow`), OMIT the `credentials` block entirely on every node — n8n renders a permanently disabled credential selector ("No credentials yet", not clickable) for unknown IDs on API-created workflows, forcing the user to rebuild the node. An absent block shows a normal clickable dropdown. If n8n-mcp is available, `n8n_manage_credentials({action: "list"})` can discover real credential IDs to embed instead.

---

## Phase 4 — JSON Construction

Build the complete n8n workflow JSON:

```json
{
  "name": "<workflow name — bump version, e.g. v3 if original was v2>",
  "nodes": [ ... all nodes ... ],
  "connections": { ... all wires ... },
  "active": false,
  "settings": { "executionOrder": "v1" },
  "versionId": "<new UUID>",
  "id": "<new UUID>",
  "meta": { "instanceId": "" }
}
```

**Connections format:**
```json
"connections": {
  "Node Name": {
    "main": [
      [ { "node": "Next Node Name", "type": "main", "index": 0 } ]
    ]
  },
  "IF Node Name": {
    "main": [
      [ { "node": "True Branch Node", "type": "main", "index": 0 } ],
      [ { "node": "False Branch Node", "type": "main", "index": 0 } ]
    ]
  }
}
```

### Output naming and folder layout (REQUIRED)

Names must mirror the ORIGINAL Zapier zap title — never invent a shorter label. The user
cross-references n8n against the Zapier account by name, so re-branding breaks that mapping.

- Filename = original zap title with filename-illegal chars swapped (`:` -> ` -`), then `_n8n.json`.
  `1. Zoho Form Populate Google Sheets: 299 Return Issues`
  -> `1. Zoho Form Populate Google Sheets - 299 Return Issues_n8n.json`
- The workflow `name` field INSIDE the JSON keeps the original title verbatim (colons fine).
- Several zaps collapsing into one workflow: keep the shared prefix, note the merge in parens —
  `3. Customer Comments add to 299 Main Logger (3a + 3b combined)`.
- Only invent a name where no original zap exists (e.g. a new shared error handler).

Every migration gets its own folder in the user's connected project folder:

```
<project folder>/
  <Migration Name>/                  e.g. "299 Return Issues"
    <workflow 1>_n8n.json
    <workflow 2>_n8n.json
```

Do NOT create README / READ ME / status text files — explain status, next
steps, and gotchas directly in chat instead.

(The skill's own directory is a read-only cache — deliverables go in the project folder.)

### Testing phase (ONE file per workflow — no separate test-variant files)

Do NOT build separate mock / fake-data / test-copy workflow files. One migrated JSON per
workflow, delivered in a "testing phase" state, adjusted in place from there:

- Ship the finished workflow with every Slack / Google Chat / email / outbound-notification
  node set `"disabled": true`. They stay off until the rest of the flow has been proven to
  work, then the user enables them in the n8n UI.
- Ask the user BEFORE building (add this to the Phase 1 AskUserQuestion call):
  "Should the workflow be built pointing at the genuine production targets (sheets, folders,
  spaces), or at testing targets if a testing sheet/folder exists for this system?"
  Options: Genuine production targets | Testing targets (I have a test sheet/folder)
  Build against whichever they choose. If they choose testing targets, get the test
  sheet/folder IDs from them and point every read/write there; switching to production later
  is an in-place edit, not a second file.
- How the JSON comes out and how it is organized is the user's call — ask, don't assume.

---

## Phase 5 — Self-Verification Loop

After writing the JSON file, run the following checks. Fix any failures and re-verify. Loop until all checks pass.

### Pass 1 — Structural Checks (run programmatically via bash/Python)

```python
import json, sys

with open('workflow.json') as f:
    w = json.load(f)

nodes = w['nodes']
node_names = {n['name'] for n in nodes}
node_ids = [n['id'] for n in nodes]

# Check 1: No duplicate node IDs
assert len(node_ids) == len(set(node_ids)), "Duplicate node IDs"

# Check 2: No duplicate node names  
assert len(node_names) == len({n['name'] for n in nodes}), "Duplicate node names"

# Check 3: All connection targets exist
for src, wires in w['connections'].items():
    assert src in node_names, f"Connection source '{src}' not in nodes"
    for branch in wires.get('main', []):
        for conn in branch:
            assert conn['node'] in node_names, f"Connection target '{conn['node']}' not in nodes"

# Check 4: No forbidden node types
forbidden = ['@n8n/n8n-nodes-langchain', 'n8n-nodes-base.openAi', 'n8n-nodes-base.anthropic']
for n in nodes:
    for f in forbidden:
        assert f not in n['type'], f"Forbidden node type '{n['type']}' in node '{n['name']}'"

# Check 5: No retired model strings
# IMPORTANT: use ensure_ascii=False so Unicode chars (em-dash, curly quotes) are not escaped
# to \uXXXX — otherwise content checks against non-ASCII strings will silently fail
import re
text = json.dumps(w, ensure_ascii=False)
assert 'claude-3-5-haiku-20241022' not in text, "Retired model string found"
assert 'claude-3-haiku' not in text, "Retired model string found"
assert 'gpt-4' not in text, "OpenAI model found — replace with claude-haiku-4-5"
assert 'gpt-3' not in text, "OpenAI model found — replace with claude-haiku-4-5"

print(f"Pass 1: {len(nodes)} nodes, {len(w['connections'])} connections — structural checks passed")
```

### Pass 2 — Content Fidelity Check (manual review against Zapier JSON)

For each Zapier AI step, verify:
- [ ] Corresponding n8n node exists with same name
- [ ] System prompt text matches verbatim (modulo whitespace)
- [ ] User content expression references the correct upstream node name
- [ ] `claude-haiku-4-5` is the model (not any deprecated string)
- [ ] NULL guard `?? []` present on any content that could be null

For each Zapier non-AI step, verify:
- [ ] Node type maps correctly to n8n equivalent
- [ ] IDs (folder IDs, sheet IDs, spreadsheet IDs) are preserved verbatim from the Zapier JSON
- [ ] Email bodies (HTML) are verbatim
- [ ] SQL queries are verbatim
- [ ] Gmail nodes: emailType set to "html" if body contains any HTML tags
- [ ] Google Sheets read nodes: sheet tab NOT set to "By ID" with a name string

### Pass 3 — Spawn a verification subagent

Use the Agent tool with this prompt:
```
Review this n8n workflow JSON file at [path]. Compare it against the original Zapier JSON at [path].

Check:
1. Every Zapier step has a corresponding n8n node (same name, same position in sequence)
2. All AI/ChatGPT steps are converted to n8n-nodes-base.httpRequest nodes using claude-haiku-4-5 — none use LangChain node types
3. No system prompts, email bodies, SQL queries, or IDs were modified (they must be verbatim)
4. All connections form a complete, unbroken chain (no dangling nodes, no missing wires)
5. The JSON is valid and parseable
6. No retired model strings (claude-3-5-haiku-20241022, gpt-4, gpt-3.5, etc.)
7. Any aggregate expressions (json_agg, etc.) feeding Claude content have null guards (?? [])

Report: PASS or FAIL for each check, with specific node names for any failures.
```

Fix all reported failures before presenting the file to the user.

### Pass 4 — n8n-mcp validation (only if the n8n-mcp server is connected)

If a `validate_workflow` MCP tool is available (tool name matches `mcp__*__validate_workflow`), run it on the built workflow JSON and fix every reported error. Two caveats:
- Validation passing means the JSON is **well-formed**, not that the migration is **faithful** — Passes 2 and 3 remain the fidelity gate and are never skipped.
- `validate_workflow` does not catch branch-reconvergence `$json` bugs, Merge `numberOfInputs` mismatches, or missing null guards — those are checked in Passes 1–3.

If no MCP tools are available, skip this pass silently; Passes 1–3 are sufficient.

---

## Phase 6 — Delivery

Present the finished JSON file. Include a brief summary covering:

1. Total node count
2. How many AI steps were converted (and to what: "7 ChatGPT steps → 7 HTTP Request nodes calling claude-haiku-4-5")
3. Credentials the user will need to attach in n8n UI (list service types only)
4. Any items that required user-specific decisions or couldn't be auto-converted (flag as "action required")

**Credential setup instructions to always include:**
- For Claude/Anthropic nodes: create one **Header Auth** credential — Name: `x-api-key`, Value: Anthropic API key (starts with `sk-ant-`). Apply this one credential to all Claude HTTP nodes.
- For Google services: select existing OAuth credentials from the dropdown in each node.

**Testing phase at delivery:** the single delivered file has all Slack/Chat/email/notification
nodes `disabled: true`, and its targets (production vs testing sheets/folders) are whatever the
user chose in Phase 1. If they chose testing targets, assert programmatically that no ENABLED
node's `url`/`jsCode` references a production ID. State in the delivery summary which nodes are
disabled and what to verify before enabling them. No separate test-copy files.

**Polling triggers:** ship with `"disabled": true` on the trigger node, and note in the summary
that the trigger goes live as soon as the workflow is saved regardless of publish state, so they
should enable it only when ready for real runs. State the chosen interval and its headroom over
expected run time. See "Scheduled / polling trigger constraints" above.

**Do NOT include** long explanations of what every node does — the user can read the workflow.

**Optional direct deploy:** if the n8n-mcp server is connected with live API access (`n8n_create_workflow` available), offer to push the workflow directly instead of file import. If the user accepts: strip ALL placeholder `credentials` blocks first (see Phase 3 credential rules), deploy with `n8n_create_workflow`, then run `n8n_validate_workflow({id})` and report the result. The file is still written to the workspace as the artifact of record either way.

---

## Phase 7 — Error Handling During Testing

When the user reports errors during n8n testing, consult this playbook before responding:

| Error | Root Cause | Fix |
|---|---|---|
| "model: claude-X not found" (404) | Deprecated model string | Change to `claude-haiku-4-5` in all HTTP nodes |
| "messages.0.content: Field required" (400) | Null passed as Claude content | Add `?? []` or `?? ''` guard on the content expression |
| "not valid JSON — Unexpected token '='" | User pasted `={{ }}` instead of `{{ }}` in expression editor | Tell user to paste only `{{ }}` without the leading `=` |
| "permission denied for view X" (Postgres) | n8n DB user lacks SELECT grant | Grant USAGE on schema + SELECT on view, or use same DB user as the Zapier integration |
| "readRows is not valid" / red Operation warning | Wrong operation name for installed n8n version | Switch to "Get Row(s) in Sheet" or equivalent |
| "At least one value has to be added under Values to Send" | Sheets append node lost mapping when pointed at new sheet | Add headers to row 1 of the target sheet, set Mapping Column Mode = "Map Automatically" |
| Node connections dropped on import | Used LangChain/Anthropic node type not installed | Rebuild as HTTP Request nodes (see Phase 3) |
| Gmail body showing raw HTML tags | Email Type set to Text | Set Email Type = HTML on each Gmail node |
| Sheet tab not resolving (no columns found) | Tab set to "By ID" but value is a name | Switch to "From list" or "By Name" mode |
| Google Sheets append silently skips columns | `autoMapInputData` couldn't match field names to headers | Ensure sheet row-1 headers exactly match Code node output field names; or use `defineBelow` mapping and set each column letter explicitly |
| Google Sheets read returns extra columns not in Zapier | No range set; reads all populated columns | Add `options.range: "A:B"` (or whatever range Zapier had) to the Sheets read node parameters |
| Google Drive upload — "binaryData is required" | Upload operation received JSON string, not binary | Add a Code node before upload: convert string to `binary.data` object with base64 + mimeType; set `binaryPropertyName: "data"` on Drive node |
| Gmail reply creates a new email thread instead of replying | `threadId` missing from send parameters | Set `options.threadId` on the Gmail send node to `={{ $('Original Send Node').first().json.threadId }}` |
| Slack / Gmail fires N times instead of once after Google Sheets loop | Multiple items from Code node flow through to notification | Use `splitInBatches` (batchSize 1) loop: Code → splitInBatches(output-0) → Sheets → back to splitInBatches; splitInBatches(output-1/done) → Slack/Gmail |
| Content checks fail for Unicode characters (em-dash, curly quotes) in programmatic verification | `json.dumps(w)` escapes non-ASCII to `\uXXXX` by default; searching for `'–'` won't find `–` | Use `json.dumps(w, ensure_ascii=False)` in all Python verification scripts |
| PDF.co HTTP node returns 401 Unauthorized | Wrong credential applied — Anthropic `httpHeaderAuth` used instead of PDF.co key | Create a separate Header Auth credential for PDF.co; do not reuse the Anthropic credential |
| Credential dropdown permanently disabled — "No credentials yet", not clickable | Workflow was created via n8n API/MCP with placeholder credential IDs | Omit `credentials` blocks entirely when deploying via `n8n_create_workflow`; placeholders are safe only for manual file import |
| Downstream node reads the wrong branch's data after IF branches reconverge | Bare `$json` after a reconvergence point — last-fired branch wins non-deterministically | Reference the source node explicitly: `$('Node Name').item.json.field` |
| Merge node silently drops the 3rd+ input | `numberOfInputs` left at its default of 2 while 3+ wires connect in | Set `parameters.numberOfInputs` on the Merge node to the actual wire count |
| "$env is not defined" or empty value in an expression | `$env.X` is not available in n8n expressions | Use `$vars.X`, or move the secret into a credential |
| Slack message posts raw JSON / plain text instead of formatted blocks | Block Kit passed as a bare array | Wrap it: `={{ { "blocks": [...] } }}` |
| Node fails hard on a transient 429/timeout that Zapier used to absorb | n8n has no automatic retries | Set `retryOnFail: true`, `maxTries: 3`, `waitBetweenTries: 2000` on the node (Phase 3 rule 6) |
| Downstream node reads `$json.<field>` as undefined immediately after an httpRequest node | httpRequest REPLACES item `$json` with the API response, dropping fields a Code node added upstream | Reference the carrier node absolutely via paired items: `$('Build Row').item.json.field`. If many branches converge, add one passthrough Code node (`return $input.all();`) before the httpRequest and reference it downstream |
| Google Sheets node can't reproduce Zapier COL$ letter mapping (headers unknown / sheet must not be edited) | n8n Sheets node maps by header NAME, not column letter | Use raw Google Sheets REST API via httpRequest with a positional values array (A=index0…AB=27), '' for blanks — see Phase 3 action table |
| Google Chat 400 "Message cannot have cards for requests carrying human credentials" | cardsV2 can only be sent by a Chat app / service account; human OAuth sends text only | Use `n8n-nodes-base.googleChat` with a `googleApi` service-account credential + `messageJson` cardsV2; or fall back to a `{text:...}` message (bold `*..*`, `\n`, `<url|label>`) |
| Workflow runs on its own every interval even though it was never published / was unpublished | Schedule/polling triggers are NOT gated by Publish or Active state — they fire as soon as the workflow is saved on the instance | Disable the trigger node, or Archive the workflow (unpublish may not stop it) |
| n8n cluster OOM / out-of-memory, other workflows affected, host crashes | Polling interval shorter than actual run time, so executions stack and run concurrently, each holding its full dataset in memory | Widen the interval with real headroom over worst-case run time, narrow the per-run read ranges, and add a concurrency guard so a new run exits if one is already in flight |
| Disabling downstream nodes doesn't stop a workflow from running repeatedly | Only the nodes after the trigger were disabled; the trigger and all upstream reads still execute every interval | Disable the trigger node itself, or Archive the workflow |

When fixing runtime errors in the live workflow:
1. Edit the saved JSON file with the fix
2. Tell the user to make the same change directly in the live node (re-importing would wipe their credentials)
3. Explain the fix in one sentence

---

## Notes for Multi-Workflow Batches

When the user has multiple Zapier workflows to migrate:
1. Process one at a time — ask which to start with
2. Re-use the same environment answers (n8n version, admin status, AI node availability) — don't re-ask
3. Re-use the same credential placeholder names across workflows (same user, same n8n instance)
4. After each workflow, ask: "Ready for the next one, or anything to change in this approach first?"
5. Number output files clearly: `workflow1_n8n.json`, `workflow2_n8n.json`, etc.

---

## Phase 8 — Memory Capture (run at end of EVERY migration session)

After the workflow is delivered and testing is complete, do all of the following automatically — no need to ask.

### Step 1 — Save new errors to memory

For every error the user encountered that is NOT already in the Phase 7 error table, append it to `feedback_n8n_common_errors.md`:

```
**Error: "<short description>"**
- Cause: <root cause>
- Fix: <fix>
- Discovered: <YYYY-MM-DD>, workflow: <workflow name>
```

### Step 2 — Save new node mappings to memory

If this migration used a Zapier app type not in the Phase 3 conversion tables, append to `feedback_n8n_node_mappings.md` (create if missing):

```
**Zapier app:** <selected_api value>
**n8n node:** <type>, typeVersion <N>
**Config notes:** <any gotchas>
**Discovered:** <YYYY-MM-DD>
```

Add a MEMORY.md index entry if the file is new.

### Step 3 — Increment migration counter

Read `project_migration_count.md`. Increment the count by 1 and write it back. Format:

```
---
name: project-migration-count
description: Running count of completed Zapier-to-n8n migrations — used to trigger periodic skill rebuilds
metadata:
  type: project
---
Completed migrations: <N>
Last migration: <workflow name>, <YYYY-MM-DD>
```

### Step 4 — Trigger skill rebuild if count is a multiple of 3

If the new count is a multiple of 3, say exactly this to the user — then immediately run Phase 9 without waiting for any response:

> "Updating the migration skill now with everything learned across the last 3 migrations..."

Do not ask for confirmation. Do not wait. Proceed directly into Phase 9.

---

## Phase 9 — Skill Self-Rebuild

This phase rewrites the skill file itself with all accumulated learnings from memory, repackages it as a `.skill` file, and presents it for reinstall. Run this when: (a) triggered automatically from Phase 8 Step 4, (b) the user says "update/rebuild/sync the skill," or (c) the migration count check at startup fires.

### Step 1 — Read current memory

Read these memory files:
- `feedback_n8n_common_errors.md` — all known errors
- `feedback_n8n_node_mappings.md` — all known node type mappings (if it exists)
- `project_n8n_env.md` — environment specifics (instance URL, installed packages, credential names)
- `project_migration_count.md` — migration count

### Step 2 — Diff against current skill

Read the current SKILL.md. Identify:
- Errors in memory that are NOT yet in the Phase 7 error table → add them
- Node mappings in memory that are NOT yet in the Phase 3 conversion table → add them
- Any environment facts in memory that should be noted as known defaults → update Phase 1 accordingly

### Step 3 — Update SKILL.md

Edit the SKILL.md file in place:
- Add new rows to the Phase 7 error table
- Add new rows to the Phase 3 node mapping tables
- Do NOT change any other section unless a memory entry explicitly contradicts existing guidance

The SKILL.md lives at the path it was originally created. Use the Edit tool to make targeted changes — do not rewrite the whole file.

### Step 4 — Repackage

Run this shell sequence:

```bash
cd /tmp
rm -rf zapier-to-n8n zapier-to-n8n.skill
mkdir zapier-to-n8n
cp "<path-to-SKILL.md>" zapier-to-n8n/
zip -r zapier-to-n8n.skill zapier-to-n8n/
cp zapier-to-n8n.skill "<workspace-folder>/"
```

The workspace folder is the user's connected project folder (e.g. `/Users/.../Documents/Claude/Projects/Zapier —> n8n/`).

### Step 5 — Present for reinstall

Call `mcp__cowork__present_files` with the new `.skill` file path. Say exactly this (one sentence):

"The skill has been updated with [N] new error patterns — click **Save skill** to reinstall it."

That's all. No further explanation needed. The user clicks Save skill and the updated version replaces the old one.

### What "reinstall" actually means

Cowork skills are installed via the Save skill button on a `.skill` file card. There is no API to reinstall silently — the user must click once. This is the minimum possible friction: Claude builds the file, presents it, user clicks one button. The old skill is replaced automatically.
