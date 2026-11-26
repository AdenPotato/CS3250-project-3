---
name: backlog
description: Sprint 0 - estimate every user story in story points, compute priority scores, rank the backlog, set team capacity, and split the stories across the sprints respecting dependencies. Fills the README Product Backlog and Sprint Planning tables. Use before Sprint 1 starts, or when the sprint plan has to change.
---

# /backlog

Fill the **Product Backlog** and **Sprint Planning** tables in [README.md](../../../README.md). They feed the **+10 user stories** line of the rubric, and every sprint is planned from them.

The team estimates; this skill runs the method and checks the rules. **Never invent an estimate.** Story points come from the team (planning poker or similar).

## Step 1 - The team

If the README's Team table still has `<name>` placeholders, ask for each member's name, GitHub login and role, and who is Scrum Master. Fill it first - the same table is used by `/new-issue`.

## Step 2 - Story points

Ask for each story's points, one story at a time, US1 to US11, on the Fibonacci scale (1, 2, 3, 5, 8, 13). If the team wants help, give a short note on what drives each story's size - US5 has five criteria and a full create/edit/delete form, US7 carries the sprint state rules (one active sprint, unfinished issues return) - but the number is theirs.

The User value column is already set by the Product Owner's priorities in the README. Change it only if a teammate says so.

## Step 3 - Priority score and rank

```
Priority score = User value / Story points
```

Two decimals. Rank highest score first; break ties by higher user value, then by fewer dependencies. Fill the table in place, keeping its columns.

## Step 4 - Capacity

Ask for the team's capacity in story points per sprint and a one or two sentence justification (team size, hours available, course load). Fill the code block.

Sanity check: the required stories (US1-US7) must fit in the planned sprints (2 or 3) x capacity. If they do not, say so now - either the estimates or the capacity is wrong, and every required story must be planned. Each required story not completed is **-5**.

## Step 5 - Split across sprints

Place stories in this order of rules, and show the reasoning:

1. **Required first.** US1-US7 all go in the planned sprints.
2. **Dependencies.** No story depends on another - each depends only on the Sprint 1 foundation (README, "Designed to be independent"). So Sprint 1 starts with the foundation task, and its capacity must leave room for it. If the team adds a dependency later, a story goes in the same sprint as its dependency only if the plan lists the dependency earlier in that sprint; otherwise a later sprint.
3. **Capacity.** No sprint exceeds it.
4. **Priority.** Within those limits, higher-ranked stories go earlier.
5. **Optional stories** only in space left over.

Fill the sprint table: in Sprint 1, the foundation first (as a note, not a story), then the stories in order, points planned, points remaining after the sprint, and `Y` in Dependencies checked only after checking every story against rule 2. Fill the Total row.

## Step 6 - Planned burndown

List the story points remaining at the start (sum of all planned stories) and after each sprint. Put them in `burndown.xlsx` as the ideal/planned line.

## Step 7 - Confirm, then write

Show both tables exactly as they will appear and write nothing until a teammate confirms. Then fill the README tables in place, keeping their format.

Propose (do not create) one GitHub milestone per sprint (`Sprint 1`, `Sprint 2`, `Sprint 3`) and one issue per Sprint 1 task, for `/new-issue`.
