---
name: new-issue
description: Open a GitHub issue for one task - titled with its user story, assigned to its owner, in its sprint milestone, and added to the board. Use when a teammate has approved tracking the work, usually at sprint planning.
---

# /new-issue

The assignment asks for **each user story to be represented by one or more GitHub issues**, titled `US#: description` when the issue supports a story, and referenced in the commits and pull requests that complete it. The board built from these issues, moved as work happens, is part of the Repository & Process grade.

**Teammate-gated.** Only run this once someone has asked for the issue or approved it. Never open one unilaterally, not even a follow-on. If you have spotted work that wants an issue, **propose it - title plus one-line scope - and wait for a go.** Claude Code has a mechanical backstop in `.claude/settings.json`; other tools may not, so the gate is on you - ask before any `gh issue create`.

## Before anything

```bash
gh repo view --json nameWithOwner -q .nameWithOwner
```

If that fails, the team repo is not set up yet - say so and stop.

## Step 1 - Draft it

**Title** is `US#: <task, lowercase>` - `US7: sprint service with one-active-sprint rule`. Work that belongs to no story uses a prefix instead: `Deploy:` (Docker, Compose), `Design:` (UML), `Test:` (white-box, black-box, coverage), `Scrum:` (records, burndown), `Fix:`.

**Body** carries four things and nothing else:

```markdown
**Story** - US# and the acceptance criteria this task satisfies, by number (US7.3, US7.4).

**Scope** - what is in, and the obvious neighbouring thing that is out.

**Deliverable** - the concrete artifact that closes this. A service that passes US7.3-7.4 with tests, a diagram that renders, a page that works in Compose.

**How it is verified** - the pytest tests, by name.
```

For a task touching `models.py`, add a **database reset** line.

Present the draft, then ask for: the **owner**, the **sprint** (milestone), and the **estimate** if the team tracks tasks in points or hours. Do not proceed until confirmed.

## Step 2 - Make sure the label and milestone exist

Idempotent; ignore "already exists":

```bash
gh label create US7 --color 1D76DB --description "As a project owner, I want to plan sprints" 2>/dev/null || true
gh api "repos/$(gh repo view --json nameWithOwner -q .nameWithOwner)/milestones" --jq '.[].title'
```

Milestones are `Sprint 1`, `Sprint 2`, `Sprint 3`. If the needed one is missing, propose creating it.

## Step 3 - Create it

```bash
gh issue create \
  --title "<title>" \
  --body "<body>" \
  --label "US#" \
  --assignee <github-login> \
  --milestone "Sprint N" \
  --project "<board name>"
```

Keep the issue URL. The card starts in **To Do**.

## Step 4 - Confirm

```bash
gh issue view <number> --json number,title,labels,assignees,milestone \
  --jq '"#\(.number) \(.title)\nlabels: \([.labels[].name] | join(", "))\nassignee: \([.assignees[].login] | join(", "))\nsprint: \(.milestone.title // "none")"'
```

Show the URL. If the work starts now, the branch is `<number>-short-slug` off `dev` (e.g. `14-sprint-model`), and the PR body says `Closes #<number>` so the issue is linked and closes on merge.

## Team

The assignee is the GitHub login. The team is listed in the README's **Team** table - read it from there; if it still has placeholders, ask for it (and fill it via `/backlog` step 1).

Board: a GitHub Projects board with the columns **To Do, In Progress, In Review, Done**. Add its link to the README's Development Process section once it exists.

## Constraints

- **One task per issue, one story per task.** A story is usually three or four tasks - model and repository, service, route and template, tests - so two people can work on it at once.
- **Every member owns issues every sprint.** `dev` must show work from everyone.
- **Do not open an issue for work already done.**
- **Cards move as work moves**, not at the end of the sprint. The board screenshot is graded on looking used.
