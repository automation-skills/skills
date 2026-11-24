---
title: Diffing old and new outputs
description: How to capture comparable outputs, normalization rules that do not hide real differences, how to classify each difference, the diff report, and a worked case.
---

## Capture

- Key every output by the input that caused it: the event id, the record id, the file name plus row. Without a shared key you are comparing piles, not pairs.
- From the old version, read what it actually wrote: the run records and the target records. From the new version in shadow, record the payload it would have written or sent, at the same point.
- Capture every side effect, not just the main one: the record write, the email, the follow-up task, the status change, the log line someone else reads.

## Normalization

Normalize only what the business does not care about. Each rule is written down and reviewed, because a normalization rule that hides a real difference is the worst bug a diff can have.

- **Timestamps:** compare at the precision the business uses (the date, or the minute), in one time zone.
- **Generated ids:** never compare them directly. Compare what they point to, by business key.
- **Order:** sort lists by a stable key before comparing, unless order carries meaning, such as line order on an invoice.
- **Text:** trim whitespace, normalize line endings and Unicode, lower-case email addresses.
- **Numbers:** compare at currency precision. Report rounding differences in their own class, because they are often real.
- **Empty, null and missing:** treat them as equal only if every downstream consumer does.
- **Never normalize away:** amounts, recipients, business ids, dates at business precision, or the presence or absence of a side effect.

## Classify every difference

| Class | Meaning | Who decides | Next |
|---|---|---|---|
| Intended | Listed before the run | Already decided | None |
| Bug in new | The new version is wrong | Builder | Fix, then re-shadow that branch |
| Bug in old | The new version is right and the old one was wrong | Process owner | Keep the fix and tell downstream teams, or reproduce the old behavior for now |
| Noise | Timing or formatting nobody consumes | Reviewer | Add a written normalization rule |

An accepted bug in the old version moves to the intended list, with the owner's decision recorded. The match rate is computed after normalization and after removing intended differences, so it measures what is still unexplained.

## The diff report

1. Window, run count, branches covered and branches replayed from history.
2. Match rate per field, with the business-critical fields first.
3. Count of differences per class.
4. For each class, up to 5 examples with input keys, the old value and the new value.
5. Open items: every unexplained difference, with an owner.

## Worked case: order-to-invoice sync rewritten

Two weeks of shadow, spanning a month-end: 3,412 runs.

| Field | Match | Differences | Class and decision |
|---|---|---|---|
| Customer id | 100% | 0 | |
| Recipient | 100% | 0 | |
| Total | 99.97% | 1 | Rounding: the old version rounded each line, the new one rounded the total. Accounting reconciles per line, so the owner chose the old behavior. Bug in new, fixed. |
| Tax | 99.59% | 14 | The old version used a tax rate that had since changed. Bug in old. The owner kept the fix and told accounting. Moved to the intended list. |
| Due date | 98.09% | 65 | The new version computed dates in UTC, so orders placed after 22:00 local time landed a day late. Bug in new, fixed. |
| Skipped orders | - | 23 | The old version silently skipped orders with no billing email. The new one sends them to a review queue. On the intended list from the start. |

The first window failed the criteria at 97.7% overall. After the two fixes, 4 more days of shadow (1,020 runs) showed zero unexplained differences. The due-date fault would have hit 2% of invoices in production and would have surfaced only as late-payment disputes weeks later.
