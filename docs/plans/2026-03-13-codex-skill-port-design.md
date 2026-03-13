# Codex Skill Port Design

## Goal

Convert this repository from a Claude Code skill pack into a Codex-only skill pack while preserving the existing skill set, browser runtime behavior, and global/project-local installation model.

## Scope

- Keep the same top-level skills:
  - `gstack`
  - `browse`
  - `qa`
  - `review`
  - `ship`
  - `plan-ceo-review`
  - `plan-eng-review`
  - `retro`
  - `setup-browser-cookies`
- Support both global install at `~/.codex/skills/gstack` and project-local install at `./.codex/skills/gstack`.
- Rewrite the skill prompts to be idiomatic for Codex workflows and tool names.
- Remove Claude-specific mechanics from runtime paths, docs, and prompt content.

## Approach

### 1. Install and runtime conversion

Update all hardcoded skill-install paths from `.claude/skills/...` to `.codex/skills/...`.

Primary files:
- `setup`
- `browse/bin/find-browse`
- `browse/src/cli.ts`
- `browse/test/commands.test.ts`

Behavior remains the same:
- `setup` builds `browse/dist/browse`
- Playwright Chromium is installed on demand
- sibling skill symlinks are created when the repo is installed inside a Codex `skills/` directory
- the browser binary resolves correctly for both global and project-local installs

### 2. Prompt and skill rewrite

Rewrite every `SKILL.md` to Codex-native wording and metadata.

Remove:
- `allowed-tools`
- slash-command framing
- `CLAUDE.md` references
- `AskUserQuestion`
- Claude-specific tool naming and assumptions

Replace with Codex-native guidance:
- `exec_command` for shell work
- `multi_tool_use.parallel` for parallel reads
- `apply_patch` for manual edits
- plain-text user questions when a decision is required
- Codex session and workflow language instead of Claude session language

### 3. Repo guidance and docs rewrite

Replace the repo’s Claude maintainer guidance with Codex maintainer guidance.

Primary changes:
- add `AGENTS.md` at repo root
- remove `CLAUDE.md`
- rewrite `README.md`
- rewrite `BROWSER.md`
- update install and troubleshooting snippets
- update changelog phrasing only where it inaccurately describes the repo as Claude-specific

## Verification Plan

- Run path-resolution tests covering `.codex/skills/gstack` installs.
- Run the full Bun test suite with Bun on `PATH`.
- Grep for stale Claude-specific references after the rewrite:
  - `.claude`
  - `CLAUDE.md`
  - `AskUserQuestion`
  - "Claude Code"
  - slash-command phrasing for these skills

## Migration Rules

- This is a Codex-only repo after the change.
- No Claude compatibility shims are kept in runtime code or docs.
- The browser architecture is unchanged apart from install-path resolution and any minimal testability cleanup needed for Codex installs.

## Non-Goals

- No new skills
- No browser feature expansion
- No dual Claude/Codex support layer
- No broad Playwright runtime refactor
