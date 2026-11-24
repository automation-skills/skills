---
title: Timeouts, audit and graduation
description: Deadlines by gate type with the reasoning, the escalation ladder, the audit record, the monthly review, rubber-stamp signals, and a worked case of a gate redesigned so it catches things again.
---

## Deadlines by gate type

| Gate | Deadline | Why |
|---|---|---|
| AI-drafted reply to a customer | 4 business hours | Customers expect a same-day answer |
| Access or permission grant | 4 business hours | Someone's work is blocked |
| Supplier payment run | 1 business day before the bank cut-off | A late approval must still leave time to fix a rejection |
| Bulk update of internal records | 2 business days | Not urgent; staleness is the bigger risk |
| Deletion or retirement | 5 business days | Never urgent; a slow yes is cheap, a fast wrong yes is not |

## Escalation ladder

1. **50% of the deadline:** remind the approver, with the screen attached, not just a link.
2. **100%:** send to the backup approver, visible to the approver's manager.
3. **3 times the deadline:** expire. Cancel the run, tell the requester and the owner, log it as expired.
4. **Never:** approve on timeout, or let the requester approve their own item once it escalates.

For time-critical gates such as payments, expiry happens before the cut-off, with a notice to the finance lead, so nobody discovers it the next morning.

An approval given more than 24 hours after the payload was built re-checks the payload against live data before execution: the record may have changed, the recipient may have left, the invoice may already be paid. If the check changes the payload, ask again.

## The audit record

One record per decision, kept for as long as the business keeps the underlying transactions:

- gate name and version
- run id and payload version or hash
- the screen as shown (or enough to rebuild it exactly)
- approver identity and channel
- decision, reason code, free-text reason
- requested at, decided at, seconds spent on the screen
- what executed, with its own payload hash
- match: executed hash equals approved hash, yes or no

A "no" in the last field is an incident, investigated the same day.

## Monthly review per gate

| Number | Healthy | Look closer when |
|---|---|---|
| Decisions | Stable with volume | Grows faster than volume: the gate's scope crept |
| Rejection rate | 2% to 15% | Under 1% for 3 months, or over 25% |
| Median time per item | 20 seconds to 2 minutes | Under 5 seconds, or over 10 minutes |
| Expired | Under 2% | Over 5%: wrong approver or deadline |
| Edits | Some | Zero for 3 months |
| Executed equals approved | 100% | Anything less |

## Rubber-stamp signals

- Median under 5 seconds per item over a rolling 200 decisions.
- Approval rate above 99% over 200 decisions with zero edits.
- Bursts: 30 or more approvals inside one minute by one person.
- Flagged items approved without being opened, if your screen tracks it.

Any two of these together: raise it with the process owner with the numbers, and decide between removing the gate (with spot checks) and redesigning the screen. Do not quietly accept it.

## Graduation

- **Proposal:** after 200 decisions with under 1% rejected and no material rejection (wrong amount, wrong recipient, wrong record) in the last 100.
- **Change:** the category becomes automatic with a 5% random spot check reviewed weekly. Outliers and flags keep the gate.
- **Approval:** the process owner, in writing, naming the category and the spot-check rate.
- **Reversal:** one material miss in a spot check puts the full gate back until another 200 clean decisions.

Bulk deletion, payments to new recipients and bank detail changes never graduate.

## Worked case: credit notes

- **Before:** every credit note was gated. 600 a month, 99.6% approved, a median of 3 seconds each. Two wrong credit notes had gone out that quarter, both approved.
- **Diagnosis:** the gate held 600 items of which perhaps 30 deserved a look. Nobody could find them.
- **Redesign:**
  - Automatic: credit notes of 200 or less on an order with a confirmed return, checked by a rule. A 5% spot check.
  - Gated: anything over 200, any credit without a confirmed return, a customer's third credit in 90 days.
- **After:** 70 gated a month. Rejection rate 9%, median 45 seconds. Approvers spend about 50 minutes a month instead of 30, and that time now catches errors instead of producing clicks.
