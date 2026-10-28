---
name: "architect-implementer-workflow"
description: "Use when starting or running a software project where Claude chat is the architect and Claude Code the implementer: Linear tickets, GitHub PRs, the review loop, CLAUDE.md, and document updates."
---

# Architect / implementer workflow

A way of building software with one human, Claude chat as the architect, and Claude Code (CC) as the implementer. Linear is the interface between the two Claudes; the human never copies text between them.

## Roles

| Who | Does |
|---|---|
| Claude chat (you) | Plans, writes tickets, reviews the result, proposes document updates. Writes no product code. |
| Claude Code | Reads a ticket from Linear, writes code, opens a PR, posts the summary as a Linear comment. |
| The human | Approves every merge, does what only a person can (accounts, secrets, hardware), decides open questions. |

## One-time setup (tell the human what to do; check what is already there before asking)

- Connect Claude chat to Linear and GitHub (Connectors). For GitHub, the Claude app must be **installed** on the account with the repo selected; authorization alone gives no repo access.
- Connect CC to Linear: `claude mcp add --transport http linear https://mcp.linear.app/mcp`, then `/mcp` in the first session to log in.
- Connect Linear to GitHub (Linear Settings → Integrations → GitHub): a PR whose branch name contains the ticket id links to it, and merging closes it.
- Protect `main`: PRs only, required status checks, "do not allow bypassing". No required approvals when the human works alone (GitHub cannot approve your own PR). Branch protection on a private repo needs GitHub Pro.
- Default merge: **squash and merge** (repo Settings → Pull Requests, disable merge commits), so one ticket = one commit on `main`.
- CI: one GitHub Actions workflow per top-level folder, running only when that folder (or a shared contracts folder) changed, plus one always-running job per workflow named `<folder>-ok` that branch protection requires. Pre-commit hooks run lint, format and type checks locally.

## Linear structure

- One initiative per product, with a target date.
- One project per deliverable of a few days to two weeks. Not one per small feature, not one for the whole product.
- One ticket per task of hours to two days; parent ticket with sub-issues for bigger items.
- Labels: a single-select Component group matching repo folders, plus Feature / Bug / Improvement.
- Dependencies (blocked by) so CC works in order.
- Statuses: Todo → In Progress (CC sets when starting) → In Review (CC sets when PR is open) → Done (automatic on merge).

## Ticket format

Write every ticket in plain English a teenager could follow. Show it to the human for approval before creating it in Linear; after approval, create it yourself.

```
> **For <human> (Claude Code: skip this block):** what the human must do before, and after.
> Suggested model: <Sonnet | Opus>, effort <low | medium | high>. Estimate: <time>. Blocked by <ticket>.

## Goal
One paragraph: what exists when the ticket is done.

## Steps
Numbered, concrete, with file names and commands.

## Done when
Checks that can be run or seen, never "works well".

## Not in this ticket
What is explicitly out, so CC does not drift.
```

Model choice: Sonnet for setup, tooling and code derived from an existing design; Opus with effort high for contracts, schemas, core logic and anything expensive to change later. Rules that repeat across tickets belong in CLAUDE.md, not in tickets.

## CLAUDE.md (root of the repo, plus a short one per main folder)

The standing instructions for CC. Claude chat writes it, the human commits it. It must contain:

- What the product is (one paragraph) and the repo layout.
- **Hard rules**: numbered, short, ending with "If a task seems to require breaking one of these, stop and say so."
- **Comment policy**: all code and comments in English; a self-contained header in every file (what, how it fits, why) with no references to design documents or decision numbers; a docstring for every public function; "why" comments wherever code is non-obvious; values likely to change marked `TUNABLE:` with unit and whether measured or guessed.
- **Tooling** per language: package manager, linter, formatter, type checker, test runner; pre-commit; CI scope rule.
- **How we work**: one ticket = one branch = one PR, branch name copied from Linear; `git checkout main && git pull` before every ticket; `main` is protected; never commit secrets; do not touch hardware unless the ticket says so; the ticket's model/effort line is to be followed, and CC says so if it is running on something else.
- **How to write to the human**: see below.

## How CC reports

- When the PR is open and checks are green, CC posts the **full summary as a comment on the Linear ticket** (same content as the PR description) and moves the ticket to In Review.
- The terminal summary is three lines: PR link, "Needs you:" line, link to the Linear comment.
- Every summary and comment starts with **"Needs you:"** listing what only the human can do, or "Needs you: nothing".
- Then: Result (one line), What changed, Not done, Choices you should know about (pinned versions, anything installed on the human's machine, deviations), Check (what the human should look at).
- Short, plain words, no jargon not already in CLAUDE.md.

## The ticket loop

1. Human: `/clear` in CC, `/model` per the ticket, then "read <TICKET> in Linear and do it".
2. CC works, opens the PR, posts the Linear comment.
3. Human: "done, check".
4. You: read the Linear comment, the PR, its checks, the file list and the diff yourself. Depth by risk: summary plus green checks for ordinary code; every line for contracts, schemas, security, tenant isolation and anything the whole system builds on. Reply with a short table (what you checked, result) and either "merge" or what to fix. If you find a mistake in your own ticket, fix the ticket before CC hits it.
5. A fix inside the same ticket: no `/clear`, CC must remember what it did.
6. Human merges with squash; Linear closes the ticket.
7. You propose the next ticket, and at the end of the day a row for the working-state document.

## Documents

- A **working-state** document with a "recent changes" table at the top (date, what happened, what it changed) and a "last updated" date. Most updates are one row.
- A **product** document (what it is), a **code architecture** document (how it is built, with a version number on its first line), and, when the product has rules or logic with edge cases, a **catalog** mapping every edge case to a named case in a regression set kept in the repo.
- Protocol: never write to the project before the human approves. Propose like a PR: which document, which section, existing text against proposed text, only what changes. A correction means propose again, not write.

## Style with the human

Match the human's language. If they write in Hebrew: English technical terms are fine mid-sentence, but no paragraph, bullet, table cell, bold lead-in or heading may begin with an English word (it flips the line to left-to-right); reword so a Hebrew word comes first. No ASCII diagrams; tables instead. Mark every unmeasured number as an estimate or guess. Keep replies short; lead with the result.