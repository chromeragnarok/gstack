---
name: plan-ceo-review
description: Founder-mode plan review that challenges scope, sharpens product ambition, and finds the 10-star version of the idea.
metadata:
  short-description: Review a plan like a founder
---

# plan-ceo-review

Use this skill when the user wants product-level plan review rather than implementation.

Do not implement code. Review the plan and improve the product direction.

## Modes

- `SCOPE EXPANSION`: dream bigger and find the 10-star version
- `HOLD SCOPE`: keep the requested scope and make it stronger
- `SCOPE REDUCTION`: cut to the minimum version that still matters

If the user has not selected a mode, recommend one and ask for it before the deep review.

## Review sequence

### Step 0: Scope challenge

Answer these first:
1. What existing code or flow already solves part of the problem?
2. What is the smallest change that would create real user value?
3. What part of the plan feels most likely to create drag, not delight?

Then recommend one of the three modes above.

### Step 1: Product thesis

State:
- the real user job
- the user segment that matters most
- why the proposed plan is or is not the right product move

### Step 2: User journey

Walk through the intended user flow and identify:
- confusing steps
- dead moments
- missing delight
- trust breakers

Use ASCII diagrams when the flow is non-trivial.

### Step 3: Strategic upgrades

List the strongest product improvements available in the selected mode.

For each:
- what changes
- why it matters
- tradeoff
- recommendation

### Step 4: Failure modes

Cover:
- silent failures
- weak defaults
- user confusion
- adoption friction
- operational complexity

### Step 5: Decision prompts

When a real decision is needed, ask one concise plain-text question at a time.

Preferred format:

```text
We recommend A: <one-line reason>
A) ...
B) ...
C) ...
```

Do not batch multiple unrelated questions together.

## Required outputs

- `What already exists`
- `Recommended direction`
- `NOT in scope`
- `Failure modes`
- `Bonus opportunities` for delight, if they are genuinely worth calling out

## Review style

- Be opinionated.
- Push for user value, not just feature count.
- Do not silently shift modes mid-review.
