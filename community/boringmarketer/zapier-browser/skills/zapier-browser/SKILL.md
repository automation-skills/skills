---
name: zapier-browser
description: Build and edit Zapier Zaps by driving the web editor with browser automation. Use when asked to create, modify, wire up or debug a Zap, add steps, map fields between steps, or when a Zap needs a step the Zapier MCP cannot add. Covers the editor's traps - virtualised pickers, hidden required fields, sync timeouts - and the raw-expression mapping technique that avoids the data picker entirely.
---

# Driving the Zapier editor

Zapier has no public write API for authoring Zaps. The only way to build one programmatically is
the web editor, and the editor is hostile to automation in specific, learnable ways. This skill is
the set of techniques that actually work.

**Read "The one technique that matters" first.** It removes most of the pain.

---

## The one technique that matters: map fields by typing a raw expression

Do **not** use the data picker to map a field between steps. Type the expression straight into the
field instead. It resolves into a proper token immediately, first time, every time:

```
{{=gives["<STEP_ID>"]["<outputFieldName>"]}}
```

- **`STEP_ID`** is in the editor URL while that step is open: `/editor/<zapId>/draft/<STEP_ID>/fields`.
  New steps look like `_GEN_1700000000000`; published ones are a plain integer.
- **`outputFieldName`** must be the *machine* name from that step's own **Data out** panel, not the
  friendly label. Guessing this is the single most common cause of a token that renders perfectly
  and evaluates to empty.

Trigger fields use a different, positional form that is useful when a column has no friendly label:

```
{{<zapId>__new__data__f7}}      // 7th field of a Zapier Tables trigger row
```

### Why not the picker

The picker is virtualised (rows are not in the DOM until scrolled into view), the panel re-scrolls
between your screenshot and your click so coordinates go stale, typing `/` to open it leaves a
literal `/` in the field, and the `+` button **adds a new row** to multi-value fields rather than
filling the empty one. Every one of these produces a silent, wrong result.

If you must use the picker, the only reliable sequence is: click the field's `+`, scroll the picker
so the rows render, then click the row **through the DOM** (see below) rather than by coordinate.

---

## Division of labour: you build structure, Copilot fills fields

The editor has a Copilot panel. Use it deliberately - it is excellent at one half of the job and
incapable of the other.

**Copilot is good at:** setting and remapping field values, finding required fields you cannot see,
resolving real output field names, editing long AI prompts, and scaffolding an entire new Zap from a
description (use the "Describe your workflow" box on a brand-new Zap - that is its strongest mode).

**Copilot cannot:** add Delay steps, add some app steps mid-Zap ("backend validation issue"). It will
tell you so and hand you manual instructions.

**So:** add the step shells yourself, then hand the fields to Copilot, then type any awkward mapping
as a raw expression. Always scope its instructions: *"Change step 4 only. Leave every other step
exactly as it is."* It will otherwise wander.

**Never click a "Copilot can fully configure your workflow / Build it" banner on an existing Zap.**
That reconfigures everything. On a new empty Zap it is exactly what you want.

---

## Clicking things that resist

Three escalating techniques:

1. **`find` + `ref`.** For dropdown options, radios and account pickers, get a ref and click that.
   Clicking option *text* by coordinate frequently does not register the selection.
2. **DOM dispatch.** For virtualised list rows, dispatch the full mouse sequence on the element
   under the row's own bounding box:
   ```js
   const row=[...document.querySelectorAll('div.list-item')]
     .find(e=>(e.textContent||'').trim().startsWith('website (new)'));
   row.scrollIntoView({block:'center'});
   const b=row.getBoundingClientRect();
   const t=document.elementFromPoint(b.left+30,b.top+b.height/2);
   ['pointerdown','mousedown','pointerup','mouseup','click']
     .forEach(n=>t.dispatchEvent(new MouseEvent(n,{bubbles:true,cancelable:true,
       clientX:b.left+30,clientY:b.top+b.height/2})));
   ```
3. **Screenshot immediately before every click.** Never reuse coordinates across tool calls. The
   panel re-scrolls on its own.

**React Flow nodes render at opacity 0** in an automated window. Inject this once per page load or
the canvas looks empty:
```js
const st=document.createElement('style');
st.textContent='.react-flow__node,.react-flow__edge{opacity:1!important;visibility:visible!important}';
document.head.appendChild(st);
```

---

## Traps that cost hours

**Required fields hide below the fold.** A disabled "Continue" with no visible error almost always
means a required field you have not scrolled to. Ask Copilot "what required field is still empty on
this step" before investigating anything else. Do not assume the field you are fighting is the
problem - verify.

**Read the test's Data in panel, always.** It shows what will actually be sent. A mapping that looks
right in the UI can resolve to empty. Real example: a digest Title mapped by friendly name rendered
as `-` because both tokens were blank, which would have merged every search into one bucket. The UI
gave no hint; Data in showed it instantly.

**Long-running actions cannot run synchronously.** Zapier's sync call times out at **30 seconds**.
Anything with container start-up (most Apify actors) will exceed it regardless of how small you make
the job. The working shape is three steps:

> Run Actor (**Run synchronously: No**) → **Delay by Zapier**, 1-2 minutes → the app's
> **Fetch/Get results** action, pointed at the first step's output id

**Aggregating many rows into one payload:** use **Digest by Zapier**. Append an entry in the
per-item Zap (Frequency: Manual), release it in a second Zap. A Tables multi-record lookup returns
one row or comes back comma-joined, which breaks mapping.

**The AI-by-Zapier prompt is silently truncated at 50,000 characters on save.** Put the most
important instruction and any data pills FIRST. Zapier also auto-infers Output Fields from the
prompt - delete them if you want one blob back instead of structured fields.

**Line items vs nested results.** When an action offers both, map the flat "Dataset Items …" style
line items, not nested "Results Items …" - the nested form arrives comma-joined.

**Stale trigger samples.** A newly added table column shows as a raw slot (`New Data F 7`) with no
label until a fresh record carries a value. Address it positionally and verify with a real row.

---

## Verification and safety

- **Test every step you touch, and read the output.** A step that goes green having sent nothing is
  worse than a red one.
- **Never test a step that sends email, posts publicly, or charges money** unless the user has said
  to. Say which steps you skipped and why.
- **Never publish without asking.** Publishing turns the Zap on. Confirm first, then name the
  version descriptively (`v4: contact enrichment`) so the history is readable and revertible.
- Drafts are safe: the published version keeps running untouched while you edit. Prefer leaving work
  as a tested draft over publishing on your own judgement.
- Copilot adds **checkpoints** with a Revert button - a cheap undo when it goes off-script.
- Copilot's "Troubleshoot" tab hallucinates. Trust the raw error text and the Data in panel over its
  diagnosis.

## Related

`tables.zapier.com` is often blocked for browser automation. Add or alter table fields through the
Zapier MCP (`zapier_tables_add_field_s`) or have the user do it, and note that the MCP needs its
Tables connection authorised first.
