---
name: mailtea-email
description: Send, schedule, and manage email with Mailtea from an AI agent. Use when the user wants to send a transactional email, send a batch, schedule a send, check delivery status / opens / clicks, list or cancel emails, manage contacts/segments, or draft and send a newsletter — via the Mailtea MCP server, the mailtea Node SDK, or the REST API.
---

# Mailtea email

Mailtea lets you send and manage email programmatically. Prefer the **MCP server**
(no code) when it's connected; fall back to the **SDK** or **REST** in code.

> Designing the email itself — layout, typography, email-safe HTML — before you
> send it? Use the **mailtea-email-design** skill first; it covers how the email
> should *look* and survive real inboxes.

## Setup (once)

Get a personal access token (prefix `mt_pat_`) from the Mailtea dashboard
(**Settings → API keys**) or `POST /v1/api-keys`. Then connect Claude Code:

```bash
claude mcp add mailtea -e MAILTEA_API_TOKEN=mt_pat_xxx -- npx -y mailtea-mcp
```

Self-hosting or local dev? add `-e MAILTEA_API_BASE_URL=http://localhost:8787`.

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

Different from transactional: `issue.create_draft` → `issue.update_draft` →
`issue.schedule` or `issue.send_now`. Inspect with `issue.list_recent`,
`issue.delivery_progress`, and `analytics.*`. Manage audience with `contact.*`.

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
