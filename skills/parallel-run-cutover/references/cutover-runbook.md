---
title: Cutover runbook
description: The cutover criteria as a checklist, the handover sequence for event, polling, scheduled and file-based triggers, the first-runs watch, rollback triggers and steps, and decommissioning.
---

## Criteria checklist

Every line is a yes before the process owner is asked to approve.

- [ ] Shadow window of at least 2 weeks, including a month-end and the weekly batch.
- [ ] At least 500 runs, or the full window if it holds more.
- [ ] Every branch exercised at least 20 times, live or replayed from history.
- [ ] At least 99.5% match on business fields after normalization and intended differences.
- [ ] Zero unexplained differences on money, recipients, business ids and dates.
- [ ] Every remaining difference classified, with the owner's decision recorded.
- [ ] New version's error rate no higher than the old one's over the same window.
- [ ] Alerts route to the owner and a backup, tested with one deliberate failure in shadow.
- [ ] Credentials belong to a service identity, not to the builder.
- [ ] Rollback rehearsed: the old version shown able to resume from a given position.
- [ ] Downstream consumers told the date and what changes for them.
- [ ] Changes to the old version frozen since the start of shadow.

## Choosing the window

- A weekday morning in the owner's time zone, with the builder and the owner both available for the next 4 hours.
- Not month-end, quarter-end, the day before a weekend or holiday, or a known peak.
- Before the old version's next scheduled run, with enough margin to stop if something looks wrong.

## Handover sequence by trigger type

**Event-driven (webhook, listener, message).** The new version has been receiving every event in shadow, so it already has a log of them.
1. Turn off the old version's side effects.
2. Note the last event the old version fully processed.
3. Let the new version process, from its shadow log, every event after that one, deduping on the event key.
4. Turn on the new version's side effects.
The gap between steps 1 and 4 loses nothing, because step 3 catches it. Overlapping the two live versions instead risks processing an event twice.

**Polling.**
1. Stop the old version after a completed poll.
2. Read its last processed position: the id, the timestamp or the cursor.
3. Start the new version from exactly that position. Start a few minutes earlier only if it dedupes on the business key.
Never start from now (items in between are lost) or from the beginning (everything is processed again).

**Scheduled.**
1. Find the next fire time of both versions, in their configured time zones.
2. Disable the old schedule after its last run before cutover. Enable the new one before its first run after.
3. Check that the new scheduler will not catch up on runs it thinks it missed.

**File or batch based.** Hand over the processed-files marker or folder, and confirm the new version skips everything the old one already processed.

## The first runs

Open each of the first 10 live runs and read them: the input, the output, the side effect. Check one record in each downstream consumer. Keep the owner's attention for the first full business day, and for the first run of each periodic event (the first month-end on the new version gets its own check).

## Rollback triggers, written before cutover

- Any wrong amount or wrong recipient.
- Error rate above twice the old version's baseline for an hour.
- A missed scheduled run.
- A downstream consumer reports broken or missing input.

Any one of these means rolling back first and diagnosing afterwards.

## Rollback steps

1. Turn off the new version's side effects.
2. Read its last processed position.
3. Turn the old version back on from that position.
4. List every record the new version wrote or message it sent since cutover, from its run log. Review the list with the owner and correct what needs correcting, with approval for any bulk correction.
5. Announce the rollback to the same people who were told about the cutover.

## Decommissioning

After 30 days or two full business cycles, whichever is longer, with no rollback trigger, and with the owner's approval:

- Remove the old version's schedules.
- Remove its listeners and webhook subscriptions at the source systems. A subscription registered on the source keeps calling even after the automation behind it is off.
- Revoke the credentials only the old version used, after checking that nothing else depends on them.
- Archive the definition and its run history for as long as the business keeps the records it produced.
- Update the company's list of automations and tell the downstream teams it is done.
