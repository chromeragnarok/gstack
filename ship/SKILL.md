---
name: ship
description: Ship workflow that syncs main, runs tests, reviews the diff, updates release metadata, and opens a PR.
metadata:
  short-description: Ship the current feature branch
---

# ship

Use this skill when the branch is ready to go through the full pre-merge workflow.

This is an execution workflow, not a planning workflow. Run it straight through unless a stop condition is hit.

## Stop conditions

- current branch is `main`
- merge conflict cannot be resolved safely
- tests fail
- critical review issue requires a fix pass
- a version bump larger than patch needs user input

## Workflow

### 1. Pre-flight

Run:

```bash
git branch --show-current
git status
git diff main...HEAD --stat
git log main..HEAD --oneline
```

If on `main`, stop.

### 2. Merge `origin/main` first

Run:

```bash
git fetch origin main
git merge origin/main --no-edit
```

Resolve simple mechanical conflicts if they are obviously safe. Stop on ambiguous conflicts.

### 3. Run tests on the merged result

Use the project’s real test commands. If the project has multiple suites, run all relevant ones before continuing.

If any suite fails:
- show the failures
- stop

### 4. Run pre-landing review

Read `review/checklist.md`, review `git diff origin/main`, and report findings.

If critical issues exist:
- ask one concise plain-text question per issue
- use:
  - `A) Fix it now`
  - `B) Acknowledge and ship anyway`
  - `C) False positive`

If any issue is fixed, commit the review fixes and stop so the user can re-run `ship`.

### 5. Version and changelog

- read `VERSION`
- choose micro vs patch automatically from diff size and scope
- ask the user only if minor or major feels warranted
- update `CHANGELOG.md` from the full branch diff, not just the last commit

### 6. Commit remaining release metadata

Create logical commits if the branch still has uncommitted release metadata changes.

### 7. Push and create PR

Run:

```bash
git push -u origin <branch>
gh pr create ...
```

Create a concise PR body with:
- summary
- test plan
- review findings if any

## Rules

- Do not ask for approval on routine steps.
- Do not ignore failing tests.
- Do not silently skip the review.
- Do not ship from `main`.
