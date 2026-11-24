# Temporal Go starter

A complete, runnable Temporal application: Workflow, Activities, Worker, client,
and tests. Verified against `temporal server start-dev` (CLI v1.8.1 / Server
1.31.2) with Go SDK v1.47.0.

`IngestDocumentWorkflow` validates a document, extracts its text, waits for human
approval, then indexes it.

## Run it

Three terminals:

```bash
# 1. Dev server — Web UI at http://localhost:8233
temporal server start-dev

# 2. Worker
go run ./worker

# 3. Client: starts the Workflow, queries status, sends the approval Signal
go run ./starter
```

Expected output from the starter:

```
started workflow workflowID ingest-doc-42 runID ...
queried status: extracting
sent approval signal
workflow completed: {DocumentID:doc-42 IndexID:... WordCount:40 Approved:true}
```

The Worker log shows `ExtractText` failing attempts 1 and 2 before succeeding on
attempt 3 — the retry policy doing its job. Open the Web UI to see the Event
History, including the retries and the Signal.

```bash
go test ./...
```

## What's here

| File | Contents |
| --- | --- |
| `workflow.go` | Workflow definition, Query handler, Signal-vs-timer Selector, `SideEffect` for a deterministic ID |
| `activities.go` | Three Activities: non-retryable validation, heartbeating extraction, idempotent indexing |
| `worker/main.go` | Worker setup and registration |
| `starter/main.go` | Client: start, Query, Signal, await result |
| `workflow_test.go` | Workflow tests with mocked Activities and skipped time; an Activity test |
| `replay_test.go` | Replay test against a recorded history |
| `testdata_history.json` | Real Event History exported from a run, used by the replay test |

## Patterns worth copying

**Per-Activity options.** Each Activity gets its own timeouts and retry policy,
derived from what it does — a fast validation gets 10s and 3 attempts; a slow
network fetch gets 5 minutes, heartbeating, and 5 attempts.

**Non-retryable errors.** `ValidateDocument` returns
`temporal.NewNonRetryableApplicationError(..., ErrInvalidDocument, nil)` so
malformed input fails immediately instead of consuming the retry budget.

**Heartbeating.** `ExtractText` calls `activity.RecordHeartbeat` and honours
`ctx.Done()`, paired with `HeartbeatTimeout: 30s`.

**A status Query.** Included to demonstrate the handler; it is *optional*, not a
required part of every Workflow. Add one when something outside needs to observe
progress, and drop it otherwise.

**Waiting for a human, with a deadline.** `waitForApproval` races a Signal
channel against a 48-hour timer in a `workflow.Selector`. The test exercises both
outcomes in milliseconds thanks to time skipping.

**Deterministic IDs.** The index ID comes from `workflow.SideEffect`, not a bare
`uuid.New()`, and is passed *into* the Activity so a retry writes under the same
key.

**Idempotency.** `IndexDocument` takes the ID from the caller rather than
generating one, so re-execution is safe.

## Adapting it

1. Rename the module in `go.mod` and the import in `worker/`, `starter/`.
2. Replace the Activities with real work; keep the timeout and retry structure.
3. Delete `SimulateTransientFailures` from `activities.go` — it exists only to
   make retries visible in the demo.
4. Regenerate the replay fixture once your Workflow shape is settled:
   ```bash
   temporal workflow show --workflow-id <id> --output json > testdata_history.json
   ```
