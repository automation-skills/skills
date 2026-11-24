# Go SDK idioms

Import paths (module `go.temporal.io/sdk`, verified against v1.47.0):

```go
import (
    "go.temporal.io/sdk/activity"  // Activity context helpers
    "go.temporal.io/sdk/client"    // starting/signalling from outside
    "go.temporal.io/sdk/temporal"  // RetryPolicy, error constructors
    "go.temporal.io/sdk/testsuite" // test environments
    "go.temporal.io/sdk/worker"    // Worker, replayer
    "go.temporal.io/sdk/workflow"  // Workflow-side API
)
```

## Shapes

A **Workflow** takes `workflow.Context` first and returns `error`, optionally
with one result value before it:

```go
func MyWorkflow(ctx workflow.Context, input MyInput) (*MyResult, error)
```

An **Activity** takes a standard `context.Context` first, same return shape:

```go
func MyActivity(ctx context.Context, input MyInput) (*MyResult, error)
```

Inputs and outputs are serialized (JSON by default). Use exported struct fields,
and prefer a **single struct parameter** over positional arguments — adding a
field to a struct is backward-compatible, adding a parameter is not.

## Worker

```go
c, err := client.Dial(client.Options{}) // defaults to localhost:7233
defer c.Close()

w := worker.New(c, "my-task-queue", worker.Options{})
w.RegisterWorkflow(MyWorkflow)
w.RegisterActivity(MyActivity)

err = w.Run(worker.InterruptCh()) // blocks until interrupted
```

Registering by function value means the Workflow Type name is the function name
(`MyWorkflow`). To decouple the wire name from the Go identifier — so renaming
the function doesn't orphan running Executions — register explicitly:

```go
w.RegisterWorkflowWithOptions(MyWorkflow, workflow.RegisterOptions{Name: "MyWorkflow"})
w.RegisterActivityWithOptions(MyActivity, activity.RegisterOptions{Name: "MyActivity"})
```

Activities can also be registered as a struct, which is the cleanest way to
inject dependencies (DB handles, HTTP clients, config):

```go
type Activities struct{ DB *sql.DB }
func (a *Activities) Charge(ctx context.Context, in ChargeInput) error { ... }

w.RegisterActivity(&Activities{DB: db}) // registers each method by name
```

Workflow code has no access to that instance, so reference the method through a
**nil-receiver method value** — the SDK only reads it to derive the Activity
name, and dispatches to the registered instance at runtime:

```go
func MyWorkflow(ctx workflow.Context, in ChargeInput) error {
    var a *Activities // nil on purpose; never dereferenced
    return workflow.ExecuteActivity(ctx, a.Charge, in).Get(ctx, nil)
}
```

This keeps compile-time checking of the name and signature. A string name
(`workflow.ExecuteActivity(ctx, "Charge", in)`) also works but loses both.

## Calling Activities

`ActivityOptions` are mandatory — at minimum `StartToCloseTimeout` or
`ScheduleToCloseTimeout`. The SDK errors out if neither is set.

```go
ao := workflow.ActivityOptions{
    StartToCloseTimeout: 30 * time.Second,
    RetryPolicy: &temporal.RetryPolicy{
        InitialInterval:        time.Second,
        BackoffCoefficient:     2.0,
        MaximumInterval:        time.Minute,
        MaximumAttempts:        5,
        NonRetryableErrorTypes: []string{"InvalidInput"},
    },
}
ctx = workflow.WithActivityOptions(ctx, ao)

var result MyResult
err := workflow.ExecuteActivity(ctx, MyActivity, input).Get(ctx, &result)
```

`ExecuteActivity` returns a `Future` immediately; `.Get(ctx, &out)` blocks. To
fan out, start everything first, then `Get` each future (see `determinism.md`).

Pass the **function reference** (`MyActivity`) rather than a string name where
possible — the compiler then catches typos and signature drift.

### Local Activities

For very short, low-latency operations on the same process (a quick in-memory
lookup, a fast local computation) `workflow.ExecuteLocalActivity` skips the task
queue round trip. It has weaker guarantees — no independent Worker routing, and
limited retry across Worker restarts — so use it only for sub-second work.

```go
lao := workflow.LocalActivityOptions{StartToCloseTimeout: time.Second}
err := workflow.ExecuteLocalActivity(workflow.WithLocalActivityOptions(ctx, lao), fn, arg).Get(ctx, &out)
```

## Errors

```go
// Permanent failure: stop retrying immediately.
return temporal.NewNonRetryableApplicationError("card declined", "CardDeclined", nil)

// Retryable failure with a type name (matchable in NonRetryableErrorTypes).
return temporal.NewApplicationError("gateway timeout", "GatewayTimeout")
```

Inspecting an Activity failure in Workflow code:

```go
var appErr *temporal.ApplicationError
if errors.As(err, &appErr) && appErr.Type() == "CardDeclined" {
    // compensate rather than retry
    reason := appErr.Message() // just the message; Error() adds type/retryable
}

var timeoutErr *temporal.TimeoutError
if errors.As(err, &timeoutErr) { ... }
```

Activity errors arrive wrapped in `*temporal.ActivityError`; always use
`errors.As` rather than type assertions.

### What you get when retries run out

When an Activity exhausts `MaximumAttempts` (or its `ScheduleToCloseTimeout`
budget), the Workflow sees **the last attempt's error**, wrapped:

```
*WorkflowExecutionError → *ActivityError → *ApplicationError
```

`errors.As` unwraps all of it, so the same check matches whether the failure was
permanent on the first try or retryable and exhausted after five. **That means
`errors.As` alone cannot tell you which happened.** If the distinction matters —
say, "declined" should refund but "gateway kept timing out" should alert — check
explicitly:

```go
var appErr *temporal.ApplicationError
if errors.As(err, &appErr) {
    switch {
    case appErr.NonRetryable():
        // Permanent: failed fast, no point trying again.
    default:
        // Retryable, but the budget ran out.
    }
}
```

If the Activity was still running when a timeout struck, you get a
`*temporal.TimeoutError` instead — with `TimeoutType()` distinguishing
StartToClose from Heartbeat. Check `TimeoutError` *before* falling through to a
generic error branch.

## Compensation (the Saga pattern)

Temporal gives you retries for free, but it does not undo completed steps. In a
multi-step business transaction — reserve inventory, charge card, ship — a
permanent failure at step 3 leaves steps 1 and 2 committed. You have to unwind
them yourself.

The idiomatic Go approach is a compensation stack, run in reverse on failure:

```go
func ProcessOrder(ctx workflow.Context, order Order) (err error) {
    var compensations []func(workflow.Context)

    // Runs on any error return, in reverse order.
    defer func() {
        if err == nil {
            return
        }
        // Use a disconnected context so compensation still runs even if the
        // Workflow is being cancelled.
        cctx, cancel := workflow.NewDisconnectedContext(ctx)
        defer cancel()
        for i := len(compensations) - 1; i >= 0; i-- {
            compensations[i](cctx)
        }
    }()

    var reservation Reservation
    if err = workflow.ExecuteActivity(ctx, ReserveInventory, order).Get(ctx, &reservation); err != nil {
        return err
    }
    compensations = append(compensations, func(ctx workflow.Context) {
        // Log, don't return: a failing compensation must not mask the original
        // error, but it does need to be visible.
        if e := workflow.ExecuteActivity(ctx, ReleaseInventory, reservation).Get(ctx, nil); e != nil {
            workflow.GetLogger(ctx).Error("compensation failed", "step", "ReleaseInventory", "error", e)
        }
    })

    var charge Charge
    if err = workflow.ExecuteActivity(ctx, ChargePayment, order).Get(ctx, &charge); err != nil {
        return err
    }
    compensations = append(compensations, func(ctx workflow.Context) {
        if e := workflow.ExecuteActivity(ctx, RefundPayment, charge).Get(ctx, nil); e != nil {
            workflow.GetLogger(ctx).Error("compensation failed", "step", "RefundPayment", "error", e)
        }
    })

    return workflow.ExecuteActivity(ctx, ShipOrder, order).Get(ctx, nil)
}
```

### Business rejection vs. operational failure

A declined card is a normal business outcome, not a system fault. Failing the
Workflow for it means retries, alerts, and an "errored" Execution for something
that worked exactly as designed.

Compensate either way, but decide deliberately how the Workflow *ends*: return a
result describing the rejection, or return an error. A useful split:

- **Expected business outcome** (declined, out of stock, rejected by a rule) →
  compensate, then **complete successfully** with a result carrying the outcome.
- **Operational failure** (gateway unreachable after the retry budget, database
  down) → compensate, then **return the error** so it surfaces and pages someone.

```go
var appErr *temporal.ApplicationError
if errors.As(chargeErr, &appErr) && appErr.NonRetryable() {
    // Declined: compensate, then report a clean, well-formed outcome.
    return &OrderResult{Status: "payment_declined", Reason: appErr.Message()}, nil
}
return nil, chargeErr // infrastructure gave up; let it fail loudly
```

Model the result type so it can express both — a `Status` field with explicit
terminal values beats overloading `error`.

Points that matter:

- **Compensating Activities need their own retry policies**, and usually more
  generous ones than the forward step: a failed refund leaves money in the wrong
  place, while a failed charge leaves nothing behind. A reasonable starting
  point is `MaximumAttempts: 10` with a `ScheduleToCloseTimeout` of an hour or
  more, versus ~5 attempts over minutes for the forward path. Tune to how bad
  the un-compensated state is.
- **Make them idempotent too**, keyed off the original operation's ID.
- **Use `workflow.NewDisconnectedContext`** for compensation. If the Workflow is
  failing because it was cancelled, `ctx` is already cancelled and every
  compensating Activity would fail immediately.
- **Not everything can be undone.** Some steps need a forward fix (a
  notification, a manual-review task) rather than a reversal. Decide per step.
- Appending the compensation *after* the step succeeds is deliberate — never
  compensate work that never happened.

## Heartbeating

Long-running Activities should heartbeat so failures are detected quickly and
retries can resume from progress. Requires `HeartbeatTimeout` in the options.

```go
for i, item := range items {
    activity.RecordHeartbeat(ctx, i) // details are checkpointed
    select {
    case <-ctx.Done():
        return ctx.Err() // honour cancellation
    default:
    }
    process(item)
}
```

Resuming on a retry:

```go
start := 0
if activity.HasHeartbeatDetails(ctx) {
    var completed int
    if err := activity.GetHeartbeatDetails(ctx, &completed); err == nil {
        start = completed + 1
    }
}
```

## Signals, Queries, Updates

**Signal** — asynchronous, one-way, mutates state, no return value.

```go
// Workflow side
ch := workflow.GetSignalChannel(ctx, "approve")
var in ApproveInput
ch.Receive(ctx, &in) // blocks

// Client side
err := c.SignalWorkflow(ctx, workflowID, runID, "approve", ApproveInput{...})
```

Prefer `SignalWithStartWorkflow` when the Workflow may not exist yet — it starts
it and delivers the Signal atomically.

**Query** — synchronous read-only. Must not mutate state, execute Activities, or
block. Not recorded in history.

```go
err := workflow.SetQueryHandler(ctx, "status", func() (string, error) {
    return status, nil
})

resp, err := c.QueryWorkflow(ctx, workflowID, runID, "status")
var status string
err = resp.Get(&status)
```

**Update** — synchronous, *can* mutate state and return a value, with an optional
validator that rejects bad requests before anything is recorded.

```go
err := workflow.SetUpdateHandlerWithOptions(ctx, "setLimit",
    func(ctx workflow.Context, newLimit int) (int, error) {
        old := limit
        limit = newLimit
        return old, nil
    },
    workflow.UpdateHandlerOptions{
        Validator: func(ctx workflow.Context, newLimit int) error {
            if newLimit < 0 {
                return fmt.Errorf("limit must be non-negative")
            }
            return nil
        },
    })
```

The validator must be side-effect free — it runs on every replay path.

```go
handle, err := c.UpdateWorkflow(ctx, client.UpdateWorkflowOptions{
    WorkflowID:   workflowID,
    UpdateName:   "setLimit",
    WaitForStage: client.WorkflowUpdateStageCompleted,
    Args:         []any{500},
})
var previous int
err = handle.Get(ctx, &previous)
```

**Drain handlers before returning.** If a Workflow completes while a Signal or
Update handler is still in flight, the work is silently lost. Guard the exit:

```go
if err := workflow.Await(ctx, func() bool { return !processing }); err != nil {
    return err
}
```

## Waiting on conditions

`workflow.Await` blocks until a predicate over Workflow state becomes true. The
predicate is re-evaluated after each event, so it must be pure and cheap.

```go
err := workflow.Await(ctx, func() bool { return approvals >= 2 })

ok, err := workflow.AwaitWithTimeout(ctx, time.Hour, func() bool { return done })
```

## Child Workflows

`ParentClosePolicy` lives in the API module, not the SDK — a common stumble:

```go
import enumspb "go.temporal.io/api/enums/v1"

cwo := workflow.ChildWorkflowOptions{
    WorkflowID: "shipment-" + orderID,
    TaskQueue:  "shipping",
    // What happens to the child if the parent completes first.
    ParentClosePolicy: enumspb.PARENT_CLOSE_POLICY_TERMINATE,
}
ctx = workflow.WithChildOptions(ctx, cwo)

var out ShipResult
err := workflow.ExecuteChildWorkflow(ctx, ShipWorkflow, in).Get(ctx, &out)
```

To wait only until the child has *started*, use `.GetChildWorkflowExecution()`.

Reach for a child Workflow when you need a separate Execution — its own history
budget, its own retry/versioning lifecycle, or routing to a different Task Queue.
Otherwise prefer an Activity; child Workflows add moving parts.

## ContinueAsNew

Event History is capped (51,200 events / 50 MB). Long-lived or looping Workflows
must periodically restart with a fresh history, carrying state forward as input:

```go
if workflow.GetInfo(ctx).GetContinueAsNewSuggested() {
    return workflow.NewContinueAsNewError(ctx, MyWorkflow, carryForwardState)
}
```

`GetContinueAsNewSuggested()` is the SDK's own signal that history is getting
large — prefer it to a hand-rolled iteration counter.

## Client

```go
we, err := c.ExecuteWorkflow(ctx, client.StartWorkflowOptions{
    ID:        "order-" + orderID, // business ID; dedupes retries
    TaskQueue: "orders",
    // Optional. Workflows do NOT retry by default; usually leave it that way.
    WorkflowExecutionTimeout: 24 * time.Hour,
}, MyWorkflow, input)

var result MyResult
err = we.Get(ctx, &result) // blocks until completion
```

`ExecuteWorkflow` returns as soon as the Workflow is *started*. Use a meaningful
business ID: Temporal rejects a duplicate ID while an Execution with that ID is
running, which makes double-submission a no-op instead of a duplicate order.

## Version pinning

- Go SDK: `go.temporal.io/sdk v1.47.0`
- CLI (dev server): `temporal` v1.8.1, bundling Server 1.31.2

Updates and `GetContinueAsNewSuggested` need reasonably recent SDK and Server
versions; on an older self-hosted Server, check support before relying on them.
