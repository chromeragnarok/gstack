---
name: setup-browser-cookies
description: Import cookies from a real Chromium browser into the gstack browse session.
metadata:
  short-description: Import browser cookies for authenticated QA
---

# setup-browser-cookies

Use this skill when browser QA needs an authenticated session without manually logging in through headless Chromium.

## Setup

Find the browser binary first:

```bash
B=$(browse/bin/find-browse 2>/dev/null || ~/.codex/skills/gstack/browse/bin/find-browse 2>/dev/null)
if [ -n "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If `NEEDS_SETUP`:
1. Ask for permission to run `./setup`.
2. Run `./setup`.
3. Retry the lookup before continuing.

## Standard flow

### 1. Open the picker UI

```bash
$B cookie-import-browser
```

Tell the user:

`Cookie picker opened. Select the domains you want to import, then tell me when you're done.`

### 2. Direct import when the user names a domain

```bash
$B cookie-import-browser comet --domain github.com
```

Swap `comet` for the requested browser if specified.

### 3. Verify import

```bash
$B cookies
```

Summarize imported domains and counts without exposing values.

## Notes

- The first import may trigger a macOS Keychain prompt.
- Imported cookies persist in the `browse` session.
- Use this before `qa` or `browse` against authenticated pages.
