---
title: Regression runs and drift monitoring
description: What triggers a regression run, the release gate, the report format, drift signals with thresholds, the weekly review, and two worked cases.
---

## What triggers a full regression run

- Any prompt edit, including "just a wording fix".
- A new model, a new model version, or a change to settings such as temperature or output length.
- A change to the output schema or to the rule checks that follow the model.
- A change to the tools an agent can call, or to their descriptions.
- A change to the retrieval data or knowledge the step reads.
- A new input format upstream: a new form, a new document template, a new source system.
- A provider alias that may point at a new model. If you cannot pin the version, run weekly.

## Release gate

| Check | Blocks the release when |
|---|---|
| Critical errors | Any new one |
| Automatic-slice precision | Falls more than 1 point, or below the target |
| Any category or branch | Falls more than 5 points |
| Flipped cases, pass to fail | Any not read and explained |
| Output-shape failures | Above 0.5% |
| "Unclear" rate | Leaves its agreed band |
| Holdout versus tuning set | Gap of 6 points or more |
| Cost per run, 95th percentile latency | Above budget |

## The report

1. Set version, prompt version, model version, settings, date of the baseline it is compared with.
2. The release gate table with each result.
3. Every case that flipped from pass to fail: the input reference, the expected value, the new output, the cause.
4. Cases that flipped from fail to pass, in one line each.
5. Flaky cases (passed some of the 3 runs, not all).
6. The verdict: release, release with a named risk accepted by the owner, or blocked.

## Drift signals in production

Set each baseline from the first 4 weeks of live traffic.

| Signal | Alert when |
|---|---|
| Human override rate in the review queue | Doubles, or rises 3 points week over week |
| "Unclear" and fallback rate | Rises 5 points over baseline |
| Output-shape failures | Above 1% in a day |
| Output mix | Any category moves more than 10 points week over week |
| Input mix (length, language, new senders or templates) | A new template or source passes 5% of volume |
| Cost per run, 95th percentile latency | Up 50% |
| Weekly sample of 30 automatic cases | Any critical error, or more than 1 other error |

The override rate is the most direct signal: it is people telling you the step is wrong. Make overriding easy and record a reason with each one.

## The weekly review

- Draw 30 random cases from the automatic slice and have the decision owner check them. About 20 minutes.
- Read every override from the week, grouped by reason.
- Add new error patterns to the golden set.
- If any alert fired, run the regression on the current set before changing anything, so you know whether the step changed or the inputs did.

## Worked case: a model upgrade that looked better

- **Set:** 280 cases, version 7.
- **Totals:** accuracy rose from 94.1% to 95.6%, automatic-slice precision from 99.2% to 99.5%. 11 cases flipped from fail to pass.
- **The catch:** 4 cases flipped from pass to fail. Three were amounts written with a decimal comma, such as 1.234,56, read as 1.234. One of them sat in the automatic slice: a critical error. The fourth was a day-month date read as month-day.
- **Verdict:** blocked, on the critical-error rule, despite the better totals.
- **Fix:** an explicit number-format instruction, plus a rule check that line items sum to the total, which would have routed all three to the review queue anyway.
- **Rerun:** 3 runs, zero critical errors, every flip explained. 15 decimal-comma cases added as version 8.

## Worked case: drift from one sender

- **Signal:** the "unclear" rate went from 6% to 14% in a week. Overrides clustered on one supplier.
- **Cause:** that supplier moved to a new document template. Nothing in the step had changed.
- **Fix:** 20 cases of the new template added to the set, the prompt adjusted with an example of the new layout, full regression run, released. "Unclear" back to 7% the following week.
- **Lesson:** without the input-mix signal, this reads as "the model got worse" and someone switches models, which would have fixed nothing.
