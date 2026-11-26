---
name: qa-steps
description: Write a numbered QA script for a branch - every step with an exact action, an exact expected result, and a blank for the result, keyed to the story's acceptance criteria. Use when handing a PR to the reviewer, or when a QA round found problems and the next round has to be precise about what to re-check.
---

# /qa-steps

Turn "ready to review" into a script a teammate can run without designing the test themselves.

**Why this exists.** Every PR needs approval from someone other than its author, and the reviewer's job is to pull the branch and **run** it. "Have a click around" produces "looks fine". A numbered script produces "4b failed, the unfinished issue stayed in the completed sprint" - which is a defect report and a reproduction at once.

## When to run it

- A PR is open and about to be reviewed.
- A QA round found problems and the next round has to state exactly what to re-check.

Skip it for a change with no observable surface: a scrum record, a diagram, a README edit.

## Step 1 - Preconditions

**Before writing a single step, say what the tester will be looking at and confirm it is this branch's code.**

```bash
git fetch && git switch <branch> && git pull
docker compose down -v               # fresh seeded database - ask the tester first, it wipes local data
docker compose up --build            # http://localhost:5000
pytest                               # the automated tests pass before anyone clicks
```

Then name, concretely:

- **The accounts** - the seed accounts from the README, plus whichever extra ones the access steps need: a project **owner**, a **member** who is not the owner, and a **non-member**. Give the usernames and passwords to create.
- **The data** - which seeded project, issues and sprint to use, or what to create first.

## Step 2 - Derive the steps from the criteria and the diff

In this order:

1. **Each acceptance criterion of the story**, in its numbered order - so step numbers line up with criteria where possible.
2. **The failures the criteria name** - duplicate project key, empty title, invalid story points, a second active sprint.
3. **The access surface** - signed out, member but not owner, non-member with a project id in the URL.
4. **What must not have changed** - the previous stories this one builds on (a sprint change can break the backlog list from US5).

## Step 3 - Write the steps

**Numbering is the contract.** Top-level steps are numbers; sub-steps are letters, so "4c failed" means the same thing to both people.

Every step has three parts:

- **Do** - one concrete action.
- **Expect** - the exact observable result. Not "the issue moves" but "`WEB-3` is in the In Progress column after a refresh".
- **Result** - blank, for the tester.

Rules: one assertion per sub-step; exact values and exact message text; say what the near-miss failure looks like; never write a step whose result you already know.

## Step 4 - Render it

This goes in the PR description or a comment, not a file.

```markdown
## QA script - <branch> (US#)

**Preconditions**
- Branch: `<branch>`, pulled; `pytest` passes
- `docker compose down -v && docker compose up --build` -> http://localhost:5000
- Accounts: owner `<user>` / `<pw>`, member `<user>` / `<pw>`, non-member `<user>` / `<pw>`
- Roughly: <minutes>

### 1. US#.1 - <criterion short name>
**Do:** <action>
**Expect:** <exact observable>
**Result:**

### 2. US#.2 - <criterion>
**Do:** <action>
- **2a. Expect:** <one observable>
  **Result:**
- **2b. Expect:** <another observable>
  **Result:**

### 5. Regression - <what must not have changed>
...

**Report back as:** the step number, what you saw, and a screenshot for anything visual.
```

## Step 5 - Close the loop

1. **A failed step becomes a test.** Before the fix, write the pytest test that reproduces it (`/testing`), so it cannot come back.
2. **Re-issue only the affected steps**, plus any regression step the fix could touch.
3. **The reviewer approves the PR only when every step passed.**
