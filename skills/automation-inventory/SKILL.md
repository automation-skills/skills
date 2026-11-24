---
name: automation-inventory
title: Automation inventory
description: |
  Use this skill when a company does not know everything that runs on its behalf, or must decide what to keep: "list all our automations", "what integrations do we have", "find shadow automations", "who owns this workflow", "the builder left, what breaks", "prepare for the audit", "consolidate our automation tools", "clean up old workflows". Produces a register of every automation with its owner, the identity and credentials it runs as, its blast radius and bus factor, and a keep, fix, merge or retire decision for each, approved by the owners before anything is changed.
category: Automation operations
tags: [Operations, Security, Risk]
author: Itay Guttman
authorTitle: CEO
authorCompany: Engini
---

Applies before an audit or a platform consolidation, after an incident caused by an automation nobody knew about, and when a builder leaves. Produces a register that names a person for every automation and a decision for each one.

## The play

1. **Search where automations leave traces, not where people say they are.** The full checklist is in `references/discovery-sources.md`.
   - Every automation platform account and workspace, including personal and free accounts opened with a company email.
   - The connected apps and granted access in each core system: customer records, email, file storage, finance, HR, support. Third-party apps holding tokens are the best detector of shadow automations.
   - API keys, integration users, and audit log entries made by non-human identities.
   - Webhooks and outbound integrations configured inside each system.
   - Scheduled jobs on servers, scripts in code repositories and shared drives, spreadsheet scripts and macros, mail rules that forward or auto-reply.
   - Card and invoice spend on automation and integration subscriptions.
   - Then ask every team: "what happens in your work that nobody clicks?" The list commonly grows by half again after this step.
2. **Record each one in the register:** what it does in one line, trigger, systems read and written, owner, builder, the identity it runs as, last successful run, runs per month, failure rate over 30 days, where its alerts go, and the business process it serves.
3. **Score the blast radius** on four questions: how many records or people it touches per run, whether its writes can be undone, how soon a failure would be noticed, and whether it reaches outside the company (customers, suppliers, money, public content). The rubric is in `references/register-and-decisions.md`.
4. **Find the bus factor.** How many people could explain and fix it today? One is a risk. Zero is an orphan. An automation running under a person's own account is critical twice over: it breaks when they leave, or keeps running with their access after they have gone.
5. **Check the credentials.** Personal tokens, shared passwords, admin scope where read access would do, tokens that never expire, tokens of people who have left. Do not test or use any credential you find. Record where it is and tell the owner of that system.
6. **Decide for each automation:**
   - **Retire** when nothing has run successfully in 90 days, or nobody consumes the output. Confirm with the consumer, not only the owner.
   - **Merge** when two or more act on the same trigger or write the same records. Two automations writing one field is a race, not redundancy.
   - **Fix** when it is valuable but has a bus factor of 1, a personal credential, no alerts, no owner, or a failure rate above 5%.
   - **Keep** when it has an owner, alerts, a service identity, a written description and a clean last 30 days.
7. **Get approval before changing anything.** Present the decisions to each owner, and orphans to the owner of the business process. Retire in two stages: turn it off, announce it, and wait 2 to 4 weeks before deleting, with the definition archived. Map what depends on a credential before revoking it, or you cause the outage you were preventing.
8. **Rank the risks.** Blast radius times ownership risk. The top ten fit on one page for leadership, each with an owner and a date.
9. **Keep it alive.** Register every new automation before it is turned on. Add "automations running as this person" to the leaver checklist. Review the register every quarter: owners still employed, credentials, failure rates, anything unused for 90 days.

## What good looks like

- **The tell:** the best lead starts with token grants and audit logs, because they show what runs whether or not anyone remembers building it.
- **The common mistake:** a spreadsheet filled in from a survey, with "IT" as the owner of everything and no column for credentials. It is out of date within weeks and misses exactly the automations that cause incidents.
- **The quiet miss:** retiring on a scream test alone. Switching off a monthly or quarterly job hears no scream until it is too late. Check the schedule and the consumer first.
- **Good output:** every row names a person, the identity it runs as, a blast radius and a decision, and the top risks are on one page with owners and dates.

## Rules

- MUST name a person as owner of every automation, or mark it orphaned.
- MUST record the identity and credential type each automation runs as.
- MUST get the owner's approval, or the process owner's for an orphan, before turning anything off.
- MUST turn off and announce before deleting, and archive the definition.
- NEVER use, test or copy a credential found during discovery.
- NEVER revoke a credential before mapping what depends on it.
