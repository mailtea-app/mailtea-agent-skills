---
name: mailtea-email
description: Send, schedule, and manage email with Mailtea from an AI agent. Use when the user wants to send a transactional email, send a batch, schedule a send, check delivery status / opens / clicks, list or cancel emails, manage contacts/segments, or draft and send a newsletter — via the Mailtea MCP server, the mailtea Node SDK, or the REST API.
---

# Mailtea email

Mailtea lets you send and manage email programmatically. Prefer the **MCP server**
(no code) when it's connected; fall back to the **SDK** or **REST** in code.

> **Designing** the email rather than sending it? Use **mailtea-email-design**
> first — it covers how the email should *look* and survive real inboxes, and the
> structured ops path that is faster than writing HTML. Building the publication's
> public **website**? Use **mailtea-site-design**.

## Setup (once)

Get a personal access token (prefix `mt_pat_`) from the Mailtea dashboard
(**Settings → API keys**) or `POST /v1/api-keys`. Then connect Claude Code:

```bash
claude mcp add mailtea -e MAILTEA_API_TOKEN=mt_pat_xxx -- npx -y mailtea-mcp
```

Self-hosting or local dev? add `-e MAILTEA_API_BASE_URL=http://localhost:7787`.

## Send a transactional email — `email.send`

A one-shot email to specific recipients (NOT a newsletter to the whole list).

- Required: `from` (must use a verified domain), `to` (string or array, ≤50),
  `subject`.
- Body: provide `html` and/or `text`, **or** a `template` reference — not both.
- Optional: `cc`, `bcc`, `reply_to`, `scheduled_at` (ISO 8601 → schedule),
  `tags` (`[{name,value}]`), `headers`, `attachments` (`{filename, content(base64)}`).

Returns `{ id }`.

## Manage email

- `email.list` — list emails (filter by `status`, `tag_name`/`tag_value`, date
  range; paginate with `limit`/`offset`).
- `email.get` — delivery status (`last_event`) + `open_count` / `click_count`.
- `email.batch` — up to 100 emails in one call (no attachments/scheduling).
- `email.reschedule` / `email.cancel` — for still-`scheduled` emails.

## Send a newsletter (to the whole publication list)

Different from transactional: `issue.create_draft` → build the body →
`issue.schedule` or `issue.send_now`. Inspect with `issue.list_recent`,
`issue.delivery_progress`, and `analytics.*`. Manage audience with `contact.*`.

**To build or change the body, use `issue.apply_ops`, not `issue.update_draft`.**
`update_draft` replaces the whole document; `apply_ops` edits it surgically —
rewrite one paragraph, recolour one button, reorder sections — and returns an
addressable outline plus a report of anything it skipped. `issue.get_editor`
reads the current outline, styles and headers back. `email.lint` checks the
result against the real caniemail matrix before you send. See
**mailtea-email-design**.

## Templates (reusable designs)

`template.create` → `template.publish`, then any send can reference it by id with
variables. `template.list` / `get` / `update` / `duplicate` / `delete`, and
`template.versions` / `template.restore_version` for history.

**Editing a published template unpublishes it.** Any content change — including
the subject line — returns it to draft, and automations and the API STOP sending
it until you `template.publish` again. The response tells you: `unpublished:
true`. Re-publish, or tell the user you left it as a draft on purpose.

## Images

`site.asset_upload` puts an image in the publication's library and returns the
permanent URL an image block needs; `site.asset_list` shows what is already
there; `site.asset_delete` retires one (the file keeps resolving, so already-sent
email does not break). CLI: `mailtea assets upload|list|delete`. PNG/JPEG/GIF/WebP,
5 MB max — SVG is refused because it can carry script.

## Website

`site.get`, `site.page_upsert`, `site.apply_ops`, `site.presets_list`,
`site.design_brief_set`, `site.publish`. Edits land on a DRAFT until you publish.
See **mailtea-site-design**.

## In code instead of MCP

Node SDK (`npm i mailtea-sdk`):

```ts
import { Mailtea } from "mailtea-sdk";
const mailtea = new Mailtea(process.env.MAILTEA_API_KEY);
const { id } = await mailtea.emails.send({
  from: "you@yourdomain.com",
  to: "user@example.com",
  subject: "Hello",
  html: "<p>Hi from Mailtea</p>"
});
const email = await mailtea.emails.get(id); // email.status
```

Python SDK (`pip install mailtea`):

```python
from mailtea import Mailtea
mailtea = Mailtea()  # reads MAILTEA_API_KEY
sent = mailtea.emails.send({"from": "you@yourdomain.com", "to": "user@example.com",
                            "subject": "Hello", "html": "<p>Hi from Mailtea</p>"})
mailtea.emails.get(sent["id"])  # ["status"]
```

REST (same shapes): `POST /v1/emails`, `GET /v1/emails`, `GET /v1/emails/:id`,
`POST /v1/emails/batch`, `PATCH /v1/emails/:id`, `POST /v1/emails/:id/cancel`,
all with `Authorization: Bearer mt_pat_...`.

## Gotchas

- `from` must use a domain verified in the workspace, or the send is rejected.
- Use either inline content (`html`/`text`) **or** a `template`, never both.
- `email.send` is one-shot transactional; `issue.send_now` mails the entire list.
- A newsletter's **subject is its title** — `issue.apply_ops` with `set_headers`
  moves both. If an operator has that email open in the Visual Email Designer,
  have them reload before typing, or their session's copy of the subject wins.
- Site edits go to a draft. Nothing reaches visitors until `site.publish`.
