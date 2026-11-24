---
title: AI step guardrails
description: Output schemas, rule checks after the model, how to set confidence cut-offs from labelled cases, the fallback ladder, untrusted input, and agent budgets.
---

## Output schema

- Enums over free text wherever the set of answers is known. Every enum includes `unclear`.
- Required fields are required in the schema. A missing value is `null`, never an empty string or a placeholder like "N/A" that a later step will treat as data.
- For extraction, ask for the source quote next to each critical value. A rule then checks that the quote appears verbatim in the input. This catches invented values cheaply.
- Keep one call to about 15 output fields. Past that, accuracy and schema failures both get worse in practice. Split into two calls.
- Validate in this order: parse, schema, then business checks. Each failure has its own route.

## Rule checks after the model

Pick the ones that apply. Each is a few lines of logic and catches a class of model error.

- Totals: lines sum to the total, tax matches rate times base within rounding.
- Existence: every id, code or email domain exists in master data.
- Range: dates not in the future and not older than the business allows; amounts within, say, 5 times the counterparty's largest previous amount.
- Consistency: currency allowed for this counterparty; the category agrees with the department.
- Grounding: quoted evidence appears in the input.

## Setting confidence cut-offs

A model's self-reported confidence is a signal, not a probability. Calibrate it before you route on it.

1. Run the step over at least 200 labelled real cases.
2. Record for each case: correct or not, the self-reported confidence, whether all rule checks passed, and whether the evidence quotes were found.
3. Group the cases into buckets and measure precision per bucket.
4. Choose the cut-off where everything above it meets the target precision.

Example from 620 labelled cases:

| Bucket | Cases | Correct | Route |
|---|---|---|---|
| All checks pass and confidence 0.9 or more | 412 | 99.3% | Automatic |
| All checks pass and confidence 0.7 to 0.9 | 160 | 93% | Human queue |
| Any check fails, or confidence under 0.7 | 48 | 71% | Manual path |

Result: 66% of cases go automatic at 99.3% precision. To claim 99% with confidence you need roughly 300 cases above the cut-off: with zero errors in n cases, the true error rate is still only bounded at about 3 in n.

Recheck the buckets on every prompt or model change. A cut-off set for one model version means nothing for the next.

For money fields, a second independent extraction (another prompt, or the same prompt run twice) that must agree with the first is a strong signal. It doubles the cost of that field, so use it where an error is expensive.

## Fallback ladder

| Failure | Do | Never |
|---|---|---|
| Timeout or transient error | Retry once after a short wait, then the human queue | Retry in a tight loop |
| Schema failure | One retry with the validation error included, then the queue | Accept a partial object |
| Business check failure | Queue at once, naming the failed check | Retry: the model will often repeat the same answer |
| Provider outage | Queue everything; alert when the queue passes 50 items or the oldest is 2 hours old | Switch to an untested second model |
| Low confidence | Human queue | Fill a default category |

A second model is a valid fallback only after it has passed the same labelled set with the same cut-offs.

## Untrusted input

Any document, email, form or web page can contain text written to steer the model. Prompt wording alone does not stop it. The structure does:

- The model's output is constrained to the schema, so it cannot emit an action.
- The AI step has no tools. It returns data.
- Recipients, amounts and accounts are checked against records by rules before anything executes.
- Bank details, credentials and permissions are never taken from input content.
- Count inputs that contain instruction-like text. A spike is worth a look.

## Agent budgets and stops

- **Tools:** an explicit allowlist, read-only by default. Each write tool needs its own justification and a human gate.
- **Step budget:** 15 tool calls. A task that has not converged by then is looping more often than progressing.
- **Time budget:** 5 minutes for interactive work, longer only for batch work that nobody waits on.
- **Spend budget:** a fixed cap per run, set at about 3 times the median run cost seen in testing.
- **Loop stop:** the same tool with the same arguments twice ends the run.
- **Final status:** resolved, could not resolve, or needs a person, always with the evidence read and the steps tried. All three are valid outcomes. Only the first may continue without a person.
- **Memory:** nothing carries over between runs unless it was designed and reviewed. Leftover context from one customer's run must never reach another's.
