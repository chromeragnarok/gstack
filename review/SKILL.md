---
name: review
description: Pre-landing diff review focused on structural bugs, trust boundaries, and hidden risks.
metadata:
  short-description: Review the branch diff before landing
---

# review

Use this skill to review the current branch against `origin/main` before landing changes.

## Workflow

### 1. Check branch state

Run:

```bash
git branch --show-current
git fetch origin main --quiet
git diff origin/main --stat
```

If the branch is `main` or there is no diff, stop with:

`Nothing to review — you're on main or have no changes against main.`

### 2. Read the checklist

Read `review/checklist.md`.

Do not proceed without the checklist.

### 3. Read the full diff

Run:

```bash
git diff origin/main
```

Review the complete diff before commenting. Do not flag issues already fixed in the diff.

### 4. Review in two passes

Pass 1:
- SQL & Data Safety
- Race Conditions & Concurrency
- LLM Output Trust Boundary

Pass 2:
- all informational categories from `review/checklist.md`

### 5. Output findings

Use the checklist output format exactly.

If critical issues exist:
- show all findings first
- then ask one concise plain-text question per critical issue
- include the problem, recommended fix, and options:
  - `A) Fix it now`
  - `B) Acknowledge and leave it`
  - `C) False positive`

If the user chooses `A`, apply the fix. Otherwise, leave the code unchanged.

If only non-critical issues exist, just report them.

If no issues exist, output:

`Pre-Landing Review: No issues found.`

## Rules

- Read-only by default.
- Be terse and specific.
- Cite file and line when possible.
- Only flag real issues.
