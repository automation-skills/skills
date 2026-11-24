# Testing Temporal Go code

Three layers, each catching a different class of bug:

1. **Workflow unit tests** — logic and orchestration, Activities mocked, time skipped.
2. **Activity unit tests** — the Activity function in isolation.
3. **Replay tests** — current code against a real recorded history, to catch
   determinism regressions before deploy.

All of it lives in `go.temporal.io/sdk/testsuite` and needs no running server.

## Workflow unit tests

`TestWorkflowEnvironment` runs the Workflow in-process with a **time-skipping
clock**: a `workflow.Sleep(ctx, 30*24*time.Hour)` returns instantly. Tests of
month-long Workflows run in milliseconds.

```go
func TestIngest_ApprovedPath(t *testing.T) {
    env := (&testsuite.WorkflowTestSuite{}).NewTestWorkflowEnvironment()

    env.OnActivity(ValidateDocument, mock.Anything, mock.Anything).Return(nil)
    env.OnActivity(ExtractText, mock.Anything, mock.Anything).
        Return(&ExtractResult{Text: "hello world", WordCount: 2}, nil)
    env.OnActivity(IndexDocument, mock.Anything, mock.Anything).Return(nil)

    env.ExecuteWorkflow(IngestDocumentWorkflow, IngestInput{DocumentID: "doc-1"})

    require.True(t, env.IsWorkflowCompleted())
    require.NoError(t, env.GetWorkflowError())

    var result IngestResult
    require.NoError(t, env.GetWorkflowResult(&result))
    require.True(t, result.Approved)
    env.AssertExpectations(t)
}
```

`ExecuteWorkflow` blocks until the Workflow finishes (or the test environment's
own timeout trips). Always assert `IsWorkflowCompleted` before reading results.

### Mocking Activities

The mock's argument list is `(context, args...)`, so the first matcher is always
for the context:

```go
env.OnActivity(ChargeCard, mock.Anything, mock.Anything).Return(nil)

// Match a specific input.
env.OnActivity(ChargeCard, mock.Anything, ChargeInput{Amount: 100}).Return(nil)

// Return an error.
env.OnActivity(ChargeCard, mock.Anything, mock.Anything).
    Return(temporal.NewNonRetryableApplicationError("declined", "CardDeclined", nil))

// Dynamic behaviour — the func signature must match the Activity's.
var attempts int
env.OnActivity(ChargeCard, mock.Anything, mock.Anything).
    Return(func(ctx context.Context, in ChargeInput) error {
        attempts++
        if attempts < 3 {
            return errors.New("transient")
        }
        return nil
    })

// Constrain call counts.
env.OnActivity(ChargeCard, mock.Anything, mock.Anything).Return(nil).Times(1)
```

`Return(func(...))` is the way to assert **retry behaviour**: count invocations
and check the Workflow's retry policy actually produced the attempts you expect.

If you register an Activity without mocking it, the real implementation runs:

```go
env.RegisterActivity(ValidateDocument) // real
env.OnActivity(ChargeCard, ...).Return(nil) // mocked
```

### Driving Signals, Queries, and time

Callbacks are scheduled on the **skipped** clock, so `time.Minute` here costs no
real time. Use them to deliver a Signal once the Workflow is parked:

```go
env.RegisterDelayedCallback(func() {
    env.SignalWorkflow(ApproveSignal, ApproveInput{Approved: true})
}, time.Minute)
```

Querying mid-run:

```go
env.RegisterDelayedCallback(func() {
    res, err := env.QueryWorkflow(StatusQuery)
    require.NoError(t, err)
    var status string
    require.NoError(t, res.Get(&status))
    require.Equal(t, "awaiting_approval", status)
}, time.Minute)
```

Sending an Update:

```go
env.RegisterDelayedCallback(func() {
    env.UpdateWorkflow("setLimit", "update-1", &testsuite.TestUpdateCallback{
        OnAccept:   func() {},
        OnReject:   func(err error) { t.Fatal(err) },
        OnComplete: func(v converter.EncodedValue, err error) { require.NoError(t, err) },
    }, 500)
}, time.Minute)
```

Testing a timeout path is just *not* sending the Signal — the Workflow's timer
wins the race and the test still finishes instantly.

Other useful controls:

```go
env.SetStartTime(time.Date(2026, 1, 1, 0, 0, 0, 0, time.UTC)) // pin workflow.Now
env.SetTestTimeout(10 * time.Second)                          // wall-clock guard
env.CancelWorkflow()                                          // exercise cancellation
```

## Activity unit tests

`TestActivityEnvironment` runs one Activity directly, with a working
`activity.GetLogger` / `RecordHeartbeat` context:

```go
func TestExtractText(t *testing.T) {
    env := (&testsuite.WorkflowTestSuite{}).NewTestActivityEnvironment()
    env.RegisterActivity(ExtractText)

    val, err := env.ExecuteActivity(ExtractText, IngestInput{DocumentID: "doc-4"})
    require.NoError(t, err)

    var res ExtractResult
    require.NoError(t, val.Get(&res))
    require.Equal(t, 40, res.WordCount)
}
```

To exercise resume-from-heartbeat logic, seed the details:

```go
env.SetHeartbeatDetails(11) // Activity sees HasHeartbeatDetails == true
```

Note there is **no API to set the attempt number** — `activity.GetInfo(ctx).Attempt`
is always 1 here. If your Activity branches on `Attempt`, put that behind a
variable or parameter so it stays testable (the starter template does this).

## Replay tests

The most important test you can add when changing an existing Workflow. It runs
today's code against a history recorded from a real Execution and fails if the
command sequence diverges.

Export a history:

```bash
temporal workflow show --workflow-id ingest-doc-42 --output json > testdata_history.json
```

```go
func TestReplayWorkflowHistory(t *testing.T) {
    replayer := worker.NewWorkflowReplayer()
    replayer.RegisterWorkflow(IngestDocumentWorkflow)

    err := replayer.ReplayWorkflowHistoryFromJSONFile(nil, "testdata_history.json")
    require.NoError(t, err)
}
```

Activities are never executed during replay — their results come from the
history — so no mocks are required.

**Practices that make this pay off:**

- Commit histories from representative Executions (happy path, timeout path,
  cancellation) as fixtures and replay them all in CI.
- When a Workflow fails in production with a non-determinism error, download
  *that* Execution's history and replay it locally. You get the exact divergence
  instead of guessing.
- After intentionally changing Workflow code, a failing replay test means you
  need `workflow.GetVersion` (see `pitfalls.md`) — not a refreshed fixture.
  Only regenerate the fixture once the change is properly patched.

`ReplayWorkflowHistory` (taking a history object) and `ReplayWorkflowExecution`
(fetching directly from a live server by Workflow ID) are also available.

## Integration tests against a real server

`testsuite.StartDevServer` boots a real dev server in-process — slower, but
exercises the actual client, Worker, and server together:

```go
srv, err := testsuite.StartDevServer(context.Background(), testsuite.DevServerOptions{
    ClientOptions: &client.Options{HostPort: ""}, // "" picks a free port
})
require.NoError(t, err)
defer func() { _ = srv.Stop() }()

c := srv.Client()
w := worker.New(c, "test-queue", worker.Options{})
w.RegisterWorkflow(IngestDocumentWorkflow)
w.RegisterActivity(ExtractText)
require.NoError(t, w.Start())
defer w.Stop()

we, err := c.ExecuteWorkflow(context.Background(),
    client.StartWorkflowOptions{ID: "it-1", TaskQueue: "test-queue"},
    IngestDocumentWorkflow, IngestInput{DocumentID: "doc-1"})
require.NoError(t, err)
```

Reserve this for wiring checks — registration names, serialization, Task Queue
mismatches. There is no time skipping here, so keep timers short. Use the
in-process `TestWorkflowEnvironment` for logic.
