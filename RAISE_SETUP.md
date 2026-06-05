# `raise.html` — community-raise interest list, setup

A pre-launch waitlist for nar's upcoming community raise on a regulated
platform (Republic Europe). **It is not an investment offer.** The page only
collects a first name + email + GDPR consent so you can notify people when the
regulated raise opens. No amounts, no pledges, no payment are taken here — by
design (that all belongs on the regulated platform's page).

Live page: `/raise.html` → https://narwomenshealth.com/raise.html
Linked from: the homepage founder section, the homepage footer ("The raise"),
and the sitemap. Point your Instagram "link in bio" at the URL above.

---

## Backend: why a hosted provider (not a serverless function)

This site is **plain GitHub Pages** (there's a `CNAME`, no `netlify.toml` /
`vercel.json` / `wrangler.toml`). GitHub Pages serves static files only — there
is no server here to run a form function. So submissions go directly from the
browser to a hosted email provider.

**Wired provider: Mailchimp** (free plan, ~500 contacts, no credit card).
Because Mailchimp doesn't send CORS headers, the page talks to its **JSONP
`post-json` endpoint** — the long-standing technique for custom forms on static
sites. It still lets the page read the result (success / already-subscribed /
bad email) and show inline errors **without a page reload**.

**No API key is in the repo.** The only value you paste is Mailchimp's public
embedded-form URL — the same one Mailchimp prints for anyone to drop on a page.
Safe to commit.

---

## What you need to do (Mailchimp — ~5 minutes)

1. Create a free account at <https://mailchimp.com> (Free plan, ~500 contacts).
   Confirm your own email so the audience is active.
2. **Audience → Signup forms → "Embedded forms" → choose "Condensed."**
3. In the generated code, copy the value of the `<form action="...">` URL.
   It looks like:
   ```
   https://YOURDC.list-manage.com/subscribe/post?u=abc123def456...&id=789ghi...&f_id=...
   ```
   (`YOURDC` is your datacenter, e.g. `us21`.)
4. Open **`raise.html`**, find the `CONFIG` block near the bottom (in the
   `<script>`), and replace:
   ```js
   const MAILCHIMP_FORM_ACTION = 'REPLACE_WITH_MAILCHIMP_FORM_ACTION';
   ```
   with that whole URL, e.g.
   ```js
   const MAILCHIMP_FORM_ACTION = 'https://us21.list-manage.com/subscribe/post?u=abc123...&id=789ghi...';
   ```
5. Commit + push. That's the whole wiring.

> The page derives Mailchimp's required anti-bot honeypot field (`b_<u>_<id>`)
> and the `post-json` endpoint automatically from that one URL, so it's the only
> value you ever touch.

> Until you paste a real URL, the form validates but shows a gentle "the list
> isn't connected yet" message instead of silently losing a signup.

### Double opt-in (recommended, GDPR-clean — on by default)
Mailchimp's embedded forms use **double opt-in by default**: each person gets a
confirmation email and is only added once they click it. The page already tells
them to check their inbox. To turn it off: Audience → Settings → "Enable double
opt-in", and set `DOUBLE_OPT_IN = false` in `raise.html` (line just under the
form URL) to hide the "check your inbox" note.

### Exporting / emailing the list later
- **Export:** Audience → **All contacts → Export Audience** (CSV).
- **Broadcast:** when the raise opens, **Campaigns → Create → Email**, send to
  this audience with the link to the regulated platform. That's the whole point
  of routing into an email tool rather than a spreadsheet.

---

## Validation & accessibility (already built in)

- **Client-side:** first name required, email format checked, consent box must
  be ticked before submit. Inline errors appear under each field (`role="alert"`,
  `aria-invalid`, `aria-describedby`), and focus jumps to the first bad field.
- **Server-side:** Mailchimp validates the email on its end; if it rejects the
  address, the page surfaces that message inline on the email field. "Already
  subscribed" is treated as success (they're on the list).
- **No reload:** on success the form is swapped for the success state via an
  `aria-live` region.
- **Accessible & responsive:** labels tied to inputs, visible focus rings,
  keyboard-navigable custom checkbox, AA-contrast oxblood-on-parchment, mobile
  layout down to 320px, and `prefers-reduced-motion` honoured.

---

## GDPR notes

- Collects **only** first name + email. No investment amounts, no brackets —
  intentionally, for compliance.
- Consent checkbox is **required** (unticked by default — no pre-ticked boxes).
- The consent text states they can unsubscribe anytime; every Mailchimp email
  carries an unsubscribe link automatically.
- Double opt-in gives you a timestamped confirmation record per subscriber.
- The form links to `/privacy.html` right beneath it.

---

## Switching providers later (optional)

The page submits in one small block in `raise.html`. To move off Mailchimp,
change the `CONFIG` block and the submit handler.

- **Kit (ConvertKit)** — if their free tier returns: POST form-encoded
  `email_address` + `first_name` to `https://app.kit.com/forms/<id>/subscriptions`
  with a normal `fetch()` (Kit allows cross-origin and returns JSON). No JSONP
  needed.
- **MailerLite** — free up to ~1,000 subscribers. Uses a JSONP subscribe
  endpoint similar to Mailchimp, or their universal embed script.
- **Buttondown** — free up to ~100 subscribers. POST to
  `https://buttondown.com/api/emails/embed-subscribe/<username>` (often
  `mode: 'no-cors'`, i.e. optimistic success).

For any of these, keep the rule that holds for all of them: **only the public
form ID / URL goes in the file — never a secret API key.**

---

## Files touched
- `raise.html` — the page (form + CONFIG block + Mailchimp JSONP submit).
- `index.html` — quiet CTA in the founder section + a footer column ("The raise").
- `sitemap.xml` — added `/raise.html`.
