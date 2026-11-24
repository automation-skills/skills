---
title: Scoring and payback
description: The quick triage score for a backlog, the full payback math, where each number comes from, and three worked cases with verdicts.
---

Use the triage score to cut a long list down. Use the payback math to decide. Never ship a verdict from the triage score alone.

## Quick triage score

Score each factor 0, 1 or 2 from what you can learn in a 15-minute conversation plus a look at the record.

| Factor | 0 | 1 | 2 |
|---|---|---|---|
| Volume | Under 10 runs a month | 10 to 100 | Over 100 |
| Hands-on minutes per run | Under 3 | 3 to 15 | Over 15 |
| Exception rate | Over 30% | 10% to 30% | Under 10% |
| Error exposure | A wrong run is costly and irreversible, or invisible for weeks | Costly but reversible, or caught by an existing check | Cheap and visible within a day |
| Stability | 3 or more changes in 12 months, or a change planned in the next 6 | 1 or 2 changes | No changes |
| Input quality | Free text, scans, scattered across places | Mixed | Structured fields from one system |
| Build effort | New integrations, unclear access, over 3 weeks | 1 to 3 weeks | Days, systems already connected |

Total out of 14:

- **10 or more:** do the full payback now.
- **7 to 9:** measure properly (count, time, exceptions) before deciding.
- **6 or less:** park it with the reason written down.
- **Any 0 on error exposure or stability:** read the do-not-automate list before anything else.

## Getting the numbers

- **Volume.** Count from the system of record over 4 weeks, then check the previous 3 months for peaks. Use the 3-month average for savings and the peak month for capacity. Month-end and quarter-end often carry 2 to 3 times the normal load.
- **Minutes.** Time at least 3 real runs. Self-reported minutes are often off by a third in either direction: people remember the clean runs and forget the lookups.
- **Exceptions.** Ask "show me the last one that went wrong" and "which customers or suppliers are always different". Then count them in the sample of 30.
- **Loaded hourly rate.** Annual salary x 1.3 / 1,700. Benefits, taxes and overhead add roughly 25% to 40% on top of salary, and a year holds about 1,700 productive hours after holidays, leave and meetings. A 52,000 salary gives a rate of about 40 an hour.
- **Build cost.** The builder's estimate, times 1.5 for every system not integrated before (access, authentication and data quirks eat the time), plus testing equal to a third of the build.
- **Monthly cost.** Run fees, plus maintenance at about 1 hour per connected system per month, plus 2 to 4 hours per AI step (reviewing samples, adjusting to input drift), plus the time reviewers spend at approval gates and spot checks.

## Payback math

```
gross monthly saving = runs per month x minutes per run x (1 - exception rate) / 60 x loaded rate
monthly net saving   = gross monthly saving - run fees - maintenance - review time
payback in months    = build cost / monthly net saving
```

If the net saving is zero or negative, the time case is dead. Only a priced risk can carry it, and the verdict must say so.

## Worked case 1: supplier invoice coding (automate now)

- **Volume:** 640 invoices a month, counted from 4 weeks of the payables record.
- **Minutes:** 4.5, from three timed runs of 3.8, 4.6 and 5.1 minutes.
- **Exceptions:** 7 of 30 sampled (23%): missing purchase order, amount mismatch.
- **Error cost:** a wrong expense code is caught at month-end close and takes about 20 minutes to fix. Reversible and visible.
- **Stability:** no changes in 12 months, none planned.
- **Rates:** clerk 40 an hour, builder 85 an hour.
- **Build:** 36 hours, both systems already connected: 3,060.
- **Monthly cost:** run fees 40, maintenance (2 systems at 1 hour, 1 AI step at 3 hours) 425, a 5% spot check of 32 invoices at 1 minute each 21. Total 486.
- **Gross saving:** 640 x 4.5 x 0.77 / 60 = 37 hours x 40 = 1,478.
- **Net saving:** 1,478 - 486 = 992 a month. **Payback:** 3,060 / 992 = 3.1 months.

The naive estimate (640 x 4.5 / 60 x 40 = 1,920 a month) was almost twice the real saving. The verdict holds either way, but the 30-day check is measured against 992, not 1,920.

## Worked case 2: leaver access removal (automate a slice, on risk)

- 5 leavers a month, 50 hands-on minutes each across 6 systems, 10% exceptions. IT rate 55.
- Gross saving: 5 x 50 x 0.9 / 60 = 3.75 hours x 55 = 206.
- Full build: 35 hours x 1.5 (4 systems not yet connected) = 52.5 hours x 85 = 4,463.
- Monthly cost: maintenance 6 hours x 85 = 510, run fees 20. Net saving: 206 - 530 = **minus 324**. It never pays back on time.
- The real driver is risk: the last access review found 3 of 41 leavers with an active account weeks after their last day.
- **Verdict:** automate a slice. A daily reconciliation compares the leaver list with the identity directory and the two systems holding customer data, and opens a task per open account. Removal stays a human action. Build 12 hours (1,020), maintenance 3 hours a month (255). The security owner signs that one orphaned account with customer-data access is worth far more than about 4,000 a year. The verdict states that it is a risk decision, not a time saving.

## Worked case 3: sales proposals (automate a slice, on time)

- 45 proposals a month, 70 minutes each, 60% exceptions (custom terms, legal review). Full automation fails the exception threshold.
- The slice: assemble the account data, the pricing table and the approved standard sections into a draft. That saves about 25 of the 70 minutes on every proposal, exceptions included, because the preparation is the same for all of them.
- Gross saving: 45 x 25 / 60 = 18.75 hours x 70 = 1,313.
- Build 30 hours x 85 = 2,550. Monthly cost: maintenance 2 systems plus 1 AI step = 5 hours x 85 = 425, run fees 30. Net: 858.
- **Payback:** 2,550 / 858 = 3.0 months. **Verdict:** automate the slice now. The proposal and its terms stay with the account owner.

## Speed and quality value

Count speed only when you can name the metric that moves and the person who will measure it, for example "first response to an inbound lead drops from 3 hours to 5 minutes, measured weekly by the sales lead". If someone attributes revenue to it, use their number at half value until 60 days of real data confirm it. Count error reduction only when you have the current error rate from the record, not from memory.

## The 30-day check

Recompute with real numbers 30 days after go-live. If volume, exceptions or minutes are more than 25% off the estimate, rewrite the scorecard. If the payback now exceeds 12 months, raise it with the owner: shrink the scope, simplify, or turn it off. An automation that does not pay for itself still costs its maintenance every month.
