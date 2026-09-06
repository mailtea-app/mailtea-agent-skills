# Mailtea Agent Skills

Agent skills that teach AI coding agents (Claude Code, Cursor, or any
SKILL.md-compatible harness) how to work with
[Mailtea](https://mailtea.app) — send, schedule and manage email, design emails
and templates that survive real inboxes, and build the publication's website.

## Skills

| Skill | What it does |
|-------|--------------|
| [`mailtea`](./mailtea/SKILL.md) | Send, schedule, and manage email — transactional sends, batches, delivery status, contacts, segments, newsletters — via the Mailtea MCP server, the `mailtea-sdk` Node SDK, or the REST API. |
| [`mailtea-email-design`](./mailtea-email-design/SKILL.md) | Design the email itself — the structured block/ops path (email-safe by construction, and the fast one), images, brand colour, and hand-written email-safe HTML when you own the whole document. |
| [`mailtea-site-design`](./mailtea-site-design/SKILL.md) | Build and restyle the publication's public website: pages, section presets, theme, the design brief, and the draft → publish flow. |

## Install

**Preferred (portable):** use the [Mailtea Agent Plugin](../agent-plugin/) — these
skills plus the Mailtea MCP server in one package. It carries a manifest for
every client that has its own format, so Codex, Claude Code, Cursor, Grok Build,
VS Code with GitHub Copilot, Kiro, and the Gemini CLI all load the same package:

```bash
npx plugins add mailtea-app/mailtea-agent-plugin
```

**Skills only:** copy the skill directories into your agent's skills folder, e.g. for Claude Code:

```bash
cp -r mailtea mailtea-email-design mailtea-site-design ~/.claude/skills/
```

Or add them per-project under `.claude/skills/`.

## Requirements

A Mailtea personal access token (prefix `mt_pat_`) from Mailtea Studio
(**Settings → API keys**). The `mailtea` skill covers connecting the MCP server.

## License

[MIT](./LICENSE)

