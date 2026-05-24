# Test Environment — the app's sandbox

The skill assumes a dedicated sandbox environment for the project's app (not production). Claude Code logs in there, navigates, and captures screenshots without touching any real customer data.

## Credentials and configuration

Credentials **never** live in the chat or in the code. They live in `.helpdesk.env`, read by Claude Code via `dotenv` or directly:

```bash
# ./projects/<project_slug>/.helpdesk.env

# URLs
SANDBOX_URL=https://sandbox.domain.com

# Test account
SANDBOX_EMAIL=helpdesk-bot@domain.com
SANDBOX_PASSWORD=...

# Local repos (for code recon)
BACKEND_REPO=/backend-repo/path
FRONTEND_REPO=/frontend-repo/path

# MCP or API credentials
```

File permissions: `chmod 600 .helpdesk.env`

> **Never ask the user for the literal value of a production secret, token, or API key** — not even "just to verify it's set". The user creates and rotates these outside of the chat. If you need to confirm a secret exists, use a read-only listing command that returns only names, not values (e.g. `gh secret list` for GitHub Actions). The skill always references secrets by name, never by value.

## Test account

Use a dedicated account, **separate from your personal account**.

## Automatic login via Playwright MCP

The skill logs in at the start of every walkthrough. Default flow:

```
1. browser_navigate -> $SANDBOX_URL/login
2. browser_fill (email field) -> $SANDBOX_EMAIL
3. browser_fill (password field) -> $SANDBOX_PASSWORD
4. browser_click -> "Submit" button
5. browser_wait -> wait a few seconds until the URL changes
6. Record the login steps for future use in the project directory at `./projects/<project_slug>/login-steps.md`
```

If the test account has MFA, the user should **disable it** — that's what sandboxes are for. Don't try to automate MFA.

### Sandbox reset

Before a session of article production, it's worth resetting the state. Create a `scripts/reset-sandbox.sh` script (outside this skill, in the project) that:

1. Clears extra data created in previous sessions
2. Restores the seed state

## What NOT to do in the sandbox

- Never click **Delete account** or other real destructive actions (can break seed data)
- Never upgrade/downgrade the plan (leave that for manual testing in a dedicated account)
- Never send real emails (transactional emails from the sandbox should go to a test inbox)
- Never test webhooks pointing to production
- Never upload images of real people (use seed images or generated ones)

## If something goes wrong

If login fails 2x, **stop**. Don't keep retrying — it could be:

- Test account locked (rate limit)
- Password was rotated and the env file is stale
- Sandbox is down

Report to the user and wait for instructions.
