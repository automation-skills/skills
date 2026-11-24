# Determinism in Temporal Go Workflows

## Why replay demands determinism

Temporal does not snapshot your Workflow's memory. It stores an **Event
History** — the ordered list of commands your Workflow issued (timers started,
Activities scheduled, Signals received) and their results.

When a Worker picks up a Workflow whose state it doesn't have in memory — after
a crash, a deploy, a rebalance, or just a long sleep — it **replays** the
history: it runs your Workflow function again from line one, intercepting each
Workflow API call and feeding back the recorded result instead of doing the work
again. Once the recorded history is exhausted, execution continues live.

For this to reconstruct the same state, the replayed run must issue **the same
commands in the same order**. If it doesn't, Temporal detects the divergence and
fails the Workflow Task with a non-determinism error.

Two consequences worth internalizing:

- Your Workflow function may execute **many times** over its life. Anything with
  a visible side effect will happen on every replay unless Temporal records it.
- Replay uses **today's code** against **yesterday's history**. Deploying a
  change that alters command order breaks Workflows that are already running.
  See `pitfalls.md` for versioning.

**Go has no workflow sandbox.** Unlike the TypeScript SDK, which patches `Date`
and `Math.random` inside an isolated context, the Go SDK cannot stop you from
calling `time.Now()`. It compiles and runs fine, and the damage surfaces only on
the first replay — often days later, in production. The discipline is entirely
on you and on code review.

## What must stay stable

The SDK tracks the sequence of these operations. Reordering, adding, or removing
any of them mid-flight breaks replay:

- Scheduling or cancelling Activities and child Workflows
- Starting or cancelling Timers
- Signalling external Workflows
- `SideEffect` and `MutableSideEffect`
- `GetVersion` calls
- Upserting Search Attributes or Memos
- Completing, failing, cancelling, or continuing-as-new

Things you *can* change freely: timer durations, Activity options (timeouts,
retry policies), Activity input values, and logging.

## Wrong vs. right

### Clock

```go
// WRONG — a different value on every replay.
deadline := time.Now().Add(24 * time.Hour)
if time.Now().After(cutoff) { ... }
```

```go
// RIGHT — workflow.Now returns the recorded Workflow Task time, stable on replay.
deadline := workflow.Now(ctx).Add(24 * time.Hour)
if workflow.Now(ctx).After(cutoff) { ... }
```

### Sleeping and timers

```go
// WRONG — blocks the Worker thread; invisible to Temporal; dies with the process.
time.Sleep(30 * time.Minute)

// WRONG — real timer inside a real select.
select {
case <-time.After(time.Hour):
case msg := <-someGoChannel:
}
```

```go
// RIGHT — durable timer; survives Worker restarts, costs nothing while pending.
if err := workflow.Sleep(ctx, 30*time.Minute); err != nil {
    return err // context cancelled
}

// RIGHT — race a Signal against a durable timer.
sel := workflow.NewSelector(ctx)
sel.AddFuture(workflow.NewTimer(ctx, time.Hour), func(workflow.Future) {
    timedOut = true
})
sel.AddReceive(workflow.GetSignalChannel(ctx, "approve"), func(c workflow.ReceiveChannel, _ bool) {
    c.Receive(ctx, &approval)
})
sel.Select(ctx)
```

A `workflow.Sleep` of a month consumes no resources — the Worker holds nothing
in memory while it waits.

### Randomness and UUIDs

```go
// WRONG — new value each replay; downstream writes land under different keys.
id := uuid.New().String()
if rand.Float64() < 0.1 { ... }
```

```go
// RIGHT — SideEffect records the value in history and replays the recorded one.
var id string
if err := workflow.SideEffect(ctx, func(workflow.Context) any {
    return uuid.New().String()
}).Get(&id); err != nil {
    return err
}
```

`SideEffect` never re-executes on replay. It is *not* an Activity: no retries, no
timeouts, no visibility into failure. Keep it to cheap, in-process, always-succeeds
computation. Anything that can fail belongs in an Activity.

Note that the Workflow ID and Run ID are already stable and available via
`workflow.GetInfo(ctx)` — prefer deriving deterministic keys from them over
generating fresh UUIDs:

```go
info := workflow.GetInfo(ctx)
idempotencyKey := fmt.Sprintf("%s-payment", info.WorkflowExecution.ID)
```

### I/O

```go
// WRONG — network call in Workflow code. Repeats on replay, can't be retried,
// blocks the Workflow Task, and will trip the deadlock detector if slow.
resp, err := http.Get("https://api.example.com/rate")
row := db.QueryRow("SELECT ...")
key := os.Getenv("API_KEY")
```

```go
// RIGHT — an Activity. Retried, timed out, and recorded.
ao := workflow.ActivityOptions{
    StartToCloseTimeout: 30 * time.Second,
    RetryPolicy: &temporal.RetryPolicy{MaximumAttempts: 5},
}
var rate float64
err := workflow.ExecuteActivity(workflow.WithActivityOptions(ctx, ao), FetchRate).Get(ctx, &rate)
```

Configuration (like `API_KEY`) should be read by the **Worker** at startup and
closed over by the Activity, or passed as Workflow input — never read from the
environment inside Workflow code.

### Concurrency

```go
// WRONG — real goroutines and channels are invisible to the SDK; the scheduler
// can't replay their interleaving, and blocking on them deadlocks the task.
go doSomething()
ch := make(chan int)
var mu sync.Mutex
```

```go
// RIGHT — SDK primitives run on a single deterministic scheduler.
workflow.Go(ctx, func(ctx workflow.Context) {
    _ = workflow.ExecuteActivity(ctx, Notify).Get(ctx, nil)
})

ch := workflow.NewChannel(ctx)
mu := workflow.NewMutex(ctx)
```

To run Activities in parallel, start them without blocking and collect the
futures:

```go
futures := make([]workflow.Future, 0, len(items))
for _, item := range items {
    futures = append(futures, workflow.ExecuteActivity(ctx, Process, item))
}
for _, f := range futures {
    if err := f.Get(ctx, nil); err != nil {
        return err
    }
}
```

### Iteration order

```go
// WRONG — Go randomizes map iteration order; a different order each replay
// means Activities are scheduled in a different sequence.
for id, item := range itemsByID {
    workflow.ExecuteActivity(ctx, Process, id, item)
}
```

```go
// RIGHT — impose a stable order.
ids := make([]string, 0, len(itemsByID))
for id := range itemsByID {
    ids = append(ids, id)
}
sort.Strings(ids)
for _, id := range ids {
    workflow.ExecuteActivity(ctx, Process, id, itemsByID[id])
}
```

### Global state

```go
// WRONG — shared across every Workflow Execution on the Worker, and not
// reconstructed on replay.
var counter int
func MyWorkflow(ctx workflow.Context) error {
    counter++
}
```

Keep all state in local variables inside the Workflow function. They *are*
reconstructed by replay; package-level variables are not.

### Logging

`workflow.GetLogger(ctx)` suppresses output during replay, so recovery doesn't
produce duplicate log lines. Plain `log.Println` or `fmt.Println` will print
again on every replay.

```go
logger := workflow.GetLogger(ctx)
logger.Info("order received", "orderID", order.ID)
```

## Recognizing a determinism failure

Symptoms: the Workflow stalls with a `Workflow Task` failing repeatedly, and the
UI or Worker logs show:

```
NonDeterministicError: unable to find command ... history event ...
```

Typical causes, in rough order of frequency:

1. Code was deployed that added/removed/reordered Activity or timer calls in a
   Workflow with running Executions → needs `workflow.GetVersion` (`pitfalls.md`).
2. Non-deterministic call in Workflow code (the table in `SKILL.md`).
3. Map iteration or other unordered traversal driving command order.
4. The Workflow's behaviour depending on Worker-local state — an env var, a
   package variable, a config file read at replay time.

Reproduce it locally with a replay test against the failing Execution's history
rather than guessing — see `testing.md`.
