# {Name}

{One sentence: the job, and the file that leaves this folder when a run is done.}

This folder is the agent. Read the file for the step you are on and nothing else. Built with Agentless on ICM™ (Interpretable Context Methodology™).

## Where things live

| Path | What it holds |
|---|---|
| `CONTEXT.md` | the stages on one screen |
| `input/` | this run's material |
| `stages/` | the work, in order; each stage writes to its own `output/` |
| `_shared/rules.md` | the rules the work follows and the check tests |
| `_shared/done.md` | what the finished result holds and how it is laid out |
| `_archive/` | closed runs (made when the first run closes) |

## Route by what just happened

| If | Do |
|---|---|
| asked for a new run | If any `stages/*/output/` holds files, the last run was never closed: ask which files in `input/` are new, then move the other `input/` files and every output to `_archive/<today's date>/`. If `input/` is empty, ask for this run's material. Then open `stages/01_{stage}/CONTEXT.md`. |
| a person approved the output of a work stage that is not the last | Open the next numbered stage's `CONTEXT.md`. |
| the last work stage is done | Run the check right away in a fresh context: a subagent given only `stages/{NN}_check/CONTEXT.md`. With no subagents, ask the person to open a new session in this folder and say "run the check". |
| asked to run the check, in a session that did none of the work | You are the fresh context. Read `stages/{NN}_check/CONTEXT.md` and follow it. |
| the check ends in SHIP | Point the person to the result in the last work stage's `output/`. They ship it or send it back. |
| the check ends in FIX, or a person sends the result back | Open the stage named, by the check or the person. Read `stages/{NN}_check/output/check.md` and the person's notes with its inputs, even if the stage says not to load the check. Fix only what they name, in the output file as it is now. Then run the check again in a fresh context, as above. |
| the person says the run is done | Close it: move every file in `input/` and `stages/*/output/` to `_archive/<today's date>/` (add `-2` if that folder exists). |
| asked for status | List the files in `stages/*/output/`. If `check.md` exists, give its last line. |
| a person corrected an output | Add one line under "Proposed" in `_shared/rules.md`: what to do next time. The person keeps it or cuts it. |

## Rules for you

- Do one stage, then stop so a person can read its output. After the last work stage, run the check first, then stop.
- A person may edit any output. The next stage reads the file as it is now.
- Change `_shared/` only to add a line under "Proposed", or when a person tells you to.
- Make any folder that is missing.
