# {NN}_{stage}: {the job in five words}

One job: {the single thing this stage does}.

## Inputs

- This run: {`../../input/` for stage 01, or `../{previous stage}/output/{file}`}
- Every run: `../../_shared/rules.md`, `../../_shared/done.md`
- Every run: {`references/{file}.md`, if this stage has know-how notes}

Do not load: {what an eager agent would pull in by mistake, such as `../../_archive/` or another stage's references}.

## Steps

1. {Read the inputs.}
2. {The method, in order. Point to `rules.md` and `done.md`; never restate them.}
3. {The shape of the output. In the last work stage, this step is: Write the result in the shape `done.md` sets.}

## Output

- `output/{file}.md`

## A person checks

{One act a person does, such as reading each verdict next to its draft.}
