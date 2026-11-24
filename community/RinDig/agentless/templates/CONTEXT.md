# {Name}: the stages

{The flow in one line, in the person's own words.}

| Stage | Job | Reads | Writes |
|---|---|---|---|
| `01_{stage}` | {five words} | `input/`, `_shared/` | `output/{file}` |
| `02_{stage}` | {five words} | `01_{stage}/output/{file}`, `_shared/` | `output/{file}` |
| `{NN}_check` | tests the result against `_shared/` | the last work stage's output, `input/`, `_shared/` | `output/check.md` |
