# 01_review: a verdict on every draft

One job: read each draft reply next to its review and give it a verdict: send as is, send with an edit, or hold.

## Inputs

- This run: `../../input/` (this week's doc: each new App Store review with its draft reply)
- Every run: `../../_shared/rules.md`, `../../_shared/done.md`
- Every run: `references/voice.md`

Do not load: `../../_archive/`, `../02_check/`.

## Steps

1. Read the inputs. List every review in the doc with its draft.
2. (assumed) If the doc can't be read, or a draft can't be matched to its review, stop and ask the person.
3. Test each draft against every rule in `rules.md`.
4. Pick its verdict by the lines in `done.md`.
5. (assumed) For an edit, keep the drafter's words where they pass and change only what breaks a rule. `references/voice.md` shows how we sound.
6. Write the result in the shape `done.md` sets.

## Output

- `output/verdicts.md`

## A person checks

Read each verdict next to its draft before anything is pasted into App Store Connect.
