# CS3250 Project 3 - TaskBoard

A small Jira-style project management web app. Users register and log in, create projects, add members, keep stories, bugs and tasks in a backlog, plan them into sprints and move them across a To Do / In Progress / Done board. Built for CS3250 (Software Development Methods and Tools) under **Scrum**: a planning Sprint 0, then two or three one-week sprints, each ending in a tagged release (v1, v2, v3).

**Stack:** Python 3, Flask (Flask-SQLAlchemy, Flask-Login, Flask-WTF), PostgreSQL, Docker Compose with at least two containers (`db`, `web`), and a three-layer architecture - presentation, business logic, data access. The assignment only requires a Python web backend with registration and login deployed with Docker; PostgreSQL, Compose and the three layers are what earn the **+5 bonus**, so treat them as fixed. SQLite is not used.

This file is the entry point for every AI coding agent (Codex, Gemini, Cursor, Copilot, Claude Code and others). `CLAUDE.md` imports it, so edit the rules here, not there.

## Where things are

| Path | What |
|---|---|
| [README.md](README.md) | The project README, built from `README_template.md`: mission statement, use case diagram, user stories with numbered acceptance criteria, product backlog, sprint plan, diagrams, testing, deployment, burndown. **The source of truth for what to build.** |
| [ASSIGNMENT.md](ASSIGNMENT.md) | The instructor's assignment text and **rubric**. The source of truth for what is graded. |
| `src/app/` | The application package (planned layout below). Run from `src/` with `flask --app app run`. |
| `tests/` | pytest tests: the white-box test, the black-box test and the rest. |
| `htmlcov/` | The coverage report. **Committed** - the assignment requires it in the repo. |
| `uml/use-case.wsd` | Use case diagram, embedded in the README. |
| `src/app/static/css/theme.css` | The shadcn-style light/dark design tokens. Every colour in the app comes from these variables. |
| `uml/class.wsd` | Model classes only: User, Project, ProjectMember, Issue, Sprint, Comment. |
| `uml/sequence-US7.wsd` | One story's flow through all three layers to PostgreSQL. Must not be a registration or login story. |
| `scrum/sprint_#/` | `planning.md`, `daily_scrums.md`, `review.md`, `retrospective.md`, `board.png`. |
| `burndown.xlsx`, `burndown.png` | Ideal and actual lines; the PNG is embedded in the README. |
| `Dockerfile`, `docker-compose.yml` | `db` (postgres + named volume) and `web` (waits for `db`). Planned. |
| `requirements.txt` | Frozen dependencies (`pip freeze`). |
| `.env.example` | Placeholder `POSTGRES_*` and `SECRET_KEY` values. `.env` itself is never committed. |
| `.agents/skills/` | Workflow skills, one `SKILL.md` per folder. |
| `.agents/agents/` | Role briefs: `app`, `ui`, `release`, `scrum`. |

### Planned `src/app/` layout (three layers)

| Layer | Path | What lives there |
|---|---|---|
| Presentation | `routes/` (one blueprint per story area: `auth` US1-2, `projects` US3, `members` US4, `issues` US5, `board` US6, `sprints` US7, `comments` US8, `search` US9, `my_work` US10), `forms.py`, `templates/<blueprint>/`, `static/` | Request parsing, form validation, rendering, redirects. No queries, no business rules. |
| Business logic | `services/` (one per story area: `auth_service.py`, `project_service.py`, `member_service.py`, `issue_service.py`, `board_service.py`, `sprint_service.py`, `comment_service.py`, `search_service.py`, `my_work_service.py`) | The rules from the acceptance criteria: who may do what, issue keys, one active sprint, returning unfinished issues. Raises domain exceptions the routes turn into messages. |
| Data access | `models.py`, `repositories/` | SQLAlchemy models and the queries. Nothing above this layer touches `db.session`. |

### The Sprint 1 foundation

Built first, by one or two people, before story work starts; everything else builds on it and on nothing else. It is not a user story and has no points. It contains: the app factory and config, Docker Compose, **all models** (so stories never change the schema), idempotent seed data (users, a project with an owner and a member, issues in every status, a planned sprint), Flask-Login wiring, the shared `require_member` / `require_owner` checks, `base.html` with the navigation and `theme.css`, the dashboard, a stub page for every navigation route, `services/errors.py`, and the test fixtures (`conftest.py` with an app, a test database and logged-in clients for an owner, a member and a non-member).

## Rules that bind every session

1. **The acceptance criteria are the definition of done.** A story is done only when every one of its criteria (numbered `US#.n` in the README) passes in the Docker Compose deployment. Every task maps to a user story (US1-US11), the Sprint 1 foundation, or a rubric line in [ASSIGNMENT.md](ASSIGNMENT.md#rubric). Work that maps to neither is out of scope until a teammate says otherwise.
2. **Required before optional.** US1-US7 are required, and each one not completed is **-5**. US8-US11 are +1 each (bonus capped at 5). No optional story is started while a required one in the current sprint is unfinished.
3. **Keep stories independent.** No story depends on another - only on the Sprint 1 foundation (models, seed data, login wiring, members-only check, base layout and theme, stub pages). A story owns its own blueprint routes, service, repository methods, templates and tests, and never edits another story's. If a story seems to need another story's code or a schema change, that is a design problem: stop and run `/new-design` instead of creating the dependency. A shared need goes into the foundation, agreed with the team.
4. **Every story leaves test evidence.** Each implemented story has pytest tests for its criteria. The suite includes one clearly named **white-box** test and one clearly named **black-box** test, neither on registration or login, and the coverage report in `htmlcov/` is regenerated before each release.
5. **Code quality is graded.** Every source file starts with a header comment giving a brief description and the author(s). Code is PEP 8 clean (`flake8 src tests` passes), with meaningful names, small functions, docstrings, OOP where it fits and real error handling - no bare `except`, no swallowed errors.
6. **Never commit to `main`.** `main` is reached only by merging `dev` at the end of a sprint, and the merge commit is tagged `v1`, `v2` or `v3`. Both branches must show continuous development - a commit dump at the end is **-5** per branch.
7. **Never commit secrets.** `.env` and `.venv` are in `.gitignore`. The database URL and secret key are read from the environment; `.env.example` holds placeholders only.
8. **Ask when the requirement is ambiguous.** The assignment is the source of truth; the instructor is the Product Owner. When it is silent, ask a teammate rather than inventing a requirement.
9. **AI agents never commit.** No AI agent runs `git commit`, `git push`, `git merge` or `git tag`, or merges PRs - it edits files and a teammate reviews and commits them. The one exception: when a teammate asks, an agent may open a PR into `dev` for a GitHub issue, from a branch the teammate has pushed, with the issue linked in the body. Never add a `Co-Authored-By` trailer or a "Generated with" line for any AI tool to a commit message, PR, or issue, including drafts written for a teammate.
10. **Scrum records are the team's, written on time.** An agent may draft a scrum record from what a teammate tells it; it never invents participants, dates, or what was said. A record written after the fact carries the real date of the event, not today's.
11. **Never break a graded artifact to make a change convenient.** The required repository structure (`README.md`, `Dockerfile`, `docker-compose.yml`, `requirements.txt`, `src/`, `tests/`, `uml/`, `scrum/`, `htmlcov/`), the README sections from the template and the scrum record files come from the assignment; changing one needs a stated reason.

## Git model

| Branch | Rule |
|---|---|
| `main` | Releases only. Protected: a PR from `dev` with at least one approving review. The merge commit is tagged `v1`/`v2`/`v3`. |
| `dev` | Integration. Everything lands here through a reviewed PR. |
| `<issue#>-short-slug` | One task, off `dev`, named with its GitHub issue number, e.g. `14-sprint-model` for issue #14. Deleted after merge. |

- The team repository is **public** at [AdenPotato/CS3250-project-3](https://github.com/AdenPotato/CS3250-project-3) (kept on the owner's account rather than `msu-denver`, by team decision), with every member and the instructor as collaborators.
- PRs into `dev` are reviewed and approved by a teammate **other than the author**. `dev`'s history must show work from every team member - spread the work so it does.
- Issues are titled `US#: description` (or `Deploy:`, `Design:`, `Test:`, `Scrum:`, `Fix:` for work outside a story). Each PR links the issue it closes (`Closes #12`).
- Commits and PR titles: `CS-<issue number> <message>` - one logical change, present tense, lowercase, no trailing period: `CS-14 add sprint model with status`.
- Never rewrite pushed history.

## Skills

Skills live in `.agents/skills/<name>/SKILL.md`. When a skill or doc says `/new-model`, it means "read and follow `.agents/skills/new-model/SKILL.md`". Read the whole skill before starting its first step.

| Skill | Use when |
|---|---|
| `session-start` | Opening a working session, before any code. |
| `project-status` | Reporting where the project stands against the rubric. |
| `backlog` | Sprint 0: scoring the backlog and writing the sprint plan. |
| `scrum-notes` | Recording a sprint planning, daily scrum, review or retrospective. |
| `new-design` | A requirement is ambiguous, or a change touches more than one layer at once. |
| `new-model` | Changing a SQLAlchemy model or the schema. |
| `new-route` | Implementing a user story through all three layers. |
| `uml` | Writing or refreshing the use case, class or sequence diagram. |
| `testing` | Writing tests, the white-box and black-box tests, and the coverage report. |
| `qa-steps` | Writing a QA script for the PR reviewer. |
| `new-issue` | Opening a GitHub issue a teammate approved. |
| `release` | Clean-clone Compose check, merging `dev` into `main`, tagging the sprint release. |
| `sprint-close` | Verifying a sprint's deliverables and evidence, and the final progress report. |

## Roles

For focused work, take on the matching brief in `.agents/agents/`: `app.md` for Python under `src/` (all three layers), `ui.md` for templates and CSS, `release.md` for Docker, Compose, env config and tags, `scrum.md` for the `scrum/` folder, the board, the burndown and the final report.

## Guardrails

Claude Code enforces these through `.claude/settings.json`. Other tools must follow them by hand.

- **Never read** `.env` files, `*.pem` or `*.key`.
- **Never run** `git push --force` (or `-f`, `--force-with-lease`), `git push origin main`, or `docker compose down -v` without being asked (it deletes the database volume).
- **Ask first** before `gh issue create`, `gh pr create`, `gh pr merge`, creating a branch, `git merge`, or `git tag`.
- **Free to run:** `pytest`, `coverage`, `flake8`, `docker compose` (`up`, `down`, `ps`, `logs`, `exec`, ...), `docker build`, `pip install -r requirements.txt`, and read-only git (`status`, `diff`, `log`, `branch`). `docker compose down -v`, `docker volume rm` and `docker system prune` stay denied - they delete the database.
- **Clean up after yourself.** An agent that started containers runs `docker compose down` when its task is done. The volume is kept; only a teammate deletes it.

## Running the app

From the repo root, once the Compose setup exists:

```bash
cp .env.example .env
docker compose up --build         # http://localhost:5000
docker compose down               # stop, keep the data
docker compose down -v            # stop and wipe the database volume - after a model change
```

Tests and coverage:

```bash
coverage run -m pytest tests && coverage html    # writes htmlcov/
flake8 src tests
```

## Writing style

### Look and feel

Styled after shadcn/ui: the neutral **zinc** palette, `0.5rem` radius, 1px borders, subtle shadows, a system font stack, light and dark mode. Colours only come from the CSS variables in `theme.css` - never a hard-coded colour in a template or another stylesheet. The `ui` agent brief has the tokens. No CSS framework or CDN: one stylesheet, plain CSS.

### Prose

Plain, specific, lowercase-first prose in docs and comments. No em-dashes - use a hyphen with spaces. Say what the code does and why it is there; skip decoration. Comments cap at three lines; anything longer belongs in a docstring, the README or a scrum record.

Every Python file opens with:

```python
"""
<one or two sentences: what this module does>.

Authors: <name>, <name>
"""
```
