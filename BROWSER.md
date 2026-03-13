# Browser — technical details

This document covers the command reference and internals of gstack's headless browser for Codex skills.

## Command reference

| Category | Commands | What for |
|----------|----------|----------|
| Navigate | `goto`, `back`, `forward`, `reload`, `url` | Get to a page |
| Read | `text`, `html`, `links`, `forms`, `accessibility` | Extract page content |
| Snapshot | `snapshot [-i] [-c] [-d N] [-s sel] [-D] [-a] [-o] [-C]` | Get refs, diffs, and annotations |
| Interact | `click`, `fill`, `select`, `hover`, `type`, `press`, `scroll`, `wait`, `viewport`, `upload` | Use the page |
| Inspect | `js`, `eval`, `css`, `attrs`, `is`, `console`, `network`, `dialog`, `cookies`, `storage`, `perf` | Debug and verify |
| Visual | `screenshot`, `pdf`, `responsive` | Capture visual state |
| Compare | `diff <url1> <url2>` | Spot differences between environments |
| Dialogs | `dialog-accept [text]`, `dialog-dismiss` | Control alert, confirm, and prompt handling |
| Tabs | `tabs`, `tab`, `newtab`, `closetab` | Multi-page workflows |
| Cookies | `cookie-import`, `cookie-import-browser` | Import cookies from file or a real browser |
| Multi-step | `chain` (JSON from stdin) | Batch commands in one call |

All selector arguments accept CSS selectors, `@e` refs after `snapshot`, or `@c` refs after `snapshot -C`.

## How it works

gstack's browser is a compiled CLI binary that talks to a persistent local Chromium daemon over HTTP. The CLI is a thin client: it reads a state file, sends a command, and prints the response to stdout. The server does the real work via [Playwright](https://playwright.dev/).

```text
┌─────────────────────────────────────────────────────────────────┐
│  Codex session                                                  │
│                                                                 │
│  "use browse to test https://staging.myapp.com"                 │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    HTTP POST     ┌──────────────┐                 │
│  │ browse   │ ──────────────── │ Bun HTTP     │                 │
│  │ CLI      │  localhost:9400  │ server       │                 │
│  │          │  Bearer token    │              │                 │
│  │ compiled │ ◄──────────────  │  Playwright  │──── Chromium    │
│  │ binary   │  plain text      │  API calls   │    (headless)   │
│  └──────────┘                  └──────────────┘                 │
│   ~1ms startup                  persistent daemon               │
│                                 auto-starts on first call       │
│                                 auto-stops after 30 min idle    │
└─────────────────────────────────────────────────────────────────┘
```

## Lifecycle

1. First call: the CLI checks `/tmp/browse-server.json` for a running server. If none exists, it spawns `bun run browse/src/server.ts`, launches headless Chromium, writes the state file, and starts accepting commands.
2. Subsequent calls: the CLI reads the state file, sends an HTTP POST with the bearer token, and prints the response. Typical round-trip is around 100-200ms.
3. Idle shutdown: after 30 minutes with no commands, the server shuts down and cleans up the state file.
4. Crash recovery: if Chromium dies, the CLI detects the dead server on the next call and starts a fresh one.

## Source map

```text
browse/
├── src/
│   ├── cli.ts                  # Reads state file, sends HTTP, prints response
│   ├── server.ts               # Bun HTTP server and command router
│   ├── browser-manager.ts      # Chromium lifecycle, tabs, refs, crash handling
│   ├── snapshot.ts             # Accessibility tree, refs, diffing, annotations
│   ├── read-commands.ts        # Non-mutating commands
│   ├── write-commands.ts       # Mutating commands
│   ├── meta-commands.ts        # Snapshot routing, chain, diff, server helpers
│   ├── cookie-import-browser.ts
│   ├── cookie-picker-routes.ts
│   ├── cookie-picker-ui.ts
│   └── buffers.ts
├── test/                       # Integration tests + fixtures
└── dist/
    └── browse                  # Compiled binary
```

## Snapshot system

The browser's main interaction model is ref-based selection:

1. `page.locator(scope).ariaSnapshot()` returns a YAML-like accessibility tree.
2. The snapshot parser assigns refs such as `@e1` and `@e2`.
3. Each ref maps back to a Playwright `Locator`.
4. Later commands such as `click @e3` resolve through that stored map.

This avoids DOM mutation and keeps the interaction model stable across repeated commands.

Extended snapshot features:

- `-D` stores a baseline and shows a unified diff on the next diff call
- `-a` captures an annotated screenshot with ref labels
- `-C` includes cursor-interactive elements that are not obvious in the accessibility tree

## Multi-workspace support

Each workspace gets its own Chromium process, tabs, cookies, and logs.

If `CONDUCTOR_PORT` is set, the browse port is derived deterministically:

```text
browse_port = CONDUCTOR_PORT - 45600
```

| Workspace | CONDUCTOR_PORT | Browse port | State file |
|-----------|---------------:|------------:|------------|
| Workspace A | 55040 | 9440 | `/tmp/browse-server-9440.json` |
| Workspace B | 55041 | 9441 | `/tmp/browse-server-9441.json` |
| No Conductor | — | 9400 (scan) | `/tmp/browse-server.json` |

You can also set `BROWSE_PORT` directly.

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `BROWSE_PORT` | `0` | Fixed port for the HTTP server |
| `CONDUCTOR_PORT` | unset | If set, browse port = this - 45600 |
| `BROWSE_IDLE_TIMEOUT` | `1800000` | Idle shutdown timeout in ms |
| `BROWSE_STATE_FILE` | `/tmp/browse-server.json` | Path to the server state file |
| `BROWSE_SERVER_SCRIPT` | auto-detected | Path to `server.ts` |

## Why CLI over MCP

For local browser automation, a CLI keeps the interface small and cheap:

- lower context overhead than a full MCP protocol exchange
- simpler failure modes than a long-lived browser protocol connection
- easy to compose from Codex through `exec_command`

Codex already has a terminal tool. A compiled CLI that prints plain text is the thinnest useful layer on top of it.

## Development

### Prerequisites

- [Bun](https://bun.sh/) v1.0+
- Playwright Chromium, installed automatically by `./setup` or `bunx playwright install chromium`

### Quick start

```bash
bun install
bun test
bun run dev goto https://example.com
bun run build
```

### Running tests

```bash
bun test
bun test browse/test/commands
bun test browse/test/snapshot
bun test browse/test/cookie-import-browser
```

Tests spin up a local HTTP server from `browse/test/test-server.ts`, serve HTML fixtures from `browse/test/fixtures/`, and exercise the CLI commands against those pages.

## Deploying to the active skill

The active skill lives at `~/.codex/skills/gstack/`. After making changes:

1. Push your branch.
2. Refresh the active skill checkout: `cd ~/.codex/skills/gstack && git fetch origin && git reset --hard origin/main`
3. Rebuild: `cd ~/.codex/skills/gstack && bun run build`

Or copy the binary directly:

```bash
cp browse/dist/browse ~/.codex/skills/gstack/browse/dist/browse
```

## Adding a new command

1. Add the handler in `read-commands.ts` or `write-commands.ts`.
2. Register the route in `server.ts`.
3. Add a test in `browse/test/commands.test.ts`, plus a fixture if needed.
4. Run `bun test`.
5. Run `bun run build`.
