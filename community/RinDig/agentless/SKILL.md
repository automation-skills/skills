---
name: agentless
description: Build an agent or an agent team as a folder of markdown files that Claude walks one stage at a time, with no agent runtime. Use when the user types /agentless, asks to build, spin up or make an agent, assistant, specialist or team for a recurring job, or asks to turn this session into an agent. Asks two to four questions, then writes numbered stage folders with contracts, shared rules and a check stage.
---

# Agentless

Build a specialist or a team for a job someone does again and again. What you hand back is a folder, and the folder is the agent: any model that reads markdown walks it one stage at a time, and a person reads the output at each stop.

Method: ICM™ (Interpretable Context Methodology™), Van Clief and McDermott, arXiv:2603.16021. More shapes and a restructure mode live in icm-architect (github.com/RinDig/icm-architect).

## Routes

The text after `/agentless` picks the route.

- A sentence about a job: Build.
- `session`: turn the work done in this conversation into a folder. See Session.
- Nothing: ask for the job in one sentence, then Build.

## Build

### 1. Say back what the sentence settles

Restate the job in one line. Note what the sentence already answers (the job, the input, maybe the output). Ask only about the rest.

### 2. Ask one batch of two to four questions

Pick the questions the sentence leaves open:

| Ask | The answer becomes |
|---|---|
| Walk me through one run. Where do you stop and look before going on? | each stop ends a stage; what they look at becomes rules the check tests |
| What stays the same every time, and what changes each run? | stable rules go in `_shared/rules.md`; know-how and examples go in the `references/` of the stage that uses them; this run's things go in `input/` |
| What leaves the folder at the end, and how do you know it's good? | the lines in `_shared/done.md` |
| Does any step need a different expert, tool or source? | that step gets its own stage, or a note in a stage's `references/` |

Use the AskUserQuestion tool if you have it. Its options are your best guesses at the answer, in the words of the person's field; they can always type their own. Without it, ask in a short numbered list and offer the same guesses. Ask for the names of secrets, never their values. Aim for one batch, and ask a second only when an answer opens a question that would change the folder.

"You decide" is a fine answer. Pick a sane default and write it where it belongs. Start each line that holds a default you chose (a rule, a limit, a step) with `(assumed)`, and mark an example you made up once, at its heading. Names of stages and files need no mark. When the person confirms or replaces a default, drop the mark.

### 3. Pick the shape

| Shape | When | What to write |
|---|---|---|
| Saved prompt | it runs now and then, nothing needs keeping between runs, and the output needs no second look | no folder; say so, offer a saved prompt or skill, and stop |
| Specialist | it repeats, has rules or examples worth keeping, and one expert can do it in one sitting | one work stage, then a check stage |
| Team | a person stops to look before the end, or steps need different expertise | one stage per stop (most jobs need two to four), then a check stage |

The check stage runs right after the last work stage, so a person reads the result and the verdict together at one stop. Split work stages where a person pauses, and nowhere else. If the unit is a record that grows (a client, a person) or a body of knowledge, use icm-architect. If the job needs agents talking in real time, many users on one pipeline, or the system picking its own path mid-run, tell the person a folder is the wrong tool.

### 4. Write the folder

Make a new folder named for the job (kebab-case) in the current directory, unless the person names another place. Never write into a folder that already holds files. Copy from this skill's `templates/` folder (`${CLAUDE_SKILL_DIR}/templates/` in Claude Code). Fill each `{...}` with real content or cut it; plain text stays.

```
{job-slug}/
├─ CLAUDE.md          templates/CLAUDE.md: the map and routing table
├─ AGENTS.md          templates/AGENTS.md: one line, for tools that read AGENTS.md (Codex, Cursor and others)
├─ CONTEXT.md         templates/CONTEXT.md: the stages on one screen
├─ input/             empty; this run's material goes here
├─ _shared/
│  ├─ rules.md        templates/rules.md
│  └─ done.md         templates/done.md
└─ stages/
   ├─ 01_{stage}/     CONTEXT.md from templates/stage-CONTEXT.md, an empty output/, references/ if needed
   ├─ 02_{stage}/     Team only: the same shape
   └─ {NN}_check/     CONTEXT.md from templates/check-CONTEXT.md, an empty output/
```

The first closed run makes `_archive/`; don't make it now. Name each stage folder with two digits, an underscore and its job in one or two kebab-case words: `01_review`, `02_write-draft`, `03_check`.

While filling, give each thing one home:

- `_shared/rules.md`: every rule the result must follow. The person's rules in their words; outside rules (a platform's, a law's) quoted, with the source and the date you checked it. A length cap is a rule.
- `_shared/done.md`: what the finished result holds and how it is laid out, as lines that pass or fail.
- `references/` in a stage: know-how and examples only that stage uses and the check never tests. Write them the way an expert briefs a new hire: short notes, plus a good and a bad example. If the person has none, make up one of each.
- Stage steps: the method only. They point to `rules.md` and `done.md` and never restate them. If the check must test something, it belongs in `_shared/`.

When a fact can go stale (a price, a law, a platform rule), look it up before you write it. Keep CLAUDE.md under 60 lines and each contract under about 40.

### 5. Walk it cold

Give the folder to a subagent that has not seen this chat. If you can't, reread it yourself, starting at CLAUDE.md and opening only the paths the files name, and answer each line below from the files alone. It passes when:

- From CLAUDE.md plus at most two more files, it can say where a new run starts and what to do when the check says FIX.
- Every stage contract names its inputs, its one job, its output file and one check a person does.
- The check can test everything `done.md` asks for without opening a work stage's contract or references.
- Status can be read by listing the files in `stages/*/output/`.
- No rule or instruction appears in two files. Names, paths and examples may repeat.
- CLAUDE.md, one stage contract and that stage's every-run files stay under about 4,000 tokens, which leaves room for this run's material.

When a line fails, fix the folder (move, split or cut files) until the walk works.

### 6. Hand it over

Show the tree and the file count. Tell the person how to run it: open the folder in Claude Code (or any agent that reads AGENTS.md), drop this run's material in `input/`, and say "new run". When a result is good, they say the run is done, and it moves to `_archive/`. Offer to do the first run now.

## Session

1. Read this conversation. In chat, write a short report: the job we did, the steps in order, where the person stopped to check or correct, what they corrected, and which files came out good.
2. Ask the person to fix the report, and where the folder should go (default: a new folder in the current directory).
3. Build from the report with steps 3 to 6. Steps become stages. Each correction becomes a line in `_shared/rules.md`. Good outputs become examples in the `references/` of the stage that made them.
4. Leave out secrets, private paths, personal data and the raw transcript.
