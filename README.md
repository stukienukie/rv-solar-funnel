# Monox AI — RV Solar Installer Funnel (v2)

A 2-page lead-gen funnel, static HTML for Vercel. No build step, no frameworks.

```
/                 index.html        Opt-in page (form gate → /watch)
/watch            watch/index.html  How-it-works video + booking
/logo.png         monox.ai wordmark (transparent)
/favicon.png      Monox "O" mark (transparent)
```

**Flow:** Paid Meta traffic lands on `/` → clicks "See How It Works" → fills the modal
form → redirected to `/watch` (with their info + attribution in the query string) →
watches the how-it-works video and books a call.

---

## What's already wired

- **Meta Pixel** `1514677843570094` — hardcoded on both pages.
- **How-it-works video** (gated): Loom `610a1353…` on `/watch`.
- **Case-study video** (free): Chris / Tucson RV Solar, Loom `2dd73e63…` on `/`.
- **Logo + favicon**: isolated to transparent PNGs.
- Fonts: **Poppins** (headlines) + **Inter** (body).

## Still to fill before going live (2 placeholders)

| Placeholder | File | Where |
|---|---|---|
| `{{GHL_INBOUND_WEBHOOK_URL}}` | index.html | `CONFIG.WEBHOOK_URL` |
| `{{GHL_BOOKING_WIDGET_URL}}` | watch/index.html | booking `<iframe src>` |

---

## Conversion tracking (IMPORTANT)

The **Lead** event = your conversion. It fires **only when a prospect's contact info is
captured and confirmed received by the CRM** — never on a plain page view.

How it works:
1. On submit, page 1 POSTs the contact to `{{GHL_INBOUND_WEBHOOK_URL}}`.
2. If the CRM responds OK, page 1 redirects to `/watch?...&lead=1`.
   If the webhook fails/isn't configured, it still redirects (so the user sees the
   video) but with `lead=0`.
3. `/watch` fires **PageView** always, but fires **Lead** (once, with advanced matching
   em/ph/fn + an `eventID` for CAPI dedup) **only when `lead=1`**.

Consequences:
- **Until `{{GHL_INBOUND_WEBHOOK_URL}}` is set, `lead` is always 0 → no Lead ever fires.**
  Expect zero conversions in Meta Test Events until the webhook is live.
- `PageView` (both pages) and `ViewContent` (form-open, page 1) still fire — these are
  traffic/retargeting signals, **not** conversions. Optimize your ad set on **Lead**.

---

## Map the form into GoHighLevel (captures name, email, phone)

The form POSTs this JSON to your Inbound Webhook:

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "phone": "+15551234567",
  "installs_per_month": "3-5",
  "page": "rv-solar-funnel",
  "submitted_at": "2026-01-01T00:00:00.000Z",
  "utm_source": "...", "utm_medium": "...", "utm_campaign": "...",
  "utm_content": "...", "utm_term": "...", "fbclid": "...", "fbc": "...", "fbp": "..."
}
```

Setup:
1. GHL → **Automation → Workflows → New Workflow → Add New Trigger → "Inbound Webhook."**
2. Copy the webhook URL → paste into `CONFIG.WEBHOOK_URL` in `index.html` → redeploy.
3. Back in the trigger, click **"Fetch Sample Request,"** then submit one real test on `/`.
   GHL captures the payload so you can map fields.
4. Add action **"Create/Update Contact"** and map:
   - **Full Name** ← `{{inboundWebhookRequest.name}}`  (or First Name — GHL splits it)
   - **Email** ← `{{inboundWebhookRequest.email}}`
   - **Phone** ← `{{inboundWebhookRequest.phone}}`  (already E.164, `+1XXXXXXXXXX`)
5. Add **"Update Contact Field"** → custom field **Installs Per Month** ←
   `{{inboundWebhookRequest.installs_per_month}}` (create the field first).
6. (Optional) Save `utm_source` / `utm_campaign` / `utm_content` / `fbclid` to contact
   custom fields for attribution.
7. Add **Tag** `rv-solar-funnel-lead` and an **Internal Notification** to Luke + Isaac.

> GHL's inbound webhook returns `200` and permits CORS, so the page can read the success
> response and correctly fire the Lead conversion. Deduplicate contacts by email+phone.

---

## Booking automations (separate workflow)

Trigger **"Customer Booked Appointment"** on this funnel's calendar:
- Immediate confirmation **email + SMS**; reminders **24h** and **1h** before.
- After the slot: **no-show** → SMS *"Hey {{contact.first_name}}, we missed you — want to
  grab a new time?"* + rebooking link; **showed** → move to post-call pipeline stage.

---

## Deploy

1. `git push` (repo: `github.com/stukienukie/rv-solar-funnel`).
2. Vercel project **rv-solar-funnel** — production branch redeploys on push.
   This v2 build is on branch **`v2-rebuild`** (preview) until you merge to `main`.
3. Custom domain recommended (e.g. `go.monox.ai`) → Vercel → Domains.

## Test checklist

- [ ] Submit on `/` → contact in GHL with tag, `Installs Per Month`, UTM/fbclid fields.
- [ ] Redirect lands on `/watch` with `name`/`email`/`phone`/`installs_per_month`/`lead=1`.
- [ ] Meta **Test Events**: `PageView` (both), `ViewContent` (form open), `Lead` **once**
      on `/watch` with advanced matching — and **only** after a successful CRM capture.
- [ ] Break `CONFIG.WEBHOOK_URL` → redirect still happens, `lead=0`, **no** Lead fires.
- [ ] Refresh `/watch` → Lead does not fire again.
- [ ] Book a test appointment → confirmation + reminders queue.

## Notes

- Meta Pixel is hardcoded on-page for reliability; GTM (`GTM-NKKF36BW`) is also loaded —
  do **not** re-install the pixel in GTM (double-fires).
- Radio buttons (not `<select>`) for "installs per month" — friendlier taps for trades.
