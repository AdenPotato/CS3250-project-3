---
name: testing
description: Write and run the pytest suite - per-criterion tests, the required clearly labelled white-box and black-box tests (neither on registration or login), and the coverage report in htmlcov/ - then document them in the README Testing section. Use when building a story's tests, before a release, or when the Testing rubric line is at risk.
argument-hint: "[user story | white-box | black-box | coverage]"
---

# /testing

The Testing line is **+10**, and the coverage report in the README is another **+5**. The assignment asks for:

- At least **one clearly identified white-box test** and **one clearly identified black-box test**, each with its purpose, the functionality tested and the outcome described in the README.
- **Neither may target user registration or authentication.**
- A **coverage report** generated with Python's `coverage` tool, saved in `htmlcov/` (committed), with the overall figure in the README. There is no minimum.

Beyond the two graded tests, each story's acceptance criteria get tests - that is how a story is shown to be done (AGENTS.md rule 4).

## Setup

`requirements-dev.txt` (not the runtime image): `pytest`, `coverage`, `flake8`.

`tests/conftest.py` creates the app with a test config pointing at a **separate PostgreSQL database** (`TEST_DATABASE_URL`, e.g. a `taskboard_test` database in the Compose `db` service), creates the tables, and drops them after the session. Do not test against SQLite - the partial unique index and enum behaviour differ.

The foundation's `conftest.py` seeds the same data as `seed.py` and provides logged-in clients for an owner, a member and a non-member (logging in with `login_user` inside a request context, not through the login page), so a story's tests never depend on another story's pages.

Every test file starts with the header docstring from AGENTS.md.

## Per-criterion tests

One test per acceptance criterion, named for it, so a failure names the criterion:

```
test_us5_1_create_issue_gets_project_key_and_number
test_us5_2_invalid_story_points_rejected
test_us7_3_second_active_sprint_refused
test_us7_4_complete_sprint_returns_unfinished_issues
```

- **Services** are tested directly - they have no Flask imports, so a test builds the service with real repositories against the test database and calls it.
- **Routes** are tested through `app.test_client()`: status codes, redirects, the exact message text, and access (signed out, member not owner, non-member).

## The white-box test

A test written **from knowledge of the code's internal structure**, chosen to exercise its branches. Pick a service method with real logic - the best candidate is `SprintService.complete_sprint` (US7.4) or `SprintService.start_sprint` (US7.3).

- Name it so it cannot be missed: `tests/test_whitebox_sprint_service.py::test_whitebox_complete_sprint_branches`.
- Open it with a docstring naming it as the white-box test and listing the branches it covers (sprint not active -> error; issues in Done stay; issues not Done return to the backlog; empty sprint).
- One assertion group per branch. Check the branch coverage of that method in `htmlcov/` and say in the README that every branch is hit.

## The black-box test

A test written **only from the acceptance criteria and the HTTP interface**, with no knowledge of the code: inputs in, observable outputs checked. It drives the app through the Flask test client (or `requests` against the running Compose stack) and never imports a service, repository or model.

- Best candidates: US5 (create an issue, check it appears in the backlog as `KEY-n`, invalid points rejected) or US6 (change status, refresh the board, check the column).
- Name it: `tests/test_blackbox_issue_backlog.py::test_blackbox_create_issue_appears_in_backlog`.
- Use equivalence classes and boundaries from the criteria - valid points 1 and 13, invalid 4 and 0, an empty title.
- Open it with a docstring naming it as the black-box test.

## Coverage

```
coverage run --source=src -m pytest tests
coverage report -m
coverage html            # writes htmlcov/
```

Use `--branch` (or `branch = True` in `.coveragerc`) so the white-box claim can be checked. **`htmlcov/` must not be in `.gitignore`** - the assignment requires it in the repository. Regenerate and commit it before each release.

## Document in the README Testing section

For each of the two graded tests: its name and file, **purpose**, **functionality tested** (the story and criteria), and **outcome** (passed, and what it showed). Then the overall coverage percentage, a link to `htmlcov/index.html` and the commands above. Optionally a screenshot of the coverage summary.

## Keep it green

Any PR that changes a route, service or template runs `pytest` and `flake8 src tests` before review. A changed message text is a broken test, and the reviewer should hear it from the PR, not from the grader.
