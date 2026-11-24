---
name: temporal-go
description: Build durable applications on Temporal (temporal.io) with the Go SDK — Workflows, Activities, Workers, Signals, Queries, Updates, child Workflows, retries, and versioning. Use when writing or reviewing Temporal Go code, debugging non-determinism or replay failures, choosing timeouts and retry policies, or setting up Workflow tests.
---

# Building Temporal applications in Go

## When to use this skill

Use this when the task involves `go.temporal.io/sdk`: writing a Workflow or
Activity, setting up a Worker, wiring a client, choosing timeouts or retry
policies, adding Signals/Queries/Updates, testing Workflow code, or diagnosing
a `NonDeterministicError` / replay failure.

Not a fit for: Temporal Cloud account administration, self-hosted cluster
operations, or SDKs other than Go.

## The mental model

A **Workflow** is a durable function. Temporal records every step it takes into
an **Event History**. If the process running it crashes, Temporal replays that
history against your code on another Worker to rebuild the in-memory state,
then continues from where it left off. Your function appears to run for days or
months, uninterrupted, across deploys and machine failures.

**Replay is the whole reason determinism matters.** Replayed code must make the
same decisions it made the first time. If it doesn't — because it read the
clock, generated a UUID, or called an API — the rebuilt state diverges from the
recorded history and Temporal fails the Workflow Task.

So the code is split in two. **Workflow code** orchestrates: it decides what
runs, in what order, and what to do on failure. It must be deterministic.
**Activity code** does the actual work: HTTP calls, database writes, file I/O,
anything that can fail on its own or return a different answer twice. Activities
are ordinary Go with no restrictions, and Temporal retries them for you.

A **Worker** is your process, polling a **Task Queue** for work and executing
the Workflow and Activity code you registered on it. Temporal stores state and
hands out tasks; it never runs your code. A **client** starts Workflows and
sends them Signals, Queries, and Updates from outside.

## Determinism: read this before writing Workflow code

The single most common way to get Temporal wrong is putting non-deterministic
work directly in a Workflow function. In Go **there is no sandbox** — nothing
stops you from calling `time.Now()` in Workflow code. It compiles, it runs, and
it corrupts the Execution on the first replay, often days later in production.

Inside a Workflow function, never call:

| Never | Use instead |
| --- | --- |
| `time.Now()` | `workflow.Now(ctx)` |
| `time.Sleep()` | `workflow.Sleep(ctx, d)` |
| `time.After()` in `select` | `workflow.NewTimer(ctx, d)` + `workflow.Selector` |
| `rand.Intn()` | `workflow.SideEffect(ctx, ...)` |
| `uuid.New()` | `workflow.SideEffect(ctx, ...)` |
| `os.Getenv`, file/network/DB I/O | an Activity |
| `go func()` | `workflow.Go(ctx, ...)` |
| `sync.Mutex`, `chan`, `select` | `workflow.Mutex`, `workflow.Channel`, `workflow.Selector` |
| map iteration where order matters | sort the keys first |
| mutable package-level state | Workflow-local variables |

Two rules behind the table:

1. **Anything that can return a different answer on a second run must be
   recorded, not recomputed.** Activities and `SideEffect` record their results
   in history; replay reads the recorded value instead of running the code again.
2. **Never block a Workflow goroutine on anything but a Workflow API.** Blocking
   on a real channel, mutex, or syscall stalls the Workflow Task until the
   deadlock detector kills it (1 second of wall-clock progress by default).

### Decision checklist

For every step you are about to add, ask:

- Does it **call an external system** (HTTP, DB, queue, filesystem, another service)?
- Can it **fail independently** of the rest of the Workflow, in a way worth retrying?
- Is it **non-deterministic** (clock, randomness, UUID, env, changing data)?
- Does it take **more than a few milliseconds** of CPU, or block?

**Any "yes" → it belongs in an Activity, not in Workflow code.**

If it is pure computation over data the Workflow already holds — branching,
arithmetic, building a struct — leave it inline.

`SideEffect` is the narrow exception for cheap non-deterministic values (a UUID,
a random number) that aren't worth an Activity's round trip. It records the value
in history but gets no retries and no timeouts — never put I/O in it.

Full rules and wrong-vs-right code: `references/determinism.md`.

## Getting started

`templates/go-starter/` is a complete, runnable project — Workflow, Activities,
Worker, client, and tests — verified against `temporal server start-dev`. Copy it
as a starting point rather than assembling one from scratch:

```bash
temporal server start-dev          # UI on http://localhost:8233
cd templates/go-starter
go run ./worker                    # terminal 1
go run ./starter                   # terminal 2
go test ./...
```

It demonstrates explicit retry policies, non-retryable errors, heartbeating, a
Signal raced against a timer, a Query handler, deterministic ID generation, and
both unit and replay tests.

## Reference files

Load these as needed — don't read them all up front.

| File | Read it when |
| --- | --- |
| `references/determinism.md` | Writing/reviewing Workflow code, or debugging a replay failure |
| `references/go-sdk.md` | You need the Go API surface: options, Selectors, Signals/Queries/Updates, child Workflows, `ContinueAsNew`, **compensation/Saga** |
| `references/concepts.md` | A Temporal term is unfamiliar |
| `references/testing.md` | Writing Workflow tests, mocking Activities, or adding replay tests |
| `references/pitfalls.md` | Choosing timeouts/retries, deploying a change to running Workflows, or hitting size limits |

## Non-negotiables

- **Every Activity needs `StartToCloseTimeout`** (or `ScheduleToCloseTimeout`).
  The SDK refuses to run an Activity with neither set.
- **Set an explicit `RetryPolicy` with `MaximumAttempts`** on Activities that
  call external systems. The default is *unlimited* retries, which turns a
  permanently-broken dependency into a Workflow that hangs forever.
- **Mark permanent failures non-retryable** with
  `temporal.NewNonRetryableApplicationError(...)` so bad input fails fast.
- **Make Activities idempotent.** Delivery is at-least-once; any Activity can run
  twice with the same input.
- **Never change the sequence of Workflow API calls in code that has running
  Executions** without `workflow.GetVersion`. See `references/pitfalls.md`.
- **In any multi-step transaction, plan the failure path.** Temporal retries
  steps but never undoes completed ones — if step 3 fails permanently, steps 1
  and 2 stay committed. Order processing, payments, and bookings all need
  explicit compensation; see the Saga section in `references/go-sdk.md`.
