---
name: retro
description: Weekly engineering retrospective over commit history, coding sessions, hotspots, and contributor patterns.
metadata:
  short-description: Generate an engineering retro from git history
---

# retro

Use this skill for a retrospective over recent work in a repository.

## Supported arguments

- `retro`
- `retro 24h`
- `retro 14d`
- `retro 30d`
- `retro compare`
- `retro compare 14d`

Default window: 7 days.

If the argument is invalid, show the accepted formats and stop.

## Data collection

Use Pacific time for all time-based reporting:

```bash
TZ=America/Los_Angeles ...
```

Gather:
- commits in the window
- per-author stats
- timestamps for session detection
- file hotspots
- PR references from commit messages
- commit types by prefix

Run independent git reads in parallel when possible.

## Required sections

### Summary metrics

Include:
- commits
- contributors
- PRs merged
- insertions
- deletions
- net LOC
- test LOC ratio
- active days
- detected sessions

### Contributor leaderboard

Show:
- commits
- LOC delta
- top area touched

Label the current git user as `You`.

### Time and session analysis

Report:
- hourly histogram in Pacific time
- peak hours
- late-night clusters
- session counts and durations
- average session length

### Hotspots and ships

Report:
- most-changed files
- churn hotspots
- focus score
- ship of the week

### Team analysis

For each contributor:
- what they worked on
- one or two specific praise points
- one concrete growth opportunity

If the repo is solo, keep it personal and skip teammate sections.

### Trends

If the window is 14 days or more, include week-over-week trends.

## Style

- Ground everything in the git data.
- Be specific, not performative.
- Prefer short evidence-backed observations over vague praise.
