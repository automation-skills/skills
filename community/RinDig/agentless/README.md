# Agentless

A friendly reply to [Agentlas OS](https://github.com/agentlas-ai/Agentlas-OS).

Agentlas turns a sentence into an agent or a team of agents. It is a serious build. At commit `704ce1e` (Oct 20, 2026) the repo holds 2,058 files, 339,886 lines of its own Python (113,972 once you drop the copies), 88 JSON schemas, and a package contract that requires 28 files for a single agent and 31 for a team in its standard profile. (I read a good chunk of it. The changelog alone is nearly 200 KB.)

Their README has this line:

> An agent package is not a worker with capabilities. It is a **method document**.

I agree, so I took it at its word. A method document is a folder of markdown, and a model that can read can walk a folder. That is ICM™ (Interpretable Context Methodology™): numbered folders hold the order, each folder's `CONTEXT.md` holds its one job, and the files on disk hold the state.

Agentless is one Claude Code skill. Tell it the job, answer a few questions, and it writes the folder. The folder is the agent.

## Install

```bash
git clone https://github.com/RinDig/agentless ~/.claude/skills/agentless
```

That's the installer. It writes one folder. (For the Claude apps, zip a folder named `agentless` holding these files and upload it at [claude.ai/customize/skills](https://claude.ai/customize/skills).)

## Use

```
/agentless an agent that reviews App Store replies before they go out
/agentless session
```

It asks two to four questions, then writes the folder. The first line is the example from the Agentlas README, and the folder it built is in [`examples/app-store-reply-review`](examples/app-store-reply-review), with one finished run inside so you can read every step. (The app, the reviews and the drafts are made up.) The second line turns the work you just did in a chat into a folder.

## What you get

```
app-store-reply-review/
├─ CLAUDE.md                  the map: where things live, what to do next
├─ AGENTS.md                  one line pointing at CLAUDE.md, for tools that read AGENTS.md
├─ CONTEXT.md                 the stages on one screen
├─ input/drafts.md            this week's drafts
├─ _shared/
│  ├─ rules.md                house rules and Apple's rules, plus lessons a person chose to keep
│  └─ done.md                 what the finished result holds; the check tests against it
└─ stages/
   ├─ 01_review/
   │  ├─ CONTEXT.md           the job, its inputs, the one thing a person checks
   │  ├─ references/voice.md  how we sound, with a good and a bad reply
   │  └─ output/verdicts.md   send, edit or hold, with each fixed reply ready to copy
   └─ 02_check/
      ├─ CONTEXT.md
      └─ output/check.md      19 passed, 0 failed, SHIP
```

Put this week's drafts in `input/`, open the folder in Claude Code and say "new run". It works one stage at a time and stops where you would stop to look. The last stage checks the result against `done.md` in a fresh context, so the context that wrote the work never grades it, and you read the result and the verdict together. Edit any output you like; the next stage reads whatever is there. Ask for status and it lists what is in the `output/` folders.

## All of it

```
agentless/
├─ SKILL.md          the method: the questions, the shapes, the walk test
├─ templates/        the seven files every folder is copied from
└─ examples/         the folder it built for the App Store example, after one run
```

`SKILL.md` and the templates come to 225 lines of markdown. No Python, no JSON schemas, no installer. The model reading the folder is the runtime.

Before publishing, four fresh agents with no other context built the App Store example from `SKILL.md` alone, and each round's failures went back into the templates. Then one fresh agent ran a week of drafts through the folder, and another, in a new session, ran the check.

## Side by side

| Agentlas OS | Agentless |
|---|---|
| `/hep-build`: a first batch of 8 to 12 questions, then follow-ups until a weighted ambiguity score falls to 0.2 | `/agentless`: one batch of 2 to 4 questions about how one run goes |
| task split into roles with handoffs | numbered stage folders; 01's output is 02's input |
| a team whose review loops back is rejected before it runs | stages run in number order; the check can send work back a stage, but only after it stops and a person reads the verdict |
| an independent verifier, plus receipts | a check stage in a fresh context; the files in `output/` are the receipts |
| Memory Events, Memory Tickets, a Memory Curator and a PM Soul | a "Proposed" list in `rules.md`; a person keeps or cuts each line |
| `/hep-build session` | `/agentless session` |
| adapters for 11 agent hosts | `AGENTS.md`, one line, for the tools that read it |
| Agentlas Hub, Agent Cloud | a public repo, a private repo |
| `/hep-upload`, `/hep-call` | `git push`, open the folder |
| `hephaestus contract verify`: 28 to 31 required files, checked for placeholders, valid JSON, schema and length | the walk test: a fresh agent must find where a run starts from `CLAUDE.md` plus at most two more files |
| `curl ... \| bash`, writes into every host it finds | `git clone`, writes one folder |

Fair credit: the question step borrows from their builder gate (say back what the request settles, ask only about the rest, mark the defaults you chose, ask for the names of secrets and never their values), and `/agentless session` follows their session route.

## Same lessons, fewer files

Agentlas measured its way to the things ICM™ starts from, and published the numbers, which I respect. From their own docs:

- A 54-word instruction that points at rule files scored 7.00 of 7 on two models. 652 words scored 6.33 and 5.00. 4,115 words, with the rule sheets pasted in, scored 5.67 and 6.00. (They call the sample small and note that one author wrote both the rules and the checks. Fair.) ICM™ points at the rule files instead of pasting them in.
- "0 of 32 live teams carried the canonical body, so every copy was a divergent rewrite." Inside a folder, ICM™ gives each fact one home and links to it.
- A `capabilities` field, auto-filled from each agent's Responsibilities section in 130 of 130 packages, was "never once informative." Agentless fills each slot with real content or cuts it.

## Where Agentlas wins

A folder won't find you an agent you have never heard of. To staff a job from hundreds of strangers' agents, with signed identity and pinned versions, you need a registry and a protocol, and Agentlas built one (with no ranking on purpose, which is a good call). Folders also lose when agents must talk in real time, when many users share one pipeline, or when the system must pick its own path mid-run. The ICM™ paper says so too.

For your own work, the job you do every week, a folder is enough. You can read every step, edit any output, and see the state of a run by opening it.

## How I counted

<details>
<summary>Commands, so you can check me</summary>

Pinned to commit `704ce1e`. The Python counts leave out the jsonschema, attrs and pyrsistent libraries they bundle (44,175 more lines).

```bash
git init Agentlas-OS && cd Agentlas-OS
git fetch --depth 1 https://github.com/agentlas-ai/Agentlas-OS 704ce1ead2a5bac712cf835b290bde12053a5db1
git checkout -q FETCH_HEAD
git ls-files | wc -l                                                    # 2058
git ls-files -z '*.py' ':!*/_vendor/*' | xargs -0 cat | wc -l           # 339886
git ls-files -z '*.py' ':!*/_vendor/*' | xargs -0 sha256sum \
  | sort -k1,1 -u | awk '{print $2}' | xargs cat | wc -l                # 113972, unique by content
ls schemas/*.schema.json | wc -l                                        # 88
python3 -c "import json; a=json.load(open('package-contract.json'))['artifacts']; \
  print(sum(x['required'] and 'single' in x['modes'] for x in a), sum(x['required'] and 'team' in x['modes'] for x in a))"   # 28 31
```

</details>

## The method

- Paper: [Interpretable Context Methodology™: Folder Structure as Agent Architecture](https://arxiv.org/abs/2603.16021), Van Clief and McDermott
- The full builder, with six workspace shapes and a restructure mode: [icm-architect](https://github.com/RinDig/icm-architect)
- Paper repo: [Interpretable Context Methodology™ on GitHub](https://github.com/RinDig/Interpretable-Context-Methodology)

MIT licensed. Made by Jake Van Clief at [Eduba](https://eduba.io).
