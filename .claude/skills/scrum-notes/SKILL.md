---
name: scrum-notes
description: Record a Scrum event - sprint planning, daily scrum, sprint review or retrospective - in the right scrum/sprint_#/ file, with its real date, participants and content. Use right after the event, from notes a teammate gives you.
argument-hint: "[planning|daily|review|retro] [sprint number]"
---

# /scrum-notes

Every Scrum event must leave a record in `scrum/sprint_#/`, using the provided templates - that is **+10 Scrum notes** in the rubric, and the sprint summaries in the final progress report are built from them. This skill writes one record.

**The rule that matters: never invent the record.** Dates, participants and what was said come from a teammate. If the notes are thin, ask; if they stay thin, the record stays thin (AGENTS.md rule 10).

## Step 1 - Which event, which sprint

From `$ARGUMENTS`, or ask. The file:

| Event | File |
|---|---|
| planning | `scrum/sprint_#/planning.md` |
| daily | `scrum/sprint_#/daily_scrums.md` |
| review | `scrum/sprint_#/review.md` |
| retro | `scrum/sprint_#/retrospective.md` |

Read the file first. Fill the template's fields (`Date/Time:`, `Participants:`, `Goal Statement:` and so on) and keep its headings. Do not restructure the template.

## Step 2 - Ask for what the record needs

Ask for the date and time of the event (not today, unless it was today) and who attended. Then:

**Planning**
- The sprint goal, one sentence.
- The stories pulled in, from the sprint plan in the README. If they differ from the plan, why.
- The **task breakdown**: each story split into tasks, each with an owner and an estimate. Each task becomes a GitHub issue titled `US#: ...` (`/new-issue`).
- Capacity this sprint, if it changed.

**Daily** - one entry per scrum, appended at the end. Per person: done since last scrum, doing next, blockers. In person or async (Discord, Teams) - say which.

**Review**
- What was demoed, against which acceptance criteria, and whether each planned story is done (every criterion passing in Compose) or not.
- Unfinished stories go back to the backlog - say where they move.
- Points completed - the **actual** line in the burndown.
- Product Owner feedback, if the instructor gave any.

**Retro**
- **From Sprint 2 on, open with the previous sprint's actions** (read the previous `retrospective.md`): each one done, partly, or not, and why.
- What went well, what did not.
- Actions for next sprint: concrete, each with an owner.
- Lessons learned worth carrying into the final progress report.

## Step 3 - Preview, then write

Show the record exactly as it will appear. Write nothing until it is confirmed.

Format dates as `MM/DD/YY-HH:MM`. Daily entries:

```markdown
# 12/02/26-09:00

Participants: <names>   (async, Discord)

- <name> - done: project model and repository (#8). Next: project service (#9). Blockers: none.
- <name> - done: ... Next: ... Blockers: ...
```

## Step 4 - Follow-ups

- Planning: propose the issues for each task, and remind the team to screenshot the board at the end of the sprint.
- Review: update the burndown's actual line, or remind the Scrum Master to.
- Retro: if this closes the sprint, point to [`/sprint-close`](../sprint-close/SKILL.md).
