---
name: new-route
description: Implement a user story end to end through all three layers - repository query, service rule, route with its Flask-WTF form and access check, Jinja template, and tests for every acceptance criterion. Use when building a story or one task of a story.
---

# /new-route

Build one story (or one task of it) through every layer it touches. Stay inside the story's own files: its blueprint, service, repository methods, templates and tests. Replace the foundation's stub page for the route; do not edit another story's files. Take the story from the request; if none is given, take the current issue's `US#` title.

## Before writing

1. Read the story's acceptance criteria in [README.md](../../../README.md#user-stories). They are numbered `US#.n`; every one must be satisfied by what you build, and the PR will list them.
2. Check the foundation is merged into `dev`. Stories do not depend on each other (AGENTS.md rule 3) - build against the seed data, not another story's pages.
3. Read `uml/class.wsd` and the current `models.py`. The foundation created every model; if the story seems to need a schema change, stop and raise it with the team (`/new-model`), because it affects everyone.
4. If a criterion is ambiguous, stop and run `/new-design`.

## Build, bottom up

Every new file starts with the header docstring from AGENTS.md (description and authors).

1. **Data access** - `src/app/repositories/<thing>_repository.py`. One class per aggregate (`IssueRepository`), methods named for what they return (`list_backlog(project_id)`, `next_number(project_id)`). The only layer that touches `db.session`.

2. **Business logic** - `src/app/services/<thing>_service.py`. The rules the criteria state live here, once: "only the owner manages members" (US4.4), "one active sprint" (US7.3), "unfinished issues return to the backlog" (US7.4), "assignee must be a member" (US5.5). Raise a domain exception (`PermissionDenied`, `NotFound`, `ValidationError` from `services/errors.py`) instead of returning flags. No Flask imports here - that is what makes the service testable on its own.

3. **The form**, if the route takes input. A `FlaskForm` in `src/app/forms.py` with validators that mirror the criteria: `Length(min=8)` on the password (US1.3), `Regexp('^[A-Z]{2,6}$')` on the project key (US3.1), a `SelectField` limited to 1, 2, 3, 5, 8, 13 for story points (US5.2), "end date after start date" for sprints (US7.1).

4. **Presentation** - the view function in the matching blueprint under `src/app/routes/`.
   - `@login_required` on everything except register and login (US2.3).
   - Call the service; catch its exceptions and turn them into a 403, a 404 or a message on the page. No queries and no rules in the route.
   - `form.validate_on_submit()` gates the POST branch. Redirect after a successful POST; re-render with errors after a failed one, using the exact message text the criteria name.
   - `url_for`, never a hardcoded path. Pass `title` to every `render_template`.

5. **The template** in `src/app/templates/`, extending `base.html`. Every field the criterion lists, `{{ form.hidden_tag() }}` in every form, errors beside their fields, the empty state the criteria name, stable `id`s on anything a test looks for.

6. **Seed data**, if the story needs something to look at on a fresh start (a project with members, issues, a planned sprint).

## Verify

- **Tests** (`/testing`): a test per criterion, named `test_us<n>_<m>_<what>`. Services get unit tests without Flask; routes get tests through the Flask test client.
- `flake8 src tests` clean.
- Rebuild and run: `docker compose up --build`, http://localhost:5000. Walk each criterion: the happy path, every failure the criteria name, signed out, a member who is not the owner, and a non-member editing a URL.

## Finish

- If the flow differs from the sequence diagram, fix the diagram in the same change.
- Commit per [AGENTS.md](../../../AGENTS.md#git-model), PR into `dev` with `Closes #<issue>`, the criteria list, a screenshot, and a QA script for the reviewer (`/qa-steps`). Move the card to **In Review**.
