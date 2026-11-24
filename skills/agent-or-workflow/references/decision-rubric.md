---
title: Decision rubric and worked decompositions
description: Signs that point to each rung, two worked decompositions (one with no agent, one where an agent earns its place), and the anti-patterns to catch in review.
---

## Signs for each rung

| Sign in the step | Points to | Why |
|---|---|---|
| The rule fits in one sentence on existing fields | Rule | A model adds cost, latency and a new way to be wrong |
| Math, dates, totals, comparisons, lookups by id | Rule | Models are unreliable at arithmetic and date logic; code is exact |
| Free text, scans or images in, fixed fields out | AI step | Rules break on the tenth layout; one extraction handles them all |
| Choose one of 3 to 8 known branches from messy input | AI step with an enum | The branch set is fixed, so the paths can still be tested |
| The next lookup depends on what the last one returned, and the paths cannot be listed | Agent | A fixed workflow would need a branch for every combination |
| The step writes, pays, deletes or messages someone outside | Rule, behind a check | The decision may be the model's, the execution never is |
| Two careful people would disagree on the answer | Fix the definition first | No rung can be tested against an answer nobody agrees on |

## Worked case 1: supplier invoices arriving by email (no agent)

The request was "an AI agent that processes our supplier invoices". Decomposed, it has 10 steps, 2 of them AI, and no agent.

| # | Step | Rung | Why | Checks and route |
|---|---|---|---|---|
| 1 | New message in the payables inbox | Rule (trigger) | Event | |
| 2 | Dedupe on message id plus attachment hash | Rule | Exact key | Duplicate: stop and log |
| 3 | Has a document attachment? | Rule | File type | None: human queue |
| 4 | Classify: invoice, credit note, statement, reminder, other, unclear | AI step, enum | Free text and layouts vary | Unclear or low confidence: human queue |
| 5 | Extract supplier tax id, invoice number, date, currency, total, lines, each with a quoted source | AI step, schema | Many layouts | Schema fail: one retry, then queue |
| 6 | Find the supplier by tax id in master data | Rule | Exact key, never name similarity | No match: queue |
| 7 | Lines sum to the total within 0.01, date not in the future and within 120 days, currency allowed for this supplier, invoice number not already booked | Rule | Arithmetic and lookups | Any fail: queue, naming the failed check |
| 8 | Match to a purchase order on number and amount within 2% | Rule | Tolerance rule | No match: queue |
| 9 | Create a draft bill | Rule (write) | Validated fields only | |
| 10 | Post the bill | Rule (write) | | Human approval above the single-approver spend limit or for a first invoice from a new supplier |

Bank details are never taken from an invoice or an email. A change to them goes to a person who verifies it through a contact already on file. It is the most common fraud route in payables, and a model cannot detect a well-made fake.

Cost: two AI calls per invoice. An agent doing the same job would make 10 to 20 calls per invoice, take longer, and be far harder to test, for no gain: every path above can be listed.

## Worked case 2: "why is this order stuck?" (an agent earns its place)

Support staff spend 20 to 40 minutes per case reading the order system, the warehouse system, carrier tracking and the payment status. The cause can sit in any of them, and what to read next depends on what the last system showed. The paths cannot be listed in advance.

- **Rung:** agent, read-only.
- **Tools:** 4 read tools, one per system. No write tools.
- **Budget:** 12 tool calls, 3 minutes per case.
- **Output schema:** cause from an enum (payment hold, stock shortage, address problem, carrier delay, warehouse exception, unknown), evidence as links to the records read, and one proposed action from an enum (release, reship, refund, contact customer, escalate).
- **Execution:** a support person approves the proposed action. The action runs as a rule step using the ids from the evidence, not from the agent's text.
- **Expansion criterion:** proposals accepted unchanged on at least 80% of 200 cases before the team considers making any action automatic, and never refunds.

## Anti-patterns to catch in review

- **A model doing arithmetic or date logic.** Move it to a rule.
- **A model deciding something a field already says** ("is this a priority customer?"). Read the field.
- **Matching records by name similarity in a model.** Match on ids. If fuzzy matching is truly needed, the model proposes candidates and a rule confirms on a second key.
- **An agent used as a router** among 4 known workflows. That is an AI step with an enum.
- **Regular expressions across 30 document layouts.** That is an extraction step.
- **"Let the agent decide whether to email the customer."** The agent can draft and recommend. Sending is a rule behind a gate.
- **A long prompt in place of steps.** If the prompt reads like a procedure with numbered steps and conditions, those conditions are workflow logic.
