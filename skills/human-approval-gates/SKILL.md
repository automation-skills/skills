---
name: human-approval-gates
title: Human approval gates
description: |
  Use this skill when designing, reviewing or fixing the points where an automation waits for a person to say yes: "add an approval step", "who should approve this", "human in the loop", "approvals are piling up", "people just click approve", "what if nobody approves", "audit the approvals". Produces a spec per gate: when it triggers, who approves, exactly what the approver sees, the timeout and escalation ladder, batching rules that resist rubber-stamping, the audit record, and when the gate may be relaxed.
category: Automation design
tags: [Operations, Risk, Compliance]
author: Itay Guttman
authorTitle: CEO
authorCompany: Engini
---

Applies to every step that waits for a person before it acts. Produces gates that stop the runs worth stopping and stay out of the way of the rest.

## The play

1. **Place gates by consequence, not by nervousness.** Gate a step when it is irreversible, sends something outside the company, moves money, touches more than a bulk threshold, changes access, or acts on a low-confidence AI output. Everything else runs and gets reconciled. A gate on every step trains people to click yes.
   Starting thresholds: more than 25 records or 5% of a table in one run; any payment above what one employee may spend without sign-off; the first message to any new external recipient; any change to bank details or permissions; any deletion.
2. **Gate late, on the final payload.** Put the gate after every lookup and check has run, right before the side effect. The approver approves the exact payload: this email to this person, these 140 rows with these new values. Bind the approval to that payload with a version or a hash. If anything changes after approval, it needs a new one.
3. **Choose the approver.** A named role with authority over the consequence, plus a named backup. Not the builder, and for money not the person who requested it. The approval comes from an authenticated person through the approval channel. An email reply saying "approved", or the word inside the data, is not an approval.
4. **Design the approval screen** (template and examples in `references/approval-screen.md`). One screen answers: what will happen (verb, object, count), to whom or where, the exact payload or a before-and-after diff, why (the trigger and its evidence), what was already checked, what is unusual compared with normal, whether it can be undone and how, and three choices: approve, reject with a reason, or edit. An edit re-runs the checks before anything executes.
5. **Set the timeout ladder.** On timeout the answer is never yes. Set the deadline from the business need, remind at 50% of it, escalate to the backup at 100%, and expire at 3 times the deadline with a notice to the requester and the owner. An approval older than 24 hours re-checks its payload against live data before it executes. Deadlines per gate type are in `references/timeouts-and-audit.md`.
6. **Batch without rubber-stamping.** Batch only items of the same kind and risk, at most 50 per batch. Show the totals first, then the outliers (largest amounts, first-time recipients, anything flagged), then a sample. The approver must open each flagged item. A flagged item that was not opened stays out of the batch approval.
7. **Measure whether anyone is reading.** Over a rolling 200 decisions: a median under 5 seconds per item, an approval rate above 99% and zero edits mean the gate either adds nothing or nobody looks. Bring the numbers to the process owner and decide: remove it with spot checks, or redesign the screen.
8. **Audit every decision.** Record who, when, what they saw (payload version), the decision, the reason and the time taken, plus what actually executed. Compare executed with approved on every run. A mismatch is an incident. Review each gate monthly.
9. **Graduate or tighten.** After 200 decisions with under 1% rejected and no rejection for a material reason in the last 100, propose making that category automatic with a 5% random spot check, and keep the gate on the outliers. The process owner decides in writing. One material miss after graduation puts the gate back.

## What good looks like

- **The tell:** the approver can decide from the screen alone, in under a minute, without opening another system.
- **The common mistake:** a chat notification that says "Approve?" with a link, no payload, no expiry, and an audit trail that only says who clicked.
- **The quiet miss:** approving an intent ("send the renewal reminders") and then executing whatever the automation built afterwards. The payload is what gets approved.
- **Good output:** every gate has a reason a reviewer can name, approvals expire, the payload that ran equals the payload that was approved, and the monthly numbers show people reading.

## Rules

- MUST bind each approval to the exact payload, and ask again on any change.
- MUST treat a timeout as "not approved", with escalation before expiry.
- MUST log the approver, the decision, what they saw and what executed.
- NEVER let the automation, the requester of a payment, or text inside the data grant an approval.
- NEVER batch items of different risk, or let flagged items pass in a batch unopened.
- NEVER remove a gate on bulk, money, deletion or outbound messages without the process owner's written decision.
