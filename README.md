# Neuralyn — Claude Code Skills

Open-source [Claude Code](https://claude.com/claude-code) skills built and maintained by Neuralyn. Drop a skill into `~/.claude/skills/` and Claude Code activates it automatically whenever a matching request comes in.

> **What's a skill?** A reusable, self-contained workflow Claude Code loads on demand. Each skill ships an entry point (`SKILL.md`), reference docs, and any supporting scripts. Skills let you encode multi-step, opinionated processes that go beyond what a single prompt can express — and reuse them across sessions and projects.

## Available skills

| Skill | What it does |
|---|---|
| [**helpdesk-writer**](helpdesk-writer/) | Drafts user-facing helpdesk articles for a SaaS product by cross-referencing the codebase and the live UI in a sandbox. Captures annotated screenshots via Playwright MCP, then publishes to a helpdesk platform via MCP/API. Human review gate before publication. |

More skills will land here over time. Each lives in its own directory and installs independently.

## Installing a skill

```bash
# Copy the skill folder into Claude Code's global skills directory
cp -r <skill-name> ~/.claude/skills/
```

Most skills need additional setup — MCP servers, environment variables, Python dependencies, etc. Open the skill's own `README.md` for the specifics.

## Repo layout

```
.
├── README.md            # this file
├── LICENSE.md           # MIT
└── <skill-name>/        # one directory per skill
    ├── README.md            # install + usage
    ├── SKILL.md             # entry point loaded by Claude Code
    ├── references/          # supporting docs the skill reads on demand
    ├── scripts/             # helper scripts
    ├── playbooks/           # reusable per-task scripts (optional)
    └── examples/            # canonical examples (optional)
```

## Contributing

Issues and pull requests welcome. If you'd like to propose a new skill, open an issue first so we can discuss scope and fit.

## License

[MIT](LICENSE.md) — free to use, modify, and redistribute.
