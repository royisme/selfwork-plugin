# selfwork-plugin

`selfwork-plugin` packages your existing selfwork orchestration into a reusable Claude Code plugin.

## Included components

- Skill: `selfwork`
- Commands:
  - `/selfwork`
  - `/selfwork:status`
  - `/selfwork:queue`
  - `/selfwork:clean`
- Hook:
  - `Stop` hook via `hooks/selfwork.ts`

## Structure

```text
selfwork-plugin/
├── plugin.json
├── README.md
└── .claude-plugin/
    ├── commands/
    ├── skills/
    └── hooks/
```

## Local test

Run Claude Code with this plugin directory:

```bash
claude --plugin-dir selfwork-plugin
```

## Notes

- Hook command uses `${CLAUDE_PLUGIN_ROOT}` for portability.
- Existing repo-level `.claude/settings.local.json` hook can be removed after you confirm plugin hook behavior is working.
