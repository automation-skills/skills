---
name: parallel-run-cutover
title: Parallel run and cutover
description: |
  Use this skill when an automation moves to another platform, gets rewritten, or is replaced by a new version that must do the same job: "migrate our workflows", "rebuild this on the new platform", "rewrite the integration", "replace the old automation", "run both side by side", "cut over safely", "how do we roll back". Produces an inventory of what the old automation really does, a shadow-run plan with output diffing, written cutover criteria, an approved cutover sequence, and a rehearsed rollback.
category: Automation delivery
tags: [Operations, Engineering]
author: Itay Guttman
authorTitle: CEO
authorCompany: Engini
---

Applies whenever a new automation takes over a job an existing one does. Produces a cutover that takes minutes, backed by a diff in which every difference has a class and a decision.

## The play

1. **Inventory what the old one really does, from its run history.** Not its documentation. Take 30 to 90 days of runs: volume per day and the peaks, every branch taken, every side effect, every downstream consumer. List the quirks people rely on without knowing it: it trims whitespace, rounds to 2 decimals, skips records with no email, sends at 9:00 in one time zone, retries twice. Each quirk is kept or dropped on purpose.
2. **List the intended differences before running anything.** Every change the new version makes on purpose. Differences that match the list are expected. Everything else is a finding.
3. **Run in shadow first.** The new version receives the same inputs, runs all its logic, and records what it would have written or sent, with every side effect off. If a step cannot run without its side effect, point it at a sandbox target or stop just before it and diff the payload.
4. **Never let both versions write.** Two versions writing to the same target or messaging the same people is how customers get two emails and records get updated twice. Shadow first. If a staged rollout follows, split by a stable key (for example 10% of customers by id) so each item has exactly one live owner.
5. **Diff the outputs per input key.** Normalize first: timestamps, generated ids, ordering, whitespace, number formats, time zones. Compare field by field and sort every difference into intended, bug in new, bug in old, or noise. A bug found in the old one goes to the process owner: fix it now, or reproduce it so downstream consumers keep working. Rules and a worked diff are in `references/diffing.md`.
6. **Cover the full cycle.** At least 2 weeks and one of each periodic event: month-end, the weekly batch, a quiet day, a peak day. Every branch exercised at least 20 times. Branches that did not occur in the window get replayed from historic inputs, in shadow.
7. **Meet the written cutover criteria:**
   - at least 99.5% exact match on business fields after normalization and after removing listed intended differences, over at least 500 runs or the full window, whichever is larger
   - zero unexplained differences on money, recipients, ids and dates
   - every remaining difference classified and accepted by the process owner
   - an error rate no higher than the old one's
   - alerts, owner and credentials moved to the new version, under a service identity
   - rollback rehearsed once
8. **Plan the handover moment** (sequence per trigger type in `references/cutover-runbook.md`). Pick a low-volume window: not month-end, not before a weekend or holiday. Freeze changes to the old version from the start of shadow. For events, the new version dedupes on the event key so anything in flight is processed once. For polling, it starts from the old one's last processed position, never from now and never from the beginning. For schedules, the next fire happens on exactly one side.
9. **Get the cutover approved** by the process owner, with the diff summary and the criteria in front of them. Then switch, and watch the first 10 live runs one by one.
10. **Keep rollback ready and decommission deliberately.** The old version stays intact and off for 30 days or two full business cycles, with rollback triggers written in advance. After that window, and with the owner's approval, remove its schedules, listeners and credentials, and archive its definition and run history.

## What good looks like

- **The tell:** the best migrator asks "what does the old one do that nobody wrote down?" and answers it from the run history.
- **The common mistake:** rebuilding from the documentation, testing ten happy-path cases, switching on a Friday and deleting the old one the same day. The month-end branch fails three weeks later with nothing to compare against.
- **The quiet miss:** a forgotten listener on the old version that wakes up months later and duplicates everything.
- **Good output:** a diff report where every difference has a class and an owner's decision, a cutover nobody noticed, and a rollback nobody needed but anyone on the team could run.

## Rules

- MUST keep the new version's side effects off until cutover.
- MUST list intended differences before diffing.
- MUST get the process owner's approval for the cutover and for decommissioning.
- NEVER let both versions write to the same target or message the same people.
- NEVER start a polling replacement from now or from the beginning.
- NEVER delete the old version before the rollback window ends.
