---
title: Discovery sources
description: Where automations leave traces, what to look for in each place, the questions to ask teams, and the signs of an automation nobody registered.
---

Work through the sources in this order. The first ones find what exists whether or not anyone remembers it. The interviews come last, to explain what you found and to catch the rest.

## 1. Granted access in core systems

For each system that holds customer, financial, employee or product data, open its list of connected apps, granted tokens and integration users. Record for each:

- the app or integration name and who granted it
- the scopes it holds (read, write, admin)
- when it was granted and when it was last used
- whether the person who granted it still works here

An app with write scope, granted by someone who has left, used in the last week, is an automation running on a departed person's authority. Put it at the top of the list.

## 2. Audit logs by non-human actors

In each core system, filter the audit log to the last 30 days and group actions by actor. Look for:

- actors named like "integration", "api", "sync", "bot", "service" or a shared mailbox
- a person's account acting at the same minute every hour, or through the night
- bursts of identical changes across many records within seconds
- records created with a source field that points to an outside app

## 3. Automation platform accounts

- Every workspace of every automation platform the company pays for, including the ones a single team bought.
- Personal and free-tier accounts opened with a company email address. The platforms' admin or domain views often list them, and expense reports show the paid ones.
- For each automation: on or off, last run, run count, the connections it uses and whose they are.

## 4. Configured outbound integrations

Inside each system, open the webhook and outbound integration settings. Every registered destination is something that receives company data. Record the destination, what is sent, and who set it up. A destination nobody can name is a security finding, not just an inventory line.

## 5. Code and scripts

- Scheduled jobs on servers and in cloud schedulers.
- Repositories with scripts that call internal or external systems, especially ones with no recent commits but recent runs.
- Scripts and macros attached to spreadsheets and documents.
- Scripts on shared drives and on the laptops of known power users, found by asking, not by searching their machines.

## 6. Mail and calendar rules

Forwarding rules, auto-replies and filters that move or send mail, in shared mailboxes first. A rule forwarding a shared mailbox to an outside address is an exfiltration risk and goes straight to the security owner.

## 7. Spend

Card transactions and supplier invoices for automation, integration, scraping, data enrichment and AI tools. A small monthly charge on one person's card is often the only trace of a team's automations.

## 8. Team interviews

Ask each team lead, and one person who does the daily work:

- What happens in your work that nobody clicks?
- What would break if a specific colleague were away for a month?
- Which reports, emails or records appear on their own?
- What did someone build to save time that everyone now relies on?
- Have you ever had to ask someone to "turn it back on"?

Show them what you already found for their team. People recognize faster than they recall.

## Signs of an unregistered automation

- A field that fills in by itself, always at the same time.
- Emails sent from a person's address at exact intervals, or while they are on leave.
- A spreadsheet column nobody types into that stays up to date.
- Duplicate records that appear in pairs, seconds apart: two automations doing the same job.
- A status that flips back and forth on the same record within a minute: two automations disagreeing.
