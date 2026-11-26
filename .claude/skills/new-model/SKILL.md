---
name: new-model
description: Change a SQLAlchemy model - edit models.py and its repository, reset or migrate the PostgreSQL schema, update the seed data, and update the class diagram together. Use for the foundation's models, or any later change to User, Project, ProjectMember, Issue, Sprint, Comment or their relationships.
---

# /new-model

The Sprint 1 foundation creates **every** model, so stories are not supposed to change the schema - a change here touches every teammate. Agree it with the team first.

A model change is not just an edit to `models.py`. The database lives in a Docker volume, the repositories query it, the seed data writes to it, and the class diagram is graded (**+5**) for describing it. All of them move together.

## Before changing anything

Ask whether it should change at all, and which acceptance criterion needs it. A change that is convenient for one route is usually the route's problem. State the reason.

Know how the schema is created in this repo - check `src/app/__init__.py`:

- **`db.create_all()`** creates missing tables and **silently ignores a changed column**. Every teammate must wipe the volume after pulling.
- **Flask-Migrate** generates a migration the container applies on start (`flask db upgrade`). No wipe needed, but the migration file must be committed.

## Steps

1. **Edit `src/app/models.py`.** Constraints the criteria depend on belong in the database, not only in the form:
   - `User.username` and `User.email` unique (US1.2).
   - `Project.key` unique, 2-6 uppercase letters (US3.1, US3.2).
   - `ProjectMember` unique on `(user_id, project_id)`, with a `role` of OWNER or MEMBER (US4.2).
   - `Issue` unique on `(project_id, number)` so `WEB-3` is never issued twice; `story_points` limited to 1, 2, 3, 5, 8, 13 or null (US5.2); `type`, `priority`, `status` as enums.
   - `Issue.sprint_id` nullable - null means the issue is in the backlog (US5.6, US7.2, US7.4).
   - At most one ACTIVE sprint per project (US7.3) - a partial unique index on PostgreSQL, and the service checks it with a clear message.
   - Deleting a project or issue cascades to what it owns; deleting a user does not silently delete issues.
   - Datetimes timezone-aware (`DateTime(timezone=True)`).

2. **Update the repository** in `src/app/repositories/` for any new query. Services never build queries themselves.

3. **Apply it.**
   - `create_all`: `docker compose down -v && docker compose up --build` (ask first - it deletes the local data).
   - Flask-Migrate: `flask db migrate -m "<what changed>"` inside the `web` container, read the generated file, then restart.

   Confirm the container starts with no traceback and the seed accounts still log in.

4. **Check what the change broke.**
   ```
   grep -rn "<field>" src/ tests/
   ```
   Repositories, services, forms, templates, tests and `seed.py` all touch model fields.

5. **Update `src/seed.py`** if the field is required, so a fresh start still seeds cleanly.

6. **Update `uml/class.wsd`** to match (`/uml class`). Model classes only - no repositories or services in it.

7. **Run the tests** (`pytest`) and walk the stories already done. A model change breaks things far from where it was made.

## The PR

Say in the first line: **"Model change - run `docker compose down -v && docker compose up --build` after pulling."** (or "migration included" with Flask-Migrate). Mention it at the next daily scrum too.
