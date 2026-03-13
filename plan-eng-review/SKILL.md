---
name: plan-eng-review
description: Engineering-manager-mode plan review focused on architecture, data flow, edge cases, and test coverage.
metadata:
  short-description: Review a plan like a tech lead
---

# plan-eng-review

Use this skill to harden an implementation plan before coding starts.

Do not implement code. Review the plan.

## Review sequence

### Step 0: Scope challenge

Always start here:
1. What existing code already solves each sub-problem?
2. What is the minimum set of changes that achieves the goal?
3. If the plan touches more than 8 files or introduces more than 2 new abstractions, is it overbuilt?

Then recommend one of:
- `SCOPE REDUCTION`
- `BIG CHANGE`
- `SMALL CHANGE`

### Step 1: Architecture review

Evaluate:
- system boundaries
- dependency graph
- data ownership
- coupling
- migration risk

Use ASCII diagrams for any non-trivial flow.

### Step 2: Data flow and failure modes

Trace the happy path and shadow paths:
- nil or empty input
- invalid input
- upstream failure
- concurrency conflict
- partial persistence

For each new path, note:
- what fails
- how it is handled
- what the user sees
- whether a test should exist

### Step 3: Test review

Produce a test matrix covering:
- happy path
- boundary cases
- failures
- regressions
- authorization or trust-boundary behavior if relevant

### Step 4: Performance and operations

Check:
- expensive loops
- N+1 risk
- retries and backoff
- logging and observability
- rollback or partial-state risk

### Step 5: Decisions

If a real choice remains, ask one concise plain-text question at a time:

```text
We recommend A: <one-line reason>
A) ...
B) ...
C) ...
```

Do not batch unrelated decisions.

## Required outputs

- `What already exists`
- `NOT in scope`
- `Architecture diagram`
- `Test matrix`
- `Failure modes`
- `Open decisions`

## Review style

- Prefer minimal diff.
- Prefer explicitness over cleverness.
- Be ruthless about hidden complexity.
- Once scope is chosen, optimize within it instead of re-litigating it.
