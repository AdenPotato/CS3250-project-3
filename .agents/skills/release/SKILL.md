---
name: release
description: Cut a sprint release - run the Docker Compose deployment from a clean clone, run the tests and regenerate coverage, walk the release checklist, then merge dev into main and tag v1, v2 or v3. Use at the end of each sprint, and for any change to the Dockerfile or docker-compose.yml.
---

# /release

Get `main` into the state the instructor receives it in. Deployment is **+5**, Compose with a multi-layer architecture is the **+5 bonus**, and every story only counts if it works in the deployment. Continuous development on `main` is graded too - a release every sprint is how `main` avoids the **-5**.

The rule that makes this skill worth running: **run it from a fresh clone, not your working tree.** The uncommitted file is the most common release failure and it is invisible from where you are sitting.

## 1. The deployment files

If they do not exist yet, write them. Non-negotiables:

- **`docker-compose.yml`** with at least two containers: `db` (official `postgres` image, **named volume**) and `web` (built from the `Dockerfile`, **waiting for `db` to be ready** - a `pg_isready` healthcheck plus `condition: service_healthy`), `web` on port `5000:5000`.
- **`Dockerfile`**: `requirements.txt` copied and installed before the source; `--host=0.0.0.0`; schema created and seeds loaded on start.
- **`requirements.txt` frozen** - exact versions from `pip freeze` in a clean venv, not hand-written ranges.
- **Config from the environment**: `DATABASE_URL`, `SECRET_KEY`. `.env.example` has placeholders; `.env` and `.venv` are in `.gitignore`.
- **`.dockerignore`** covering `.venv/`, `.git/`, `__pycache__/`, `.env`, `htmlcov/`.

## 2. Run clean

```
git clone -b dev <repo-url> /tmp/release-check
cd /tmp/release-check
cp .env.example .env
docker compose up --build
```

A failure here is almost always a missing commit. Fix it by committing the file, not by copying it in. Then `docker compose down -v` when done.

## 3. Test, cover, lint

```
coverage run -m pytest tests
coverage report
coverage html                  # regenerates htmlcov/ - commit it
flake8 src tests
```

Then, in the browser at http://localhost:5000, log in with the seed accounts and walk every acceptance criterion of every story implemented so far. A story that fails here is not in this release - say so at the review rather than tagging it as done.

## 4. The checklist

Report each item; do not mark one done without checking it.

- [ ] `docker compose up --build` from a clean clone serves http://localhost:5000
- [ ] Registration and login work (each broken is **-10**)
- [ ] Seed accounts log in; sample project, issues and sprint are visible
- [ ] Data survives `docker compose down` then `up`
- [ ] Every story planned for this sprint passes all its criteria in Compose, with tests
- [ ] `pytest` passes; `htmlcov/` regenerated and committed; coverage figure updated in the README
- [ ] `flake8 src tests` clean; every source file has its header comment
- [ ] README has the run commands and the seed accounts
- [ ] `.env`, `.venv`, `__pycache__` absent from `git ls-files`; `.env.example` present
- [ ] `uml/` diagrams match the code
- [ ] This sprint's `scrum/sprint_N/` records are filled and `board.png` is in

## 5. Merge and tag

Only once `dev` passes the checklist. **A teammate runs these** - agents never merge or tag (AGENTS.md rule 9).

`main` is protected, so merge `dev` into `main` through a PR with an approving review, then tag the resulting merge commit:

```
gh pr create --base main --head dev --title "release vN: sprint N"
# after approval and merge:
git switch main && git pull
git tag -a vN -m "sprint N release"
git push origin vN
```

Then: `git tag --list` shows `vN`, and `git log main --oneline -1` is the merge.

## 6. Report

State what passed, what failed, and what is outstanding on the checklist. If a planned story is not releasable, say so plainly - it goes back to the backlog at the review and shows up on the burndown's actual line. Finding that out is the point of this skill.
