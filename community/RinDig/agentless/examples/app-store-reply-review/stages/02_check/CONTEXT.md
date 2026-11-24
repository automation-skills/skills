# 02_check: test the result before it leaves

One job: say whether `01_review`'s result meets `_shared/done.md` and `_shared/rules.md`. You did not write it. Do not fix it.

## Inputs

- This run: `../01_review/output/verdicts.md`
- This run: `../../input/` (to confirm nothing was dropped)
- Every run: `../../_shared/done.md`, `../../_shared/rules.md`

Do not load: the work stages' contracts or references, `../../_archive/`.

## Steps

1. Treat each line of done.md, and each rule in rules.md, as a test. Test everything the result puts forward, including input it passes on unchanged (a draft approved as is). Input quoted only for context, such as the message a reply answers, is exempt.
2. Run every test, on every item if the result is a batch. Write each FAIL with its item and a quote of the part that fails. Then one line: how many tests passed and how many failed.
3. End with one word on its own line: SHIP if nothing failed, FIX if anything did. For a FIX, the line above it names the stage to run again.

## Output

- `output/check.md`

## A person checks

Read the result and the FAILs together. Ship it, or send it back.
