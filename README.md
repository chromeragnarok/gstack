# gstack

**gstack turns Codex from one generic assistant into a team of specialists you can summon on demand.**

Eight opinionated workflow skills for Codex. Plan review, code review, shipping, browser automation, QA testing, cookie import, and retrospectives, all packaged as installable skills with a fast headless browser.

## Skills

| Skill | Mode | What it does |
|-------|------|--------------|
| `plan-ceo-review` | Founder / CEO | Rethink the problem. Find the 10-star product hiding inside the request. |
| `plan-eng-review` | Eng manager / tech lead | Lock in architecture, data flow, diagrams, edge cases, and tests. |
| `review` | Paranoid staff engineer | Find the bugs that pass CI but still explode in production. |
| `ship` | Release engineer | Sync main, run tests, push, and open a PR for ready branches. |
| `browse` | QA engineer | Give the agent eyes. It logs in, clicks through your app, and captures evidence. |
| `qa` | QA lead | Systematic QA testing with reports, health scores, screenshots, and regression tracking. |
| `setup-browser-cookies` | Session manager | Import cookies from your real browser into the headless session. |
| `retro` | Engineering manager | Team-aware retrospective with work patterns, praise, and growth areas. |

## Why gstack

Without gstack:

- the agent takes your request literally
- review depth varies from run to run
- shipping turns into a long back-and-forth
- browser QA is still manual
- the model can write code but cannot see your app

With gstack:

- you can ask for founder taste, engineering rigor, or release discipline explicitly
- browser QA becomes scriptable and repeatable
- the same repo works for global and project-local Codex installs
- each workspace gets its own isolated browser session

## Install

**Requirements:** Codex, Git, and [Bun](https://bun.sh/) v1.0+. The `browse` runtime compiles a native binary and works on macOS and Linux (x64 and arm64).

### 1. Install globally

```bash
git clone https://github.com/garrytan/gstack.git ~/.codex/skills/gstack
cd ~/.codex/skills/gstack
./setup
```

Then add a `gstack` section to your global `AGENTS.md`:

```md
## gstack

- Prefer the `browse` skill from gstack for browser QA and dogfooding.
- Available skills: `plan-ceo-review`, `plan-eng-review`, `review`, `ship`, `browse`, `qa`, `setup-browser-cookies`, `retro`.
- If a gstack skill is missing or stale, run `cd ~/.codex/skills/gstack && ./setup`.
```

### 2. Add gstack to a project

```bash
mkdir -p .codex/skills
cp -Rf ~/.codex/skills/gstack .codex/skills/gstack
rm -rf .codex/skills/gstack/.git
cd .codex/skills/gstack
./setup
```

Then add a project-level `gstack` section to `AGENTS.md`:

```md
## gstack

- Prefer the `browse` skill from `.codex/skills/gstack` for browser QA and dogfooding.
- Available skills: `plan-ceo-review`, `plan-eng-review`, `review`, `ship`, `browse`, `qa`, `setup-browser-cookies`, `retro`.
- If a gstack skill is missing or stale, run `cd .codex/skills/gstack && ./setup`.
```

Project-local installs commit real files to the repo, not a submodule. Teammates only need to run `cd .codex/skills/gstack && ./setup` once to build the browser binary and register sibling skill links.

### What gets installed

- skill files in `~/.codex/skills/gstack/` or `./.codex/skills/gstack/`
- sibling skill links such as `~/.codex/skills/browse` pointing into the gstack directory
- `browse/dist/browse` compiled binary
- `node_modules/` for the browser runtime
- `.context/retros/` snapshots when `retro` stores historical data in a project

Everything lives under `.codex/`. Nothing touches your global PATH or runs in the background permanently.

## Using the skills

The simplest way to invoke gstack is to mention the skill by name in your request:

```text
Use plan-ceo-review on this feature idea.
Use review on my current branch.
Use browse to test the signup flow on staging.
Use qa in quick mode against http://localhost:3000.
```

Typical flow:

1. Use `plan-ceo-review` to challenge the product shape.
2. Use `plan-eng-review` to lock the buildable architecture.
3. Implement the change.
4. Use `review` to catch structural issues.
5. Use `ship` when the branch is ready.
6. Use `browse` or `qa` to verify the live experience.

## Browser QA

`browse` is the differentiator. It gives the agent a persistent headless Chromium session with:

- navigation and interaction commands
- accessibility snapshots with reusable refs
- console and network inspection
- screenshots and responsive captures
- cookie import from real browsers

That means the agent can verify a live flow instead of guessing from code.

See [BROWSER.md](BROWSER.md) for the full command reference and internals.

## Running many sessions

gstack works well in one Codex session. It scales even better when you run multiple isolated sessions or worktrees in parallel.

[Conductor](https://conductor.build) pairs especially well with this setup. Each workspace gets its own browser instance, cookies, tabs, and logs, so one session can run `qa` while another uses `review` or implements a feature.

## Development

```bash
bun install
bun test
bun run dev goto https://example.com
bun run build
```

The browser CLI sources from `browse/src/` and compiles to `browse/dist/browse`.

## Troubleshooting

**A skill is not showing up in Codex**

Run:

```bash
cd ~/.codex/skills/gstack && ./setup
```

For project installs:

```bash
cd .codex/skills/gstack && ./setup
```

**`browse` cannot find the binary**

Run:

```bash
cd ~/.codex/skills/gstack && bun install && bun run build
```

If Bun is installed outside your shell PATH, call it explicitly with `~/.bun/bin/bun`.

**Refresh a project-local copy from the global install**

```bash
for s in browse plan-ceo-review plan-eng-review review ship retro qa setup-browser-cookies; do rm -f .codex/skills/$s; done
rm -rf .codex/skills/gstack
cp -Rf ~/.codex/skills/gstack .codex/skills/gstack
rm -rf .codex/skills/gstack/.git
cd .codex/skills/gstack
./setup
```

**Update gstack**

```bash
cd ~/.codex/skills/gstack
git fetch origin
git reset --hard origin/main
./setup
```

If the current project also vendors gstack, refresh the project copy after updating the global install.

**Uninstall gstack**

```bash
for s in browse plan-ceo-review plan-eng-review review ship retro qa setup-browser-cookies; do rm -f ~/.codex/skills/$s; done
rm -rf ~/.codex/skills/gstack
```

If a project also vendors gstack, remove `.codex/skills/gstack` and the matching `AGENTS.md` section from that repo.
