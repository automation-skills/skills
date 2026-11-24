# Agent tools

## Naming
- verb_noun, stable, unique
- No overlapping tools with ambiguous boundaries

## Description checklist
- What it does / does not do
- Required inputs + types
- Side effects
- Error modes the model should expect

## Workflow-as-tool
- Trigger: Execute Workflow Trigger with **Define Below** inputs when used as tool (clear schema)
- Return a small JSON the model can read
- Tag workflow `tool` + `subworkflow`
