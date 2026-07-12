---
name: mailtea-email-design
description: Design a beautiful, on-brand, deliverable email or template BEFORE creating it in Mailtea. Use when an agent is about to write the HTML for a Mailtea template, transactional email, or newsletter post — to get email-safe layout, typography, color, and a render/QA loop right the first time. Pairs with the mailtea-email skill (which covers sending/managing) — this one covers how the email should LOOK.
---

# Mailtea email design

`mailtea-email` teaches you how to *send*. This teaches you how to make the email
*look good* — and survive real inboxes. Email is not the web: no flexbox, no CSS
variables, no web fonts you can rely on, and Outlook renders with Word. Design
*within* those constraints, not against them.

Work in four passes (borrowed from impeccable's shape → craft → critique → polish,
adapted for email): **frame → build → lint → render-and-compare.**

## 0. Pick the surface — this decides whether your design survives

Mailtea has two content models. Using the wrong one is the #1 cause of "it looked
different when sent / when I opened it in the editor."

| | **Complete document** | **Newsletter fragment** |
|---|---|---|
| What it is | A whole self-contained email: `<!DOCTYPE html><html><head>…fonts/styles…</head><body>…own background/footer…</body>` | BODY content only — no `<html>`/`<head>`/`<body>`, no outer background, no footer |
| Ships via | **Transactional** — `POST /v1/emails` (html or template). Delivered **verbatim** (sanitize + tracking only). | **Newsletter/broadcast post** — `POST /v1/posts`. Mailtea **wraps it** in the publication's email shell (`<Container>` + unsubscribe footer). |
| Use for | Pixel-controlled designs, receipts, review requests, anything where YOU own the whole layout. | Regular editions where Mailtea's shell + compliant unsubscribe footer is wanted. |
| No-code editor | Opens **as one faithful "Custom HTML" block** — preview == inbox; edit the raw HTML. Block editing is intentionally NOT offered (it can't change a complete document without losing it). | Maps cleanly to editor **blocks** (text, image, button…); the shell is added at send. |

Rules of thumb:
- **A complete `<!DOCTYPE html>` document → send it transactionally.** If you post it as a
  newsletter it gets double-wrapped (a full `<html>` nested inside Mailtea's shell) and
  the sent email won't match your design.
- **Writing a newsletter post → emit a body fragment**, not a full document. Don't include
  `<html>`/`<head>`/`<body>`, your own page background, or an unsubscribe footer — Mailtea
  supplies the shell and a compliant footer. Fragments also survive the block editor.
- Either way, opening custom HTML in the no-code editor converts it into an **editable
  block-based design** (high-fidelity — tables/text/images/buttons become real blocks, the
  page background/width carry over), so operators can keep editing what you produced.

## 1. Frame (shape)

Before any HTML, decide:

- **Type** — transactional (one-shot, e.g. receipt/review-request) or marketing
  (newsletter/broadcast). Transactional leans focused + single-action; marketing
  can carry more sections.
- **One job per email.** A single primary action. Everything else is support.
- **Variables** — what's personalized (`{{first_name}}`, `{{review_url}}`,
  `{{unsubscribe_url}}`). Plan these as `{{snake_case}}` tokens now.

## 2. Build — the email-safe contract (non-negotiable)

These aren't style preferences; break them and the layout collapses in a major client.

- **Tables, not divs, for layout.** `<table role="presentation" cellpadding="0"
  cellspacing="0">`. No `<div>` grids.
- **Inline every style** (`style="…"` on the element). `<style>` in `<head>` is
  for `@media` and `:hover` only — Gmail strips much of it.
- **Fixed width ~600px**, centered, with a `@media (max-width:620px){ width:100%
  !important }` fallback for mobile.
- **Never emit these** — Mailtea lints serialized HTML against the real
  caniemail matrix (`apps/web/app/lib/caniemail/caniemail-lint.ts`,
  `lintEmailHtml()`), strict clients = **Apple Mail, Gmail, Outlook desktop**:

  | Severity | Don't use | Use instead |
  |----------|-----------|-------------|
  | **fail** (breaks) | `display:flex` / `inline-flex` | nested tables / `align`, `valign` |
  | **fail** | `position:absolute` / `fixed` | document flow only |
  | **fail** | `gap` | cell `padding` / spacer rows |
  | **fail** | `transform` | pre-positioned markup; a pre-rendered **image** for arched/rotated wordmarks |
  | **fail** | `aspect-ratio` | explicit `width`/`height` |
  | **fail** | `clamp()` | fixed px + a `@media` override |
  | **fail** | `vh` / `vw` units | `px` / `%` |
  | **fail** | `var(--…)` | literal values inline |
  | **warn** (degrades) | `box-shadow`, `*-gradient()` | flat fills; treat as enhancement only |

- **Fonts: declare a web font, always supply a system fallback.** Web fonts load
  only in Apple Mail / some clients; everywhere else the stack must still look
  intentional. `font-family:'Source Serif 4', Georgia, 'Times New Roman', serif`.
- **Logos / wordmarks → a hosted image, NOT inline `<svg>`.** Gmail and Outlook strip
  inline SVG entirely (it renders only in Apple Mail and in-app previews), so a `<textPath>`
  arched wordmark vanishes for most recipients. Pre-render it to a PNG/SVG file and reference
  it with `<img src="…" alt="…">`. (Inline SVG is also dropped when the email is opened in the
  no-code editor, for the same reason.)
- **Social / brand icons → hosted `<img>`, never icon fonts or bare glyphs.** A footer
  row of `f  𝕏  in` typed as font-glyphs or characters renders as plain text in most
  clients (no icon), looking broken. Each icon is a small hosted PNG/SVG
  `<img width height alt style="display:block;border:0">` inside its own cell — see the
  social-row recipe below.
- **Brand-critical display/poster type → a hosted image when the weight/tracking carries
  the brand.** A heavy condensed poster headline (e.g. a giant `BIG COZY`) falls back to a
  generic bold across clients and loses its punch. If the exact face matters, pre-render the
  headline to an `<img>` (as with arched wordmarks); otherwise pick the closest web-safe
  stack and accept the fallback.
- **Images** need `width`, `height`, `alt`, and `style="display:block;border:0"`.
  Don't depend on them: many clients block images by default, so the email must
  read with images off. Add a hidden **preheader** span as the first body node.
- **Dark mode**: Apple Mail/Outlook may invert. Keep the email canvas warm/light
  by design; don't rely on a dark background you didn't set.
- **Accessibility**: real text (not text-in-images), `lang` on `<html>`,
  `role="presentation"` on layout tables, ≥14px body, AA contrast.

## 3. Design judgment (typeset · layout · colorize · distill · polish)

Constraints handled — now make it *good*. Mailtea's brand is **precise, calm,
recipient-faithful** (see `CLAUDE.md` → Design Context):

- **typeset** — One serif for body (`Source Serif 4`), one sans for UI/labels
  (`Space Grotesk`); `Instrument Serif` for display moments. Body 16–19px,
  line-height ~1.6–1.75, generous paragraph spacing (margin, since `gap` is out).
  Don't mix four fonts.
- **layout** — Vertical rhythm via consistent cell padding. "One column" governs the
  *outer* frame — but faithful designs often have multi-column *regions* (a 2-up product
  grid, an image-beside-text row, a card collage). Build those as nested tables with
  fixed-percentage cells (`<td width="50%">`), and add a `@media (max-width:620px)` rule
  that stacks them (`display:block;width:100%`) on mobile. Don't flatten a genuine
  side-by-side layout into a single stacked column. Let the white canvas breathe; hairline
  dividers over boxes.
- **colorize** — Restraint. Near-neutral text on a warm/white canvas, a *single*
  committed accent for the one CTA. Status meaning stays (scheduled blue, sent
  green, failed red). No garish gradients (and they're `warn`-listed anyway).
- **distill** — Cut to one message and one action. If a section doesn't serve the
  primary job, remove it.
- **polish** — The last 10%: optical alignment, consistent corner radii
  (`rounded-md`/`-lg`, not pill-everything), an underlined text link with
  `text-underline-offset` instead of a heavy button when the tone is personal,
  a real signature, a complete footer (address + unsubscribe).

## 4. Render and compare (critique)

Never ship unrendered. The loop:

1. **Lint** — run the HTML past `lintEmailHtml()`; resolve every `fail`.
2. **Render with variables** — Mailtea substitutes `{{var}}` and `{{{var}}}`
   placeholders (`apps/api/src/template-render.ts`). On the **posts** path only
   *passed* variables are replaced, so pass them all; the **transactional** send
   path also applies a variable's declared `fallback_value`.
3. **See it in a real client** — send to Mailpit
   (`pnpm --filter @mailtea/web exec tsx app/../scripts/send-templates-to-mailpit.ts`,
   web UI at `http://localhost:8026`), or open the rendered HTML in a browser.
4. **Compare to intent** — if you were given a reference, screenshot the render and
   check fidelity (wordmark, fonts, spacing, color, footer). Fix the gaps, re-render.

## Create it in Mailtea

Once it renders right, persist it as a reusable **server template** (then any
send can reference it by id + variables):

```bash
# MCP (no code): tools template.create → template.publish
# CLI:
mailtea templates create --publication-id pub_123 --name "Review request" \
  --subject "A small favor" --html "$(cat email.html)"
mailtea templates publish <template-id> --publication-id pub_123

# Use it (substitutes server-side):
mailtea posts create --publication-id pub_123 --subject "A small favor" \
  --template-id <template-id> \
  --template-var first_name=Friend --template-var review_url=https://…
```

Declare each `{{variable}}` (with `type`, optional `fallback_value`) on create so
the template is self-documenting. REST equivalents: `POST /v1/templates`,
`POST /v1/templates/:id/publish?publication_id=…`, `POST /v1/posts`. For transactional
sends, pass a `template` object on `POST /v1/emails` instead — see `mailtea-email`.

## Starter skeleton (correct by construction)

A 600px card that satisfies the contract. Start here, then design into it.

```html
<!DOCTYPE html>
<html lang="en"><head>
<meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<meta name="x-apple-disable-message-reformatting">
<style>
  body{margin:0;padding:0;background:#efeeec}
  table{border-collapse:collapse}
  img{border:0;line-height:100%;outline:none;text-decoration:none}
  @media (max-width:620px){.card{width:100%!important}.pad{padding-left:28px!important;padding-right:28px!important}}
</style></head>
<body>
  <span style="display:none;visibility:hidden;opacity:0;height:0;width:0;font-size:1px;color:#efeeec;">Preheader — the inbox preview line.</span>
  <table role="presentation" width="100%" cellpadding="0" cellspacing="0" style="background:#efeeec">
    <tr><td align="center" style="padding:36px 16px">
      <table role="presentation" class="card" width="600" cellpadding="0" cellspacing="0" style="width:600px;max-width:600px;background:#fff">
        <tr><td class="pad" style="padding:48px 56px;font-family:'Source Serif 4',Georgia,serif;font-size:18px;line-height:1.72;color:#1b1b1b">
          <p style="margin:0 0 24px">Hi {{first_name}}!</p>
          <p style="margin:0 0 24px">Your message. One clear idea.</p>
          <p style="margin:0"><a href="{{cta_url}}" style="color:#1b1b1b;text-decoration:underline;text-underline-offset:3px">One clear action →</a></p>
        </td></tr>
        <tr><td align="center" class="pad" style="padding:18px 56px 48px;font-family:'Source Serif 4',Georgia,serif;font-size:13px;color:#9b9b9b">
          Your Company · City, Country<br>
          <a href="{{unsubscribe_url}}" style="color:#b4b4b4;text-decoration:none">Unsubscribe</a>
        </td></tr>
      </table>
    </td></tr>
  </table>
</body></html>
```

## Recipes (copy these shapes for faithful recreations)

These are the three shapes agents most often get wrong when reproducing a real design.
All survive the email-safe lint AND the no-code editor's HTML→blocks import.

**Social icon row** — hosted images in cells, never glyphs:

```html
<table role="presentation" cellpadding="0" cellspacing="0" align="center"><tr>
  <td style="padding:0 8px"><a href="{{facebook_url}}"><img src="https://cdn.example.com/ic-facebook.png" width="28" height="28" alt="Facebook" style="display:block;border:0"></a></td>
  <td style="padding:0 8px"><a href="{{instagram_url}}"><img src="https://cdn.example.com/ic-instagram.png" width="28" height="28" alt="Instagram" style="display:block;border:0"></a></td>
  <!-- one <td> per icon -->
</tr></table>
```

**Two-column region** — nested table with `%` cells + a mobile stack fallback (put the
`@media` rule in `<head>`; give each column class `col`):

```html
<style>@media (max-width:620px){.col{display:block!important;width:100%!important}}</style>
…
<table role="presentation" width="100%" cellpadding="0" cellspacing="0"><tr>
  <td class="col" width="50%" valign="top" style="padding:12px">…left…</td>
  <td class="col" width="50%" valign="top" style="padding:12px">…right…</td>
</tr></table>
```

**Card with chrome** — accent bar as a colored row, outline + radius inline on the cell
(inline `border`, `border-radius`, `background` all survive the editor import; the bare
`border`/`bordercolor` HTML attributes are also honored):

```html
<table role="presentation" width="100%" cellpadding="0" cellspacing="0"
       style="border:1px solid #e5e7eb;border-radius:8px">
  <tr><td style="height:6px;background:#2680eb;border-radius:8px 8px 0 0;font-size:0;line-height:0">&nbsp;</td></tr>
  <tr><td style="padding:0"><img src="…" width="252" height="160" alt="…" style="display:block;border:0;width:100%"></td></tr>
  <tr><td style="padding:14px 16px;font-weight:600;text-decoration:underline">Card label</td></tr>
</table>
<!-- pill/badge: bg + radius + padding on an inline-block-ish cell -->
<span style="display:inline-block;padding:4px 12px;background:#111;color:#fff;border-radius:999px;font-size:12px">New</span>
```

## Don't

- Don't paste a web/marketing-page layout into an email — flex/grid/`var()` will
  collapse in Outlook and Gmail.
- Don't type social icons as font glyphs or characters — use hosted `<img>` icons.
- Don't flatten genuine side-by-side layouts into one stacked column — use a `%`-cell
  nested table with a `@media` mobile stack.
- Don't ship without rendering in a client.
- Don't leave `{{placeholders}}` unsubstituted, or a footer without an unsubscribe link.
- Don't reach for `box-shadow`/gradients/buttons by reflex; flat and typographic
  usually reads calmer and survives more clients.
