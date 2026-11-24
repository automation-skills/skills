---
name: decide-what-to-automate
title: Decide what to automate
description: |
  Use this skill when someone asks whether a process is worth automating, or has a list of candidates and must choose: "should we automate this", "is this worth building", "what should we automate first", "prioritize the automation backlog", "what's the ROI of automating this", "build it or keep doing it by hand". Produces a scorecard per candidate (volume, hands-on minutes, exception rate, error cost, process stability, build and run cost), a payback period with its assumptions, and a verdict: automate now, automate a slice, fix the process first, or do not automate.
category: Automation design
tags: [Operations, Strategy]
author: Itay Guttman
authorTitle: CEO
authorCompany: Engini
---

Applies before anything is specced or built. Produces a verdict per candidate that the process owner and whoever pays for the build would both sign.

## The play

1. **Count from the record, not the story.** Pull the last 30 instances, or the last 4 weeks, from the system where the work lands: tickets, invoices, rows, sent emails. "We do this all the time" often turns out to be 14 a month. If there is no record, have the people doing it log every instance for two weeks before you score it.
2. **Time the hands-on minutes.** Watch two or three real runs, or ask for start and stop times. Count the minutes a person works, not the elapsed time while the work waits in a queue. Waiting is a speed problem and gets valued separately, if at all.
3. **Count the exceptions.** Of those 30, how many left the standard path: missing data, a judgment call, a back-and-forth, a special customer? Exceptions still reach a person after you automate, so they come off the saving.
4. **Price one wrong outcome.** What does a single error cost, who notices, and how late? A wrong internal tag costs a minute. A wrong payment, a wrong customer email or a missed access revocation can cost more than a year of savings, and it forces an approval gate whose review time you must count.
5. **Check stability.** How many times did the steps, rules or underlying systems change in the last 12 months, and is a migration, reorg or policy rewrite planned in the next 6? An automation of a moving process gets rebuilt before it pays back.
6. **Price the build and the run.** Builder hours times a loaded rate, times 1.5 for the first build against a system you have not integrated before. Then monthly cost: run fees, about 1 hour of maintenance per connected system, 2 to 4 more hours for any AI step, and the reviewer time of every approval gate.
7. **Compute payback.** Monthly net saving = runs per month x hands-on minutes x (1 - exception rate) / 60 x loaded hourly rate, minus the monthly cost. Payback in months = build cost / monthly net saving. Formulas, rates and three worked cases are in `references/scoring.md`.
8. **Apply the thresholds.**
   - Under 6 months: automate now.
   - 6 to 12 months: automate only if it also removes a risk you can price or a bottleneck someone is waiting on.
   - Over 12 months: do not build it as scoped. Most automations need a significant rework within 12 to 18 months as the systems around them change, so a longer payback rarely arrives.
   - Exception rate over 30%: automate the slice (gather the data, draft, route) and leave the decision with a person. Over 50%: standardize the process first.
9. **Run the do-not-automate list** below, even when the payback looks good.
10. **Write the verdict** with the assumptions that drove it and a check 30 days after go-live that compares real volume, exceptions and minutes with the estimate.

For a backlog of more than five candidates, shortlist with the quick triage score in `references/scoring.md` and do the full payback only on the top ones.

## Do not automate

- Work that runs fewer than 2 times a month with a low error cost. A checklist is cheaper.
- A process no two people do the same way. You would be encoding one person's habits.
- A step that is itself the control: the second pair of eyes on payments, access grants or contract terms. Automate the preparation, not the check.
- A decision someone must own by name: a dismissal, a credit limit, a pricing exception. Automate the evidence pack.
- A process due to change within 6 months.
- A one-time cleanup or migration. Run a reviewed script once, with approval, and leave no standing automation behind.
- Anything with no owner willing to receive its failure alerts.

## What good looks like

- **The tell:** the best lead asks "how many last month, and how many of those were weird?" before anyone mentions a tool. The answer usually halves the estimate.
- **The common mistake:** savings computed as volume times minutes, with no exceptions, no maintenance and no review time. That number is often close to twice the real one.
- **The quiet miss:** ranking by excitement. The best first candidates are boring hand-offs between two teams, with high volume and clean inputs.
- **Good output:** every number has a source (a count, a timed run, a named estimate), the verdict follows from the thresholds, and a stranger could redo the math and get the same answer.

## Rules

- MUST count volume and exceptions from real records, or mark the score provisional.
- MUST subtract exceptions, maintenance and approval-review time from the saving.
- MUST put a price on the cost of one error, even a rough one, and say who would notice it.
- NEVER recommend automating a control step or a decision that needs a named owner.
- NEVER present a saving without its assumptions and the 30-day check.
