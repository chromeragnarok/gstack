---
name: gstack
description: Fast headless browser workflow for QA, dogfooding, screenshots, and browser-state inspection in Codex.
metadata:
  short-description: Use gstack browse from Codex
---

# gstack

Use this skill when the user wants browser-driven QA, dogfooding, screenshots, console/network inspection, or evidence for a frontend bug.

`gstack` is the umbrella browser skill. It uses the compiled `browse` binary in this repo, not Codex browser MCP tools.

## Core rule

Prefer the local `browse` binary through `exec_command`. Do not switch to other browser tooling unless the user explicitly asks for something `browse` cannot do.

## Setup

Before the first browser command, check whether the binary exists:

```bash
B=$(browse/bin/find-browse 2>/dev/null || ~/.codex/skills/gstack/browse/bin/find-browse 2>/dev/null)
if [ -n "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If the result is `NEEDS_SETUP`:
1. Ask the user for permission to run `./setup`.
2. Run `./setup` from the skill directory.
3. If Bun is missing, install it first.

## Standard flow

1. Resolve `B`.
2. Navigate: `$B goto <url>`
3. Understand the page: `$B snapshot -i`
4. Interact with `@e` refs: `$B click @e3`, `$B fill @e4 "value"`
5. Verify results with snapshots, assertions, console, network, and screenshots.

## Most useful commands

```bash
$B goto https://example.com
$B snapshot -i
$B snapshot -D
$B console --errors
$B network
$B is visible ".selector"
$B screenshot /tmp/page.png
$B responsive /tmp/layout
```

## Common workflows

### Smoke-check a page

```bash
$B goto https://example.com
$B text
$B console --errors
$B network
$B is visible "main"
```

### Walk a user flow

```bash
$B goto https://app.example.com/login
$B snapshot -i
$B fill @e3 "user@example.com"
$B fill @e4 "password"
$B click @e5
$B snapshot -D
$B screenshot /tmp/after-login.png
```

### Collect bug evidence

```bash
$B snapshot -i -a -o /tmp/annotated.png
$B screenshot /tmp/plain.png
$B console --errors
$B network
```

## Related skills

- Use `browse` for the same workflow under the explicit browser-skill name.
- Use `qa` for a structured QA pass with a report.
- Use `setup-browser-cookies` when authenticated pages need real browser cookies.
