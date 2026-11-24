# App Store Reply Review: the stages

Once a week our support person drafts replies to new App Store reviews in a doc; before anything goes live, each one is read; then we paste them into App Store Connect.

| Stage | Job | Reads | Writes |
|---|---|---|---|
| `01_review` | a verdict on every draft | `input/`, `_shared/` | `output/verdicts.md` |
| `02_check` | tests the result against `_shared/` | the last work stage's output, `input/`, `_shared/` | `output/check.md` |
