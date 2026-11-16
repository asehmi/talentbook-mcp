# talentbook-mcp

A Claude Code plugin that connects an agent to your own [Talentbook](https://talentbook.io)
data over MCP — goals, skills, OKRs, journal, guided consultations, team summaries, and (for
org-admin tokens) org roster/role-baseline/work-job-template management.

This plugin does **not** implement an MCP server. Talentbook already is one
(`lib/mcp_tools.py`/`api/mcp.py` in the main [`talentbook`](https://github.com/asehmi/talentbook)
repo, mounted at `/mcp/`). This plugin is the client-side half: it bundles the connection
definition and a skill that teaches an agent to use the ~70-tool catalog well — its scopes,
its idempotency-key and rate-limit rules, and the draft-then-approve pattern for anything it
writes on your behalf.

See `talentbook`'s own `_pm/MCP_CLIENT_PLUGIN.md` for the full design spec this plugin
implements, `_pm/MCP_CONNECTOR_PLAN.md`/`_pm/MCP_CONNECTOR_PRD.md` for the server side, and
`_pm/MCP_CONNECTOR_USAGE_NOTES.md` for the tool catalog reference and worked examples.

## Install

`claude plugin install` only accepts a plugin name resolved through a registered marketplace —
not a raw filesystem path, so `claude plugin install /path/to/talentbook-mcp` fails with
"not found in any configured marketplace." This repo doubles as its own single-plugin
marketplace (`.claude-plugin/marketplace.json`), so either of these works:

```bash
# Persistent install, straight from GitHub (adds the marketplace and installs in one step):
claude plugin install talentbook-mcp --marketplace asehmi/talentbook-mcp

# One-off session test from a local clone, no install:
claude --plugin-dir /path/to/talentbook-mcp
```

For a persistent install from a local clone during development, add it as a marketplace first,
then install from it by name:

```bash
claude plugin marketplace add /path/to/talentbook-mcp
claude plugin install talentbook-mcp@talentbook-mcp-marketplace
```

## Configure

On enabling the plugin, Claude Code prompts for two values (declared in
`.claude-plugin/plugin.json`'s `userConfig`):

- **Talentbook server URL** — the MCP endpoint, trailing slash required
  (`https://talentbook.io/mcp/` by default; use `http://localhost:8888/mcp/` for a local dev
  server).
- **Talentbook personal access token** — a `tb_pat_...` token from Talentbook's own
  `/app/connected-agents` page. Choose scopes to match what you actually want an agent to do —
  see the bundled skill (`skills/talentbook/SKILL.md`) for what each scope unlocks. Stored in
  Claude Code's secure credential storage, never written into this plugin or a plaintext
  settings file.

Revoking the token at `/app/connected-agents` immediately disconnects this plugin, with no
plugin-side cleanup needed.

## What's in here

- `.claude-plugin/plugin.json` — manifest + the `userConfig` prompts above.
- `.mcp.json` — the bundled MCP server definition, substituting the configured URL/token.
- `skills/talentbook/SKILL.md` — usage guidance: the scope table, idempotency-key rule,
  rate-limit backoff rule, and the draft-then-approve pattern for write tools.
- `commands/talentbook-status.md` — `/talentbook-mcp:talentbook-status`, a quick connection
  check.
- `commands/talentbook-checkin.md` — `/talentbook-mcp:talentbook-checkin`, a canned weekly
  reflective journal-entry assistant.
- `commands/talentbook-consultation.md` — `/talentbook-mcp:talentbook-consultation`, drives a
  guided consultation end to end with the draft-then-approve pattern.
- `commands/talentbook-team-digest.md` — `/talentbook-mcp:talentbook-team-digest`, composes a
  written digest across a manager's direct reports' latest progress summaries.
- `commands/talentbook-goal-planner.md` — `/talentbook-mcp:talentbook-goal-planner`, drafts
  goals targeting the biggest gaps from `get_my_gap_analysis`.

## Non-goals

No new MCP tools or scopes, no change to Talentbook's auth model, and no replacement for the
standalone [`talentbook_chat`](https://github.com/asehmi/talentbook_chat) CLI/Streamlit
clients — those remain separate clients for their own use cases. This plugin specifically
targets agent harnesses, starting with Claude Code, that already speak MCP and just need to be
pointed at the server and taught how to use it well.
