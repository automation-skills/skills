# Recurring failure modes

## 1. Retry policy vs. timeout confusion

These are different knobs and they are constantly mixed up. There are four
Activity timeouts:

| Timeout | Bounds | Notes |
| --- | --- | --- |
| `StartToCloseTimeout` | **one attempt** | The one you almost always want. Required (or `ScheduleToClose`). |
| `ScheduleToCloseTimeout` | **all attempts including retries and backoff** | The wall-clock budget for the whole operation. |
| `ScheduleToStartTimeout` | time waiting in the queue before a Worker picks it up | Rarely useful; usually indicates a capacity problem you should fix instead. |
| `HeartbeatTimeout` | max gap between heartbeats | Only meaningful if the Activity calls `RecordHeartbeat`. |

The default Retry Policy is:

- `InitialInterval`: 1s
- `BackoffCoefficient`: 2.0
- `MaximumInterval`: 100 × `InitialInterval` (100s)
- `MaximumAttempts`: **unlimited**
- `NonRetryableErrorTypes`: none

**Activities retry by default. Workflows do not.**

### The classic mistakes

**Setting only `StartToCloseTimeout` and expecting the operation to give up.**
It bounds a single attempt. With the default unlimited `MaximumAttempts`, a
permanently-down dependency produces an Activity that retries forever — the
Workflow just hangs, looking "running" but making no progress.

```go
// WRONG — retries forever.
ao := workflow.ActivityOptions{StartToCloseTimeout: 30 * time.Second}
```

```go
// RIGHT — bound the attempt AND the overall effort.
ao := workflow.ActivityOptions{
    StartToCloseTimeout:    30 * time.Second, // one attempt
    ScheduleToCloseTimeout: 10 * time.Minute, // total budget
    RetryPolicy: &temporal.RetryPolicy{
        MaximumAttempts: 5,
    },
}
```

**Setting `MaximumAttempts: 1` to mean "no retry" — correct, but easy to confuse
with `0`, which means *unlimited*, not "none".**

**Assuming `MaximumAttempts` and `ScheduleToCloseTimeout` are alternatives.**
They are independent limits and **whichever is reached first stops the retries**.
With `ScheduleToCloseTimeout: 3s`, `InitialInterval: 1s`, and
`MaximumAttempts: 100`, you get about 4 attempts, not 100 — the budget expires
long before the attempt count does. Verified against SDK v1.47.0.

Two consequences:

- If you want a guaranteed *number* of attempts, make sure the budget is large
  enough to contain them, including exponential backoff.
- When the budget expires between attempts, the Workflow sees the **last
  attempt's error**, not a `TimeoutError`. A `TimeoutError` surfaces only when a
  timeout interrupts an attempt that was actually running. Don't rely on seeing
  `TimeoutError` to detect an exhausted retry budget.

**Setting `StartToCloseTimeout` shorter than the work actually takes.** Each
attempt is killed mid-flight and retried, so the Activity never completes and
side effects pile up. If duration is unpredictable, heartbeat and set a generous
`StartToCloseTimeout` with a short `HeartbeatTimeout` — that detects a dead
Worker fast without capping legitimate long work.

**Retrying non-retryable failures.** A validation error or a declined card will
fail identically on every attempt. Mark them:

```go
return temporal.NewNonRetryableApplicationError("card declined", "CardDeclined", nil)
```

...or list the type in `RetryPolicy.NonRetryableErrorTypes`.

**Putting a retry policy on the Workflow to compensate for a flaky Activity.**
Workflow-level retry restarts the *entire* Workflow from scratch, re-running
completed steps. Retry at the Activity level.

## 2. Versioning a running Workflow after a deploy

This is the failure that bites in production. Workflows started under the old
code are still running; when a Worker on the new code replays their history, any
change to the sequence of Workflow API calls diverges from what was recorded and
the Workflow Task fails.

**Safe to change freely:** timer durations, Activity options (timeouts, retry
policies), Activity input values, logging, and anything inside Activity code.

**Requires versioning:** adding, removing, or reordering Activity calls, timers,
child Workflows, `SideEffect`s, or Signal sends.

### Patching with `workflow.GetVersion`

Say v1 called `ActivityA` and you want new Executions to call `ActivityB`:

```go
// v1 — original
err := workflow.ExecuteActivity(ctx, ActivityA, data).Get(ctx, nil)
```

```go
// v2 — patched. Old Executions replay the DefaultVersion branch; new ones take v1.
v := workflow.GetVersion(ctx, "UseActivityB", workflow.DefaultVersion, 1)
if v == workflow.DefaultVersion {
    err = workflow.ExecuteActivity(ctx, ActivityA, data).Get(ctx, nil)
} else {
    err = workflow.ExecuteActivity(ctx, ActivityB, data).Get(ctx, nil)
}
```

`GetVersion` records the version in history the first time it runs, and returns
that recorded value on every replay — which is what keeps both branches
deterministic.

Once every pre-patch Execution has finished (past your Namespace retention),
raise `minSupported` to drop the dead branch:

```go
// v3 — old branch retired.
v := workflow.GetVersion(ctx, "UseActivityB", 1, 1)
err = workflow.ExecuteActivity(ctx, ActivityB, data).Get(ctx, nil)
```

Keep the `GetVersion` call itself until no history contains that marker; removing
it early is itself a breaking change.

**Rules of thumb:**

- The `changeID` must be unique and stable — never reuse one for a different change.
- Don't put `GetVersion` inside a loop or conditional whose execution differs
  between branches.
- The safest deploy for a large restructure is often not patching at all: give
  the new code a **new Workflow Type name**, route new traffic to it, and let old
  Executions drain on the old code.

**Alternative: Worker Versioning.** Instead of branching in code, pin Workers to
a Build ID so existing Executions keep running on the code they started with.
Better for frequent large changes; more deployment machinery. Temporal now
recommends this over patching for many cases — worth checking current docs when
choosing.

Always add a **replay test** (see `testing.md`) when changing Workflow code.

## 3. Oversized payloads

Limits:

| Limit | Value |
| --- | --- |
| Single payload (blob) | 2 MB |
| gRPC message / history transaction | 4 MB |
| Event History | 51,200 events or 50 MB (warnings at 10,240 / 10 MB) |

Every Activity input and output is stored in Event History **permanently**.
Passing a 50 MB file through an Activity return value will blow the limit and
fail the Workflow, possibly hours in.

**Use the claim-check pattern:** put the blob in object storage and pass the key.

```go
// WRONG — the whole document travels through history.
var content []byte
workflow.ExecuteActivity(ctx, DownloadFile, url).Get(ctx, &content)
workflow.ExecuteActivity(ctx, ProcessFile, content).Get(ctx, nil)
```

```go
// RIGHT — only the reference is recorded.
var s3Key string
workflow.ExecuteActivity(ctx, DownloadToS3, url).Get(ctx, &s3Key)
workflow.ExecuteActivity(ctx, ProcessFromS3, s3Key).Get(ctx, nil)
```

Watch history growth in general: a loop that runs thousands of Activities will
approach the event cap. Use `ContinueAsNew` (see `go-sdk.md`), driven by
`GetContinueAsNewSuggested()`.

## 4. Blocking calls inside Workflow code

The Workflow scheduler is single-threaded and cooperative. Blocking a Workflow
goroutine on anything other than a Workflow API stalls the whole Workflow Task.
The **deadlock detector fires after 1 second** of no progress
(`worker.Options.DeadlockDetectionTimeout`) and fails the task.

```go
// WRONG — all of these block the scheduler.
time.Sleep(time.Second)
resp, _ := http.Get(url)
<-someGoChannel
mu.Lock()
wg.Wait()
```

Use `workflow.Sleep`, Activities, `workflow.Channel`, `workflow.Mutex`, and
`workflow.Await` respectively. See `determinism.md`.

Heavy CPU work also counts: a tight loop over a million items inside a Workflow
will trip the detector. Move it to an Activity.

## 5. Other frequent traps

**Non-idempotent Activities.** Delivery is at-least-once. An Activity that
succeeds but whose completion is lost to a network partition *will* run again.
Pass an idempotency key derived from stable Workflow state:

```go
info := workflow.GetInfo(ctx)
key := fmt.Sprintf("%s-charge", info.WorkflowExecution.ID)
```

Don't generate the key inside the Activity — it would differ per attempt.

**Losing in-flight Signals or Updates at completion.** If the Workflow returns
while a handler is still running, that work vanishes. Drain before returning with
`workflow.Await`.

**Mutating state in a Query handler.** Queries aren't recorded in history, so
mutations there don't survive replay and silently corrupt state. Queries must be
read-only.

**Reusing a Workflow ID carelessly.** Temporal rejects a start while another
Execution with the same ID is running — usually what you want for dedupe. If you
genuinely need reruns, understand `WorkflowIDReusePolicy` before changing it.

**Worker not registered for the Task Queue.** A Workflow that stays in "Running"
with no events, or an Activity stuck in "Scheduled", almost always means no
Worker is polling that queue, or the Workflow/Activity name isn't registered.
Check the Task Queue string matches on both sides.

**Renaming a Workflow or Activity function.** The Type name defaults to the Go
function name, so a rename orphans running Executions. Register explicitly with
`RegisterOptions{Name: ...}` to decouple the two.
