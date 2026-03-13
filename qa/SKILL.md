---
name: qa
description: Systematic web QA workflow with smoke, full, and regression modes for Codex.
metadata:
  short-description: Structured browser QA in Codex
---

# qa

Use this skill when the user wants a structured QA pass rather than one-off browser actions.

## Inputs to extract from the request

- Target URL
- Mode: `full`, `quick`, or `regression`
- Scope focus, if any
- Output directory override, if any
- Auth instructions, if any

Defaults:
- Mode: `full`
- Output directory: `.gstack/qa-reports`

## Setup

Resolve the browser binary first:

```bash
B=$(browse/bin/find-browse 2>/dev/null || ~/.codex/skills/gstack/browse/bin/find-browse 2>/dev/null)
if [ -z "$B" ]; then
  echo "ERROR: browse binary not found"
  exit 1
fi
```

Create output directories:

```bash
REPORT_DIR=".gstack/qa-reports"
mkdir -p "$REPORT_DIR/screenshots"
```

## Modes

### Full

Visit the important reachable pages, explore forms and actions, document real issues with evidence, and produce a report with severity counts and a health score.

### Quick

30-second smoke test:
- homepage
- top navigation targets
- obvious console errors
- obvious broken UI or dead links

### Regression

Run a full pass, then compare results against a baseline JSON file and call out:
- new issues
- fixed issues
- score delta

## Workflow

### 1. Initialize

- Resolve `B`
- Create the output directory
- Copy or recreate a report file in the output directory

### 2. Authenticate if needed

- If credentials are required, ask the user for the missing pieces plainly.
- If the app needs existing browser sessions, use `setup-browser-cookies`.
- If the flow hits 2FA or CAPTCHA, tell the user what to do and pause.

### 3. Orient

Get a quick model of the app:

```bash
$B goto <target-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/initial.png"
$B links
$B console --errors
```

### 4. Explore

For each page or workflow:

```bash
$B goto <page-url>
$B snapshot -i -a -o "$REPORT_DIR/screenshots/page.png"
$B console --errors
```

Then check:
- layout issues
- interactive controls
- forms and validation
- empty/loading/error states
- responsive behavior where relevant

### 5. Document issues immediately

For interactive bugs, capture:
1. state before action
2. action taken
3. resulting state
4. repro steps

For static bugs, capture one annotated screenshot and describe the defect clearly.

### 6. Wrap up

Produce:
- health score
- severity counts
- top 3 issues to fix
- console health summary
- pages visited
- screenshot count
- regression summary if applicable

## Reporting standard

Every issue should include:
- short title
- severity
- category
- repro steps
- evidence path(s)
- expected behavior
- actual behavior

## Scoring guidance

Use the existing weighted categories in this repo:
- Console
- Links
- Visual
- Functional
- UX
- Performance
- Content
- Accessibility

Be consistent and explain the biggest deductions.
