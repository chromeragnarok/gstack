# gstack development

## Commands

```bash
bun install          # install dependencies
bun test             # run integration tests (browse + snapshot)
bun run dev <cmd>    # run CLI in dev mode, e.g. bun run dev goto https://example.com
bun run build        # compile binary to browse/dist/browse
```

## Project structure

```text
gstack/
├── browse/          # Headless browser CLI (Playwright)
│   ├── src/         # CLI + server + commands
│   ├── test/        # Integration tests + fixtures
│   └── dist/        # Compiled binary
├── ship/            # Ship workflow skill
├── review/          # PR review skill
├── plan-ceo-review/ # Founder-mode plan review skill
├── plan-eng-review/ # Engineering-mode plan review skill
├── retro/           # Retrospective skill
├── setup            # One-time setup: build binary + link sibling skills
├── SKILL.md         # Root gstack skill entrypoint
└── package.json     # Build scripts for browse
```

## Deploying to the active skill

The active skill lives at `~/.codex/skills/gstack/`. After making changes:

1. Push your branch.
2. Refresh the active skill checkout: `cd ~/.codex/skills/gstack && git fetch origin && git reset --hard origin/main`
3. Rebuild: `cd ~/.codex/skills/gstack && bun run build`

Or copy the binary directly: `cp browse/dist/browse ~/.codex/skills/gstack/browse/dist/browse`
