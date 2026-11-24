# Done means

The check tests the finished result against each line.

- [ ] Every review in `input/` has one entry in `verdicts.md`.
- [ ] (assumed) The file opens with one count line: entries in total, then how many of each verdict.
- [ ] (assumed) Entries follow the doc's order. Each is headed with its number and the reviewer's name as the doc shows it, then quotes the review and the draft.
- [ ] Each entry gives one verdict, in these words: Send as is, Send with an edit, or Hold.
- [ ] Send as is: the draft, which is what goes out, passes every rule in `rules.md`.
- [ ] Send with an edit: the fixed reply sits right under the verdict, whole and ready to copy, and passes every rule in `rules.md`.
- [ ] (assumed) The fixed reply sits alone in a fenced block, and one line under it says what changed.
- [ ] (assumed) Hold: one line says what must happen before anything goes out: a fact or decision only the team has, a draft that is missing, or a report to Apple in place of a reply. A draft an edit could fix is not held.
