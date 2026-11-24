# Core concepts glossary

## Execution model

**Temporal Service (Server/Cluster)** — Stores Event Histories, schedules Tasks,
enforces timeouts. It **never runs your code**; it only records what happened and
hands out work.

**Worker** — *Your* process, using the SDK to poll a Task Queue and execute the
Workflow and Activity code registered on it. Scaling throughput means running
more Workers. Nothing runs if no Worker polls the queue — the most common cause
of a Workflow that sits in "Running" with no progress.

**Task Queue** — A named channel connecting Workflow/Activity Tasks to the
Workers that can run them. The name in `client.StartWorkflowOptions.TaskQueue`
and in `worker.New(...)` must match exactly.

**Namespace** — Isolation boundary for Workflows, with its own retention period
and configuration. The dev server gives you `default`.

## Workflows

**Workflow Definition** — The function you write (`func(workflow.Context, In) (Out, error)`).
Must be deterministic; see `determinism.md`.

**Workflow Type** — The registered name, defaulting to the Go function name.
Renaming the function changes the Type and orphans running Executions unless you
register an explicit name.

**Workflow Execution** — One running instance, identified by **Workflow ID**
(yours; use a business key) plus **Run ID** (Temporal's, new for each run,
including after `ContinueAsNew` or a retry).

**Event History** — The append-only, durable log of everything the Execution did:
Activities scheduled and completed, timers, Signals received, and their results.
This *is* the Workflow's state; it's what replay reads. Capped at 51,200 events /
50 MB.

**Workflow Task** — A unit of work handed to a Worker telling it to advance the
Workflow: replay the history, run until the code blocks, and report back the new
commands. Workflow code runs inside these, which is why a Workflow Task must
finish quickly (the deadlock detector fires after 1s of no progress).

**Replay** — Re-running Workflow code from the start against recorded history to
rebuild in-memory state after a crash, deploy, or long sleep. The reason
determinism is mandatory.

**Command** — What Workflow code asks Temporal to do (schedule an Activity, start
a timer, complete the Workflow). The recorded sequence of commands is what must
stay stable across replays.

**ContinueAsNew** — Atomically ends the current Execution and starts a fresh one
with a new (empty) history, carrying state forward as input. How long-running or
looping Workflows stay under the history cap.

## Activities

**Activity Definition** — A normal Go function (`func(context.Context, In) (Out, error)`)
that does real work: I/O, API calls, computation. No determinism constraints.

**Activity Execution** — One invocation, retried per its Retry Policy.
Delivery is **at-least-once**, so Activities should be idempotent.

**Heartbeat** — `activity.RecordHeartbeat(ctx, details)` tells the server the
Activity is alive and checkpoints progress. Required for long Activities so a
dead Worker is detected quickly (via `HeartbeatTimeout`) rather than waiting out
`StartToCloseTimeout`. Details are returned to the next attempt.

**Local Activity** — Runs in the same Worker process without a Task Queue round
trip. Lower latency, weaker guarantees. Only for very short operations.

**Retry Policy** — `InitialInterval`, `BackoffCoefficient`, `MaximumInterval`,
`MaximumAttempts`, `NonRetryableErrorTypes`. Activities retry by default with
unlimited attempts; Workflows do not retry by default. See `pitfalls.md`.

**Timeouts** — `StartToClose` (one attempt), `ScheduleToClose` (all attempts
including backoff), `ScheduleToStart` (queue wait), `Heartbeat`. One of
`StartToClose` / `ScheduleToClose` is required.

## Messages

**Signal** — Asynchronous one-way message into a running Workflow. Mutates state,
returns nothing, is recorded in history. `SignalWithStartWorkflow` delivers a
Signal and starts the Workflow if it isn't running.

**Query** — Synchronous, **read-only** request for Workflow state. Must not
mutate state, execute Activities, or block. Not recorded in history — so it can
be served by replay against any Worker.

**Update** — Synchronous call that *can* mutate state and return a value,
recorded in history, with an optional **validator** that can reject the request
before it's recorded. Roughly "a Signal and a Query in one, with rejection".

Choosing between them: read-only → Query; fire-and-forget → Signal; need a
result or must reject bad input → Update.

## Composition

**Child Workflow** — A Workflow started by another Workflow, with its own
Execution, history budget, and lifecycle. **Parent Close Policy** decides what
happens to it when the parent closes. Use when you need separate history or
different Task Queue routing; otherwise prefer an Activity.

**SideEffect** — Records the result of a cheap non-deterministic computation
(UUID, random number) in history so replay reuses the value. Not an Activity: no
retries, no timeouts. Never do I/O in it. **MutableSideEffect** is a variant that
re-evaluates and only records when the value changes.

**Timer** — `workflow.Sleep` / `workflow.NewTimer`. Durable: survives Worker
restarts, consumes no resources while pending, and can safely span months.

**Selector** — `workflow.NewSelector(ctx)`, the deterministic analogue of Go's
`select`. Races futures, timers, and Signal channels.

**Await** — `workflow.Await(ctx, predicate)` blocks until a condition over
Workflow state becomes true. Re-evaluated after each event, so keep it pure.

## Operations

**Versioning / Patching** — `workflow.GetVersion` branches between old and new
code paths so a deploy doesn't break in-flight Executions. **Worker Versioning**
is the alternative: pin Workers to a Build ID and let old Executions finish on
old code. See `pitfalls.md`.

**Data Converter** — Controls serialization of inputs, outputs, and Signals
(JSON by default). The extension point for encryption or compression.

**Search Attributes / Memo** — Indexed (queryable) and non-indexed metadata
attached to an Execution, for finding Workflows in the UI or via List filters.
