---
title: Building the golden set
description: Where the cases come from, how to stratify and label them, how many you need and why, the holdout, and how to keep the set alive.
---

## Where the cases come from

- **The real input stream.** Pull 3 to 6 months of historic inputs from the place the step will read: the inbox, the form submissions, the documents, the tickets. A random draw gives you the typical mix.
- **Known hard cases.** Ask the people who do the work today: "which ones do you argue about?", "which ones did someone get wrong last quarter?". Pull those by id.
- **Edge cases.** Empty input, a very long input, another language, a poor scan or photo, two documents in one file, a forwarded chain, a reply that quotes an old message, and text that tries to give the model instructions.
- **Production errors,** once live. Each confirmed error becomes a case with its correct label.

Never let the prompt author invent cases. Invented cases share the author's blind spots, so the prompt passes them by construction.

## Stratification

| Slice | Share | Purpose |
|---|---|---|
| Typical, in the real mix | about 60% | The score you will see in production |
| Known hard | about 25% | Where the money is lost |
| Edge | about 15% | Where the step breaks loudly |

On top of the shares: at least 20 cases for every category, branch or counterparty type that matters, even when it is rare in real traffic. Report those small groups separately; never let them vanish into an average.

## Labelling

- The label comes from the person who owns the decision in the business, with a one-line reason.
- Write the labelling guide first: for each category or field, a definition, two examples, and the rule for ambiguous cases.
- A second labeller does 20% of the cases, blind. Measure agreement.
  - 95% or more: good.
  - 90% to 95%: review the disagreements and tighten the guide.
  - Under 90%: the task is not defined well enough to test. Fix it before running any model.
- When a model "fails" a case and the label turns out wrong, fix the label, log the change, and recount. Never quietly drop the case.

## How many cases

The score on a test set is an estimate. With a measured accuracy of 95%, the 95% confidence range is about:

| Cases | Range at 95% | Range at 99% |
|---|---|---|
| 50 | plus or minus 6.0 points | plus or minus 2.8 |
| 100 | plus or minus 4.3 | plus or minus 2.0 |
| 300 | plus or minus 2.5 | plus or minus 1.1 |
| 1,000 | plus or minus 1.4 | plus or minus 0.6 |

What follows from that:

- 150 to 300 cases catch big breaks reliably. They cannot prove a 1-point improvement in the total.
- To claim the automatic slice is 99% precise when you saw zero errors, you need about 300 cases in that slice. With zero errors in n cases, the true error rate is only bounded at about 3 in n.
- Regressions are easier to see than totals suggest, because you rerun the same cases and see each flip directly. Six cases flipping from pass to fail is a signal worth reading even when the total moved by less than the range.

## The holdout

Set aside 20% of the cases, chosen at random within each slice, before anyone tunes anything. Nobody reads them while iterating on the prompt. Score them at each release candidate. If the tuning set scores 6 or more points above the holdout, the prompt has been fitted to the cases and will disappoint in production.

## Keeping the set alive

- Version the set. Every report names the version it ran on.
- Add every confirmed production error, and every new input layout or template once it passes 5% of volume.
- Add 10 fresh random cases a month from production, so the typical slice keeps matching reality.
- Retire cases only when the business rule they test no longer exists, and log why.
- Store the set where the source data is allowed to live, with the same access controls. A golden set of real customer documents is customer data.
