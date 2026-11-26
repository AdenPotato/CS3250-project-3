---
name: project-status
description: Report where the project stands - the current sprint and its goal, each user story's status against its acceptance criteria, the rubric with points earned and at risk, penalties, missing Scrum evidence, and what to do next. Use at the start of a session or before a sprint review.
---

# /project-status

Read the state, report it, recommend the next move. **Read-only** - this skill changes nothing.

## Gather

1. **Sprint** - which sprint is current (the highest-numbered `scrum/sprint_#/` with a filled `planning.md` and an empty `review.md`; none filled means Sprint 0). Its goal and planned stories.
2. **Stories** - for the Sprint 1 foundation and each of US1-US11, from the code and the evidence, not from memory:
   - Is there code for it in all three layers? (`grep -rn` in `src/app/routes`, `services`, `repositories`.)
   - Is it merged into `dev`? (`git log dev --oneline --grep US#` or the linked issues.)
   - Does each acceptance criterion `US#.n` have a passing test? (`grep -rn "us<n>_" tests/`, then `pytest -q`.)
3. **README documentation** - mission statement, use case diagram (rendered image present), user stories with criteria, backlog table filled (points, priority score, rank), sprint plan filled, sequence diagram (not a registration or login story), class diagram, testing section naming the white-box and black-box tests, coverage figure, deployment instructions, burndown image.
4. **Scrum evidence** - for every sprint so far: `planning.md`, `daily_scrums.md`, `review.md`, `retrospective.md` filled and dated, and `board.png` present and not the template.
5. **Testing** - a test clearly labelled white-box and one clearly labelled black-box, neither on registration or login; `htmlcov/index.html` committed and recent.
6. **Code quality** - `flake8 src tests` result; source files missing the header comment (`grep -L "Authors:" $(git ls-files 'src/*.py' 'tests/*.py')`).
7. **Deployment** - `Dockerfile`, `docker-compose.yml` (two or more services), `requirements.txt` frozen, `.env.example`; `.env` and `.venv` absent from `git ls-files`. Three layers present for the bonus.
8. **Git** - `git branch -a` (`main` and `dev` both exist), `git tag --list`, main branch protection (`gh api repos/{owner}/{repo}/branches/main/protection` or rulesets), and `git log dev --format='%an %ad' --date=short` and the same for `main` - continuous development on both, and every member on `dev`.

## Report

Short. Six sections, no preamble:

**Sprint** - which one, its goal, days left.

**Stories** - one row per story planned so far: `US# | status | criteria passing N/M | owner`. Status is `not started`, `in progress`, `in review`, or `done` - and `done` means every criterion passes in Compose with tests.

**Rubric** - a compact table of every line in [ASSIGNMENT.md](../../../ASSIGNMENT.md#rubric) not yet earned, with points. Then one line: `earned N / at risk M / not started K` out of 100, plus bonus.

**Penalties** - check each one every time: -10 registration not working, -10 login not working, -5 per required story not completed, -5 no continuous development on `main`, -5 on `dev`, -5 no burndown chart. Near the end, the self/team evaluation (**+15**, grade held until everyone submits).

**Drift** - anywhere the plan, the board, the code and the evidence disagree. A story marked done with no tests. A class diagram that no longer matches `models.py`. A sprint with no `board.png`. A stale `htmlcov/`. Name the file.

**Next** - two or three concrete actions, highest points-per-hour first. Prefer work that unblocks others: the foundation before everything, the diagrams by the end of Sprint 1, the Compose setup before the first release.

## Judgement

- "The code is written" is `in progress`. A story is `done` only when it passes in Compose and its tests exist.
- Scrum notes cannot be reconstructed honestly after the fact. Missing daily scrums for past days are a loss to report, not a gap to back-fill.
- If the clock is short, say which stories are realistically reachable this sprint and which will slip. A team that knows it will miss a story can tell the Product Owner at the review.
