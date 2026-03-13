# Codex Skill Port Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Convert `gstack` into a Codex-only skill pack with Codex-native prompts, `.codex` install paths, and working global/project-local runtime resolution.

**Architecture:** Keep the existing browser runtime and skill layout, but replace Claude-specific install assumptions and prompt conventions end to end. Implement the port in small, test-first slices: runtime path resolution first, then prompt rewrites, then repo docs and compatibility cleanup, and verify with focused greps plus the full Bun suite.

**Tech Stack:** Markdown skills, Bash setup script, Bun, Playwright, TypeScript, GitHub

---

### Task 1: Port Runtime Path Resolution To Codex

**Files:**
- Modify: `browse/test/commands.test.ts`
- Modify: `browse/bin/find-browse`
- Modify: `browse/src/cli.ts`

**Step 1: Write the failing test**

Add or update tests in `browse/test/commands.test.ts` so server-script resolution explicitly expects `.codex/skills/gstack/...` instead of `.claude/skills/gstack/...`.

```ts
const execPath = path.join(root, '.codex/skills/gstack/browse/dist/browse');
const serverPath = path.join(root, '.codex/skills/gstack/browse/src/server.ts');
expect(resolveServerScript({ HOME: path.join(root, 'empty-home') }, '$bunfs/root', execPath)).toBe(serverPath);
```

Add a second case covering the fallback-to-home install:

```ts
const home = path.join(root, 'home');
const serverPath = path.join(home, '.codex/skills/gstack/browse/src/server.ts');
expect(resolveServerScript({ HOME: home }, '$bunfs/root', '')).toBe(serverPath);
```

**Step 2: Run test to verify it fails**

Run:

```bash
export PATH="$HOME/.bun/bin:$PATH" && ~/.bun/bin/bun test browse/test/commands.test.ts
```

Expected: path-resolution assertions fail because the code still points to `.claude`.

**Step 3: Write minimal implementation**

Update:
- `browse/src/cli.ts` legacy fallback to `~/.codex/skills/gstack/browse/src/server.ts`
- `browse/bin/find-browse` project-local lookup to `<repo>/.codex/skills/gstack/browse/dist/browse`
- `browse/bin/find-browse` home lookup to `~/.codex/skills/gstack/browse/dist/browse`

**Step 4: Run test to verify it passes**

Run:

```bash
export PATH="$HOME/.bun/bin:$PATH" && ~/.bun/bin/bun test browse/test/commands.test.ts
```

Expected: PASS.

**Step 5: Commit**

```bash
git add browse/test/commands.test.ts browse/bin/find-browse browse/src/cli.ts
git commit -m "fix: resolve gstack runtime from codex skill paths"
```

### Task 2: Rewrite Skill Prompts For Codex

**Files:**
- Modify: `SKILL.md`
- Modify: `browse/SKILL.md`
- Modify: `qa/SKILL.md`
- Modify: `review/SKILL.md`
- Modify: `ship/SKILL.md`
- Modify: `plan-ceo-review/SKILL.md`
- Modify: `plan-eng-review/SKILL.md`
- Modify: `retro/SKILL.md`
- Modify: `setup-browser-cookies/SKILL.md`

**Step 1: Write the failing structural checks**

Define the structural checks as grep-based assertions:

```bash
rg -n "\.claude|CLAUDE\.md|AskUserQuestion|allowed-tools" SKILL.md browse/SKILL.md qa/SKILL.md review/SKILL.md ship/SKILL.md plan-ceo-review/SKILL.md plan-eng-review/SKILL.md retro/SKILL.md setup-browser-cookies/SKILL.md
```

Expected current output: matches exist.

**Step 2: Run the check to verify it fails**

Run the `rg` command above.

Expected: non-empty output proving Claude-specific content is still present.

**Step 3: Write minimal implementation**

Rewrite each skill file to:
- use Codex-friendly frontmatter
- describe Codex tool usage accurately
- replace slash-command framing with “use this skill / workflow” wording
- replace `.claude` install references with `.codex`
- replace `AskUserQuestion` instructions with concise plain-text question flow

**Step 4: Run structural checks to verify they pass**

Run:

```bash
rg -n "\.claude|CLAUDE\.md|AskUserQuestion|allowed-tools" SKILL.md browse/SKILL.md qa/SKILL.md review/SKILL.md ship/SKILL.md plan-ceo-review/SKILL.md plan-eng-review/SKILL.md retro/SKILL.md setup-browser-cookies/SKILL.md
```

Expected: no output.

**Step 5: Commit**

```bash
git add SKILL.md browse/SKILL.md qa/SKILL.md review/SKILL.md ship/SKILL.md plan-ceo-review/SKILL.md plan-eng-review/SKILL.md retro/SKILL.md setup-browser-cookies/SKILL.md
git commit -m "refactor: rewrite gstack skills for codex"
```

### Task 3: Replace Repo Guidance And Install Docs

**Files:**
- Create: `AGENTS.md`
- Delete: `CLAUDE.md`
- Modify: `README.md`
- Modify: `BROWSER.md`
- Modify: `CHANGELOG.md`

**Step 1: Write the failing structural checks**

Run:

```bash
rg -n "\.claude|CLAUDE\.md|Claude Code|slash command|slash commands" README.md BROWSER.md CHANGELOG.md AGENTS.md 2>/dev/null
```

Expected current output: matches in the existing docs.

**Step 2: Run the check to verify it fails**

Run the `rg` command above before editing.

Expected: non-empty output.

**Step 3: Write minimal implementation**

- Create `AGENTS.md` with the repo-maintainer guidance currently carried by `CLAUDE.md`, rewritten for Codex.
- Remove `CLAUDE.md`.
- Rewrite `README.md` install, usage, and troubleshooting content for Codex-only installs.
- Rewrite `BROWSER.md` so architecture references describe Codex, not Claude.
- Update `CHANGELOG.md` only where old entries inaccurately market the repo as Claude-only.

**Step 4: Run structural checks to verify they pass**

Run:

```bash
rg -n "\.claude|CLAUDE\.md|Claude Code|slash command|slash commands" README.md BROWSER.md CHANGELOG.md AGENTS.md 2>/dev/null
```

Expected: no matches that describe the current repo inaccurately.

**Step 5: Commit**

```bash
git add AGENTS.md README.md BROWSER.md CHANGELOG.md CLAUDE.md
git commit -m "docs: port gstack repository guidance to codex"
```

### Task 4: Verify End-To-End Codex Port

**Files:**
- Review only: entire diff

**Step 1: Run focused compatibility checks**

Run:

```bash
rg -n "\.claude|CLAUDE\.md|AskUserQuestion|allowed-tools" .
```

Expected: no matches outside historical context you intentionally kept.

**Step 2: Run the full test suite**

Run:

```bash
export PATH="$HOME/.bun/bin:$PATH" && ~/.bun/bin/bun test
```

Expected: PASS.

**Step 3: Review install and runtime behavior**

Run:

```bash
./setup
```

Expected:
- build succeeds or is skipped if current
- Playwright check succeeds
- skill symlink message references Codex layout

**Step 4: Review git diff**

Run:

```bash
git status --short
git diff --stat
```

Expected: only the planned Codex-port files are changed.

**Step 5: Commit**

```bash
git add AGENTS.md BROWSER.md CHANGELOG.md README.md SKILL.md browse plan-ceo-review plan-eng-review qa review setup setup-browser-cookies ship docs/plans
git commit -m "feat: port gstack to codex skills"
```
