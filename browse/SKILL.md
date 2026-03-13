---
name: browse
description: Fast headless browser control for page inspection, QA, screenshots, and interaction testing in Codex.
metadata:
  short-description: Browser QA workflow for Codex
---

# browse

Use this skill when the task is primarily browser interaction: checking a page load, clicking through a flow, capturing screenshots, inspecting console/network state, or verifying DOM changes.

## Preferred execution model

Use the compiled `browse` binary through `exec_command`.

## Setup

Find the binary first:

```bash
B=$(browse/bin/find-browse 2>/dev/null || ~/.codex/skills/gstack/browse/bin/find-browse 2>/dev/null)
if [ -n "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If `NEEDS_SETUP`:
1. Ask the user for permission to run `./setup`.
2. Run `./setup` from the installed `gstack` directory.
3. Re-run the check before continuing.

## Standard operating procedure

1. Go to the page.
2. Snapshot interactive elements.
3. Use `@e` or `@c` refs for interactions.
4. Re-snapshot or diff after important actions.
5. Check console, network, and explicit assertions before claiming success.

## Core command patterns

```bash
$B goto https://example.com
$B snapshot -i
$B click @e3
$B fill @e4 "value"
$B snapshot -D
$B console --errors
$B network
$B is visible ".success"
$B screenshot /tmp/result.png
```

## High-value commands

### Understand the page

```bash
$B text
$B html
$B links
$B forms
$B accessibility
```

### Interact

```bash
$B click @e3
$B fill @e4 "text"
$B select @e5 "Option"
$B hover @e6
$B type @e7 "hello"
$B press Tab
```

### Inspect and assert

```bash
$B console --errors
$B network
$B is visible ".modal"
$B is enabled "#submit"
$B attrs "#logo"
$B css ".button" "background-color"
```

### Visual output

```bash
$B screenshot /tmp/page.png
$B snapshot -i -a -o /tmp/annotated.png
$B responsive /tmp/layout
```

## Notes

- `snapshot` is the default way to discover interactable elements.
- Refs are invalidated after navigation. Re-run `snapshot` after `goto`.
- If the user needs a systematic report rather than ad hoc browser work, use the `qa` skill instead.
