---
title: The approval screen
description: What the approver sees, as a template, with a bad and a good example for a single outbound message and for a batch of payments, and the flags worth computing.
---

The approval screen is the whole control. If the approver has to open another system to decide, most will not, and the gate becomes a formality.

## Template

```
[VERB] [COUNT] [OBJECT] in/to [TARGET]            e.g. "Send 1 email to an external contact"
Requested by: [trigger or person], at [time]     Deadline: [time], then escalates to [backup]

WHAT WILL HAPPEN
  The exact payload, or a before -> after diff per record.
  For more than 10 items: totals, then outliers, then a sample.

WHY
  The trigger and the evidence: the record, message or rule that started it.

ALREADY CHECKED
  Each check that ran and its result.

UNUSUAL
  Every flag, with the normal value beside it.

CAN IT BE UNDONE?
  Yes, how and until when / No.

[Approve]   [Reject - reason required]   [Edit - checks run again]
```

## Single item: an AI-drafted reply to a customer

**Bad:** "Reply ready for ticket 48213. Approve?" with a link to the ticket. The approver sees neither the text nor the recipient, and approves the idea of replying.

**Good:**

- Send 1 email to an external customer contact (verified on the account: yes).
- The full draft, with the two sentences that state a commitment highlighted.
- Why: the customer asked for a refund timeline; the order is 9 days late.
- Checked: the recipient belongs to the account on the ticket; the refund amount matches the order; no other customer's data appears in the draft.
- Unusual: the draft promises a date. Commitments in replies are normally given by the account manager.
- Cannot be undone once sent.

## Batch: a supplier payment run

**Bad:** "Payment run, 47 payments, 312,450 total. Approve all?"

**Good:**

- Pay 47 suppliers, 312,450 total, from the operating account, value date tomorrow.
- Compared with the last 6 runs: average 38 payments and 265,000.
- **Flagged, must be opened (3):**
  - A supplier whose bank details changed 6 days ago. Verification call logged: no.
  - A payment of 48,200 to a supplier whose largest previous payment was 9,100.
  - A first payment to a new supplier.
- **Largest 5**, then a random sample of 5 from the rest.
- Checked: every invoice approved by its budget owner; no invoice paid twice; every supplier active in master data.
- Cannot be undone after the bank cut-off at 15:00.
- Approve the 44 unflagged / decide each flagged item / reject the run.

## Flags worth computing

- An amount above 3 times the counterparty's median, or above their largest previous amount.
- A first-time recipient, supplier or destination.
- Bank details, an address or a contact changed within the last 30 days.
- A record touched by another automation in the last hour (two writers may be racing).
- An AI output routed here for low confidence: show its reason and its evidence quotes.
- A count more than 50% above the usual run.
- Anything outside working hours for the requester.

## Screen rules

- The payload on screen is the payload that will execute. Never a summary generated separately from it.
- No personal or sensitive data beyond what the decision needs. Mask account numbers to the last 4 digits.
- On a phone, a bulk or money approval shows the totals and the flags and requires opening each flagged item. If a screen cannot show that, it cannot approve that gate.
- Rejection reasons come from a short list plus free text, so they can be counted.
