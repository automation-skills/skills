# temporal-go-skill

A [Claude Code](https://claude.com/claude-code) skill for building
[Temporal](https://temporal.io) applications with the **Go SDK**.

It gives Claude the context it needs to write Temporal code that is correct on
the details that are easy to get wrong and expensive to get wrong late:
determinism, retry policies, timeouts, and versioning.

## Why this exists

Temporal has a small API surface and a large number of ways to misuse it. Most
mistakes compile cleanly, pass tests, run fine in staging, and only surface in
production — sometimes days after the deploy that caused them.

The Go SDK is especially exposed here. Unlike the TypeScript SDK, which runs
workflow code in a sandbox that replaces `Date` and `Math.random` with
deterministic versions, **the Go SDK has no sandbox**. Calling `time.Now()` or
`uuid.New()` inside a workflow is valid Go that runs correctly the first time and
corrupts the execution on the first replay.

This skill front-loads the constraints an LLM would otherwise have to infer:

- **Determinism rules**, stated as a table of what to never call and what to use
  instead — with the reasoning, so it generalizes past the listed cases.
- **Retry and timeout semantics**, including the defaults that surprise people
  (activities retry *unlimited* times by default; workflows don't retry at all).
- **A decision checklist** for the recurring judgement call: does this step
  belong in workflow code, or in an activity?
- **The failure paths**, not just the happy path — compensation, non-retryable
  errors, versioning a workflow that's already running.

Everything in it was verified against a live Temporal server rather than written
from memory. See [Accuracy](#accuracy).

## Installation

Skills are directories containing a `SKILL.md`. Install this one by cloning it
into your skills directory.

**Personal** (available in every project):

```bash
git clone https://github.com/campoy/temporal-go-skill ~/.claude/skills/temporal-go
```

**Project-scoped** (checked in, shared with your team):

```bash
git clone https://github.com/campoy/temporal-go-skill .claude/skills/temporal-go
```

Claude loads `SKILL.md` when a task looks like Temporal Go work, and pulls in
files from `references/` only as it needs them — so the detailed material costs
nothing until it's relevant.

## Usage

Just describe what you want. The skill activates on Temporal Go tasks without
being named:

```
Build a Temporal workflow in Go that processes an order with a payment step
that can be retried.
```

Other prompts it's built for:

```
This workflow started failing with a non-determinism error after last
Friday's deploy. Figure out what broke and fix it properly.
```

```
Review this workflow for determinism violations and check the retry
policies are sane.
```

```
Add a human approval step that times out after 48 hours and falls back to
auto-rejection.
```

```
Write tests for this workflow — mock the activities, and make sure the
30-day timer doesn't make the suite slow.
```

You can also invoke it explicitly if it doesn't trigger on its own: *"Using the
temporal-go skill, ..."*

### Getting a project running

The repo includes a complete working application. To try it:

```bash
temporal server start-dev              # Web UI at http://localhost:8233

cd templates/go-starter
go run ./worker                        # terminal 2
go run ./starter                       # terminal 3
go test ./...
```

## What's in here

| Path | Purpose |
| --- | --- |
| `SKILL.md` | Entry point. When to use the skill, the core mental model, the determinism rules, and the activity-vs-workflow decision checklist. Deliberately short — everything else is loaded on demand. |
| `references/determinism.md` | The determinism rules in depth, as wrong-vs-right Go snippets: clocks, timers, randomness, I/O, concurrency, map iteration, global state. Plus how to diagnose a non-determinism failure. |
| `references/go-sdk.md` | The Go API surface: workflow/activity shapes, worker setup and dependency injection, activity options, error types, heartbeating, signals/queries/updates, child workflows, `ContinueAsNew`, and the compensation (Saga) pattern. |
| `references/pitfalls.md` | The recurring failure modes: retry-policy vs. timeout confusion, versioning a running workflow after a deploy, oversized payloads, blocking calls in workflow code, non-idempotent activities. |
| `references/testing.md` | The time-skipping test environment, mocking activities, driving signals and queries from tests, replay tests, and integration tests against a real dev server. |
| `references/concepts.md` | Glossary of Temporal terminology — workers, task queues, event history, workflow tasks, replay, signals/queries/updates, and the rest. |
| `templates/go-starter/` | A complete, runnable starter project — workflow, activities, worker, client, and tests. See its own [README](templates/go-starter/README.md). |

### The template

`templates/go-starter/` is a real application, not a collection of snippets. It
implements a document ingestion workflow — validate, extract, wait for human
approval, index — chosen to exercise the patterns worth copying:

- Per-activity timeouts and retry policies, sized to what each activity does
- Non-retryable errors, so malformed input fails fast instead of retrying
- Heartbeating on a long-running activity, with cancellation handling
- A signal raced against a 48-hour timer in a `workflow.Selector`
- A query handler for progress
- Deterministic ID generation via `workflow.SideEffect`
- Unit tests with mocked activities and skipped time, plus a replay test against
  a real recorded event history

## Accuracy

Written against **Go SDK v1.47.0** and **Temporal CLI v1.8.1** (Server 1.31.2).

- Every code snippet in `SKILL.md` and `references/` was compile-checked against
  the real SDK.
- The template was run end to end against `temporal server start-dev` — including
  confirming the retry policy actually produces the retries it claims to.
- The replay test was verified to *fail* when a determinism break is injected,
  so it isn't false comfort.
- Behaviours that are easy to assume wrongly were tested rather than guessed —
  for example, that `ScheduleToCloseTimeout` and `MaximumAttempts` are
  independent limits where whichever is reached first wins, and that an
  exhausted retry budget surfaces the last attempt's error rather than a
  `TimeoutError`.

The skill was also evaluated by giving fresh Claude instances an order-processing
task with access to nothing but these files, then auditing the output for
determinism violations and retry-policy mistakes. Gaps that surfaced — missing
compensation guidance, an under-specified dependency-injection pattern — were
fixed and re-tested.

Two places where the upstream documentation is unsettled, flagged rather than
silently resolved:

- **Worker Versioning vs. patching.** Temporal now recommends Worker Versioning
  over `workflow.GetVersion` for many cases. Both are documented, with the
  trade-off stated, because the recommendation is still moving.
- **Go patching APIs.** Temporal's Go versioning docs show only
  `workflow.GetVersion`, while the TypeScript docs feature `patched()` /
  `deprecatePatch()`. This skill documents `GetVersion`, which is what the
  official Go samples use.

## Scope

Go only. It covers writing and testing Temporal applications — not Temporal Cloud
account administration or self-hosted cluster operations.

## License

[0BSD](LICENSE) (Zero-Clause BSD) — a public-domain-equivalent license.

Copy anything here into your own projects, commercial or otherwise, with no
attribution and no conditions. The starter template in particular is meant to be
lifted wholesale; you should not have to carry a copyright notice around in your
boilerplate.

**This applies to the code in this repository only.** The template's `go.mod` and
`go.sum` declare third-party dependencies that `go mod download` fetches from
their own sources — they are not distributed here and are not relicensed by this
file. They remain under their own terms: the Temporal Go SDK is MIT, and its
transitive dependencies vary. If your organization audits dependency licensing,
run that audit against the resolved module graph (`go list -m all`), not against
this license.
