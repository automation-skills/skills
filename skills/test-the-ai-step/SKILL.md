---
name: test-the-ai-step
title: Test the AI step
description: |
  Use this skill when an automation contains an AI or agent step that must be trusted before go-live, or that is changing: "test the prompt", "evaluate the AI step", "is the model good enough", "we're switching models", "the classifier got worse", "build an eval", "how do we know the agent is right". Produces a golden set built from real cases, written pass criteria with critical errors defined, a regression report for every prompt or model change that names the cases that flipped, and production drift signals with alert thresholds.
category: Automation QA
tags: [QA, Engineering, AI]
author: Itay Guttman
authorTitle: CEO
authorCompany: Engini
---

Applies before an AI or agent step goes live inside an automation, and every time its prompt, model, tools or inputs change. Produces evidence that the step is right often enough, on the cases that matter, and keeps being right.

## The play

1. **Define the job precisely.** The input, the output shape, what "correct" means for each field, and which errors are critical: a wrong amount, a wrong recipient, an invented commitment, data from another customer. If two careful people would disagree on the right answer, fix the definition before testing any model.
2. **Build the golden set from real cases.** 150 to 300 real historic inputs, never invented ones. Roughly 60% typical cases in their real mix, 25% known hard cases (ones people got wrong or argued about), 15% edge cases (empty, very long, another language, a poor scan, two documents in one, text that tries to instruct the model). At least 20 cases for every category or branch that matters. Mask personal data the test does not need. Sourcing and sizing are in `references/golden-set.md`.
3. **Label with the owner of the decision**, not the builder, with a reason per label. A second person labels 20% of the cases. Under 90% agreement between them means the task is ambiguous, and no model score means anything until it is fixed.
4. **Hold out 20%.** Nobody looks at those cases while tuning the prompt. Report both scores. A big gap means you tuned to the set.
5. **Write pass criteria before the first run.**
   - Zero critical errors.
   - Precision of the automatic slice at target: 99% where the step writes money or customer-visible data, 95% for internal labels.
   - No category more than 5 points below the overall score.
   - An "unclear" rate inside a set band, for example under 15%. A step that answers "unclear" to everything is safe and useless.
   - Valid output shape on at least 99.5% of cases.
   - Cost per run and 95th percentile latency inside budget.
6. **Grade like it matters.** Exact match for fields and categories. For drafts, a rubric of required facts, forbidden content and tone, graded by a person, or by a grader model only after it agreed with human grades on at least 90% of 50 cases. Whoever wrote the prompt does not grade it.
7. **Run 3 times when outputs vary.** A case passes only if it passes all 3. Report flaky cases on their own line: they will be flaky in production too.
8. **Read the failures, not only the score.** Group them by cause: unclear instruction, missing context, wrong label, model limit. Fix wrong labels openly and log each change.
9. **Run the regression on every change:** prompt, model or model version, settings, schema, tools, retrieval data, or a new upstream input format. Rerun the whole set and compare case by case with the last accepted run. Block the release on any new critical error, an automatic-slice precision drop over 1 point, or any category falling more than 5 points. Read every case that flipped from pass to fail, even when the total went up. Pin the model version; if you cannot, rerun the set weekly.
10. **Watch for drift in production:** the human override rate in the review queue, the "unclear" and fallback rates, output-shape failures, the output and input mix, cost and latency. Review 30 random automatically accepted cases a week. Every confirmed production error becomes a golden case. Thresholds are in `references/regression-and-drift.md`.

## What good looks like

- **The tell:** the best builder asks "where did these test cases come from?" A set written by the prompt author passes. Real traffic does not.
- **The common mistake:** one accuracy number on 30 hand-picked examples, run once on launch day and never again. The model changes, the inputs drift, and a customer notices first.
- **The quiet miss:** a better overall score hiding a new failure. A model upgrade can gain 2 points on average and start misreading every amount written with a decimal comma.
- **Good output:** a versioned golden set with labels and reasons, a pass or fail report per change that names the flipped cases, and a weekly drift line that someone reads.

## Rules

- MUST build the golden set from real inputs, labelled by the owner of the decision.
- MUST define critical errors and pass criteria before the first run.
- MUST rerun the full set on any change to prompt, model, schema, tools or retrieval data, and block on the criteria.
- MUST add every confirmed production error to the golden set.
- NEVER tune the prompt while looking at the holdout cases.
- NEVER trust a grader model that has not been checked against human grades.
- NEVER let a test run send messages or write to live systems. Test the step on its own, with side effects off.
