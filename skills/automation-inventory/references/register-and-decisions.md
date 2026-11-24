---
title: Register, blast radius and decisions
description: The register fields, the blast radius rubric, ownership risk and ranking, the decision table, a worked case of four automations, and the one-page risk summary.
---

## Register fields

| Field | Notes |
|---|---|
| Id and name | A stable id, plus the name people use |
| What it does | One line, a verb and an object |
| Trigger | Event, schedule (with time zone), poll interval, or manual |
| Reads from / writes to | Each system, with read or write |
| Process served | The business process and its owner |
| Owner | A person, never a team. Blank means orphaned |
| Builder | Who built it, and whether they are still here |
| Runs as | Service identity, a person's account, a shared account |
| Credential type and scope | Token, key, password; read, write, admin; expiry |
| Last successful run | From the run history, not from memory |
| Runs per month, failure rate over 30 days | From the run history |
| Alerts go to | A person or channel that is read today |
| Bus factor | People who could explain and fix it today |
| Blast radius | Score from the rubric below |
| Decision | Keep, fix, merge, retire, with a date and approver |

## Blast radius rubric

Score each question 0 to 3.

| Question | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| Reach per run | Under 10 records, internal | 10 to 100 | 100 to 1,000 | Over 1,000, or master data |
| Reversibility | Read only | Writes easily undone | Undone with effort | Irreversible: sends, payments, deletions |
| Time to notice a failure | Within an hour | Within a day | Within a week | Only at month-end or audit |
| External exposure | Internal only | Partner or supplier data | Customers see it | Money or regulated data |

Total out of 12: 9 or more is high, 5 to 8 medium, 4 or less low.

## Ownership risk and ranking

- **Ownership multiplier:** bus factor 2 or more: x1. Bus factor 1: x2. Orphaned: x3.
- **Personal credential:** add 3 to the result, because it fails on the day that person leaves and may keep their access alive after it.
- **Risk score** = blast radius x ownership multiplier, plus 3 for a personal credential.
- Anything scoring 20 or more is fixed this month, 10 to 19 this quarter, and under 10 at the next quarterly review.

## Decision table

| Decision | When | First action | Approval |
|---|---|---|---|
| Retire | No successful run in 90 days, or no one consumes the output | Confirm with the consumer, then turn off and announce | Owner, or process owner for an orphan |
| Merge | Same trigger or same records as another automation | Choose the survivor, move the missing logic, then retire the other | Both owners |
| Fix | Valuable, but bus factor 1, personal credential, no alerts, no owner, or failure rate above 5% | Assign an owner, move to a service identity, route alerts, write a one-page description | Owner |
| Keep | Owner, alerts, service identity, description, clean 30 days | Review next quarter | None |

Retiring is always two steps: off and announced, then deleted after 2 to 4 weeks with the definition archived. For anything that runs monthly or quarterly, wait through one full cycle.

## Worked case: four automations at a 300-person company

**A. New deal creates a delivery project.** Runs as the sales operations manager's own account. 400 runs a month, 2% failures, bus factor 1.
Blast radius: reach 1, reversibility 1, notice 1, exposure 0 = 3. Risk: 3 x 2 + 3 = 9.
**Fix:** move it to a service identity, route alerts to the delivery lead, have a second person walk through it.

**B. Nightly customer sync into billing.** A scheduled script on a server. The author left 8 months ago. It writes about 3,000 records a night, and its failure emails go to the departed author's closed mailbox.
Blast radius: 3 + 2 + 3 + 3 = 11. Risk: 11 x 3 = 33.
**Fix, this week.** The finance lead becomes the owner, alerts are routed to finance operations, and the script is documented. Not retired: billing depends on it every night.

**C. Two automations set lead status from form submissions,** one built by marketing and one by sales. 12% of new leads changed status twice within a minute, and sales reported leads "disappearing" from their queue.
**Merge:** the sales version survives because sales owns the field. The marketing logic for event sign-ups moves into it, and the marketing version is turned off, announced, and deleted after 4 weeks.

**D. A weekly summary email to a leadership group.** 3 of the 5 recipients have left. The two remaining say they do not read it.
**Retire:** turned off and announced to the two recipients. Deleted after 4 weeks, definition archived.

## One-page risk summary

For leadership, one table, at most ten rows, sorted by risk score:

| Automation | What it does | Why it is a risk | Score | Owner | Action | Due |
|---|---|---|---|---|---|---|

Under the table, three numbers: total automations found, the share with a named owner, and the share running under a service identity. Track those three every quarter.
