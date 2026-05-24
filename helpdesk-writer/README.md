# Helpdesk Writer (Claude Code skill)

A Claude Code skill for drafting helpdesk articles by cross-referencing two sources of truth — **the codebase** and **the live UI in a sandbox** — and applying a **voice/style guide** before publishing to your helpdesk platform via MCP/API.

## Installation

```bash
# 1. Copy the skill into Claude Code's global skills directory
cp -r helpdesk-writer ~/.claude/skills/

# 2. Install the required MCPs
claude mcp add playwright npx @playwright/mcp@latest
# Plus the MCP for your helpdesk platform (for publishing)

# 3. Install the Python dependency
pip install Pillow

# 4. Configure your project (DO NOT commit the .env)
mkdir -p ./projects/my-app
cat > ./projects/my-app/.helpdesk.env <<'EOF'
SANDBOX_URL=https://sandbox.domain.com
SANDBOX_EMAIL=helpdesk-bot@domain.com
SANDBOX_PASSWORD=...
BACKEND_REPO=/backend/repo/path
FRONTEND_REPO=/frontend/repo/path
# MCP or API credentials for your helpdesk platform
EOF

chmod 600 ./projects/my-app/.helpdesk.env
```

## Usage

Inside Claude Code, just ask:

> "Write a help article on how to register products in my app."

The skill activates automatically. Claude Code will:

1. Run **code recon** across your repo (limits, formats, possible errors)
2. **Log into the sandbox** via Playwright MCP
3. **Capture and annotate** screenshots
4. **Write the markdown** with evidence trails in HTML comments
5. **Show you the draft** for review
6. **Publish as a draft** through your helpdesk MCP or API

## Structure

```
helpdesk-writer/
├── README.md                 # this file
├── SKILL.md                  # entry point — master workflow
├── references/
│   ├── code-recon.md             # how to extract truth from the codebase
│   ├── style-guide.md            # voice, tone, glossary
│   ├── screenshot-conventions.md # image standards
│   ├── test-environment.md       # sandbox configuration
│   └── publishing-rules.md       # draft vs. publish-directly rules
├── scripts/
│   └── annotate.py           # image annotation (Pillow CLI)
├── playbooks/
│   └── example-add-product.md       # reusable script (example)
└── examples/
    └── example-step-by-step-article.md  # expected output
```

## Principles

The skill is built on a few non-negotiable rules — most notably, **every technical claim must have a code/UI trace** and **nothing publishes without human review**. The complete set lives in [`SKILL.md`](SKILL.md#principles).

## Customizing

To add a new type of article, create a playbook in `playbooks/`. To adjust the voice, edit `references/style-guide.md`. To change the annotation style, tweak `references/screenshot-conventions.md` and/or the defaults in `scripts/annotate.py`.

The skill is designed to be edited — it is not a closed product.
