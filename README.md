# Mailtea Agent Skills

Agent skills that teach AI coding agents (Claude Code, Cursor, or any
SKILL.md-compatible harness) how to work with
[Mailtea](https://mailtea.app) — send, schedule, and manage email, and design
emails that survive real inboxes.

## Skills

| Skill | What it does |
|-------|--------------|
| [`mailtea`](./mailtea/SKILL.md) | Send, schedule, and manage email — transactional sends, batches, delivery status, contacts, segments, newsletters — via the Mailtea MCP server, the `mailtea` Node SDK, or the REST API. |
| [`mailtea-email-design`](./mailtea-email-design/SKILL.md) | Design the email itself: layout, typography, and email-safe HTML that renders correctly across real email clients. |

## Install

Copy the skill directories into your agent's skills folder, e.g. for Claude Code:

```bash
cp -r mailtea mailtea-email-design ~/.claude/skills/
```

Or add them per-project under `.claude/skills/`.

## Requirements

A Mailtea personal access token (prefix `mt_pat_`) from the Mailtea dashboard
(**Settings → API keys**). The `mailtea` skill covers connecting the MCP server.

## License

[MIT](./LICENSE)

